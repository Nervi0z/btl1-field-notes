# SIEM Analysis

Logs from across the environment — endpoints, network, authentication — aggregated into one place with a query language to cut through them. In BTL1, that means Splunk and SPL. The skill being tested isn't knowing Splunk exists; it's being able to write a query that surfaces the five events that matter out of fifty thousand that don't.

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [01_Key_Concepts.md](01_Key_Concepts.md) | What a SIEM does, SPL basics, ECS field mapping, log sources |
| [02_Splunk_Cheatsheet.md](02_Splunk_Cheatsheet.md) | SPL command reference — stats, eval, rex, timechart, subsearches |
| [03_Common_Searches.md](03_Common_Searches.md) | Ready-to-adapt detection queries for common attack scenarios |
| [04_Practice_Resources.md](04_Practice_Resources.md) | BOTS datasets, practice platforms, Splunk training |

---

## Quick reference

**Detection queries — run these first on an unknown incident:**

```splunk
| Failed logins — who's being targeted and from where
index=* EventCode=4625
| stats count by Account_Name, IpAddress
| sort -count

| Process anomalies — Office spawning shells
index=* EventCode=4688
(ParentProcessName="*\\winword.exe" OR ParentProcessName="*\\excel.exe"
 OR ParentProcessName="*\\outlook.exe")
(NewProcessName="*\\cmd.exe" OR NewProcessName="*\\powershell.exe"
 OR NewProcessName="*\\wscript.exe")
| table _time, ComputerName, ParentProcessName, NewProcessName, CommandLine

| Encoded PowerShell
index=* EventCode=4688 CommandLine="* -enc *"
| table _time, ComputerName, User, CommandLine
```

**SPL quick syntax:**

```splunk
| base search
index="win_events" EventCode=4624 LogonType=10
| filter after pipe
| where count > 5
| aggregate
| stats count by src_ip, user | sort -count
| format output
| table _time, user, src_ip, dest_ip | head 20
```

**Time modifiers:**
```splunk
earliest=-24h latest=now
earliest=-7d@d latest=@d
earliest="01/15/2024:00:00:00" latest="01/16/2024:00:00:00"
```

---

> Start with `03_Common_Searches.md` if you need detection queries fast. Read `01_Key_Concepts.md` for SPL fundamentals and log source context.
