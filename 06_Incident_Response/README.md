# Incident Response

IR is what ties everything else together — phishing analysis identifies the initial access vector, forensics shows what the attacker did, SIEM and network analysis fill in the timeline, and IR is the process of managing the response: confirming scope, containing the threat, removing persistence, and recovering safely.

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [01_IR_Lifecycle.md](01_IR_Lifecycle.md) | Six phases, what BTL1 tests at each, common failure modes |
| [02_Live_Response_Windows.md](02_Live_Response_Windows.md) | Full PowerShell/CMD command reference for Windows triage |
| [03_Live_Response_Linux.md](03_Live_Response_Linux.md) | Full command reference for Linux triage |
| [04_Containment_Eradication.md](04_Containment_Eradication.md) | Network isolation, account lockout, persistence removal, verification |
| [05_Practice_Resources.md](05_Practice_Resources.md) | IR lab platforms, tabletop resources, NIST/SANS frameworks |

---

## Quick reference

**Windows live response — run in order:**
```powershell
# processes with full paths
Get-Process | Select-Object Name, Id, Path, StartTime | Sort-Object StartTime -Descending

# network connections with PIDs
netstat -ano

# map PID to process name
Get-NetTCPConnection | Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,State,
  @{n='Process';e={(Get-Process -Id $_.OwningProcess).Name}} | Sort-Object State

# scheduled tasks
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"} | Select-Object TaskName,TaskPath,State

# services
Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object Name,DisplayName,Status
```

**Linux live response — run in order:**
```bash
# running processes
ps aux --sort=-%mem | head 20

# network connections
ss -tulnp

# recently modified files
find / -type f -mtime -1 -not -path '/proc/*' -not -path '/sys/*' 2>/dev/null | head 30

# cron jobs all users
for user in $(cut -d: -f1 /etc/passwd); do echo "==$user=="; crontab -l -u $user 2>/dev/null; done
```

---

> Use `02_Live_Response_Windows.md` and `03_Live_Response_Linux.md` as command lookups during active triage. Start with `01_IR_Lifecycle.md` to understand the full process.
