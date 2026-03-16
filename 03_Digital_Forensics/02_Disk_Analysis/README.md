# Disk Analysis

Working from a forensic image to reconstruct system and user activity. The question at every step is the same: what does this artifact tell me about what happened, and when?

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [Windows_Artifacts.md](Windows_Artifacts.md) | Registry hives, Event IDs, Prefetch, Shimcache, Amcache, LNK, NTFS artifacts |
| [Linux_Artifacts.md](Linux_Artifacts.md) | /var/log files, bash history, cron, user accounts, /proc, suspicious locations |
| [Disk_Tools.md](Disk_Tools.md) | Autopsy, TSK commands, EZ Tools, KAPE, hex viewers |
| [File_Carving_Scalpel.md](File_Carving_Scalpel.md) | Recovering files from unallocated space using Scalpel |
| [Metadata_ExifTool.md](Metadata_ExifTool.md) | Extracting embedded metadata from files with ExifTool |

---

## Quick reference

**High-value Windows artifact locations:**

| Artifact | Path |
| :--- | :--- |
| Security event log | `C:\Windows\System32\winevt\Logs\Security.evtx` |
| Prefetch | `C:\Windows\Prefetch\` |
| SYSTEM hive | `C:\Windows\System32\config\SYSTEM` |
| SAM hive | `C:\Windows\System32\config\SAM` |
| SOFTWARE hive | `C:\Windows\System32\config\SOFTWARE` |
| NTUSER.DAT | `C:\Users\<user>\NTUSER.DAT` |
| UsrClass.dat | `C:\Users\<user>\AppData\Local\Microsoft\Windows\UsrClass.dat` |
| Amcache | `C:\Windows\AppCompat\Programs\Amcache.hve` |
| LNK files | `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\` |
| Jumplists | `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` |
| Recycle Bin | `C:\$Recycle.Bin\<user_SID>\` |

**TSK quick reference:**
```bash
# list partitions in a disk image
mmls disk.dd

# list files in a partition (offset from mmls output)
fls -r -o <offset> disk.dd

# extract a file by inode number
icat -o <offset> disk.dd <inode_number> > output_file

# recover deleted files
tsk_recover -e -o <offset> disk.dd ./recovered_files/
```

**EZ Tools — parse Prefetch:**
```powershell
PECmd.exe -d C:\Windows\Prefetch\ --csv C:\output\
```

---

> Start with `Windows_Artifacts.md` for a full artifact reference. Use `Disk_Tools.md` when you need command syntax for Autopsy, TSK, or KAPE.
