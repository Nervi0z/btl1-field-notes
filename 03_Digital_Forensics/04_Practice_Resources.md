# Practice Resources — Digital Forensics

---

## Disk images and memory dumps for practice

| Source | What's available |
| :--- | :--- |
| [Digital Corpora](https://digitalcorpora.org/) | Public repository of disk images, memory dumps, and network captures built for forensic education. Broad range of OS types and scenarios. |
| [CyberDefenders](https://cyberdefenders.org/) | CTF-style DFIR challenges with realistic evidence packages — disk, memory, PCAP, logs. The most directly relevant platform for BTL1-style practice. |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Investigation scenarios frequently require disk and memory forensics as part of a broader case. BTL1-aligned format. |
| [Hack The Box — Sherlocks](https://app.hackthebox.com/sherlocks) | DFIR challenges at higher difficulty. Good for pushing beyond BTL1 fundamentals. |
| [Volatility Memory Samples](https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples) | Public memory images from the Volatility Foundation wiki. Use these to practice specific plugins without needing a full scenario. |
| [NIST CFReDS](https://cfreds.nist.gov/) | Reference disk image datasets. Some are dated but the forensic questions are still valid for practicing artifact location. |
| [aboutDFIR Challenges](https://aboutdfir.com/challenges/) | Aggregated list of publicly available DFIR challenges across multiple platforms. |

---

## Lab platforms

| Platform | Relevant content |
| :--- | :--- |
| [TryHackMe](https://tryhackme.com/) | Volatility rooms, Autopsy walkthroughs, Windows forensics paths. Good starting point before tackling full scenarios. |
| [Hack The Box — Sherlocks](https://app.hackthebox.com/sherlocks) | Multi-artifact investigations requiring correlation across disk, memory, and logs. |

---

## Tool documentation

These are worth reading once, not just using as a reference:

- **[Autopsy User Documentation](https://docs.sleuthkit.org/autopsy/)** — covers ingest modules, timeline analysis, and keyword search in detail
- **[The Sleuth Kit Wiki](https://wiki.sleuthkit.org/)** — TSK command reference and filesystem concepts
- **[Eric Zimmerman's Tools](https://ericzimmerman.github.io/)** — each tool has its own documentation; his blog covers practical usage with real examples
- **[Volatility 3 GitHub Wiki](https://github.com/volatilityfoundation/volatility3/wiki)** — plugin list, symbol table setup, and contribution docs
- **[KAPE Documentation](https://www.kroll.com/en/insights/publications/cyber/kape-documentation)** — target and module reference; the GitHub repo has the full current list of targets
- **[ExifTool Tag Names](https://exiftool.org/TagNames/)** — complete tag reference organized by file format

---

## Blogs and ongoing learning

- **[13Cubed](https://www.13cubed.com/)** — video walkthroughs of Volatility, Plaso, and Windows artifact analysis. One of the best free resources for memory forensics specifically.
- **[This Week in 4n6](https://thisweekin4n6.com/)** — weekly roundup of DFIR news, new tools, writeups, and challenge releases.
- **[SANS DFIR Blog](https://www.sans.org/blog/topic/dfir/)** — technical posts and webcasts on current DFIR techniques.
- **[Didier Stevens Blog](https://blog.didierstevens.com/)** — deep dives on file format analysis, PDF and Office document forensics, malicious document deconstruction.
- **[aboutDFIR](https://aboutdfir.com/)** — aggregator for tools, challenges, and community resources. Good for staying current.
- **[Forensafe](https://forensafe.com/)** — Windows artifact analysis walkthroughs with real examples.
