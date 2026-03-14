# BTL1 Certification Blueprint

The Blue Team Level 1 (BTL1) by Security Blue Team is a practical evaluation framework designed for Tier 1 Security Operations Center (SOC) analysts and incident responders. This document outlines the technical scope, assessment parameters, and operational domains covered by the certification and this repository.

## Assessment Methodology

BTL1 operates strictly as a practical assessment. Candidates are deployed into a simulated corporate environment and tasked with conducting a full-scope incident investigation. The evaluation emphasizes technical proficiency and artifact correlation over theoretical knowledge.

* **Operational Focus:** Defensive operations (detection, analysis, and response) versus offensive simulation.
* **Skill Validation:** Requires hands-on execution of phishing analysis, SIEM alert triage, artifact extraction (disk/memory), and packet analysis using industry-standard tools.

## Prerequisite Knowledge

To effectively navigate the BTL1 framework and these playbooks, engineers are expected to possess foundational knowledge in:
* TCP/IP networking fundamentals.
* Windows and Linux operating system architecture.
* Command-line interfaces (Bash, PowerShell).
* Basic security logging mechanisms.

## Exam Execution Parameters

The certification assessment is conducted under the following operational constraints:

* **Timeframe:** 24-hour continuous engagement window.
* **Infrastructure:** Cloud-hosted virtual lab environment containing multi-OS endpoints, network sensors, and pre-configured SIEM infrastructure.
* **Objective:** Investigate a multi-stage intrusion across various data sources (disk images, memory dumps, PCAPs, and aggregated logs).
* **Deliverable:** A formal incident report detailing the attack timeline, compromised assets, and specific findings based on scenario injects.

## Domain Mapping

The certification framework—and consequently, the structure of this playbook repository—maps to the following core domains:

1. **Phishing Analysis:** SMTP header inspection, malicious payload extraction, and URL pivoting.
2. **Threat Intelligence:** TTP categorization (MITRE ATT&CK mapping) and IOC enrichment.
3. **Digital Forensics:** Non-volatile storage artifact recovery and volatile memory (RAM) analysis.
4. **SIEM Analysis:** Log aggregation, event correlation, and query formulation (e.g., Splunk SPL).
5. **Network Analysis:** Deep packet inspection (DPI) of PCAP files using Wireshark and `tshark`.
6. **Incident Response:** Live endpoint triage, containment concepts, and evidence preservation.
