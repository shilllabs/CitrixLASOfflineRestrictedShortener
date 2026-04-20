# Release Notes

## Citrix LAS Offline Restricted Shortener

---

## v3.0.0 — 2026-04-20 (Current)

### EULA Acceptance and Instructions Overhaul

- Added EULA/disclaimer acceptance requirement for both GUI and CLI.
- **GUI:** Displays a license agreement dialog on launch. User must click "I Accept" before the application opens. Declining or closing the dialog exits the tool.
- **CLI:** Requires `--acceptEULA` switch. Without it, the tool prints the full license terms and exits with an error. The rejection is logged.
- Completely rewritten Instructions page with detailed 3-step workflow covering encode, decode, and activation with all sub-steps, export locations, verification hash explanation, CLI usage examples, and key format details.

---

## v2.9.2 — 2026-04-20

### Activation File Type Fix

- Fixed activation file type from `.tgz` to `.zip`.
- The activation response file from the Citrix portal is named `<LSID>_activation.blob.zip`, not `.tgz`.
- Updated file browser filter to show `*_activation.blob.zip` and `*.zip` files.
- Updated all documentation references.

---

## v2.9.1 — 2026-04-20

### Path Normalization Fix

- Fixed file path normalization for the Citrix tool.
- Tkinter file dialogs return forward slashes (`C:/Users/...`) but the Citrix native exe requires Windows backslashes (`C:\Users\...`).
- All file paths are now normalized with `os.path.normpath()` before being passed to the Citrix tool or used for file operations.
- This fixes the "failed finding central directory" error when importing activation files.

---

## v2.9.0 — 2026-04-20

### 3-Step Workflow and License Proxy Activation

- Reorganized tabs into a clear 3-step workflow:
  - **Step 1: Encode** — generate and encode on the License Server
  - **Step 2: Decode** — decode the key on the machine with portal access
  - **Step 3: Activate** — import the activation response file
- Added "Step 3: License Proxy Activation" tab that imports the `activation.blob.zip` file by running:
  ```
  ctxLasOfflineActivationTool.exe -activation -response -import <filepath>
  ```
- Includes timeout handling (120 seconds), permission error handling, and success/failure output display.
- Updated Instructions page to describe all three steps in order.

---

## v2.8.0 — 2026-04-20

### LSID and Public Key Fields with Copy Buttons

- Added separate LSID and Public Key fields on the Decode tab.
- After decoding, each value appears in its own labeled text box.
- Each field has a "Copy" button to copy the value directly to the clipboard for pasting into the Citrix licensing portal.
- Public Key is displayed with proper line breaks for readability.

---

## v2.7.0 — 2026-04-20

### Verification Hash

- Added verification hash (truncated SHA-256, 8 characters) displayed during both encode and decode operations.
- Encode shows: `Verification Hash: 29DC3DF0`
- Decode shows: `Verification Hash: 29DC3DF0`
- If both hashes match, the key was typed correctly. If they don't match, there's a typo somewhere in the key entry.
- Hash is recorded in the log file for traceability.

---

## v2.6.0 — 2026-04-18

### CLI Verification and Hardening

- Verified and hardened all CLI commands (encode, decode, generate) through the compiled executable entry point.
- Full round-trip testing confirmed data integrity across encode/decode cycles via both GUI and CLI modes.
- All CLI commands produce timestamped output files and log entries consistent with GUI behavior.

---

## v2.5.0 — 2026-04-18

### Export File Preservation

- Before generating a new export, any existing `darksiteRequest.zip` at the Citrix cache location is automatically renamed with its last-modified timestamp (e.g., `darksiteRequest_20260417_143000.zip`).
- This preserves previous exports instead of overwriting them.
- The rename operation is logged to the activity log for traceability.

---

## v2.4.0 — 2026-04-17

### Security Hardening

Addresses all vulnerabilities identified in a full security audit:

- **Zip bomb protection:** Decompressed file size checked before reading (50MB limit) to prevent memory exhaustion from malicious archives.
- **JSON file size limit:** Files over 10MB are rejected before parsing.
- **LSID input validation:** Must be exactly 64 hexadecimal characters. Non-hex characters are rejected.
- **Base64 decode hardening:** Malformed PEM key data produces clear error messages instead of unhandled exceptions.
- **Sensitive data redaction:** Log entries record operation summaries and character counts, never the actual LSID or key content.
- **Restrictive file permissions:** Output files (key, data) are created with owner-only read/write access (mode 0600 on supported systems).

---

## v2.3.0 — 2026-04-17

### Timestamped Filenames and Logging System

- Output files now include date-timestamps in filenames to prevent overwriting previous results (e.g., `key_20260417_143000.txt`).
- Added logging system: activities are recorded to a timestamped log file (`shorten_YYYYMMDD_HHMMSS.log`).
- Added "Log" tab in the GUI with a real-time scrollable view of all activity for troubleshooting.
- Logs record: file reads, encode/decode operations, Citrix tool execution, file saves, and errors.

---

## v2.2.0 — 2026-04-17

### Generate Export and Encode Feature

- Added "Generate Export & Encode" feature to the Encode tab.
- Runs the Citrix `ctxLasOfflineActivationTool.exe` to generate the `darksiteRequest.zip` export file automatically.
- Copies the export file to the Desktop, then encodes it in one step.
- Also available as a CLI command: `generate`.
- Encode tab redesigned with two clearly labeled options: Option A (Generate & Encode) and Option B (Browse for file).

---

## v2.1.0 — 2026-04-17

### CLI Mode, About, and Release Notes

- Combined GUI and CLI into a single executable. Double-click for the graphical interface, or pass command-line arguments for terminal use.
- Added About tab with author information and Citrix legal disclaimer.
- Added Release Notes tab with full version history.

---

## v2.0.0 — 2026-04-17

### Graphical User Interface

- Introduced graphical user interface (GUI) with tabbed layout.
- Instructions landing page with step-by-step usage guide.
- Encode tab: browse for a file, view encoded key, auto-save `key.txt` and `data.txt` to the source file's directory.
- Decode tab: paste a key or load `key.txt`, view decoded data, auto-save `data.txt`.
- Renamed executable to `CitrixLASOfflineRestrictedShortener`.
- Version number displayed in window title bar and Instructions page.

---

## v1.1.0 — 2026-04-17

### ZIP File Support

- Added `.zip` file support for the encode command.
- The tool now accepts a `.zip` archive directly as input. It opens the archive, locates the JSON file inside, extracts the "data" segment, and encodes it automatically.
- Also accepts `.json` files directly, extracting the "data" segment without manual preparation.
- Output files (`key.txt`, `data.txt`) saved alongside the input file.

---

## v1.0.0 — 2026-04-17

### Initial Release

- Crockford Base32 chunked key encoding with DER structure stripping.
- Encodes LSID + RSA 2048 public key data into approximately 467 characters formatted across 19 lines of 5-character chunks.
- Strips 38 known bytes of ASN.1/DER structure from the public key, reducing output length.
- Forgiving decoder: accepts uppercase/lowercase, ignores dashes and spaces, auto-corrects common look-alike characters (O to 0, I/L to 1).
- 2-character position-weighted checksum detects typos before decoding.
- Supports standard RSA 2048 keys with exponent 65537 (optimized) and non-standard exponents (fallback).
- Pure Python standard library — no external dependencies.
- Command-line interface with encode and decode commands.
