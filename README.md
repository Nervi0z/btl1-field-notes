# btl1-field-notes

Operational reference for Security Blue Team Level 1 candidates and SOC analysts.

Cheat sheets, CLI references, and detection workflows across the five BTL1 domains: phishing analysis, threat intelligence, SIEM, digital forensics, and network traffic analysis. Built for use during active lab sessions — not exam prep, just operational context.

---

## Coverage

| Domain | Focus | Tools |
| :--- | :--- | :--- |
| Endpoint forensics | File system artifacts, registry hives, execution traces | KAPE, Autopsy, Sysinternals |
| Memory analysis | Process injection, rogue connections, hollowing | Volatility 2 & 3 |
| Network traffic analysis | PCAP inspection, protocol anomalies, payload carving | Wireshark, tshark, BPF |
| Log correlation / SIEM | Event correlation, SPL, ECS field mapping | Splunk, Elastic Stack |
| Malware analysis | Static analysis, file carving, metadata extraction | Scalpel, Foremost, ExifTool |

---

## Structure

```text
.
├── 00_introduction/        # incident methodology, hypothesis-driven triage
├── 01_phishing/            # header analysis, MTA logs, attachment extraction
├── 02_threat_intel/        # IoC management, MITRE ATT&CK TTP mapping
├── 03_forensics/
│   ├── disk/               # NTFS artifacts, registry hives, file carving
│   └── memory/             # Volatility profiles, injection detection
├── 04_siem/                # SPL query structures, log correlation rules
├── 05_network/             # BPF filters, protocol anomalies, PCAP carving
└── 06_incident_response/   # IR lifecycle, containment, live response
```

---

## Usage

Clone and query locally during lab work. All playbooks are plain markdown — no dependencies.

```bash
# clone
git clone https://github.com/youruser/btl1-field-notes
cd btl1-field-notes

# search by Windows Event ID
grep -rnw . -e 'EventCode=4624'

# find Volatility 3 module syntax
grep -rnw 03_forensics/ -e 'windows.malfind'

# locate SPL patterns
grep -rnw 04_siem/ -e 'stats count by'
```

---

## Contributing

Pull requests for updated SPL queries, corrected syntax, or new tool references are welcome.
Follow the branching convention and check `CONTRIBUTING.md` before opening a PR.

```bash
git checkout -b fix/spl-execution-queries
git commit -m "siem: update sysmon execution detection rules"
git push origin fix/spl-execution-queries
```

---

> General technical documentation and open-source tool references only. No proprietary exam content, lab infrastructure details, or restricted BTL1 materials — per SBT NDA. Pull requests containing such material will not be merged.

---

MIT License · See `LICENSE` for details.
