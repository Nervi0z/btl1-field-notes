# Live Response — Windows

Full command reference for Windows triage. Run these on a live system before containment — volatile data is lost on shutdown or isolation. PowerShell preferred; CMD alternatives included where useful.

> Run PowerShell as Administrator. Some commands require elevation to return complete results.

---

## Running processes

```powershell
# process list with full paths and start times
Get-Process | Select-Object Name, Id, Path, StartTime, CPU, WorkingSet |
  Sort-Object StartTime -Descending | Format-Table -AutoSize

# include parent process ID (requires WMI)
Get-WmiObject Win32_Process |
  Select-Object Name, ProcessId, ParentProcessId, CommandLine, ExecutablePath |
  Sort-Object ProcessId | Format-Table -AutoSize

# find processes running from suspicious locations
Get-Process | Where-Object {$_.Path -match 'AppData|Temp|Users|ProgramData'} |
  Select-Object Name, Id, Path

# process with specific name
Get-Process -Name powershell | Select-Object Id, Path, StartTime
```

```cmd
:: CMD alternative
tasklist /v /fo csv
tasklist /svc
wmic process get Name,ProcessId,ParentProcessId,CommandLine,ExecutablePath
```

---

## Network connections

```powershell
# all TCP connections with PID
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress,
  RemotePort, State, OwningProcess | Sort-Object State | Format-Table -AutoSize

# TCP connections with process names
Get-NetTCPConnection |
  Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State,
    @{Name='Process'; Expression={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}},
    OwningProcess |
  Sort-Object State | Format-Table -AutoSize

# established connections only
Get-NetTCPConnection -State Established |
  Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort,
    @{Name='Process'; Expression={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}}

# listening ports
Get-NetTCPConnection -State Listen |
  Select-Object LocalAddress, LocalPort,
    @{Name='Process'; Expression={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}} |
  Sort-Object LocalPort

# UDP listeners
Get-NetUDPEndpoint |
  Select-Object LocalAddress, LocalPort,
    @{Name='Process'; Expression={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}}
```

```cmd
:: CMD — all connections with PIDs
netstat -ano
netstat -anob    :: includes executable name (requires elevation)
```

---

## Recently modified files

```powershell
# files modified in the last 24 hours (system-wide — slow on large drives)
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
  Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1) -and !$_.PSIsContainer} |
  Select-Object FullName, LastWriteTime, Length | Sort-Object LastWriteTime -Descending

# faster — check specific high-value locations
$paths = @("C:\Windows\Temp", "C:\Users", "C:\ProgramData", "$env:APPDATA", "$env:TEMP")
foreach ($path in $paths) {
  Get-ChildItem -Path $path -Recurse -ErrorAction SilentlyContinue |
    Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)} |
    Select-Object FullName, LastWriteTime
}

# files modified in the last 7 days in user directories
Get-ChildItem -Path C:\Users -Recurse -ErrorAction SilentlyContinue |
  Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)} |
  Select-Object FullName, LastWriteTime | Sort-Object LastWriteTime -Descending
```

---

## Scheduled tasks

```powershell
# all non-disabled scheduled tasks
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"} |
  Select-Object TaskName, TaskPath, State | Sort-Object TaskPath

# tasks with their actions (what they run)
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"} | ForEach-Object {
  $task = $_
  $task.Actions | ForEach-Object {
    [PSCustomObject]@{
      TaskName  = $task.TaskName
      State     = $task.State
      Execute   = $_.Execute
      Arguments = $_.Arguments
    }
  }
} | Format-Table -AutoSize

# get full details on a specific task
Get-ScheduledTask -TaskName "suspicious_task" | Get-ScheduledTaskInfo
(Get-ScheduledTask -TaskName "suspicious_task").Actions
```

```cmd
:: CMD alternatives
schtasks /query /fo LIST /v
schtasks /query /fo CSV > tasks.csv
```

---

## Services

