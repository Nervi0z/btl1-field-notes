# Tools — Threat Intelligence

Grouped by investigation task. For most IOC investigations, you'll move through these in sequence: reputation first, then infrastructure, then passive history, then behavior.

---

## Reputation and context platforms

Start here for any IP, domain, URL, or file hash.

| Tool | Best for |
| :--- | :--- |
| [VirusTotal](https://www.virustotal.com/) | Cross-engine reputation, passive DNS history, file behavior, indicator relationships. The Relations tab is where pivoting starts. |
| [AbuseIPDB](https://www.abuseipdb.com/) | IP-specific reputation — abuse categories, report count, comments from reporters. |
| [URLhaus](https://urlhaus.abuse.ch/) | URLs associated with malware distribution — includes malware family tags and hosting status. |
| [OTX AlienVault](https://otx.alienvault.com/) | Community threat intel — search any IOC and find related pulses with TTPs, associated malware, and linked indicators. |
| [URLScan.io](https://urlscan.io/) | Safe URL analysis — screenshot, contacted IPs and domains, downloaded file hashes, technology fingerprint. Good for finding related infrastructure. |

---

## Domain and IP registration

| Tool | Use |
| :--- | :--- |
| `whois <domain>` (Linux) | Registration dates, registrar, nameservers, registrant (often hidden by privacy) |
| [who.is](https://who.is/) | Web-based WHOIS with clean output |
| [domaintools.com](https://whois.domaintools.com/) | Historical WHOIS data — useful when current record is privacy-protected |

**What to look for in WHOIS:**
- Creation date — very recent? (days or weeks before the incident)
- Registrar — known bulletproof hosting registrar?
- Nameservers — shared with other suspicious domains?
- Registrant email — anonymous? shared across multiple registrations?

---

## DNS and passive DNS

Current DNS tells you where a domain resolves now. Passive DNS tells you where it has resolved historically — critical for finding infrastructure that's been rotated or taken down.
```bash
# current A record
nslookup suspicious-domain.com

# current A record + all record types
dig suspicious-domain.com ANY

# MX records (useful for phishing domain investigation)
dig suspicious-domain.com MX

# reverse lookup — what domain is this IP associated with?
dig -x 192.168.1.1
```

**Passive DNS sources:**
- VirusTotal Relations tab → "Resolutions" section
- [CIRCL PDNS](https://www.circl.lu/services/passive-dns/) — free passive DNS lookup
- RiskIQ / Microsoft Defender TI — requires account but has deep PDNS history

---

## Device and service search engines

Useful when you have a suspicious IP and want to understand what's running on it.

| Tool | Use |
| :--- | :--- |
| [Shodan](https://www.shodan.io/) | Open ports, banners, running services, geolocation, ASN |
| [Censys](https://search.censys.io/) | Similar to Shodan — TLS certificates are particularly useful for pivoting |
| [ZoomEye](https://www.zoomeye.org/) | Broader internet scan data, alternative to Shodan |

**Certificate pivoting:** If a domain uses a TLS certificate with an unusual common name or organization field, search that value in Censys — you may find other IPs or domains using the same certificate, revealing related infrastructure.

---

## Malware and sample repositories

| Tool | Use |
| :--- | :--- |
| [MalwareBazaar](https://bazaar.abuse.ch/) | Hash lookup, sample download (with caution), malware family tags |
| [Hybrid Analysis](https://www.hybrid-analysis.com/) | Sandbox reports with behavioral IOCs and ATT&CK mapping |
| [Feodo Tracker](https://feodotracker.abuse.ch/) | C2 infrastructure tracking for specific banking trojans |
