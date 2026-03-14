# BTL1 Certification Blueprint

The Blue Team Level 1 (BTL1) by Security Blue Team is a practical evaluation framework designed for Tier 1 Security Operations Center (SOC) analysts and incident responders. This document outlines the technical scope, assessment parameters, and operational domains covered by the certification and this repository.

> **Operational Objective:** To validate that candidates can transition from theoretical security concepts to hands-on, terminal-based execution within a live, simulated corporate intrusion scenario.

---

## 1. Assessment Methodology

BTL1 operates strictly as a practical assessment, diverging entirely from traditional multiple-choice examinations. Candidates are deployed into a simulated corporate environment and tasked with conducting a full-scope incident investigation.

* **Operational Focus:** The evaluation is exclusively anchored in defensive operations—specifically threat detection, artifact analysis, and incident response—rather than offensive (Red Team) simulation.
* **Skill Validation:** Passing requires the successful hands-on execution of phishing analysis, SIEM alert triage, artifact extraction across both volatile and non-volatile memory, and deep packet analysis.
* **Artifact Correlation:** Evaluators expect analysts to stitch together a cohesive timeline by correlating technical evidence across multiple, isolated data telemetry sources.

## 2. Prerequisite Knowledge Baseline

To effectively navigate the BTL1 framework and utilize the playbooks within this repository, engineers must possess a foundational understanding of the following areas:

* **Networking Fundamentals:** * TCP/IP stack mechanics and DNS resolution.
  * HTTP/S traffic flow and common ports/protocols.
* **Operating System Architecture:** * *Windows:* Registry hive structures, event logging (Sysmon/Security), and process execution trees.
  * *Linux:* File system hierarchy, daemon execution, and permission models.
* **Command-Line Proficiency:** System navigation and data manipulation via Bash and PowerShell.
* **Security Telemetry:** Understanding of how logging mechanisms (Firewalls, IDS/IPS, AV/EDR) generate, format, and store event data.

---

## 3. Exam Execution Parameters

The certification assessment is conducted under strict operational constraints, simulating the SLA pressure of a live Incident Response engagement:

* **Engagement Window:** A continuous 24-hour timeframe to conduct the investigation, extract evidence, and draft the final report.
* **Infrastructure Access:** Candidates are granted access to a cloud-hosted virtual lab containing multi-OS endpoints, network sensors, and pre-configured SIEM infrastructure (e.g., Splunk/Elastic).
* **Investigation Objective:** Trace a multi-stage intrusion by analyzing varied data sources:
  * Disk images (raw `DD`/`E01` files).
  * Volatile memory dumps (`RAM`).
  * Packet captures (`PCAP`).
  * Aggregated SIEM logs.
* **Final Deliverable:** A formal, evidence-backed Incident Report detailing the attack timeline, identifying compromised assets, and addressing specific scenario injects with concrete technical proof.

---

## 4. Operational Domain Mapping

The certification framework—and consequently, the architectural structure of this playbook repository—is mapped to six core defensive domains. 

| Domain | Operational Focus | Primary Toolset / Focus Area |
| :--- | :--- | :--- |
| **Phishing Analysis** | Inspection of SMTP headers, reverse-engineering payloads, and URL pivoting. | Any.Run, PhishTool, CyberChef |
| **Threat Intelligence** | Categorization of adversary TTPs and IOC enrichment. | MITRE ATT&CK, VirusTotal, OTX |
| **Digital Forensics** | Recovery of non-volatile storage artifacts and volatile memory analysis. | Autopsy, Volatility 3, KAPE |
| **SIEM Analysis** | Log aggregation, event correlation, and advanced query formulation. | Splunk (SPL), Elastic (KQL) |
| **Network Analysis** | Deep Packet Inspection (DPI) to identify lateral movement or C2 beaconing. | Wireshark, `tshark`, Zeek |
| **Incident Response** | Live endpoint triage methodologies and evidence preservation techniques. | Sysinternals, native OS CLI |

---

## License
MIT
