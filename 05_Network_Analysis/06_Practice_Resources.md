# Practice Resources — Network Analysis

---

## PCAP repositories

| Source | What's available |
| :--- | :--- |
| [Malware Traffic Analysis](https://malware-traffic-analysis.net/) | Brad Duncan's archive of real malware infection PCAPs with analysis writeups. One of the best resources available — work through the exercises before reading the answers. |
| [Wireshark Sample Captures](https://wiki.wireshark.org/SampleCaptures) | Protocol samples across dozens of protocol types. Good for learning what normal looks like before you look for abnormal. |
| [PacketLife.net](http://packetlife.net/captures/) | Protocol-organized PCAP collection. |
| [Netresec PCAP files](https://www.netresec.com/?page=PcapFiles) | Curated list of PCAP sources — CTF captures, ICS protocol captures, and public malware samples. |
| [BTLO Challenges](https://blueteamlabs.online/) | Many BTLO challenges include PCAPs as evidence. BTL1-aligned format. |

---

## Practice platforms

| Platform | Network analysis content |
| :--- | :--- |
| [CyberDefenders](https://cyberdefenders.org/) | Strong PCAP-based challenges. Search the "Network Forensics" category. |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Investigation scenarios frequently include PCAP analysis as part of a broader case. |
| [TryHackMe](https://tryhackme.com/) | Wireshark-focused rooms and network analysis paths. |
| [PicoCTF](https://picoctf.org/) | Network forensics challenges in the forensics category — useful for fundamentals. |

---

## Protocol reference

- **[Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)** — complete list of every display filter field by protocol. Bookmark this.
- **[Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)** — thorough documentation on all features including statistics, expert analysis, and following streams.
- **[tshark man page](https://www.wireshark.org/docs/man-pages/tshark.html)** — every flag and option for command-line work.
- **[RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035)** — DNS protocol specification. Worth reading the query/response format section once.
- **[BPF Filter Syntax](https://www.tcpdump.org/manpages/pcap-filter.7.html)** — reference for capture filter syntax used in tcpdump and tshark `-f`.
