# Windows Forensic Artifacts

Windows leaves extensive traces of system and user activity across the registry, event logs, the filesystem, and NTFS metadata structures. Knowing where to look — and what each artifact actually tells you — is the core of Windows disk forensics.

---

## Registry hives

The registry is a hierarchical database that records low-level configuration for the OS, applications, and users. For forensics, it's one of the richest sources of evidence about what ran, what connected, and what persisted.

| Hive | Location | What's forensically relevant |
| :--- | :--- | :--- |
| SAM | `C:\Windows\System32\config\SAM` | Local user accounts, password hashes, group memberships |
| SECURITY | `C:\Windows\System32\config\SECURITY` | Security policies, cached logon credentials |
| SOFTWARE | `C:\Windows\System32\config\SOFTWARE` | Installed software, run keys, network history |
| SYSTEM | `C:\Windows\System32\config\SYSTEM` | Hardware config, services, USB device history, timezone |
| NTUSER.DAT | `C:\Users\<user>\NTUSER.DAT` | Per-user settings, recent files, typed paths, UserAssist |
| UsrClass.dat | `C:\Users\<user>\AppData\Local\Microsoft\Windows\UsrClass.dat` | Shellbags, COM settings |
| Amcache.hve | `C:\Windows\AppCompat\Programs\Amcache.hve` | Program execution history with SHA1 hashes |

**High-value registry keys:**

```
# Persistence — programs that run at startup
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce

# Services — each service definition, including malicious ones
HKLM\SYSTEM\CurrentControlSet\Services\

# Program execution — AppCompatCache (Shimcache)
HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache

# USB devices — vendor ID, product ID, serial number
HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR\

# Network interfaces and history
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged

# Typed URLs in IE/Edge Legacy
HKCU\SOFTWARE\Microsoft\Internet Explorer\TypedURLs

# UserAssist — GUI program execution (ROT13 encoded)
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\
```

---

## Event logs

| Log file | Records |
| :--- | :--- |
| `Security.evtx` | Logon/logoff, account management, object access, policy changes, privilege use |
| `System.evtx` | Service installs, driver loads, system errors, shutdown/startup |
| `Application.evtx` | Application-specific events and crashes |
| `Microsoft-Windows-Sysmon/Operational.evtx` | Process creation, network connections, registry changes (requires Sysmon) |
| `Microsoft-Windows-PowerShell/Operational.evtx` | PowerShell activity |
| `Microsoft-Windows-TaskScheduler/Operational.evtx` | Scheduled task creation, execution, deletion |
| `Microsoft-Windows-WMI-Activity/Operational.evtx` | WMI query and subscription activity |

**Key Event IDs:**

| Event ID | Log | What happened | Note |
| :--- | :--- | :--- | :--- |
| 4624 | Security | Successful logon | Check Logon Type — Type 3 = network, Type 10 = remote interactive |
| 4625 | Security | Failed logon | Multiple failures = brute force attempt |
| 4634 | Security | Logoff | |
| 4648 | Security | Logon with explicit credentials | Pass-the-hash, runas, lateral movement indicator |
| 4672 | Security | Special privileges assigned at logon | Admin/SYSTEM logon |
| 4688 | Security | Process creation | Requires advanced audit policy; shows parent and child process |
| 4698 | Security | Scheduled task created | |
| 4702 | Security | Scheduled task updated | |
| 4720 | Security | User account created | |
| 4726 | Security | User account deleted | |
| 4732 | Security | User added to security group | |
| 7045 | System | New service installed | Malware persistence via services |
| 1102 | Security | Security audit log cleared | Evidence of log tampering |
| 4104 | PowerShell | Script block logged | Captures full PowerShell commands including deobfuscated |
| 1 | Sysmon | Process creation | CommandLine, ParentCommandLine, Hashes |
| 3 | Sysmon | Network connection | Source/dest IP, port, process |
| 11 | Sysmon | File created | |
| 13 | Sysmon | Registry value set | |

**Logon Type reference for Event 4624:**

| Type | Meaning |
| :--- | :--- |
| 2 | Interactive (local keyboard) |
| 3 | Network (file share, net use) |
| 4 | Batch (scheduled task) |
| 5 | Service logon |
| 7 | Unlock (screen unlock) |
| 10 | Remote Interactive (RDP) |
| 11 | Cached interactive (domain user, no DC available) |

