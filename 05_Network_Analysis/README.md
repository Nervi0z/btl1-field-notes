# Network Analysis

PCAPs are a ground-truth record of what actually happened on the wire. Unlike logs, which depend on what the OS chose to record, a packet capture doesn't lie about what was sent. This section covers how to work through a PCAP efficiently — narrowing from thousands of packets to the handful that tell the story.

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [01_Key_Concepts.md](01_Key_Concepts.md) | PCAPs, protocol stack, what to look for in DNS/HTTP/SMB/FTP |
| [02_Wireshark_Tshark.md](02_Wireshark_Tshark.md) | Wireshark interface panels, tshark command structure, one-liners |
| [03_Filters_Cheatsheet.md](03_Filters_Cheatsheet.md) | Display filters by scenario, BPF capture filters, tshark equivalents |
| [04_Specific_Protocol_Analysis.md](04_Specific_Protocol_Analysis.md) | DNS, HTTP/HTTPS, SMB, FTP — normal vs malicious patterns |
| [05_Malicious_Patterns.md](05_Malicious_Patterns.md) | C2 beaconing, exfiltration, port scanning, lateral movement in PCAP |
| [06_Practice_Resources.md](06_Practice_Resources.md) | PCAP repositories, practice platforms, protocol documentation |

---

## Quick reference

**Wireshark display filters — start here:**

```
| Suspicious DNS
dns.qry.name matches ".*\.(tk|xyz|top|pw|cc)$"
dns.qry.name.len > 50

| HTTP POST requests (data being sent out)
http.request.method == "POST"

| SMB lateral movement
smb2.cmd == 0x05 && smb2.filename contains ".exe"

| Large data transfers
frame.len > 1400 && tcp

| Specific IP
ip.addr == 192.168.1.100
ip.dst == 10.0.0.1 && !ip.src == 10.0.0.0/8
```

**tshark one-liners:**

```bash
# all DNS queries in a PCAP
tshark -r capture.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name

# unique external destination IPs
tshark -r capture.pcap -T fields -e ip.dst | sort -u | grep -v '^10\.\|^192\.168\.\|^172\.'

# HTTP hosts and URIs
tshark -r capture.pcap -Y http.request -T fields -e http.host -e http.request.uri

# extract files from HTTP traffic
tshark -r capture.pcap --export-objects http,./extracted_files/
```

---

> New to network forensics? Start with `01_Key_Concepts.md`. Mid-analysis and need a filter fast? Go straight to `03_Filters_Cheatsheet.md`.
