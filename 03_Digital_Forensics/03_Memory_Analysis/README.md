# Memory Analysis

A memory dump captures the running state of a system at the moment of acquisition — processes, network connections, loaded modules, registry keys in use, credentials, and code that never touched the disk. That last point is why memory analysis finds things disk analysis misses: fileless malware, packed payloads that unpack only in RAM, and rootkits that hide themselves from the filesystem but can't hide from a raw memory scan.

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [Key_Concepts.md](Key_Concepts.md) | What lives in RAM by category, what to look for, Volatility plugin names |
| [Volatility_Tool.md](Volatility_Tool.md) | V2 vs V3, full plugin reference, filtering output, complete investigation sequences |

---

## Quick reference

**Volatility 3 — investigation starting point:**

```bash
# identify OS version and architecture
python3 vol.py -f memory.raw windows.info.Info

# process list — standard view
python3 vol.py -f memory.raw windows.pslist.PsList

# process tree — shows parent-child relationships
python3 vol.py -f memory.raw windows.pstree.PsTree

# scan for process structures (finds hidden/terminated processes)
python3 vol.py -f memory.raw windows.psscan.PsScan

# command-line arguments for all processes
python3 vol.py -f memory.raw windows.cmdline.CmdLine

# network connections and listening ports
python3 vol.py -f memory.raw windows.netscan.NetScan

# suspicious memory regions (injected code candidates)
python3 vol.py -f memory.raw windows.malfind.Malfind

# registry hives loaded in memory
python3 vol.py -f memory.raw windows.registry.hivelist.HiveList

# dump a process executable
python3 vol.py -f memory.raw -o /output/ windows.procdump.ProcDump --pid <PID>
```

**Filtering Volatility output:**
```bash
# grep for a specific process name
python3 vol.py -f memory.raw windows.pslist.PsList | grep -i 'powershell\|cmd\|wscript'

# grep for a specific port
python3 vol.py -f memory.raw windows.netscan.NetScan | grep ':4444\|:8080\|:1337'

# save output to file for analysis
python3 vol.py -f memory.raw windows.pslist.PsList > pslist_output.txt
```

---

> For Linux memory dumps, replace `windows.` plugin prefix with `linux.`. Start with `Key_Concepts.md` to understand what you're looking for before running plugins.
