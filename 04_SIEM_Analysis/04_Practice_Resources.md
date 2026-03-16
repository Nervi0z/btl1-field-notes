# Practice Resources — SIEM Analysis

---

## BOTS — Boss of the SOC datasets

Splunk's CTF datasets, released publicly after each competition. Ingest into a free Splunk instance and work through the questions — this is the closest you'll get to BTL1-style SIEM work without the actual exam.

| Dataset | Scenario | Key log sources |
| :--- | :--- | :--- |
| BOTS v1 | Web server compromise, malware, defacement | Apache, Suricata, Sysmon, WinEventLog |
| BOTS v2 | Targeted attack, phishing, ransomware | Sysmon, WinEventLog, Cisco logs, DNS |
| BOTS v3 | Cloud attack (AWS), insider threat | CloudTrail, O365, Sysmon, Zeek |

Download locations: search "Splunk BOTS dataset download" — datasets are hosted on GitHub and the Splunk community site. You'll need a free Splunk instance to ingest them (8.x or later works for all three).

---

## Setting up a free Splunk instance

```bash
# Splunk Free is limited to 500MB/day ingestion — sufficient for BOTS datasets
# Download from: https://www.splunk.com/en_us/download/splunk-enterprise.html

# After install, ingest BOTS data:
# Settings → Add Data → Upload → select the dataset archive
# or use the Splunk command line:
./splunk add index botsv2
./splunk add monitor /path/to/botsv2/ -index botsv2
```

---

## Practice platforms

| Platform | What's relevant |
| :--- | :--- |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Challenges frequently include Splunk or Elastic as the analysis tool. BTL1-aligned scenarios. |
| [CyberDefenders](https://cyberdefenders.org/) | Several challenges include pre-loaded SIEM environments. Check the "SIEM" tag. |
| [LetsDefend](https://letsdefend.io/) | SIEM module with realistic log investigation tasks. |
| [TryHackMe](https://tryhackme.com/) | Rooms on Splunk fundamentals and specific detection scenarios. |

---

## Splunk training

- **[Splunk Fundamentals 1](https://www.splunk.com/en_us/training/free-courses/splunk-fundamentals-1.html)** — Free. Gets you comfortable with the interface and basic SPL before you hit the more complex query work.
- **[Splunk Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/WhatsInThisManual)** — The authoritative SPL command documentation. Bookmark this.
- **[Splunk Security Essentials](https://splunkbase.splunk.com/app/3435/)** — Free Splunk app with pre-built detection searches mapped to MITRE ATT&CK. Good for seeing how production detection queries are structured.

---

## Elastic/Kibana note

If you encounter an Elastic Stack environment rather than Splunk, the query language is KQL (Kibana Query Language) or EQL (Event Query Language). The concepts are identical — base search, filter, aggregate — but the syntax differs.

```
| KQL equivalent of SPL index + EventCode filter
event.code : "4625" and host.name : "dc01"

| EQL — sequence detection (Elastic-specific)
sequence by host.name
  [process where process.name == "winword.exe"]
  [process where process.name == "powershell.exe" and process.parent.name == "winword.exe"]
```

Elastic's Detection Rules repository on GitHub ([github.com/elastic/detection-rules](https://github.com/elastic/detection-rules)) contains production EQL queries for hundreds of attack techniques — worth reading even if you're using Splunk, since the logic translates.
