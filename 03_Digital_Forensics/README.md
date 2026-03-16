# Digital Forensics

Disk and memory analysis — reconstructing what happened on a system after the fact. You're working from forensic images and memory dumps, not live systems, and the goal is building a timeline: what ran, what changed, what was accessed, and when.

---

## What's in this section

| File / Folder | What it covers |
| :--- | :--- |
| [01_Acquisition.md](01_Acquisition.md) | Image formats, dd/dcfldd commands, memory acquisition, integrity hashing |
| [02_Disk_Analysis/](02_Disk_Analysis/) | Windows and Linux artifacts, TSK commands, KAPE, file carving, ExifTool |
| [03_Memory_Analysis/](03_Memory_Analysis/) | What lives in RAM, Volatility 3 full plugin reference, investigation sequences |

---

## Quick reference

**Volatility 3 — first commands on a new dump:**
```bash
# identify OS, version, architecture
python3 vol.py -f memory.raw windows.info.Info

# process list
python3 vol.py -f memory.raw windows.pslist.PsList

# network connections
python3 vol.py -f memory.raw windows.netscan.NetScan

# processes with suspicious memory regions
python3 vol.py -f memory.raw windows.malfind.Malfind
```

**Key Windows artifact locations:**

| Artifact | Location |
| :--- | :--- |
| Prefetch | `C:\Windows\Prefetch\*.pf` |
| Event logs | `C:\Windows\System32\winevt\Logs\` |
| SYSTEM hive | `C:\Windows\System32\config\SYSTEM` |
| NTUSER.DAT | `C:\Users\<user>\NTUSER.DAT` |
| Amcache | `C:\Windows\AppCompat\Programs\Amcache.hve` |
| $MFT | Root of NTFS volume |
| LNK files | `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\` |

**Key Event IDs:**

| Event ID | Log | What happened |
| :--- | :--- | :--- |
| 4624 | Security | Successful logon |
| 4625 | Security | Failed logon |
| 4688 | Security | Process creation |
| 4698 | Security | Scheduled task created |
| 4720 | Security | User account created |
| 7045 | System | New service installed |
| 1102 | Security | Audit log cleared |
| 4104 | PowerShell | Script block logging |

**Disk acquisition — quick syntax:**
```bash
sudo dd if=/dev/sdb of=/mnt/evidence/disk.dd bs=4M status=progress
sha256sum /mnt/evidence/disk.dd > disk.dd.sha256
```

---

> New to forensics? Start with `01_Acquisition.md`, then `02_Disk_Analysis/Windows_Artifacts.md`. For memory work, go straight to `03_Memory_Analysis/Volatility_Tool.md`.
