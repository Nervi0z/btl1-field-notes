# Splunk Cheatsheet

SPL command reference. All examples use real syntax — adapt the index, sourcetype, and field names to your environment.

---

## Basic search structure

```splunk
index="win_events" sourcetype="WinEventLog:Security" EventCode=4625
| <command1>
| <command2>
```

---

## Time modifiers

```splunk
earliest=-15m latest=now
earliest=-1h@h latest=@h          ← last complete hour
earliest=-24h latest=now
earliest=-7d@d latest=@d          ← last 7 complete days
earliest=-30d latest=now
earliest="01/15/2024:08:00:00" latest="01/15/2024:16:00:00"
```

---

## Transforming commands

```splunk
| stats count by user, src_ip
| stats count, dc(user) as unique_users by host
| stats values(CommandLine) as cmds by ParentProcessName
| stats avg(bytes) as avg_bytes, max(bytes) as max_bytes by dest_ip
| stats earliest(_time) as first_seen, latest(_time) as last_seen by user

| chart count over _time by EventCode
| timechart span=1h count by EventCode
| timechart span=5m sum(bytes) by dest_ip
```

---

## eval — computed fields

```splunk
| eval is_encoded=if(match(CommandLine, "(?i)-enc\s"), "YES", "NO")
| eval size_mb=round(bytes/1024/1024, 2)
| eval duration=strftime(strptime(end_time, "%Y-%m-%d"), "%s") - strftime(strptime(start_time, "%Y-%m-%d"), "%s")
| eval hour=strftime(_time, "%H")
| eval combined=src_ip.":".src_port
```

---

## rex — field extraction from raw text

```splunk
| rex field=_raw "CommandLine=(?P<cmd>.+?)(?=\s[A-Z]+=|\n|$)"
| rex field=CommandLine "-EncodedCommand\s+(?P<encoded>[A-Za-z0-9+/=]+)"
| rex field=url "https?://(?P<domain>[^/]+)"
```

---

## where — post-search filtering

```splunk
| where count > 100
| where src_ip != "10.0.0.1"
| where match(CommandLine, "(?i)invoke-expression|iex|downloadstring")
| where NOT cidrmatch("10.0.0.0/8", src_ip)
| where _time > relative_time(now(), "-1h")
```

---

## table, fields, sort, head, tail

```splunk
| table _time, user, src_ip, dest_ip, EventCode, CommandLine
| fields - _raw, _indextime, punct
| sort -count
| sort 0 _time               ← sort chronologically, return all results
| head 20
| tail 10
| dedup user                 ← deduplicate by field value
| dedup 3 user               ← keep first 3 occurrences per user
```

---

## Field extraction with erex and spath

```splunk
| erex CommandLine examples="powershell -enc ABC", "powershell.exe -EncodedCommand XYZ"
| spath input=json_field output=username path=user.name
| spath output=event_data path=EventData.Data{0}.#text
```

---

## Lookups

```splunk
| lookup threat_ips.csv ip as src_ip OUTPUT threat_type
| lookup dnslookup clientip as src_ip OUTPUT clienthost
| inputlookup known_bad_hashes.csv | join sha256 [search index=* sha256=*]
```

---

## Subsearches

```splunk
index=* [search index=* EventCode=4625 | stats count by user | where count > 50 | fields user]

index=proxy
  [search index=* EventCode=4625
   | stats count by src_ip | where count > 100 | fields src_ip]
| stats count by url, src_ip
```

A subsearch runs first, generates a list of values, and that list becomes a filter for the outer search. Use sparingly — they can be slow on large datasets.

---

## Useful SPL patterns

```splunk
| top limit=20 user                     ← most frequent values
| rare limit=10 process_name            ← least frequent (outliers)
| anomalydetection action=annotate      ← built-in anomaly detection
| streamstats count as event_count by user  ← running count per user
| transaction user maxspan=1h           ← group related events into sessions
| rename Account_Name as user, IpAddress as src_ip
| fillnull value="N/A" user
| replace "N/A" with "unknown" in user
```

---

## Useful functions in eval

```splunk
| eval lower_cmd=lower(CommandLine)
| eval len=len(CommandLine)
| eval contains_ps=if(like(CommandLine, "%powershell%"), 1, 0)
| eval base64_decoded=urldecode(encoded_field)
| eval hour=tonumber(strftime(_time, "%H"))
| eval day_of_week=strftime(_time, "%A")
```
