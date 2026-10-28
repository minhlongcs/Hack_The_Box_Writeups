#sherlock #DFIR #windows-forensics #NTFS #MFT #UsnJrnl #CVE-2025-8088 #WinRAR
Difficulty: Very Easy
Categories: DFIR
Description: Susan works at the Research Lab in Forela International Hospital. A Microsoft Defender alert was received from her computer, and she also mentioned that while extracting a document from the received file, she received tons of errors, but the document opened just fine. According to the latest threat intel feeds, WinRAR is being exploited in the wild to gain initial access into networks, and WinRAR is one of the Software programs the staff uses. You are a threat intelligence analyst with some background in DFIR. You have been provided a lightweight triage image to kick off the investigation while the SOC team sweeps the environment to find other attack indicators.

- Personal Summary:
    - New terms, things to learn: CVE-2025-8088 (WinRAR path traversal via ADS), USN Journal (`$UsnJrnl:$J`), mounting a `.vhdx`, Recent-folder `.lnk` files as evidence of opening, Startup-folder persistence (T1547.009)
    - Goals: Identify the CVE, find the malicious archive + when it was created/opened, find the decoy document, the dropped backdoor, the persistence file, and when the decoy was opened.

## Learning

- **CVE-2025-8088 (WinRAR, Windows)**: a **path traversal** bug abused through **Alternate Data Streams (ADS)**. The attacker builds a RAR whose visible file is a harmless decoy, but hides extra files in ADS whose names contain `..\..\` sequences. When WinRAR extracts the archive, those streams get written **outside** the extraction folder (e.g. `%LOCALAPPDATA%`, the Startup folder). Exploited in the wild in 2025 by **RomCom** (and Paper Werewolf).
    - The "tons of errors" Susan saw = WinRAR complaining about the hidden/bogus stream paths it couldn't write, while the decoy still opened fine → classic sign of this exploit.
    - Because everything comes from ADS of the same decoy, **all dropped files are created at the same moment as the decoy** → look for FileCreate events clustered around the decoy's creation time.
    - Lots of **WinRAR WebView** file activity between the archive being opened and the document being extracted is another sign the exploit ran.
- **USN Journal (`$Extend\$UsnJrnl:$J`)**: NTFS change log. Each record = file name + **Update Reason** (`FileCreate`, `DataExtend`, `RenameNewName`, `FileDelete`, `Close`...) + **Update Timestamp**. Great for "when was X created" and for seeing everything created in a short time window. Only stores the file name + parent entry number → correlate with the MFT to get the full path.
- **MFT**: see [[BFT]]. Gives full paths and `Created0x10` / `Created0x30` timestamps.
- **Recent `.lnk` files**: when a user opens a file, Windows creates a shortcut in `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\`. The `.lnk` creation time ≈ **when the file was first opened** (used for Q5 and Q10).
- **Startup folder persistence (MITRE T1547.009 – Shortcut Modification)**: anything in `...\Start Menu\Programs\Startup\` runs at user logon. Dropping a `.lnk` there pointing at the backdoor = persistence + execution.
- **VHDX**: Hyper-V virtual disk format. KAPE-style triage images are often delivered as `.vhdx`.

## Key tools + functions

- **Mount the VHDX on Linux** (qemu-nbd):
    - `sudo modprobe nbd` (if `/dev/nbd0` doesn't exist)
    - `sudo qemu-nbd --connect=/dev/nbd0 2025-09-02T083211_pathology_department_incidentalert.vhdx`
    - `sudo fdisk -l /dev/nbd0` → find the partition (here `/dev/nbd0p1`)
    - `sudo mkdir -p /mnt/romcom && sudo mount -o ro /dev/nbd0p1 /mnt/romcom` (`-o ro` = read-only, don't touch the evidence)
    - Inside: `$MFT` and `$Extend` (holds `$J`).
    - Cleanup: `sudo umount /mnt/romcom && sudo qemu-nbd --disconnect /dev/nbd0`
- **Mount on Windows**: right-click the `.vhdx` → Mount → it shows up as a new drive.
- **MFTECmd** (Eric Zimmerman):
    - MFT: `MFTECmd.exe -f "E:\C\$MFT" --csv C:\out --csvf mft.csv`
    - USN Journal: `MFTECmd.exe -f "E:\C\$Extend\$J" --csv C:\out --csvf usn.csv` (add `-m "E:\C\$MFT"` to resolve parent paths)
- **Timeline Explorer** functions used:
    - `Extension` filter = `.rar`, `Parent Path` filter / search bar = `Susan\Documents` → the archive.
    - `Update Reasons` filter = `FileCreate` (on the USN CSV) → everything created right after the decoy.
    - `Extension` filter = `.pdf` / `.docx` in the archive's folder → decoy document.
    - Read `Created0x10` / `Update Timestamp` columns for times.
- Google / NVD (nvd.nist.gov) for the CVE; MITRE ATT&CK Enterprise matrix (Boot or Logon Autostart Execution) for the technique ID.

## Doing the sherlock

Artifacts: `RomCom.zip` (password `hacktheblue`) → `2025-09-02T083211_pathology_department_incidentalert.vhdx` (incident date = 2025-09-02, from the triage image name).

### Task 1: What is the CVE assigned to the WinRAR vulnerability exploited by the RomCom threat group in 2025?

- Google "WinRAR RomCom 2025 CVE".
- Answer: `CVE-2025-8088`

### Task 2: What is the nature of this vulnerability?

- Confirm on NVD.
- Answer: `Path Traversal`

### Task 3: What is the name of the archive file under Susan's documents folder that exploits the vulnerability upon opening the archive file?

- MFT CSV → `Extension` = `.rar` + Parent Path contains `Susan\Documents`, on the incident date.
- Answer: `Pathology-Department-Research-Records.rar`

### Task 4: When was the archive file created on the disk?

- USN CSV → record for the `.rar` with Update Reason `FileCreate` → Update Timestamp (also matches the MFT created time).
- Answer: `2025-09-02 08:13:50`

### Task 5: When was the archive file opened?

- USN (or MFT) → the `.lnk` shortcut for the archive (Recent folder) with `FileCreate` → its timestamp = time opened.
- Answer: `2025-09-02 08:14:04`

### Task 6: What is the name of the decoy document extracted from the archive file, meant to appear legitimate and distract the user?

- MFT → same Documents folder, filter for `.pdf`/`.docx` → the PDF right next to the `.rar`. Confirm with USN `FileCreate` times right after the archive was opened (lots of WinRAR WebView activity in between).
- Answer: `Genotyping_Results_B57_Positive.pdf`

### Task 7: What is the name and path of the actual backdoor executable dropped by the archive file?

- USN → `FileCreate` events created at the same time as the decoy → a suspicious `.exe`. Search its name in the MFT for the full path.
- Answer: `C:\Users\Susan\Appdata\Local\ApbxHelper.exe`

### Task 8: The exploit also drops a file to facilitate the persistence and execution of the backdoor. What is the path and name of this file?

- Same `FileCreate` cluster → a `.lnk` → full path from the MFT = Startup folder.
- Answer: `C:\Users\Susan\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\Display Settings.lnk`

### Task 9: What is the associated MITRE Technique ID discussed in the previous question?

- MITRE → Boot or Logon Autostart Execution → Shortcut Modification (procedure examples mention writing `.lnk` files to the Startup folder).
- Answer: `T1547.009`

### Task 10: When was the decoy document opened by the end user, thinking it to be a legitimate document?

- MFT → the `.lnk` shortcut for the PDF (listed right under it) → its creation time.
- Answer: `2025-09-02 08:15:05`

## Takeaways / IOCs

- Timeline (2025-09-02, UTC):
    - 08:13:50 → `Pathology-Department-Research-Records.rar` lands in `C:\Users\Susan\Documents\`
    - 08:14:04 → archive opened in WinRAR
    - extraction → decoy `Genotyping_Results_B57_Positive.pdf` + (via ADS path traversal) `C:\Users\Susan\AppData\Local\ApbxHelper.exe` + `...\Startup\Display Settings.lnk`
    - 08:15:05 → Susan opens the decoy PDF
    - next logon → `Display Settings.lnk` runs `ApbxHelper.exe` (persistence, T1547.009)
- Workflow to remember for "what did this archive drop?" cases: **mount image → MFTECmd on `$MFT` + `$J` → find archive in MFT → USN `FileCreate` cluster around the decoy → resolve full paths in MFT → Recent `.lnk` files for "when opened".**
- Defense: patch WinRAR to 7.13+, alert on new files in Startup folders / `%LOCALAPPDATA%` executables created by `WinRAR.exe`.

Source: HTB official RomCom writeup (VivisGhost, 2025-09-03).
