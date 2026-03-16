# Volatility

The de facto standard framework for memory forensics. Open-source, Python-based, and plugin-driven — each plugin extracts a specific type of information from a raw memory dump. This file covers Volatility 3, with V2 equivalents noted throughout.

---

## V2 vs V3

| | Volatility 2 | Volatility 3 |
| :--- | :--- | :--- |
| Python version | Python 2 (Python 3 forks exist) | Python 3 native |
| OS detection | Requires manual `--profile` specification | Automatic — uses symbol tables |
| Profile syntax | `--profile=Win10x64_17763` | Not required |
| Plugin naming | `pslist`, `netscan` | `windows.pslist.PsList`, `windows.netscan.NetScan` |
| Symbol tables | Not needed | Auto-downloaded or manually placed |
| Status | Legacy, still widely used | Active development, recommended |

**V2 basic syntax:**
```bash
python vol.py --profile=<Profile> -f memory.raw <plugin> [options]
# Profile examples: Win7SP1x64, Win10x64_17763, LinuxDebian10x64
python vol.py -f memory.raw imageinfo   # get suggested profiles
```

**V3 basic syntax:**
```bash
python3 vol.py -f memory.raw <namespace.PluginName> [options]
```

> Check which version the BTL1 environment provides before the exam. Syntax differs enough that running V2 commands against V3 (or vice versa) will fail silently or with confusing errors.

---

## Basic workflow

```bash
# 1. identify OS, architecture, and build
python3 vol.py -f memory.raw windows.info.Info

# for Linux dumps
python3 vol.py -f memory.raw linux.info.Info

# V2 equivalent
python vol.py -f memory.raw imageinfo
# note the "Suggested Profile(s)" line — use the first match
```

If Volatility 3 can't find symbol tables automatically:
```bash
# place downloaded symbol packs in:
volatility3/symbols/windows/   # for Windows symbols
volatility3/symbols/linux/     # for Linux symbols

# download from: https://github.com/volatilityfoundation/symbol-packs
```

---

## Full plugin reference

Organized by investigation category. V3 syntax shown; V2 equivalent in the right column.

### Processes

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| List active processes | `windows.pslist.PsList` | `pslist` |
| Process tree (parent-child) | `windows.pstree.PsTree` | `pstree` |
| Scan for hidden/terminated | `windows.psscan.PsScan` | `psscan` |
| Command-line arguments | `windows.cmdline.CmdLine` | `cmdline` |
| Loaded DLLs | `windows.dlllist.DllList` | `dlllist` |
| Open handles | `windows.handles.Handles` | `handles` |
| Environment variables | `windows.envars.Envars` | `envars` |
| Process memory map | `windows.memmap.Memmap` | `memmap` |
| Dump process executable | `windows.procdump.ProcDump` | `procdump` |
| Dump full process memory | `windows.dumpfiles.DumpFiles` | `memdump` |

### Network

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| All connections and sockets | `windows.netscan.NetScan` | `netscan` |
| Active connections | `windows.netstat.NetStat` | `connections` |
| Scan for closed TCP connections | — | `connscan` |

### Registry

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| List loaded hives | `windows.registry.hivelist.HiveList` | `hivelist` |
| Print a registry key's values | `windows.registry.printkey.PrintKey` | `printkey` |
| Dump a hive to disk | `windows.registry.hivedump.HiveDump` | `hivedump` |
| Search for a key/value | `windows.registry.userassist.UserAssist` | `userassist` |

### System modules and services

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| Loaded kernel modules | `windows.modules.Modules` | `modules` |
| Scan for hidden modules | `windows.modscan.ModScan` | `modscan` |
| Registered services | `windows.svcscan.SvcScan` | `svcscan` |
| Driver objects | `windows.driverscan.DriverScan` | `driverscan` |

### Files

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| File objects in memory | `windows.filescan.FileScan` | `filescan` |
| Extract cached files | `windows.dumpfiles.DumpFiles` | `dumpfiles` |

