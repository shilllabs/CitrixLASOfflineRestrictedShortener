# Security Documentation

## Citrix LAS Offline Restricted Shortener

**Version:** 3.0.0
**Author:** Shane Smith
**Date:** 2026-04-20

---

## 1. Purpose and Scope

This tool facilitates the offline (restricted/air-gapped) activation of Citrix License Server licenses by encoding the licensing request data (LSID + RSA public key) into a human-typeable chunked key for manual transfer between isolated machines.

The tool does **not** transmit, upload, or share any data over a network. All operations are performed locally on the machine where the executable runs.

---

## 2. Architecture Overview

| Component | Description |
|-----------|-------------|
| `shorten.py` | Core encoding/decoding engine and CLI interface |
| `shorten_gui.py` | Graphical user interface (tkinter) with 7 tabs |
| `build.spec` | PyInstaller build specification for the compiled executable |
| Compiled executable | Single-file Windows executable built with PyInstaller |

**Data flow:**
1. Input: `.zip` archive, `.json` file, or `.txt` file containing LSID + public key
2. Processing: Parse JSON, extract data segment, strip known DER/ASN.1 structure, encode as Crockford Base32
3. Output: Chunked key (text), saved to timestamped files with restrictive permissions
4. Verification: SHA-256 hash displayed on both encode and decode sides for integrity confirmation

**Application tabs:**
| Tab | Function |
|-----|----------|
| Instructions | Complete workflow documentation |
| Step 1: Encode | Generate/encode licensing request on the License Server |
| Step 2: Decode | Decode the key on a machine with portal access |
| Step 3: Activate | Import the activation response file |
| Log | Real-time activity log for troubleshooting |
| Release Notes | Full version history |
| About | Author, credits, legal disclaimer |

---

## 3. Dependency Profile

### Runtime Dependencies

This application uses **zero third-party packages**. All modules are from the Python 3.13 standard library:

| Module | Purpose | Supply Chain Risk |
|--------|---------|-------------------|
| `base64` | PEM key body encoding/decoding | None (stdlib) |
| `hashlib` | SHA-256 verification hash generation | None (stdlib) |
| `json` | JSON parsing for data extraction | None (stdlib) |
| `struct` | Binary data packing/unpacking | None (stdlib) |
| `os` | File path operations and permission management | None (stdlib) |
| `sys` | Command-line argument handling | None (stdlib) |
| `zipfile` | ZIP archive reading | None (stdlib) |
| `datetime` | Timestamp generation for filenames and logs | None (stdlib) |
| `logging` | Activity logging to file and GUI | None (stdlib) |
| `tkinter` | Graphical user interface | None (stdlib) |
| `shutil` | File copy operations | None (stdlib) |
| `subprocess` | Citrix tool execution | None (stdlib) |
| `stat` | File permission constants | None (stdlib) |

### Build-Time Dependencies (Development Only)

These packages are used only to compile the executable and are **not included in the runtime**:

| Package | Version | Purpose |
|---------|---------|---------|
| PyInstaller | 6.19.0 | Compiles Python script into standalone Windows executable |
| altgraph | 0.17.5 | Dependency graph analysis (PyInstaller dependency) |
| packaging | 26.1 | Version parsing (PyInstaller dependency) |
| pefile | 2024.8.26 | Windows PE file handling (PyInstaller dependency) |
| pyinstaller-hooks-contrib | 2026.4 | Module hooks (PyInstaller dependency) |
| pywin32-ctypes | 0.2.3 | Windows API bindings (PyInstaller dependency) |
| setuptools | 82.0.1 | Package management (PyInstaller dependency) |

---

## 4. Security Controls

### 4.1 EULA / License Acceptance

| Control | Description |
|---------|-------------|
| GUI acceptance dialog | A license agreement dialog is displayed on launch; user must click "I Accept" to proceed |
| CLI acceptance switch | The `--acceptEULA` switch is required for CLI mode; without it the tool exits with an error |
| Rejection logging | EULA rejection is logged to the activity log file |

### 4.2 Input Validation

| Control | Description | Implementation |
|---------|-------------|----------------|
| LSID hex validation | LSID must be exactly 64 hexadecimal characters | `pack_data()` in `shorten.py` |
| Base64 validation | PEM key body validated before decoding; malformed input produces clear error | `pack_data()` in `shorten.py` |
| JSON file size limit | Files larger than 10 MB rejected before parsing | `_read_from_json()` in `shorten.py` |
| Zip bomb protection | Decompressed file size checked against 50 MB limit before reading | `_read_from_zip()` in `shorten.py` |
| Checksum verification | 2-character position-weighted checksum detects typos during decode | `decode()` in `shorten.py` |
| Verification hash | 8-character SHA-256 hash for end-to-end data integrity confirmation | `compute_verification_hash()` in `shorten.py` |
| Path normalization | All file paths normalized to OS-native format before use | `os.path.normpath()` in all browse handlers |

