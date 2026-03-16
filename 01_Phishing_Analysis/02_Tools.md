# Tools — Phishing Analysis

Grouped by what you're trying to do. Use reputation services first — they're fast and often sufficient. Move to sandboxes when you need to see behavior.

---

## Reputation lookup

Start here for any URL, IP, domain, or file hash.

| Tool | Best for |
| :--- | :--- |
| [VirusTotal](https://www.virustotal.com/) | Cross-reference URLs, IPs, domains, and hashes across 70+ engines. Also shows relationships between indicators. |
| [URLhaus](https://urlhaus.abuse.ch/) | URLs associated with malware distribution. Good for context on suspicious links. |
| [AbuseIPDB](https://www.abuseipdb.com/) | IP reputation — reports of spam, C2, brute force. Use for source IPs from headers. |
| [OTX AlienVault](https://otx.alienvault.com/) | Broader threat intel — pulses with IOCs, TTPs, and MITRE ATT&CK mapping. |

---

## Header analysis

| Tool | Notes |
| :--- | :--- |
| [MxToolbox Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx) | Pastes headers and parses the full delivery path. Clear SPF/DKIM/DMARC output. |
| [Google Messageheader](https://toolbox.googleapps.com/apps/messageheader/) | Google's equivalent — works with any email header, not just Gmail. |

---

## URL inspection

| Tool | Notes |
| :--- | :--- |
| [URLScan.io](https://urlscan.io/) | Loads the URL in a safe environment and returns a screenshot, IOCs, contacted domains, and technologies. See what's behind a link without visiting it. |
| URL expanders | For shortened links — paste a `bit.ly` or `tinyurl` and get the real destination. Search "URL expander" for current options. |

---

## Sandboxes

When reputation lookup isn't enough and you need to observe actual behavior.

| Tool | Notes |
| :--- | :--- |
| [Any.Run](https://any.run/) | Interactive — you can click inside the VM. Good for observing process trees, network connections, dropped files in real time. |
| [Hybrid Analysis](https://www.hybrid-analysis.com/) | Static + dynamic. Solid free tier. MITRE ATT&CK tags, PCAP export, screenshot capture. |
| [Triage](https://tria.ge/) | Windows and Linux targets. Generous free tier. Good IOC extraction. |

> Before submitting anything to a public sandbox, consider whether the file contains sensitive data. Use private or offline analysis for confidential material.

---

## Working with `.eml` files

If you have the email as a file rather than viewing it in a client:
```bash
# read headers and body in terminal
cat suspicious.eml

# extract attachments (if python3 available)
python3 -c "
import email, sys
msg = email.message_from_file(open('suspicious.eml'))
for part in msg.walk():
    if part.get_filename():
        open(part.get_filename(), 'wb').write(part.get_payload(decode=True))
        print('Saved:', part.get_filename())
"
```