### Malware detection

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| Injected/suspicious memory | `windows.malfind.Malfind` | `malfind` |
| DLL presence check (injection) | `windows.ldrmodules.LdrModules` | `ldrmodules` |
| YARA rule scan | `yarascan.YaraScan` | `yarascan` |
| SSDT hooks | — | `ssdt` |

### User activity

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| cmd.exe command history | `windows.cmdscan.CmdScan` | `cmdscan` |
| Console output history | `windows.console.ConsoleInfo` | `consoles` |
| Clipboard content | `windows.clipboard.Clipboard` | `clipboard` |
| NTLM hash dump | `windows.hashdump.Hashdump` | `hashdump` |

### Linux-specific

| Task | V3 Plugin | V2 Plugin |
| :--- | :--- | :--- |
| Process list | `linux.pslist.PsList` | `linux_pslist` |
| Process tree | `linux.pstree.PsTree` | `linux_pstree` |
| Network connections | `linux.netstat.NetStat` | `linux_netstat` |
| Bash history | `linux.bash.Bash` | `linux_bash` |
| Loaded kernel modules | `linux.lsmod.Lsmod` | `linux_lsmod` |
| Injected code | `linux.malfind.Malfind` | `linux_malfind` |

---

## Filtering output

```bash
# filter pslist for specific process names
python3 vol.py -f memory.raw windows.pslist.PsList | grep -iE 'powershell|cmd|wscript|mshta'

# filter netscan for suspicious ports
python3 vol.py -f memory.raw windows.netscan.NetScan | grep -E ':4444|:1337|:8080|ESTABLISHED'

# filter netscan for external connections (exclude common local ranges)
python3 vol.py -f memory.raw windows.netscan.NetScan | grep -v '127\.\|0\.0\.0\.0\|::1'

# save output to file
python3 vol.py -f memory.raw windows.pslist.PsList > pslist.txt

# filter malfind results to only those with MZ header (actual executables)
python3 vol.py -f memory.raw windows.malfind.Malfind | grep -A5 'MZ'
```

---

## Complete investigation sequence — suspicious process

Starting from nothing. Adapt the PID and details to your actual dump.

```bash
# Step 1 — identify OS and confirm the dump is readable
python3 vol.py -f memory.raw windows.info.Info

# Step 2 — get process list and look for anomalies
python3 vol.py -f memory.raw windows.pslist.PsList | tee pslist.txt

# Step 3 — get process tree to see parent-child relationships
python3 vol.py -f memory.raw windows.pstree.PsTree | tee pstree.txt

# Step 4 — check command-line arguments for all processes
# look for encoded commands, unusual paths, suspicious arguments
python3 vol.py -f memory.raw windows.cmdline.CmdLine | tee cmdline.txt

# Step 5 — check network connections
# look for established outbound connections from unusual processes
python3 vol.py -f memory.raw windows.netscan.NetScan | tee netscan.txt

# Step 6 — check for injected code in the suspicious process (PID from step 2)
python3 vol.py -f memory.raw windows.malfind.Malfind --pid <PID> | tee malfind_pid.txt

# Step 7 — dump the process executable for further analysis
mkdir -p ./process_dumps/
python3 vol.py -f memory.raw -o ./process_dumps/ windows.procdump.ProcDump --pid <PID>

# hash the dumped executable and look it up in VirusTotal
sha256sum ./process_dumps/pid.<PID>.*.exe

# Step 8 — check what DLLs the process loaded (look for unsigned or unusual paths)
python3 vol.py -f memory.raw windows.dlllist.DllList --pid <PID>

# Step 9 — check registry for persistence
python3 vol.py -f memory.raw windows.registry.printkey.PrintKey \
  --key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Step 10 — check cmd history for commands the attacker ran
python3 vol.py -f memory.raw windows.cmdscan.CmdScan
python3 vol.py -f memory.raw windows.console.ConsoleInfo
```