### 4.3 Data Protection

| Control | Description |
|---------|-------------|
| No network access | Tool operates entirely offline; no data transmitted |
| Sensitive data redaction | Log entries record operation summaries and character counts only; LSID and key values are never written to logs |
| Restrictive file permissions | Output files created with owner-only read/write permissions (mode 0600) via `secure_write()` |
| Timestamped filenames | Output files include timestamps to prevent accidental overwriting |
| Export file preservation | Existing export files renamed with last-modified timestamp before generating new exports |

### 4.4 Process Execution

| Control | Description |
|---------|-------------|
| No shell execution | Subprocess calls use list form (not `shell=True`), preventing command injection |
| Path validation | Citrix tool path verified to exist before execution |
| Execution timeout | Export commands: 60-second timeout; Activation import: 120-second timeout |
| Hardcoded tool path | Only the known Citrix tool path is executed; no user-supplied commands |

### 4.5 Encoding Security

| Property | Description |
|----------|-------------|
| Algorithm | Crockford Base32 encoding with DER structure stripping |
| Verification | SHA-256 hash (truncated to 8 hex characters) for integrity confirmation |
| Reversibility | Fully reversible — this is encoding, not encryption |
| No secrets embedded | The executable contains no keys, passwords, or secrets |
| Deterministic output | Same input always produces the same encoded output and verification hash |

---

## 5. Threat Model

### 5.1 In-Scope Threats (Mitigated)

| Threat | Mitigation |
|--------|------------|
| Zip bomb / resource exhaustion | 50 MB decompressed size limit |
| Oversized JSON / memory exhaustion | 10 MB file size limit before parsing |
| Malformed input causing crashes | Input validation on LSID hex, base64, and file sizes |
| Log file data leakage | Sensitive values redacted from all log output |
| Output file exposure | Restrictive file permissions on generated files |
| Previous export overwriting | Automatic rename with timestamp before generating new exports |
| Typos during manual key entry | Position-weighted checksum with warning on mismatch |
| Silent data corruption | SHA-256 verification hash for end-to-end integrity |
| Command injection via subprocess | List-form subprocess execution, no shell interpretation |
| Unauthorized CLI usage | EULA acceptance required (`--acceptEULA` switch) |
| Path format issues | All paths normalized with `os.path.normpath()` |

### 5.2 Out-of-Scope Threats

| Threat | Reason |
|--------|--------|
| Network-based attacks | Tool has no network functionality |
| Encryption of data at rest | Tool performs encoding (reversible by design), not encryption |
| Physical access to machine | Physical security is the responsibility of the deployment environment |
| Privilege escalation | Tool runs with the permissions of the invoking user |
| Tampering with the executable | Code signing is the responsibility of the deploying organization |

---

## 6. Known Limitations

1. **Not encryption.** The encoded output is reversible by anyone with this tool. It is encoding for transport convenience, not confidentiality.
2. **Windows file permissions.** On Windows NTFS, `os.chmod()` has limited effect. File access is controlled by Windows ACLs, which default to user-profile-based access.
3. **Single-user tool.** The tool is designed for individual use, not multi-user or server deployment.
4. **Citrix tool dependency.** Steps 1 (Generate Export) and 3 (Activate) require the Citrix Licensing Server and `ctxLasOfflineActivationTool.exe` to be installed. Step 2 (Decode) has no external dependencies.

---

## 7. Security Audit History

| Version | Date | Audit Performed |
|---------|------|-----------------|
| v2.4.0 | 2026-04-17 | Full code review for OWASP categories: command injection, path traversal, zip slip, JSON parsing, log injection, file permissions, sensitive data exposure, denial of service. All identified vulnerabilities addressed. |
| v2.5.0 | 2026-04-18 | Added export file preservation to prevent data loss. |
| v2.7.0 | 2026-04-20 | Added SHA-256 verification hash for data integrity. |
| v2.9.1 | 2026-04-20 | Fixed path normalization for Windows native tool compatibility. |
| v3.0.0 | 2026-04-20 | Added EULA acceptance requirement for GUI and CLI. Rewritten instructions with complete security-aware workflow. |

---

## 8. Contact

For security concerns, contact:

**Shane Smith**
