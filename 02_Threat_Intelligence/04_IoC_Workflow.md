# IOC Investigation Workflow

A practical sequence for enriching indicators extracted from phishing analysis, SIEM alerts, forensic artifacts, or network traffic. The goal is always the same: understand what the indicator is, what it's connected to, and what else it leads you to.

Document every step. Pivoting is only useful if you can trace how you got from A to B.

---

## Investigating an IP address
```
[ ] VirusTotal — detection score, community comments, Relations tab (passive DNS history)
[ ] AbuseIPDB — abuse categories, number of reports, reporter comments
[ ] OTX — any associated pulses? Linked TTPs or malware families?
[ ] WHOIS — which ASN/ISP owns this block? Which country? Expected or suspicious?
[ ] Passive DNS (VT Relations → Resolutions) — what domains have pointed here?
    → For each suspicious domain found, run the domain workflow below
[ ] Shodan/Censys — what ports and services are exposed? Any unexpected banners?
```

**Pivot signal:** If VT passive DNS shows 10 domains resolving to this IP and 3 of them match the naming pattern of known phishing infrastructure, that's your next thread.

---

## Investigating a domain
```
[ ] VirusTotal — detection, passive DNS (what IPs has it resolved to?), associated URLs and files
[ ] URLhaus — any malware distribution activity?
[ ] OTX — related pulses, TTPs, associated indicators
[ ] WHOIS — creation date, registrar, nameservers, registrant (if visible)
    → Recent creation? Suspicious registrar? Shared NS with other bad domains?
[ ] Current DNS resolution — what IP does it resolve to now?
    nslookup suspicious-domain.com
    → Run IP workflow on the result
[ ] URLScan.io — screenshot, contacted infrastructure, tech stack, downloaded file hashes
[ ] VT Relations → Subdomains — are there malicious subdomains under this domain?
```

**Pivot signal:** Domain registered 3 days before the incident, nameservers shared with 15 other newly-registered domains, resolves to an IP on AbuseIPDB with 200+ phishing reports.

---

## Investigating a file hash
```
[ ] VirusTotal — detection ratio, malware family name, first/last seen
    → Behavior tab: process tree, network connections, dropped files, ATT&CK TTPs
    → Relations tab: URLs this was downloaded from, IPs contacted, related files
    → Community tab: analyst notes and campaign context
[ ] OTX — associated pulses, linked IOCs
[ ] Hybrid Analysis / Triage — additional sandbox reports if VT behavior is limited
[ ] MalwareBazaar — sample context, tags, associated campaigns

If hash has 0 detections but came from a confirmed phishing email:
[ ] Submit to sandbox (Any.Run, Triage, Hybrid Analysis)
[ ] Note confidentiality before submitting to public services
```

**What to extract from sandbox output:**
- Process tree — what did the file spawn?
- Network connections — any C2 callbacks? To what IP/domain/port?
- Dropped files — secondary payloads? Their hashes become new IOCs
- Registry modifications — persistence mechanisms?
- ATT&CK technique IDs assigned by the sandbox

---

## Investigating a URL
```
[ ] Extract domain and run domain workflow
[ ] Extract IP the domain resolves to and run IP workflow
[ ] VirusTotal full URL lookup — detection, associated files, community notes
[ ] URLhaus — malware distribution history?
[ ] URLScan.io — submit for safe rendering
    → Screenshot: fake login? Redirect chain? Exploit kit landing page?
    → Contacted domains and IPs during page load → new IOCs
    → Files downloaded by the page → hash those
[ ] If shortened URL: expand first, then analyze the destination
```

---

## Pivoting

Pivoting is how a single IOC becomes a map of an actor's infrastructure.

**Common pivot paths:**

| Starting point | Pivot to | Via |
| :--- | :--- | :--- |
| Malicious IP | Related domains | Passive DNS (VT Relations) |
| Domain | Hosting IP | Current DNS resolution |
| Domain | Related domains | Shared nameservers, registrant email |
| File hash | Download URL | VT Relations → URLs |
| File hash | C2 IP/domain | Sandbox network connections |
| TLS certificate | Related IPs/domains | Censys certificate search |
| Malware family name | Known IOCs | OTX pulses, Feodo Tracker, MalwareBazaar |

Document every pivot step — where you started, what tool you used, what you found, and what it led to next.
