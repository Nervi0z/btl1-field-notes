# Disk Analysis Tools

---

## Forensic suites

### Autopsy

Free, open-source, and the most accessible disk forensics platform for BTL1 work. It's a GUI front-end for The Sleuth Kit with additional modules for automated artifact extraction.

What it does well: loading disk images in multiple formats, navigating the filesystem including deleted files, keyword searching, timeline generation, and running ingest modules that automatically extract browser history, registry data, recent files, and metadata.

```
Basic workflow:
1. New Case → add Data Source (select disk image)
2. Configure Ingest Modules — enable what you need: Recent Activity,
   Hash Lookup, Keyword Search, EXIF Parser
3. Wait for processing
4. Explore: Data Artifacts, Web History, Recent Documents, Installed Programs
5. Use the Timeline view to correlate timestamps across artifact types
```

### FTK Imager

Free from Exterro (not the same as full FTK, which is commercial). Used primarily for acquisition and image previewing. Can mount images, preview file contents, capture memory, and export files — without modifying the image.

Not a full analysis suite, but useful for quickly browsing an image or exporting specific files for analysis with other tools.

---

## The Sleuth Kit (TSK)

The command-line engine behind Autopsy. Use TSK directly when you need precision, scripting, or when you want to inspect something at a lower level than Autopsy's GUI exposes.

```bash
# show file system information — type, block size, cluster size
fsstat -o <partition_offset> disk.dd

# list partitions in a disk image — get offsets for other TSK commands
mmls disk.dd

# list files and directories (including deleted — shown with *)
# -r = recursive, -l = long format, -o = partition offset in sectors
fls -r -l -o 2048 disk.dd

# list only deleted files
fls -r -d -o 2048 disk.dd

# extract a specific file by inode number (get inode from fls output)
icat -o 2048 disk.dd 12345 > recovered_file.exe

# find filename for a given inode
ffind -o 2048 disk.dd 12345

# recover all deleted files from a partition
tsk_recover -e -o 2048 disk.dd ./recovered_output/

# extract all MACB timestamps from the MFT
tsk_gettimes -o 2048 disk.dd > timeline.csv

# calculate the offset in bytes (multiply sector offset × 512)
# e.g., offset 2048 sectors × 512 = 1048576 bytes
```

> The `-o` offset comes from `mmls` output — it's the starting sector of the partition you want to analyze.

---

## Eric Zimmerman's EZ Tools

Free Windows tools that parse specific artifact types far faster and more thoroughly than anything built into Autopsy. The go-to for Windows artifact analysis.

| Tool | Parses |
| :--- | :--- |
| `PECmd.exe` | Prefetch files |
| `AmcacheParser.exe` | Amcache.hve |
| `AppCompatCacheParser.exe` | Shimcache from SYSTEM hive |
| `EvtxECmd.exe` | Windows Event Logs (.evtx) |
| `MFTECmd.exe` | $MFT, $UsnJrnl, $LogFile |
| `LECmd.exe` | LNK shortcut files |
| `JLECmd.exe` | Jumplists |
| `SBECmd.exe` | Shellbags |
| `RBCmd.exe` | Recycle Bin $I files |
| `Registry Explorer` / `RECmd.exe` | Registry hives |

```powershell
# Prefetch — parse all files in directory, output CSV
PECmd.exe -d C:\Windows\Prefetch\ --csv C:\output\ --csvf prefetch.csv

# Event logs — parse a single EVTX, output CSV
EvtxECmd.exe -f Security.evtx --csv C:\output\ --csvf security.csv

# MFT — parse and output CSV
MFTECmd.exe -f '$MFT' --csv C:\output\ --csvf mft.csv

# Amcache — parse hive
AmcacheParser.exe -f Amcache.hve --csv C:\output\

# Registry hive — search for a specific key
RECmd.exe -f NTUSER.DAT --kn "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# LNK files — parse all in a directory
LECmd.exe -d "C:\Users\user\AppData\Roaming\Microsoft\Windows\Recent\" --csv C:\output\
```

---

## KAPE (Kroll Artifact Parser and Extractor)

KAPE is a collection framework that automates two things: collecting specific artifacts from a live system or mounted image (Targets), and processing those artifacts with analysis tools (Modules). It's faster than manual collection and keeps everything organized.

```powershell
# collect registry hives from C: drive
kape.exe --tsource C: --tdest C:\output\collected\ --target RegistryHives

# collect from a mounted forensic image
kape.exe --tsource D: --tdest C:\output\collected\ --target !SANS_Triage

# collect and immediately process Prefetch with PECmd
kape.exe --tsource C: --tdest C:\output\collected\ --target Prefetch `
         --mdest C:\output\processed\ --module PECmd

# useful target names
# !SANS_Triage       — broad triage collection (many artifact types)
# RegistryHives      — all registry hives
# Prefetch           — prefetch files
# EventLogs          — Windows event logs
# $MFT               — Master File Table
# BrowserHistory     — Chrome, Firefox, Edge artifacts
```

KAPE targets and modules are community-maintained. Check the [KAPE GitHub](https://github.com/EricZimmerman/KapeFiles) for the full current list.

---

## Hex viewers

Use when you need to look at raw file content — verifying file type by magic bytes, identifying file format structures, finding hidden data, or examining unknown binary formats.

| Tool | Platform | Notes |
| :--- | :--- | :--- |
| HxD | Windows | Free. Fast, straightforward. Handles files up to several GB. |
| 010 Editor | Windows/Mac/Linux | Commercial, more powerful. Binary templates parse known file formats automatically. Worth having for complex formats. |
| `xxd` | Linux | Command-line hex dump. Good for scripting and piping. |
| `hexdump` | Linux | Similar to xxd. Built into most distributions. |

```bash
# hex dump with xxd
xxd suspicious_file.bin | head -20

# look for magic bytes at the start of a file
xxd suspicious_file.bin | head -2

# common magic bytes:
# FF D8 FF           → JPEG
# 25 50 44 46        → PDF (%PDF)
# 50 4B 03 04        → ZIP (and docx/xlsx/pptx)
# 4D 5A              → PE executable (MZ header)
# 7F 45 4C 46        → ELF (Linux executable)
# 89 50 4E 47        → PNG
```
