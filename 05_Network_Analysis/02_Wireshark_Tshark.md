# Wireshark and tshark

---

## Wireshark interface

| Panel | What it shows |
| :--- | :--- |
| Packet list (top) | One row per packet — time, source, destination, protocol, length, info summary |
| Packet details (middle) | Hierarchical protocol breakdown — expand any layer to see field values |
| Packet bytes (bottom) | Raw hex and ASCII of the selected packet |

**Key menus for investigation:**

- **Statistics → Conversations** — list of all TCP/UDP/IP conversations, sortable by byte count. Start here to identify the busiest connections.
- **Statistics → Protocol Hierarchy** — breakdown of what protocols are in the capture by percentage.
- **Statistics → I/O Graph** — traffic volume over time. Spikes correlate with events.
- **Statistics → Capture File Properties** — file size, packet count, time window.
- **Analyze → Follow → TCP Stream** — reassemble and display the full content of a selected TCP session. Essential for reading HTTP, FTP, SMTP in plaintext.
- **File → Export Objects → HTTP** — export all files transferred over HTTP in the capture.

---

## tshark command structure

`tshark` is the command-line version of Wireshark. Same dissectors, same filters, scriptable.

```bash
tshark -r <file.pcap>                          # read a pcap
       -Y "<display filter>"                   # filter packets (Wireshark display filter syntax)
       -T fields                               # output only specific fields
       -e <field.name>                         # field to extract (use with -T fields)
       -w <output.pcap>                        # write filtered results to a new pcap
       -q                                      # quiet mode (suppress packet output)
       -z <statistics_type>                    # generate statistics
       --export-objects <protocol>,<dir>       # extract transferred files
```

---

## Common tshark one-liners

```bash
# list all DNS queries (names only, no responses)
tshark -r capture.pcap -Y "dns.flags.response == 0" \
  -T fields -e dns.qry.name | sort -u

# DNS queries with response IPs
tshark -r capture.pcap -Y "dns.flags.response == 1" \
  -T fields -e dns.qry.name -e dns.a | sort -u

# all HTTP requests — method, host, URI
tshark -r capture.pcap -Y "http.request" \
  -T fields -e http.request.method -e http.host -e http.request.uri

# all HTTP responses with status codes
tshark -r capture.pcap -Y "http.response" \
  -T fields -e http.host -e http.response.code -e http.content_type

# unique destination IPs (all protocols)
tshark -r capture.pcap -T fields -e ip.dst | sort | uniq -c | sort -rn | head 20

# unique external destination IPs (exclude RFC1918)
tshark -r capture.pcap -T fields -e ip.dst | sort -u | \
  grep -vE '^10\.|^192\.168\.|^172\.(1[6-9]|2[0-9]|3[01])\.|^127\.'

# extract all files transferred over HTTP
tshark -r capture.pcap --export-objects http,./http_objects/

# follow a TCP stream (stream index from Wireshark or tshark conversation stats)
tshark -r capture.pcap -Y "tcp.stream eq 5" -T fields -e data.data

# filter and save to a new PCAP
tshark -r capture.pcap -Y "ip.addr == 185.220.101.47" -w filtered_output.pcap

# show all unique User-Agent strings
tshark -r capture.pcap -Y "http.user_agent" \
  -T fields -e http.user_agent | sort -u

# show SSL/TLS server names (SNI — what domain the client expected)
tshark -r capture.pcap -Y "ssl.handshake.extensions_server_name" \
  -T fields -e ssl.handshake.extensions_server_name | sort -u

# conversation statistics — top talkers by bytes
tshark -r capture.pcap -q -z conv,tcp | head -30

# protocol hierarchy
tshark -r capture.pcap -q -z io,phs

# packets per second over time
tshark -r capture.pcap -q -z io,stat,1
```

---

## Working with large PCAPs

```bash
# split a large pcap by packet count
editcap -c 100000 large_capture.pcap split_output.pcap

# split by file size (in kilobytes)
editcap -b 100000 large_capture.pcap split_output.pcap

# merge multiple pcaps
mergecap -w combined.pcap capture1.pcap capture2.pcap

# convert between formats
editcap -F pcap input.pcapng output.pcap

# strip payload — keep only headers (useful for sharing captures safely)
editcap -s 100 capture.pcap headers_only.pcap
```