```powershell
# running services
Get-Service | Where-Object {$_.Status -eq "Running"} |
  Select-Object Name, DisplayName, Status | Sort-Object Name

# all services including stopped
Get-Service | Select-Object Name, DisplayName, Status, StartType | Sort-Object Status

# services with executable paths (requires WMI)
Get-WmiObject Win32_Service |
  Select-Object Name, DisplayName, State, StartMode, PathName |
  Sort-Object State | Format-Table -AutoSize

# recently installed services (check System event log for Event ID 7045)
Get-WinEvent -LogName System |
  Where-Object {$_.Id -eq 7045} |
  Select-Object TimeCreated, Message | Sort-Object TimeCreated -Descending | Select-Object -First 20
```

---

## Startup entries

```powershell
# registry Run keys — current user
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"

# registry Run keys — all users (HKLM)
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"
Get-ItemProperty "HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"

# startup folders
Get-ChildItem "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup"
Get-ChildItem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

```cmd
:: Autoruns (Sysinternals) — most complete startup enumeration
autorunsc.exe -a * -ct -h -s -u -vt > autoruns_output.csv
```

---

## Local users and groups

```powershell
# local users
Get-LocalUser | Select-Object Name, Enabled, LastLogon, PasswordLastSet, Description

# local administrators group members
Get-LocalGroupMember -Group "Administrators"

# all local groups and members
Get-LocalGroup | ForEach-Object {
  $group = $_.Name
  Get-LocalGroupMember -Group $group -ErrorAction SilentlyContinue |
    Select-Object @{n='Group';e={$group}}, Name, ObjectClass
}
```

```cmd
net user
net localgroup administrators
net localgroup
```

---

## Recent logon events

```powershell
# successful logons — last 50 events
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]" -MaxEvents 50 |
  ForEach-Object {
    $xml = [xml]$_.ToXml()
    [PSCustomObject]@{
      Time        = $_.TimeCreated
      User        = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'TargetUserName'} | Select-Object -ExpandProperty '#text'
      LogonType   = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'LogonType'} | Select-Object -ExpandProperty '#text'
      SourceIP    = $xml.Event.EventData.Data | Where-Object {$_.Name -eq 'IpAddress'} | Select-Object -ExpandProperty '#text'
    }
  } | Format-Table -AutoSize

# failed logons — last 50
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]" -MaxEvents 50 |
  Select-Object TimeCreated, Message | Sort-Object TimeCreated -Descending
```

---

## PowerShell history

```powershell
# current user PowerShell history file location
(Get-PSReadLineOption).HistorySavePath

# read the history file
Get-Content (Get-PSReadLineOption).HistorySavePath

# all users' PowerShell history files
$users = Get-ChildItem C:\Users -Directory
foreach ($user in $users) {
  $histPath = "$($user.FullName)\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"
  if (Test-Path $histPath) {
    Write-Host "=== $($user.Name) ===" -ForegroundColor Yellow
    Get-Content $histPath
  }
}
```

---

## Prefetch quick check

```powershell
# list Prefetch files sorted by last write time (most recently executed first)
Get-ChildItem C:\Windows\Prefetch -Filter *.pf |
  Sort-Object LastWriteTime -Descending |
  Select-Object Name, LastWriteTime | Head 30

# check if a specific executable has a prefetch entry
Get-ChildItem C:\Windows\Prefetch -Filter "POWERSHELL*"
Get-ChildItem C:\Windows\Prefetch -Filter "CMD*"
Get-ChildItem C:\Windows\Prefetch -Filter "MIMIKATZ*"
```

---

## ARP cache

```powershell
# ARP cache — recently connected hosts
Get-NetNeighbor | Select-Object IPAddress, LinkLayerAddress, State | Sort-Object IPAddress
```

```cmd
arp -a
```

---

## DNS cache

```powershell
# cached DNS entries — shows what domains this host has resolved recently
Get-DnsClientCache | Select-Object Name, Type, Data, TimeToLive | Sort-Object Name
```

```cmd
ipconfig /displaydns
```

---

## Quick hash check on suspicious files

```powershell
# hash a file for VirusTotal lookup
Get-FileHash -Algorithm SHA256 C:\Users\user\AppData\Temp\suspicious.exe
Get-FileHash -Algorithm MD5 C:\Users\user\AppData\Temp\suspicious.exe

# hash all executables in a directory
Get-ChildItem C:\Windows\Temp -Filter *.exe |
  Get-FileHash -Algorithm SHA256 | Select-Object Hash, Path
```