---

## Program execution artifacts

### Prefetch

Location: `C:\Windows\Prefetch\*.pf`

Records: Executable name, run count, last 8 execution timestamps, files and directories loaded at runtime.

Note: Disabled by default on SSDs in some Windows versions. Check `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters`.

```powershell
# Parse with EZ Tools
PECmd.exe -f "C:\Windows\Prefetch\POWERSHELL.EXE-XXXXXXXX.pf"
PECmd.exe -d "C:\Windows\Prefetch\" --csv C:\output\
```

### Shimcache (AppCompatCache)

Location: `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache`

Records: Full path and last modified timestamp of executables. Does not record execution time — only that the file *was* on the system and was examined by the compatibility layer. Read from memory with Volatility `windows.shimcachemem` plugin or from the hive offline.

```powershell
AppCompatCacheParser.exe -f SYSTEM --csv C:\output\
```

### Amcache

Location: `C:\Windows\AppCompat\Programs\Amcache.hve`

Records: Program execution evidence with SHA1 hash of the executable, install time, file path, publisher. The SHA1 hash is the most forensically valuable field — look it up in VirusTotal even if the file has been deleted.

```powershell
AmcacheParser.exe -f Amcache.hve --csv C:\output\
```

### UserAssist

Location: `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\`

Records: GUI program execution — run count and last run time. Keys are ROT13 encoded.

```powershell
# Parse with Registry Explorer (Eric Zimmerman)
# or use RegRipper with the userassist plugin
```

---

## File and folder access

### LNK files

Location: `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\`

Windows automatically creates LNK shortcut files when a user opens a file or application. LNK files contain: the original file path, timestamps of the target file at time of access, and volume information (drive type, serial number) — even if the target was on a removable drive that's since been removed.

```powershell
LECmd.exe -d "C:\Users\user\AppData\Roaming\Microsoft\Windows\Recent\" --csv C:\output\
```

### Jumplists

Location: `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\`

Similar data to LNK files, organized per application. Each file is named by AppID of the application that generated it.

```powershell
JLECmd.exe -d "C:\Users\user\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\" --csv C:\output\
```

### Shellbags

Location: `NTUSER.DAT` and `UsrClass.dat`

Records folder navigation through Windows Explorer — including folders that have since been deleted or were on removable media. This is evidence that a user *browsed* a specific directory, which can corroborate file access or data staging activity.

```powershell
SBECmd.exe -d C:\Users\user\ --csv C:\output\
```

---

## NTFS artifacts

### $MFT (Master File Table)

The index of the entire NTFS volume. Every file and directory has at least one MFT record containing: filename, timestamps (Created, Modified, Accessed, Entry Modified — MACB), size, attributes, and for small files, the data itself. MFT records persist after deletion (marked as unused, not immediately overwritten) — a key source for file recovery and timeline building.

```powershell
MFTECmd.exe -f "$MFT" --csv C:\output\
```

### $UsnJrnl (Update Sequence Number Journal)

Location: `$Extend\$UsnJrnl`

A log of changes to files and directories — creation, deletion, renaming, attribute changes. Useful for tracking file activity over a time period even when other artifacts have been cleared. The `$J` data stream contains the actual records.

```powershell
MFTECmd.exe -f "$UsnJrnl" --csv C:\output\
```

### ADS (Alternate Data Streams)

NTFS allows multiple data streams per file. Malware uses ADS to hide executables or configuration data — the file shows normal in Explorer but carries additional content that won't be visible without specifically querying for streams.

```cmd
# list ADS on a file (CMD)
dir /r suspicious_file.txt

# list ADS with PowerShell
Get-Item -Path .\suspicious_file.txt -Stream *

# read a specific ADS
Get-Content -Path .\suspicious_file.txt -Stream hidden_stream
```

---

## Recycle Bin

Location: `C:\$Recycle.Bin\<user_SID>\`

Contains `$I` files (metadata: original path and deletion timestamp) paired with `$R` files (actual file content). Even after a user empties the recycle bin, forensic recovery of `$I` and `$R` pairs from unallocated space is possible.

```powershell
RBCmd.exe -d "C:\$Recycle.Bin\" --csv C:\output\
```
