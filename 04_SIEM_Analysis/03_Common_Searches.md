# Common Detection Searches — Splunk SPL

Ready-to-adapt queries. Adjust `index`, `sourcetype`, and field names to match your environment. Field names vary between Splunk deployments depending on how logs are parsed.

---

## Authentication and brute force

```splunk
| Failed logins by user — find accounts being targeted
| Adjust threshold: >10 failures is suspicious in most environments
index=* EventCode=4625
| stats count by Account_Name, IpAddress, Workstation_Name
| where count > 10
| sort -count

| Failed logins by source IP — find attackers spraying credentials
index=* EventCode=4625
| stats count, dc(Account_Name) as unique_accounts by IpAddress
| where count > 20
| sort -count

| Successful login after multiple failures — brute force success
| Threshold: >5 failures before success from same source
index=* (EventCode=4624 OR EventCode=4625)
| stats count(eval(EventCode=4625)) as failures,
        count(eval(EventCode=4624)) as successes by Account_Name, IpAddress
| where failures > 5 AND successes > 0
| sort -failures

| RDP brute force — Logon Type 10 failures
index=* EventCode=4625 LogonType=10
| stats count by Account_Name, IpAddress
| where count > 5
| sort -count

| Password spray — one source targeting many accounts
index=* EventCode=4625
| stats dc(Account_Name) as accounts_targeted, count by IpAddress
| where accounts_targeted > 10
| sort -accounts_targeted
```

---

## Process creation anomalies

```splunk
| Office applications spawning shells — strong malware indicator
| Adjust: add mspub.exe, msaccess.exe if in scope
index=* EventCode=4688
(ParentProcessName="*\\winword.exe" OR ParentProcessName="*\\excel.exe"
 OR ParentProcessName="*\\outlook.exe" OR ParentProcessName="*\\powerpnt.exe")
(NewProcessName="*\\cmd.exe" OR NewProcessName="*\\powershell.exe"
 OR NewProcessName="*\\wscript.exe" OR NewProcessName="*\\cscript.exe"
 OR NewProcessName="*\\mshta.exe" OR NewProcessName="*\\rundll32.exe")
| table _time, ComputerName, Account_Name, ParentProcessName, NewProcessName, CommandLine

| Encoded PowerShell — base64 payload being executed
index=* EventCode=4688 (CommandLine="* -enc *" OR CommandLine="* -EncodedCommand *"
                        OR CommandLine="* -e JAB*" OR CommandLine="* -e SQB*")
| table _time, ComputerName, Account_Name, CommandLine

| PowerShell download cradles — remote code execution via download
index=* EventCode=4688 CommandLine IN ("*Invoke-WebRequest*", "*DownloadString*",
  "*DownloadFile*", "*IEX*", "*Invoke-Expression*", "*Net.WebClient*")
| table _time, ComputerName, Account_Name, CommandLine

| Living-off-the-land binary abuse — LOLBins running unusual commands
index=* EventCode=4688
NewProcessName IN ("*\\certutil.exe", "*\\bitsadmin.exe", "*\\regsvr32.exe",
                   "*\\msiexec.exe", "*\\wmic.exe", "*\\rundll32.exe")
| table _time, ComputerName, Account_Name, NewProcessName, CommandLine

| Sysmon variant — same logic with Event ID 1
index=* source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
(ParentImage="*\\winword.exe" OR ParentImage="*\\excel.exe")
(Image="*\\powershell.exe" OR Image="*\\cmd.exe")
| table _time, Computer, User, ParentImage, Image, CommandLine
```

---

## Persistence

```splunk
| Scheduled task creation
index=* EventCode=4698
| table _time, ComputerName, Account_Name, TaskName, TaskContent
| sort -_time

| New service installation
index=* EventCode=7045
| table _time, ComputerName, ServiceName, ServiceFileName, ServiceType, StartType
| sort -_time

| Registry Run key modification — persistence via autorun
| Adjust: add HKCU path if needed
index=* EventCode=4657
ObjectName IN ("*\\CurrentVersion\\Run*", "*\\CurrentVersion\\RunOnce*",
               "*\\Winlogon*", "*\\Image File Execution Options*")
| table _time, ComputerName, Account_Name, ObjectName, NewValue

| Sysmon registry persistence detection
index=* source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=13
TargetObject IN ("*\\CurrentVersion\\Run*", "*\\CurrentVersion\\RunOnce*",
                  "*\\Services\\*", "*\\Winlogon\\*")
| table _time, Computer, User, TargetObject, Details
```

---

## Lateral movement

```splunk
| PsExec usage — remote execution via ADMIN$ share
index=* EventCode=4688
(NewProcessName="*\\PSEXESVC.exe" OR CommandLine="*psexec*" OR CommandLine="*\\ADMIN$*")
| table _time, ComputerName, Account_Name, NewProcessName, CommandLine

| WMI remote execution
index=* EventCode=4688
(NewProcessName="*\\WmiPrvSE.exe" OR ParentProcessName="*\\WmiPrvSE.exe")
| table _time, ComputerName, Account_Name, ParentProcessName, NewProcessName, CommandLine

| Pass-the-hash indicator — network logon with no Kerberos
index=* EventCode=4624 LogonType=3 AuthenticationPackageName="NTLM"
NOT Account_Name IN ("ANONYMOUS LOGON", "*$")
| stats count by Account_Name, IpAddress, WorkstationName
| sort -count

| Unusual SMB connections between internal hosts
| Adjust: replace 10.0.0.0/8 with your internal range
index=* (EventCode=4624 OR EventCode=5140) LogonType=3
NOT (IpAddress IN ("10.*") OR IpAddress="127.0.0.1" OR IpAddress="::1")
| table _time, ComputerName, Account_Name, IpAddress, ShareName
```

---

## Data staging and exfiltration

```splunk
| Large file transfers — adjust threshold to match environment baseline
index=proxy bytes > 50000000
| stats sum(bytes) as total_bytes by src_ip, dest_ip, url
| eval total_mb=round(total_mb/1024/1024, 2)
| sort -total_bytes

| DNS queries with abnormally long hostnames — potential DNS tunneling
| Adjust: threshold of 50 chars catches most tunneling
index=dns
| eval query_length=len(query)
| where query_length > 50
| stats count by query, src_ip
| sort -count

| Repeated outbound connections to the same external IP — C2 beaconing
index=proxy
| stats count, dc(url) as unique_urls by src_ip, dest_ip
| where count > 100 AND unique_urls < 5
| sort -count

| Unusual outbound ports from internal hosts
index=firewall action=allowed
NOT (dest_port IN (80, 443, 53, 25, 587, 993, 995))
| stats count by src_ip, dest_port, dest_ip
| sort -count
```

---

## Log clearing and defense evasion

```splunk
| Security log cleared — high priority alert
index=* EventCode=1102
| table _time, ComputerName, Account_Name, SubjectUserName

| System log cleared
index=* EventCode=104
| table _time, ComputerName, Account_Name

| Sysmon service stopped or uninstalled
index=* EventCode=7036 ServiceName="Sysmon*" Message="*stopped*"
| table _time, ComputerName

| Shadow copy deletion — ransomware precursor
index=* EventCode=4688
(CommandLine="*vssadmin*delete*shadows*" OR CommandLine="*wmic*shadowcopy*delete*"
 OR CommandLine="*bcdedit*/set*recoveryenabled*no*")
| table _time, ComputerName, Account_Name, CommandLine
```

---

> All thresholds are starting points. Tune based on your baseline — what looks like brute force in a 10-person org might be normal authentication noise in an enterprise. Document the threshold you used and why in your report.
