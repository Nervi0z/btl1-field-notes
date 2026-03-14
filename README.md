# BTL1 Operations Framework

Operational playbooks, technical cheat sheets, and threat hunting workflows mapped to the Security Blue Team Level 1 (BTL1) domains. 

This repository serves as a quick-reference framework for junior security analysts, incident responders, and BTL1 certification candidates handling endpoint telemetry, network traffic analysis, and log correlation.

## Core Capabilities

The documentation is built around standard Security Operations Center (SOC) workflows and covers the following toolsets:

* **Endpoint Telemetry & Forensics:** KAPE, Autopsy, The Sleuth Kit (TSK), Sysinternals.
* **Memory Analysis:** Volatility 2 / 3.
* **Network Traffic Analysis (NTA):** Wireshark, Tshark, PCAP inspection methodologies.
* **Log Aggregation & SIEM:** Splunk (SPL optimization), Elastic Stack (ECS standard).
* **Malware & File Analysis:** Scalpel, Foremost, ExifTool, static analysis sandboxes (Any.Run, Hybrid Analysis).

## Repository Architecture

```text
.
├── 00_Introduction/         # Exam strategy and critical thinking models
├── 01_Phishing/             # Email header analysis, MTA logs, attachment extraction
├── 02_Threat_Intel/         # IoC management, TTP mapping (MITRE ATT&CK)
├── 03_Forensics/
│   ├── 02_Disk_Analysis/    # NTFS artifacts, registry hives, file carving
│   └── 03_Memory_Analysis/  # Volatility profiles, process hollowing detection
├── 04_SIEM/                 # SPL query structures, log correlation rules
├── 05_Network/              # Protocol anomalies, BPF filters
└── 06_Incident_Response/    # IR lifecycle, containment strategies, live response
```

## Usage & Quick Search

This repository is designed to be accessible directly via CLI during lab exercises or live investigations. Clone the repository and use standard GNU tools to query the playbooks:

```bash
# Clone the playbooks
git clone [https://github.com/yourusername/btl1-ops-framework.git](https://github.com/yourusername/btl1-ops-framework.git)
cd btl1-ops-framework

# Quick search for a specific Splunk command or Windows Event ID
grep -rnw '.' -e 'EventCode=4624'
grep -rnw '04_SIEM/' -e 'stats count by'

# Find Volatility 3 specific commands
grep -rnw '03_Forensics/' -e 'vol3'
```

## Maintenance & Contributions

Standard operating procedures (SOPs) evolve. To contribute updates, new SPL queries, or correct syntax errors:
1. Fork the repository.
2. Create a feature branch (`git checkout -b update/spl-queries`).
3. Commit changes (`git commit -m "docs: update Splunk cheatsheet with rare execution queries"`).
4. Open a Pull Request referencing the modified domain.

See `CONTRIBUTING.md` for strict formatting guidelines.

## Compliance & NDA Notice

This repository contains general technical documentation, standard cybersecurity operating procedures, and open-source tool usage guides. **It strictly adheres to the Security Blue Team Non-Disclosure Agreement (NDA).** No proprietary exam questions, specific lab infrastructure details, or restricted testing materials are included or will be accepted via Pull Requests.

## License

MIT License. See `LICENSE` for details.
