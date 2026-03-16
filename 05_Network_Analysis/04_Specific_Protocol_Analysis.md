# Protocol Analysis

---

## DNS

DNS is the first place to look in most network-based investigations. Every domain name a host resolves leaves a query in the PCAP — and malware has to resolve its C2 domain before it can talk to it.

**What normal DNS looks like**: short hostname queries (20-30 chars), standard record types (A, AAAA, MX, TXT), reasonable response times (< 100ms for cached records), responses pointing to expected IP ranges for the domain.

**DGA (Domain Generation Algorithm) indicators**:
```
Wireshark filter: dns.qry.name.len > 40 && dns.flags.response == 1 && dns.count.answers == 0
```
DGA malware generates hundreds of pseudo-random domain names and queries them until one resolves — the one the attacker has registered that day. In a PCAP, DGA looks like: repeated NXDOMAIN responses, short hostname strings with high entropy (lots of consonants, no recognizable words), all to the same nameserver.

**DNS tunneling indicators**:
```
Wireshark filter: dns.qry.type == 16    ← TXT records
                  dns.qry.name.len > 60
```
DNS tunneling encodes data into DNS queries and responses — data exfiltration via DNS, or C2 communication using DNS as the transport. Indicators: unusually long subdomains (`aGVsbG8gd29ybGQ=.evil.com`), TXT record queries from workstations, high query volume to a single parent domain, base64 or hex-encoded looking subdomains.

```bash
# extract all unique DNS queries and sort by length
tshark -r capture.pcap -Y "dns.flags.response == 0" \
  -T fields -e dns.qry.name | sort -u | awk '{ print length, $0 }' | sort -rn | head 20
```

---

## HTTP and HTTPS

**HTTP** is fully readable in a PCAP. Focus on:
- **Follow TCP Stream** (right-click a packet → Follow → TCP Stream) to read the full HTTP exchange
- POST body — what data is being sent? Is it base64, encoded, structured (JSON, form data)?
- Response content — what did the server return? An executable? An HTML redirect?

```bash
# extract HTTP objects (files transferred over HTTP)
tshark -r capture.pcap --export-objects http,./extracted/

# read all HTTP headers
tshark -r capture.pcap -Y http -T fields \
  -e http.request.method -e http.host -e http.request.uri \
  -e http.user_agent -e http.response.code | column -t
```

**HTTPS** — you can't read the content without the session key or private key, but you can see:

- **SNI (Server Name Indication)**: The hostname the client is connecting to, sent in the ClientHello before encryption starts. Use `ssl.handshake.extensions_server_name` in Wireshark.
- **Certificate fields**: Subject CN, SAN (Subject Alternative Names), Issuer, validity dates. Self-signed certs on non-standard ports for non-browser traffic are suspicious.
- **JA3 fingerprint**: A hash of TLS parameters from the ClientHello. The same malware family often produces the same JA3 across different C2 IPs. Look up JA3 hashes at [ja3er.com](https://ja3er.com/).

```
Wireshark filter for TLS certificate inspection:
tls.handshake.certificate   ← select a certificate record, expand in detail panel
```

---

## SMB

SMB is the Windows file sharing protocol — and the primary tool for lateral movement in Windows environments.

**Authentication**: Modern Windows environments use Kerberos for SMB authentication. Seeing NTLM authentication (look for `ntlmssp` in Wireshark) on a domain-joined network is suspicious — it can indicate pass-the-hash or hash relay attacks.

**Useful SMB filters**:
```
smb2                                        ← all SMBv2 traffic
smb2.cmd == 0x05                            ← file create/open operations
smb2.filename                               ← filter to packets with a filename field
smb2.filename contains "ADMIN$"            ← ADMIN$ share access
smb2.filename contains "C$"               ← C$ administrative share
ntlmssp && smb2                            ← NTLM auth in SMB sessions
```

**Lateral tool transfer**: An attacker using PsExec or impacket's smbexec copies an executable to ADMIN$ or a temp location via SMB, then creates a service to run it. In a PCAP, this looks like:
1. NTLM authentication to a target host
2. SMB2 Create request to `\ADMIN$\filename.exe` or `\C$\Windows\Temp\`
3. SMB2 Write — the binary being transferred
4. Follow-on service creation (may appear in Windows Event logs)

```bash
# extract files transferred over SMB
tshark -r capture.pcap --export-objects smb,./smb_objects/
```

---

## FTP

FTP sends credentials and file content in cleartext over separate control (port 21) and data (port 20 or negotiated passive port) channels.

**Credentials are visible in plaintext**:
```
Wireshark filter: ftp.request.command == "USER" || ftp.request.command == "PASS"
```

In the Wireshark packet list, FTP control commands show in the Info column. Follow the TCP stream on the control channel to see the full session including filenames.

**Data channel**: In passive FTP, the data channel uses a dynamically negotiated port. Wireshark usually reassociates these automatically — look for FTP-DATA as the protocol.

```bash
# list FTP commands
tshark -r capture.pcap -Y "ftp" -T fields -e ftp.request.command -e ftp.request.arg

# extract files transferred over FTP
tshark -r capture.pcap --export-objects ftp-data,./ftp_objects/
```
