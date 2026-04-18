Citrix LAS Offline Restricted Shortener v2.6.0


PURPOSE

This tool was built to solve a specific problem: transferring structured
data (an LSID and RSA public key) between two machines that cannot share
files directly. Instead of copying long strings of cryptographic data by
hand, this tool compresses the data into a chunked key — similar to a
software license key — that a person can read, type, or dictate to
someone at the other machine.

The encoded key uses Crockford Base32, an alphabet specifically designed
to avoid characters that look alike (no O/0 confusion, no I/1/L mix-ups).
It also includes a checksum so typos are caught before decoding.



HOW IT WORKS

The tool strips known structural overhead from the data (ASN.1 headers,
PEM formatting, hex encoding) and packs only the truly unique bytes into
a compact binary form. That binary is then encoded as printable characters
and formatted into numbered, chunked lines for easy reading.

The decoder reverses every step exactly, rebuilding the full original
data string from the typed key.



EXPORT LOCATION

  The Citrix LAS Offline Activation Tool generates the export file
  (darksiteRequest.zip) to the following location:

    C:\Program Files (x86)\Citrix\Licensing\LS\resource\cache

  This is where the tool writes the darksiteRequest.zip file when
  Option A ("Generate Export & Encode") is used. You can also
  navigate to this folder manually to find the file for Option B.



STEP-BY-STEP INSTRUCTIONS

  Encoding (on the machine where the data file lives):

    Option A — Generate and encode in one step:
    1. Click the "Encode" tab above.
    2. Click "Generate Export & Encode". This runs the Citrix LAS
       Offline Activation Tool to create the darksiteRequest.zip
       file at the export location shown above, then immediately
       encodes it.
    3. The encoded key will appear on screen.
    4. A timestamped copy of the export file, along with key.txt
       and data.txt, are saved to your Desktop.

    Option B — Encode an existing file:
    1. Click the "Encode" tab above.
    2. Click "Browse" and select your input file. You can navigate
       to the export location above, or select any of these:
       - A .zip archive containing a JSON request file
       - A .json request file directly
       - A .txt file containing the "data":{...} segment
    3. Click the "Encode" button.
    4. The encoded key will appear on screen, formatted as numbered
       lines of chunked characters (e.g., 01: ABCDE-FGHJK-...).
    5. Timestamped output files are saved automatically to the same
       folder as your input file:
       - key_YYYYMMDD_HHMMSS.txt  — the encoded key
       - data_YYYYMMDD_HHMMSS.txt — the extracted data segment

    6. Communicate the key to the person at the other machine.
       You can read it aloud, send it over chat, or write it down.
       The key is designed to be easy to read and type.


  Decoding (on the other machine):

    1. Click the "Decode" tab above.
    2. Enter the encoded key using one of these methods:
       - Click "Load key.txt" to browse for a saved key file, OR
       - Type or paste the key lines directly into the text box
    3. Click the "Decode" button.
    4. The original data will appear on screen.
    5. A data.txt file is saved automatically containing the
       decoded data.

  Tip: The decoder is forgiving about formatting. You can type the key
  with or without line numbers, dashes, or spaces. Upper and lowercase
  both work. Even common look-alike mistakes (typing O instead of 0, or
  I instead of 1) are handled automatically.



KEY DETAILS

  - Encoded keys are approximately 467-473 characters, formatted
    across 19 lines of 5-character chunks.
  - The last 2 characters of the key are a checksum. If you make a
    typo, the decoder will warn you before attempting to decode.
  - No internet connection is required. Everything runs locally.
  - No data is sent anywhere. The tool is fully offline.
  - The source code (shorten.py) is fully commented and open for
    inspection.


COMMAND LINE (CLI) MODE

  This tool can also be run from the command line without the GUI.
  Open a terminal, navigate to the folder containing the executable,
  and pass arguments directly:

  Encode:
    CitrixLASOfflineRestrictedShortener_v2.6.0.exe encode -f request.zip
    CitrixLASOfflineRestrictedShortener_v2.6.0.exe encode -f data.json
    CitrixLASOfflineRestrictedShortener_v2.6.0.exe encode -f data.txt
    CitrixLASOfflineRestrictedShortener_v2.6.0.exe encode

  Decode:
    CitrixLASOfflineRestrictedShortener_v2.6.0.exe decode -f key.txt
    CitrixLASOfflineRestrictedShortener_v2.6.0.exe decode "01: XXXXX-..."

  When run with no arguments, the GUI opens (this window).
  When run with arguments, it operates in CLI mode with output
  printed to the terminal.


