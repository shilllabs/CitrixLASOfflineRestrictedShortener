# Security Documentation

## Citrix LAS Offline Restricted Shortener

**Version:** 2.5.0
**Author:** Shane Smith, Senior Director of North America Government Technical Strategy
**Date:** 2026-04-18

---

## 1. Purpose and Scope

This tool encodes structured licensing data (LSID and RSA public key) into a human-typeable chunked key format for transfer between air-gapped machines. It is designed for offline use in restricted environments where file transfer is not possible.

The tool does **not** transmit, upload, or share any data over a network. All operations are performed locally on the machine where the executable runs.

---

## 2. Architecture Overview

| Component | Description |
|-----------|-------------|
| `shorten.py` | Core encoding/decoding engine and CLI interface |
| `shorten_gui.py` | Graphical user interface (tkinter) |
| Compiled executable | Single-file Windows executable built with PyInstaller |

**Data flow:**
1. Input: `.zip` archive, `.json` file, or `.txt` file containing LSID + public key
2. Processing: Parse JSON, extract data segment, strip known DER/ASN.1 structure, encode as Crockford Base32
3. Output: Chunked key (text), saved to timestamped files with restrictive permissions

---

## 3. Dependency Profile

### Runtime Dependencies

This application uses **zero third-party packages**. All modules are from the Python 3.13 standard library:

| Module | Purpose | Supply Chain Risk |
|--------|---------|-------------------|
| `base64` | PEM key body encoding/decoding | None (stdlib) |
| `json` | JSON parsing for data extraction | None (stdlib) |
| `struct` | Binary data packing/unpacking | None (stdlib) |
| `os` | File path operations | None (stdlib) |
| `sys` | Command-line argument handling | None (stdlib) |
| `zipfile` | ZIP archive reading | None (stdlib) |
| `datetime` | Timestamp generation | None (stdlib) |
| `logging` | Activity logging | None (stdlib) |
| `tkinter` | Graphical user interface | None (stdlib) |
| `shutil` | File copy operations | None (stdlib) |
| `subprocess` | Citrix tool execution | None (stdlib) |
| `stat` | File permission constants | None (stdlib) |
| `secrets` | Cryptographic random generation (unused in current version) | None (stdlib) |
| `string` | Character set constants (unused in current version) | None (stdlib) |

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

### 4.1 Input Validation

| Control | Description | Implementation |
|---------|-------------|----------------|
| LSID hex validation | LSID must be exactly 64 hexadecimal characters | `pack_data()` in `shorten.py` |
| Base64 validation | PEM key body validated before decoding; malformed input produces clear error | `pack_data()` in `shorten.py` |
| JSON file size limit | Files larger than 10 MB rejected before parsing | `_read_from_json()` in `shorten.py` |
| Zip bomb protection | Decompressed file size checked against 50 MB limit before reading | `_read_from_zip()` in `shorten.py` |
| Checksum verification | 2-character position-weighted checksum detects typos during decode | `decode()` in `shorten.py` |

### 4.2 Data Protection

| Control | Description |
|---------|-------------|
| No network access | Tool operates entirely offline; no data transmitted |
| Sensitive data redaction | Log entries record operation summaries and character counts only; LSID and key values are never written to logs |
| Restrictive file permissions | Output files created with owner-only read/write permissions (mode 0600) via `secure_write()` |
| Timestamped filenames | Output files include timestamps to prevent accidental overwriting |
| Export file preservation | Existing export files renamed with last-modified timestamp before generating new exports |

### 4.3 Process Execution

| Control | Description |
|---------|-------------|
| No shell execution | Subprocess calls use list form (not `shell=True`), preventing command injection |
| Path validation | Citrix tool path verified to exist before execution |
| Execution timeout | Subprocess calls have a 60-second timeout to prevent hangs |
| Hardcoded tool path | Only the known Citrix tool path is executed; no user-supplied commands |

### 4.4 Encoding Security

| Property | Description |
|----------|-------------|
| Algorithm | Crockford Base32 encoding with DER structure stripping |
| Reversibility | Fully reversible — this is encoding, not encryption |
| No secrets embedded | The executable contains no keys, passwords, or secrets |
| Deterministic output | Same input always produces the same encoded output |

---

## 5. Threat Model

### 5.1 In-Scope Threats (Mitigated)

| Threat | Mitigation |
|--------|------------|
| Zip bomb / resource exhaustion | 50 MB decompressed size limit |
| Malformed input causing crashes | Input validation on LSID, base64, and file sizes |
| Log file data leakage | Sensitive values redacted from all log output |
| Output file exposure | Restrictive file permissions on generated files |
| Previous export overwriting | Automatic rename with timestamp before generating new exports |
| Typos during manual key entry | Position-weighted checksum with warning on mismatch |
| Command injection via subprocess | List-form subprocess execution, no shell interpretation |

### 5.2 Out-of-Scope Threats

| Threat | Reason |
|--------|--------|
| Network-based attacks | Tool has no network functionality |
| Encryption of data at rest | Tool performs encoding (reversible by design), not encryption |
| Physical access to machine | Physical security is the responsibility of the deployment environment |
| Privilege escalation | Tool runs with the permissions of the invoking user |

---

## 6. Known Limitations

1. **Not encryption.** The encoded output is reversible by anyone with this tool. It is encoding for transport convenience, not confidentiality.
2. **Windows file permissions.** On Windows NTFS, `os.chmod()` has limited effect. File access is controlled by Windows ACLs, which default to user-profile-based access.
3. **Single-user tool.** The tool is designed for individual use, not multi-user or server deployment.

---

## 7. Security Audit History

| Version | Date | Audit Performed |
|---------|------|-----------------|
| v2.4.0 | 2026-04-17 | Full code review for OWASP categories: command injection, path traversal, zip slip, JSON parsing, log injection, file permissions, sensitive data exposure, denial of service. All identified vulnerabilities addressed. |
| v2.5.0 | 2026-04-18 | Added export file preservation to prevent data loss. |

---

## 8. Contact

For security concerns, contact:

**Shane Smith**
Senior Director of North America Government Technical Strategy
