# SIEM — Key Concepts

---

## What a SIEM actually does

A SIEM ingests log data from every source in the environment — endpoints, firewalls, DNS servers, proxies, authentication systems — and puts it all in one place with a consistent schema and a query language to search it.

The four functions that matter for investigation work:

**Aggregation**: Logs from a Windows endpoint, a Palo Alto firewall, and a DNS server don't arrive in the same format. The SIEM collects them all. Without aggregation, correlation is impossible.

**Normalization**: Raw log data gets parsed into structured fields. A Windows Security Event log line and a Cisco ASA syslog line both contain source IPs — normalization maps both to a consistent field name so you can search across both sources with one query.

**Correlation**: Rules that watch for patterns across multiple events — five failed logins followed by a successful one, or a PowerShell process spawned by Word, or an outbound connection to a known malicious IP. Correlation is what generates alerts, but during investigation you're usually writing your own correlation queries rather than waiting for a rule to fire.

**Retention**: Logs are searchable over time. When you're investigating an incident that started three weeks ago, the SIEM is what makes it possible to see what was happening then.

---

## SPL basics

Splunk's Search Processing Language. Every query starts with a base search (what events you want) and passes results through a pipeline of commands.

**Search syntax fundamentals:**
```splunk
index="win_events" host="dc01" EventCode=4625
```

- `index=` — which data store to search. Required, or use `index=*` to search all (slower)
- `sourcetype=` — the type of log (e.g., `WinEventLog:Security`, `syslog`, `cisco:asa`)
- `host=` — which host generated the log
- bare keywords — full-text search across the raw event

**Boolean operators:**
```splunk
EventCode=4624 OR EventCode=4625
index=* NOT (host="internal-monitor")
(user="admin" OR user="administrator") AND src_ip!="10.0.0.0/8"
```

**Pipes — the pipeline:**
```splunk
index=* EventCode=4625
| stats count by Account_Name, IpAddress
| where count > 10
| sort -count
| head 20
```

Each `|` passes the current results to the next command. Order matters.

**Time:**
```splunk
earliest=-24h latest=now
earliest=-7d@d latest=@d                          ← start/end of yesterday
earliest="01/15/2024:00:00:00" latest="01/16/2024:00:00:00"
```

Always set the time range — either in the search bar or with the time picker. Searching `index=*` with no time range on a production Splunk instance returns too much data.

---

## ECS field mapping — Sysmon to common field names

Depending on how the Splunk environment is configured, the same data may use different field names. Sysmon events and Windows Security events have different native field names that get mapped to normalized names.

| Data | Sysmon field | Windows Security field | Common normalized |
| :--- | :--- | :--- | :--- |
| Process name | `Image` | `NewProcessName` | `process_name` |
| Parent process | `ParentImage` | `ParentProcessName` | `parent_process` |
| Command line | `CommandLine` | `CommandLine` | `process_command_line` |
| Source IP | `SourceIp` | `IpAddress` | `src_ip` |
| Destination IP | `DestinationIp` | — | `dest_ip` |
| User | `User` | `Account_Name` | `user` |
| Host | `Computer` | `ComputerName` | `host` |

Check your field names with `| fieldsummary` or by clicking on an event and expanding the field list.

---

## Log sources that matter in BTL1

**Windows Security Log** (`WinEventLog:Security`):
- Authentication events (4624, 4625, 4648, 4672)
- Account management (4720, 4732, 4728)
- Process creation if advanced auditing is on (4688)
- Log clearing (1102)

**Sysmon** (`XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`):
- Process creation with full command line (Event ID 1)
- Network connections (Event ID 3)
- File creation (Event ID 11)
- Registry changes (Event ID 13)
- Far more detail than native Windows auditing

**Windows System Log** (`WinEventLog:System`):
- Service installation (7045)
- Driver loads

**PowerShell** (`WinEventLog:Microsoft-Windows-PowerShell/Operational`):
- Script block logging (Event ID 4104) — captures full deobfuscated PowerShell

**DNS** — Query logs show what domains endpoints are trying to resolve. Long subdomains, high-entropy names, and unusual TLDs are DGA or tunneling indicators.

**Firewall/proxy** — Outbound connection logs. Look for connections to newly registered domains, direct IP connections (no domain lookup), and unusual user agents.
