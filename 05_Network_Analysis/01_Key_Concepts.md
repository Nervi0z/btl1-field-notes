# Network Analysis — Key Concepts

---

## What PCAPs are and how you get them

A PCAP (Packet Capture) file contains a timestamped record of every packet that passed through a network interface during the capture window. Unlike logs, which record what the OS decided was worth logging, a PCAP records everything — headers, payloads, connection setup, teardown, and retransmissions.

In BTL1, PCAPs are provided as evidence files. In a real SOC, they come from: network taps on segment boundaries, SPAN ports on managed switches, inline capture devices (like Security Onion sensors), or endpoint-based capture tools like Wireshark or `tcpdump`.

A PCAP without a timeframe context is harder to work with. Know when the incident occurred before you open the file — use the Statistics → Capture File Properties menu in Wireshark to see the PCAP's time window, then narrow your filter to the relevant period.

---

## Protocol stack — what you're analyzing at each layer

| Layer | Protocols | What you analyze |
| :--- | :--- | :--- |
| Layer 3 (Network) | IP, ICMP | Source/destination IPs, ICMP type codes, TTL values |
| Layer 4 (Transport) | TCP, UDP | Ports, connection state (SYN/ACK/RST), session reconstruction |
| Layer 7 (Application) | DNS, HTTP, SMB, FTP, TLS | Protocol-specific content — queries, requests, credentials, filenames |

Most investigation work happens at Layer 7. But Layer 4 is where you see port scans, lateral movement patterns, and C2 connection timing. Layer 3 is where you identify which hosts are communicating.

---

## DNS — normal vs suspicious

Normal DNS: short hostnames, known TLDs (.com, .net, .org, .io), TTL values consistent with the record type, responses pointing to expected IPs.

Suspicious DNS patterns:
- **High-entropy hostnames**: `a3f9b2c1d8e7.evil.com` — random-looking subdomains are characteristic of DGA (Domain Generation Algorithm) malware
- **Abnormally long query names**: legitimate domains rarely exceed 50 characters; DNS tunneling tools encode data in subdomains and produce very long names
- **High query volume to a single domain**: a host querying the same domain hundreds of times per hour suggests C2 keepalive or tunneling
- **Unusual TLDs**: `.tk`, `.xyz`, `.top`, `.pw`, `.cc` are heavily abused; their presence isn't conclusive but warrants a lookup
- **NXDOMAIN floods**: many consecutive failed lookups suggest DGA — the malware is cycling through generated domains looking for one that's registered

---

## HTTP — normal vs suspicious

HTTP is cleartext and fully inspectable in a PCAP. Inspect:
- **Request method**: GET retrieves, POST sends data. Large POST bodies with no corresponding user action are exfiltration candidates
- **User agent**: legitimate browsers send consistent, recognizable user agents. `python-requests/2.28`, `curl/7.68`, or empty user agents in browser traffic are anomalous
- **URI patterns**: `/gate.php`, `/panel/`, `/upload/`, `/update/` are common C2 endpoint names
- **Response codes**: 200 is success, 404 is not found — C2 infrastructure often returns 404 for anything except the configured endpoint

HTTPS: you can't see the content without the private key or a MITM proxy, but you can see: the server's certificate (CN, SAN, issuer), SNI (Server Name Indication) showing what hostname the client expected, and JA3/JA3S fingerprints of the TLS handshake.

---

## SMB — normal vs suspicious

SMB is used for Windows file sharing and is the primary protocol for lateral movement in Windows environments.

Normal SMB: hosts accessing mapped network drives, print spooling, domain controller SYSVOL/NETLOGON access.

Suspicious SMB patterns:
- Connections from non-server hosts to other workstations (workstation-to-workstation SMB is unusual in most environments)
- Access to ADMIN$, C$, or IPC$ shares from unusual sources — these are administrative shares used by tools like PsExec and impacket
- SMB authentication using NTLM instead of Kerberos on a domain (potential pass-the-hash)
- File access to `.exe`, `.ps1`, or `.dll` files via SMB (lateral tool transfer)

---

## FTP — cleartext risk

FTP sends credentials and data in cleartext. If you see FTP in a PCAP:
- Filter for the control channel (port 21) and look for `USER` and `PASS` commands — both are visible as plaintext
- The data channel (port 20, or negotiated in passive mode) carries the actual file transfer — follow the TCP stream to see file content
- Exfiltration via FTP is unsophisticated but still occurs; look for unusual file sizes or transfer destinations
