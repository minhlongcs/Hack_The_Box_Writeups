#sherlock #DFIR #windows-forensics #NTFS #MFT
Difficulty: Very Easy

Description: Simon Stark was targeted by attackers on 13 Feb 2024. He downloaded a ZIP from a link in an email. We only get the `$MFT` from a KAPE triage collection (`2024-02-13T164623_MFT_TRIAGE.zip`) and must use it to trace the download, find the stager, and recover the C2 address.

- Personal Summary:
    - New terms, things to learn: Master File Table (MFT), NTFS attributes, Alternate Data Streams (ADS) / Zone.Identifier, MFT resident files, MFT record offsets
    - Goals: Find the downloaded ZIP + its source URL, find the malicious file, build a timeline, recover the stager's contents (C2 IP:port) straight from the MFT.

## Learning

- **MFT (Master File Table)**: the "database" of an NTFS volume. Each file/folder = 1 MFT entry (record), identified by an **Entry Number**. Every record is **1024 bytes**. Entries of deleted files often survive → useful for recovery.
- **Key MFT attributes**:
    - `$STANDARD_INFORMATION` (0x10): Created / Modified / Accessed / Entry-Modified timestamps → timeline. Easy to timestomp.
    - `$FILE_NAME` (0x30): file name, parent directory record, its own set of timestamps (harder to tamper with → compare with 0x10 to spot timestomping).
    - `$DATA`: the file content itself (small files) or pointers to clusters (big files).
    - Others: `$LOGGED_UTILITY_STREAM` (TxF), `$BITMAP`, `$SECURITY_DESCRIPTOR`, `$VOLUME_INFORMATION`, `$INDEX_ROOT`/`$INDEX_ALLOCATION` (directory indexes).
- **Alternate Data Stream (ADS)**: an extra hidden data stream attached to a file. When a browser downloads a file, Windows adds the `Zone.Identifier` ADS (Mark-of-the-Web) holding `ZoneId`, `ReferrerUrl` and **`HostUrl`** → where the file came from (great IOC).
- **Resident file**: if a file is small enough (~< 900 bytes, depends on metadata), its content lives *inside* its MFT record → can be read from `$MFT` alone, even without the disk.
- **Record offset** = `Entry Number × 1024` (decimal) → convert to hex to jump there in a hex editor.

## Key tools + functions

- **MFTECmd** (Eric Zimmerman): parse a raw `$MFT` into CSV.
    - `MFTECmd.exe -f "C:\path\$MFT" --csv "C:\output\dir" --csvf mft.csv`
    - `-f` input file, `--csv` output folder, `--csvf` output file name.
- **Timeline Explorer** (Eric Zimmerman): open the CSV and filter/sort. Functions used:
    - Column filter on `Created0x10` / `Created0x30` → pick a date (13/02/2024).
    - `Extension` filter = `.zip` → find downloaded archives (ignore `KAPE.zip` = collection tool).
    - `File Name` filter = `stage` → find the `...:Zone.Identifier` ADS entry, read the **Zone Id Contents** column → HostUrl.
    - `Parent Path` filter = `stage` → list everything extracted from the ZIP → spotted `invoice.bat`.
    - Read `Entry Number`, `File Size`, `Created0x30` columns of the suspicious file.
- **MFTExplorer**: GUI alternative for browsing MFT records (slow to load on big MFTs).
- **HxD** (hex editor): open the raw `$MFT` → `Search > Go To` → enter the hex offset → read the resident `$DATA` of the record (the .bat content).
- Calculator / any dec→hex converter for the offset.
- (Linux alternative for the same job: `analyzeMFT`, and `xxd -s 0x16E3000 -l 1024 '$MFT'` instead of HxD.)

## Doing the sherlock

### Question 1: Name of the ZIP downloaded from the email link?

- Filter `Created0x10` = 2024-02-13 + `Extension` = zip → `Stage-20240213T093324Z-001.zip`, `invoices.zip` (nested inside Stage), `KAPE.zip` (ignore).
- Answer: `Stage-20240213T093324Z-001.zip`

### Question 2: Full HostUrl the ZIP was downloaded from?

- Filter File Name = `stage` → entry with extension `.Identifier` (Zone.Identifier ADS) → Zone Id Contents.
- Answer: `https://storage.googleapis.com/drive-bulk-export-anonymous/20240213T093324.039Z/4133399871716478688/a40aecd0-1cf3-4f88-b55a-e188d5c1c04f/1/c277a8b4-afa9-4d34-b8ca-e1eb5e5f983c?authuser` (Google Drive used to host it)

### Question 3: Full path of the malicious file that connected to C2?

- Filter Parent Path = `stage` → batch file.
- Answer: `C:\Users\simon.stark\Downloads\Stage-20240213T093324Z-001\Stage\invoice\invoices\invoice.bat`

### Question 4: Created0x30 timestamp of that file?

- Answer: `2024-02-13 16:38:39`

### Question 5: Offset (hex) of the stager's MFT record?

- Entry Number = 23436 → 23436 × 1024 = 23998464 (dec) → hex
- Answer: `16E3000`

### Question 6: C2 IP and port from the stager's contents?

- invoice.bat is tiny → MFT resident. HxD → Go To `16E3000` → see `invoice.bat` file name + a PowerShell one-liner reverse shell.
- Answer: `43.204.110.203:6666`

## Takeaways / IOCs

- Attack chain: phishing email link → ZIP from Google Drive → nested `invoices.zip` → `invoice.bat` → PowerShell → C2 `43.204.110.203:6666`.
- Workflow to remember for MFT cases: **MFTECmd → Timeline Explorer (date + extension + path filters) → Zone.Identifier for source URL → Entry Number × 1024 → hex editor for resident content.**
