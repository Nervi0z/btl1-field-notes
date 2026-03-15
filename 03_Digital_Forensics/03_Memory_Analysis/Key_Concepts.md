# Memory Analysis — Key Concepts

What you find in a memory dump, organized by category — with what to look for in each and the Volatility plugins that surface it.

---

## Processes

**What's there**: Every process running at the moment of acquisition — its name, PID, parent PID, creation time, and exit time if it's terminated. Process structures persist in memory even after a process exits, which is why `psscan` often finds more processes than `pslist`.

**What to look for**:
- Processes with names that mimic system processes with slight variations (`svchost.exe` is legitimate, `svch0st.exe` is not)
- `cmd.exe` or `powershell.exe` spawned by unusual parents — `winword.exe`, `excel.exe`, `outlook.exe` spawning a shell is a strong malware indicator
- Multiple instances of processes that should only run once
- Processes running from unexpected paths — `svchost.exe` should run from `C:\Windows\System32\`, not `C:\Users\user\AppData\`
- Processes with no parent (PPID doesn't match any known process) — sign of process injection or rootkit behavior

| Plugin (V3) | Plugin (V2) | What it returns |
| :--- | :--- | :--- |
| `windows.pslist.PsList` | `pslist` | Active process list from the doubly-linked list |
| `windows.pstree.PsTree` | `pstree` | Same data in tree format — shows parent-child |
| `windows.psscan.PsScan` | `psscan` | Scans for EPROCESS structures — finds hidden/terminated |
| `windows.cmdline.CmdLine` | `cmdline` | Command-line arguments used to launch each process |
| `windows.dlllist.DllList` | `dlllist` | DLLs loaded by each process |
| `windows.handles.Handles` | `handles` | Open handles: files, registry keys, mutexes, threads |
| `windows.procdump.ProcDump` | `procdump` | Dump process executable from memory to disk |

---

## Network activity

**What's there**: Active TCP and UDP connections, listening ports, and recently closed connections (in varying states depending on OS). Each entry includes source/dest IP:port, connection state, and the PID of the owning process.

**What to look for**:
- Connections to external IPs on unusual ports — 4444, 1337, 8080, 8443 are common C2 ports
- Processes that shouldn't have network access making outbound connections — `svchost.exe` connecting outbound to a non-Microsoft IP on port 443 is suspicious
- Listening ports on high port numbers tied to unusual processes
- UDP connections — often used for DNS-based C2

| Plugin (V3) | Plugin (V2) | What it returns |
| :--- | :--- | :--- |
| `windows.netscan.NetScan` | `netscan` | All TCP/UDP connections and sockets |
| `windows.netstat.NetStat` | `connections` | Active connections (fewer states than netscan) |

---

## Registry (Windows)

**What's there**: Registry hives loaded into memory — the live, in-use versions of the hive files on disk. These may contain data not yet flushed to the disk hive files, and can be extracted and analyzed offline.

**What to look for**:
- Run keys with suspicious values — base64-encoded PowerShell, paths to temp directories
- Services referencing executables in unusual locations

| Plugin (V3) | Plugin (V2) | What it returns |
| :--- | :--- | :--- |
| `windows.registry.hivelist.HiveList` | `hivelist` | Lists registry hives and their memory addresses |
| `windows.registry.printkey.PrintKey` | `printkey` | Prints values of a specific key |
| `windows.registry.hivedump.HiveDump` | `hivedump` | Dumps a full hive to disk for offline analysis |

```bash
# print Run key values from memory
python3 vol.py -f memory.raw windows.registry.printkey.PrintKey \
  --key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

---

## Credentials

**What's there**: Password hashes for local accounts, extracted from the SAM hive as loaded in memory. On systems running older credential caching, cleartext or reversibly-encrypted credentials may also be present.

**What to look for**:
- NTLM hashes for accounts — crack offline or use in pass-the-hash analysis
- Multiple accounts with the same hash (password reuse)

| Plugin (V3) | Plugin (V2) | What it returns |
| :--- | :--- | :--- |
| `windows.hashdump.Hashdump` | `hashdump` | LM/NTLM hashes from SAM in memory |

---

## User activity

**What's there**: Commands typed into `cmd.exe` or PowerShell consoles, clipboard content at the time of acquisition.

**What to look for**:
- Encoded PowerShell commands (`-EncodedCommand` or `-enc` flag followed by base64)
- Credential-related commands: `net user`, `mimikatz`, `whoami /all`
- Download commands: `Invoke-WebRequest`, `certutil -urlcache`, `bitsadmin`
- Clipboard content that shows recently copied data

| Plugin (V3) | Plugin (V2) | What it returns |
| :--- | :--- | :--- |
| `windows.cmdscan.CmdScan` | `cmdscan` | Commands typed in cmd.exe consoles |
| `windows.console.ConsoleInfo` | `consoles` | Full console output history |
| `windows.clipboard.Clipboard` | `clipboard` | Clipboard contents at time of dump |

---

## Malware indicators

**What's there**: Memory regions that don't map to known files on disk, or have unusual permission combinations (executable AND writable is abnormal — legitimately loaded code should be executable but not writable). These indicate code injection.

**What to look for**:
- Memory regions marked `PAGE_EXECUTE_READWRITE` — legitimate mapped code is almost never writable
- Regions in process space that don't correspond to a file on disk (`ldrmodules` shows this gap)
- Process hollowing artifacts — a process whose memory content doesn't match its on-disk executable

| Plugin (V3) | Plugin (V2) | What it returns |
| :--- | :--- | :--- |
| `windows.malfind.Malfind` | `malfind` | Suspicious memory regions with MZ headers or abnormal permissions |
| `windows.ldrmodules.LdrModules` | `ldrmodules` | DLLs not in the loader list (potential injection) |
| `windows.filescan.FileScan` | `filescan` | File objects in memory — includes deleted or hidden files |
| `windows.dumpfiles.DumpFiles` | `dumpfiles` | Extract cached file contents from memory |
| `yarascan.YaraScan` | `yarascan` | Scan memory with YARA rules |

---

## Why memory finds what disk analysis misses

A rootkit that hooks the filesystem API will hide its files from any tool that uses the normal Windows file enumeration calls — but a memory dump bypasses the OS entirely. Volatility reads raw memory structures directly, not through the OS layer that the rootkit has compromised.

Fileless malware that runs only in memory and never writes a file to disk leaves no artifact for disk forensics to find. The process, its network connections, and its command-line arguments exist only in RAM.

Packed malware unpacks itself into memory before executing. The on-disk binary is obfuscated; the in-memory version is the actual executable code. `malfind` and `procdump` give you access to the unpacked version.
