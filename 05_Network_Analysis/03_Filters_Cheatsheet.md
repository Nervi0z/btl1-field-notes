# Filters Cheatsheet

Wireshark display filters and BPF capture filters. Display filters work in Wireshark and tshark `-Y`. BPF filters work with tcpdump and tshark `-f` at capture time.

---

## Display filters — by scenario

### Suspicious DNS

```
dns.qry.name.len > 50
dns.qry.name matches ".*\.(tk|xyz|top|pw|cc|ru)$"
dns.qry.type == 16                                  ← TXT records (used in DNS tunneling)
dns.flags.response == 0 && !dns.qry.name contains "."   ← malformed queries
dns.count.answers == 0 && dns.flags.rcode != 0     ← NXDOMAIN responses
```

### HTTP analysis

```
http.request.method == "POST"
http.request.method == "POST" && http.content_length > 10000
http.response.code == 200
http.response.code >= 400                           ← client/server errors
http.user_agent contains "python" || http.user_agent contains "curl"
http.user_agent == ""                               ← empty user agent
http.request.uri contains "/gate" || http.request.uri contains "/panel"
http.request.uri matches ".*\.(php|asp|aspx)\?.*"  ← dynamic pages with params
http && frame.len > 10000                           ← large HTTP transfers
```

### SMB activity

```
smb || smb2
smb2.filename contains ".exe" || smb2.filename contains ".ps1"
smb2.cmd == 0x05                                    ← SMB2 Create (file open/create)
smb2.cmd == 0x09                                    ← SMB2 Write
smb.cmd == 0x25                                     ← SMBv1 Trans2 (file operations)
smb2.filename contains "ADMIN$" || smb2.filename contains "C$"
ntlmssp                                             ← NTLM auth in SMB sessions
```

### Large transfers and exfiltration

```
frame.len > 1400 && tcp
tcp.len > 1400
ip.dst == <external_ip> && frame.len > 100000
```

### Specific IP or port

```
ip.addr == 192.168.1.100
ip.src == 192.168.1.100
ip.dst == 185.220.101.47
ip.addr == 10.0.0.0/8                              ← subnet
tcp.port == 4444
tcp.dstport == 8080
udp.port == 53
!(ip.addr == 10.0.0.0/8)                           ← all non-RFC1918 traffic
```

### Protocol anomalies

```
tcp.flags.syn == 1 && tcp.flags.ack == 0           ← SYN only (scan or connection attempt)
tcp.flags.rst == 1                                  ← connection resets
tcp.analysis.retransmission                         ← retransmissions
icmp.type == 8                                      ← ICMP echo requests
icmp && frame.len > 100                             ← large ICMP (possible tunneling)
ssl.handshake.type == 1                             ← TLS Client Hello
ssl.handshake.extensions_server_name               ← has SNI field
```

### Beaconing patterns

```
ip.dst == <suspected_c2> && tcp.flags.syn == 1     ← connection attempts over time
ip.dst == <suspected_c2>                           ← all traffic to a suspected C2
http.request && ip.dst == <suspected_c2>           ← HTTP C2 traffic
```

---

## BPF capture filters

These run at capture time and limit what gets written to the PCAP. Use them when you know what you're looking for and want to reduce file size.

```bash
# capture only traffic to/from a specific host
tcpdump -i eth0 -w output.pcap host 192.168.1.100

# capture only DNS traffic
tcpdump -i eth0 -w dns.pcap port 53

# capture only HTTP traffic
tcpdump -i eth0 -w http.pcap port 80 or port 8080

# capture traffic to an external network (not RFC1918)
tcpdump -i eth0 -w external.pcap 'not (net 10.0.0.0/8 or net 172.16.0.0/12 or net 192.168.0.0/16)'

# capture SYN packets only (see connection attempts)
tcpdump -i eth0 -w syns.pcap 'tcp[tcpflags] & tcp-syn != 0'

# capture ICMP
tcpdump -i eth0 -w icmp.pcap icmp

# capture traffic on multiple ports
tcpdump -i eth0 -w multi.pcap port 80 or port 443 or port 8080
```

---

## tshark display filter equivalents

```bash
# DNS queries (tshark)
tshark -r capture.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name

# HTTP POST requests
tshark -r capture.pcap -Y "http.request.method == POST" \
  -T fields -e http.host -e http.request.uri -e http.content_length

# large frames
tshark -r capture.pcap -Y "frame.len > 1400" -T fields -e ip.src -e ip.dst -e frame.len

# suspicious user agents
tshark -r capture.pcap -Y 'http.user_agent contains "python" or http.user_agent == ""' \
  -T fields -e ip.src -e http.user_agent

# all SMB2 file operations
tshark -r capture.pcap -Y "smb2.cmd == 5" -T fields -e ip.src -e ip.dst -e smb2.filename
```
