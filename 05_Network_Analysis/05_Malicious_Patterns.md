# Malicious Traffic Patterns

Recognizing what malicious network activity looks like in a PCAP — before applying filters.

---

## C2 beaconing

Malware beacons home to its C2 server at regular intervals to receive commands or confirm the implant is still alive. In a PCAP, beaconing produces a recognizable pattern: repeated connections from the same source to the same destination at consistent time intervals.

**What to look for**:
- Same source IP connecting to the same external IP every N seconds/minutes
- Consistent byte counts in the requests and responses (keepalive packets are often identical)
- Connection intervals that are *too* regular — human-driven traffic has natural variation; automated beacons don't
- Jitter: sophisticated C2 frameworks add randomness to intervals (e.g., ±30% of the base interval) to evade interval-based detection

**Common beacon intervals**: 30 seconds, 60 seconds, 5 minutes, 15 minutes. Cobalt Strike's default is 60 seconds; many operators change it.

**Detection approach**:
```
Wireshark filter: ip.dst == <suspected_c2> && tcp.flags.syn == 1

Use Statistics → Conversations, sort by packets or bytes
Look for connections with very regular packet timing
```

```bash
# extract connection timestamps to a specific IP for interval analysis
tshark -r capture.pcap -Y "ip.dst == 185.220.101.47 && tcp.flags.syn == 1" \
  -T fields -e frame.time_relative | awk 'NR>1{print $0-prev}{prev=$0}'
```

---

## Data exfiltration

**DNS tunneling**:

Data encoded in DNS query subdomain strings, exfiltrated to an attacker-controlled nameserver that decodes the queries. The C2 can respond via DNS answers.

```
Wireshark filter: dns.qry.name.len > 60
                  dns.qry.type == 16          ← TXT queries
```

Signs: extremely long subdomains that look like base64 or hex, consistent parent domain, TXT record queries from workstations, nameserver for the domain resolves to an unusual location.

**HTTP POST exfiltration**:

Data sent out via POST requests, often to a legitimate-looking domain or compromised site.

```
Wireshark filter: http.request.method == "POST" && http.content_length > 50000
```

Follow the TCP stream to see what's in the POST body. Base64-encoded content, compressed data, or JSON with unusual field values are all worth examining.

**Large single transfers**:
```
Wireshark filter: tcp.len > 100000
                  frame.len > 1400 && ip.dst != (internal range)
```

Use Statistics → Conversations sorted by bytes to find the largest transfers in the capture.

---

## Port scanning

A port scan from outside produces a predictable pattern: one source IP sending SYN packets to many destination ports on the same host, in rapid succession, with mostly RST+ACK responses.

**SYN scan pattern** (nmap default):
- One SYN packet per port
- Closed ports respond with RST+ACK
- Open ports respond with SYN+ACK (sometimes followed by RST from the scanner)
- Very short time between each probe

```
Wireshark filter: tcp.flags == 0x002 && ip.src == <scanner_ip>    ← all SYN from one source
                  tcp.flags.syn == 1 && tcp.flags.ack == 0         ← SYN only packets
```

```bash
# count SYN packets per destination port from a suspected scanner
tshark -r capture.pcap -Y "ip.src == 10.10.10.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0" \
  -T fields -e tcp.dstport | sort | uniq -c | sort -rn | head 20
```

A service discovery scan looks the same but targets many hosts on a single port.

---

## Lateral movement in PCAP

Lateral movement produces network artifacts depending on the technique used.

**PsExec / SMB lateral movement**:
```
Wireshark filter: smb2 && (smb2.filename contains "ADMIN$" || smb2.filename contains "C$")
                  ntlmssp
```
Look for: NTLM authentication to a workstation IP, SMB2 Create/Write to ADMIN$ or C$, followed by service creation (may be visible as additional SMB traffic).

**WMI remote execution**:
- WMI uses DCOM, which runs over dynamic high ports (TCP 49152+)
- Look for connections from one host to another on ports above 49000 with DCOM/RPC signatures
- In Wireshark: `dcerpc` or `epm` protocol labels

**RDP (Remote Desktop)**:
```
Wireshark filter: tcp.dstport == 3389 || tcp.srcport == 3389
```
RDP is encrypted and you can't read session content, but you can see: which hosts connected to RDP on which targets, timing of connections, duration of sessions, and data volume (session interactive work vs. file transfer have different traffic profiles).

**Pass-the-hash via SMB**:
- NTLM authentication to a remote host without a preceding Kerberos exchange
- The NTLM challenge-response is visible in the PCAP
- Correlate with Windows Event ID 4624 LogonType=3 with NtlmV2 authentication
