[Home](#) | [Examples](./EXAMPLES.md) | [SECURITY](./SECURITY.md)

# Citrix LAS Offline Restricted Shortener

**Version:** 3.0.0  
**Author:** Shane Smith  
**Credits:** Robert Jaudon

## Overview

This tool facilitates the offline (restricted/air-gapped) activation of Citrix License Server licenses when the server cannot reach the internet. It encodes the licensing request data (LSID + RSA public key) into a short, human-typeable chunked key that can be manually transferred between the isolated License Server and a machine with internet access to the Citrix licensing portal.

## The Problem

Citrix License Servers in restricted environments (air-gapped networks, government systems, classified enclaves) cannot contact the Citrix licensing portal directly. The licensing request data is a 554-character JSON string containing a 64-character hex LSID and a full RSA 2048-bit public key. Manually typing this accurately between machines is error-prone and impractical.

## The Solution

This tool compresses and encodes the licensing data into **19 lines of chunked characters** (approximately 467-473 characters total) using Crockford Base32 — an alphabet specifically designed to avoid visually ambiguous characters. The encoded key includes a checksum for typo detection and a verification hash for end-to-end confirmation.

## 3-Step Workflow

### Step 1: Encode (on the Citrix License Server)

Generate the licensing request and encode it into a typeable key.

1. Run the tool on the License Server
2. Click **"Generate Export & Encode"** (or browse for an existing darksiteRequest.zip)
3. The encoded key appears as 19 numbered lines
4. Note the **Verification Hash** (8 characters)
5. Communicate the key to the machine with internet access

### Step 2: Decode (on a machine with portal access)

Type the encoded key and recover the LSID + public key.

1. Run the tool on the internet-connected machine
2. Type or paste the 19-line key
3. Click **Decode**
4. Verify the hash matches Step 1
5. Copy the LSID and Public Key into the Citrix licensing portal
6. Download the activation response file (`<LSID>_activation.blob.zip`)

### Step 3: Activate (on the Citrix License Server)

Import the activation response to complete licensing.

1. Transfer the `.blob.zip` file to the License Server
2. Run the tool, select the file, click **Import Activation**
3. The license is now active

## Installation

No installation required. The tool is a single standalone Windows executable:

```
CitrixLASOfflineRestrictedShortener_v3.0.0.exe
```

Requirements:
- Windows 10/11 or Windows Server 2016+
- Citrix Licensing Server installed (for Steps 1 and 3)
- No internet connection required on the License Server
- No Python installation needed (bundled in the exe)

## Usage

### GUI Mode (recommended)

Double-click the executable. A license agreement dialog appears first — click **"I Accept"** to proceed. The main window opens with tabs for each step.

### CLI Mode

```cmd
CitrixLASOfflineRestrictedShortener_v3.0.0.exe --acceptEULA encode -f darksiteRequest.zip
CitrixLASOfflineRestrictedShortener_v3.0.0.exe --acceptEULA decode -f key.txt
CitrixLASOfflineRestrictedShortener_v3.0.0.exe --acceptEULA generate
```

The `--acceptEULA` switch is required for CLI mode.

## Output Files

All output files include timestamps to prevent overwriting:

| File | Description |
|------|-------------|
| `key_20260417_143000.txt` | The encoded chunked key (19 lines) |
| `data_20260417_143000.txt` | Extracted LSID + pubkey data |
| `shorten_20260417_143000.log` | Activity log for troubleshooting |
| `darksiteRequest_20260417_143000.zip` | Timestamped copy of the export |

## Encoded Key Format

The output looks like a software license key:

```
01: 9503N-GTNCA-4YJB5-V0XSK-JASYC
02: CMS36-BMNAS-KMPZD-FG8FA-46BEJ
03: NXBZT-X9PPM-ZPP1N-HH3HC-FHG7N
...
19: RTF0V-YHHFW-9FVW4-1JBYZ-1DE

Verification Hash: 29DC3DF0
```

**Alphabet:** Crockford Base32 (0-9, A-H, J-K, M-N, P-T, V-Z)  
**Excluded characters:** I (looks like 1), L (looks like 1), O (looks like 0), U  
**Forgiving decoder:** O is treated as 0, I/L as 1, case-insensitive

## Verification Hash

An 8-character SHA-256 hash is displayed during both encode and decode:
- If the hashes **match**: the key was typed correctly
- If they **don't match**: there is a typo somewhere

## Export Location

The Citrix LAS Offline Activation Tool reads and writes files at:

```
C:\Program Files (x86)\Citrix\Licensing\LS\resource\cache\
```

## Security

- **Zero third-party dependencies** — uses only Python standard library modules
- **No network access** — fully offline operation
- **Input validation** — LSID hex validation, base64 validation, file size limits
- **Zip bomb protection** — 50MB decompression limit
- **Sensitive data redaction** — logs record character counts, never LSID/key values
- **Restrictive file permissions** — output files set to owner-only access
- **EULA acceptance required** — both GUI and CLI enforce license agreement

See [SECURITY.md](SECURITY.md) for full security documentation.  
See [sbom.cdx.json](sbom.cdx.json) for the Software Bill of Materials.

## Dependencies

**Runtime:** None beyond the Python 3.13 standard library (bundled in the exe).

**Build-time only:** PyInstaller 6.19.0 (not included in the distributed executable).

## Building from Source

```cmd
pip install pyinstaller
python -m PyInstaller --clean build.spec
```

The executable is produced at `dist/CitrixLASOfflineRestrictedShortener_v3.0.0.exe`.

## Legal Disclaimer

This software / sample code is provided to you "AS IS" with no representations, warranties or conditions of any kind. You may use, modify and distribute it at your own risk. CITRIX DISCLAIMS ALL WARRANTIES WHATSOEVER, EXPRESS, IMPLIED, WRITTEN, ORAL OR STATUTORY, INCLUDING WITHOUT LIMITATION WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NONINFRINGEMENT. Without limiting the generality of the foregoing, you acknowledge and agree that (a) the software / sample code may exhibit errors, design flaws or other problems, possibly resulting in loss of data or damage to property; (b) it may not be possible to make the software / sample code fully functional; and (c) Citrix may, without notice or liability to you, cease to make available the current version and/or any future versions of the software / sample code. In no event should the software / code be used to support of ultra-hazardous activities, including but not limited to life support or blasting activities. NEITHER CITRIX NOR ITS AFFILIATES OR AGENTS WILL BE LIABLE, UNDER BREACH OF CONTRACT OR ANY OTHER THEORY OF LIABILITY, FOR ANY DAMAGES WHATSOEVER ARISING FROM USE OF THE SOFTWARE / SAMPLE CODE, INCLUDING WITHOUT LIMITATION DIRECT, SPECIAL, INCIDENTAL, PUNITIVE, CONSEQUENTIAL OR OTHER DAMAGES, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGES. Although the copyright in the software / code belongs to Citrix, any distribution of the code should include only your own standard copyright attribution, and not that of Citrix. You agree to indemnify and defend Citrix against any and all claims arising from your use, modification or distribution of the code.
