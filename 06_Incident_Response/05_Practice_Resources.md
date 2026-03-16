# Practice Resources — Incident Response

---

## IR-focused lab platforms

| Platform | What it offers |
| :--- | :--- |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Investigation scenarios that walk through full IR cases — phishing to forensics to containment. The closest free equivalent to the BTL1 lab format. |
| [CyberDefenders](https://cyberdefenders.org/) | Multi-stage IR challenges with realistic evidence packages. "Endpoint Forensics" and "Network Forensics" tags are most relevant. |
| [LetsDefend](https://letsdefend.io/) | Simulated SOC environment with alert triaging and incident management workflow. |
| [TryHackMe](https://tryhackme.com/) | IR-specific learning paths and rooms — Incident Response, Windows Forensics, Linux forensics. |
| [Hack The Box — Sherlocks](https://app.hackthebox.com/sherlocks) | DFIR and IR challenges focused on evidence analysis. Higher difficulty than BTLO/CyberDefenders. |

---

## Tabletop and scenario resources

Tabletop exercises walk through IR scenarios in discussion format — useful for building decision-making instincts without needing a full lab environment.

- **[CISA Tabletop Exercise Packages (CTEPs)](https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages)** — Free scenario packages for ransomware, insider threat, and supply chain attacks. Download the exercise guide and work through the scenario.
- **[MITRE ATT&CK Evaluations](https://attackevals.mitre-engenuity.org/)** — Published results of detection evaluations against known adversary TTPs. Useful for understanding what an attack chain looks like end-to-end.
- **[The DFIR Report](https://thedfirreport.com/)** — Real intrusion case studies from initial access through post-exploitation. Read these as tabletop scenarios — try to answer "what would I look for next?" before reading the next section.

---

## IR frameworks and documentation

| Resource | What it covers |
| :--- | :--- |
| [NIST SP 800-61r2](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final) | NIST's Computer Security Incident Handling Guide — the reference framework for IR lifecycle and documentation. Free PDF. |
| [SANS Incident Handler's Handbook](https://www.sans.org/white-papers/33901/) | Practical incident handling guide from SANS. More operational than NIST, more narrative. Free with registration. |
| [CISA IR Playbooks](https://www.cisa.gov/sites/default/files/publications/Federal_Government_Cybersecurity_Incident_and_Vulnerability_Response_Playbooks_508C.pdf) | Federal incident response playbooks — ransomware and vulnerability response. Concrete decision trees. |
| [MITRE D3FEND](https://d3fend.mitre.org/) | Defensive techniques mapped to ATT&CK. Useful for identifying containment and eradication actions tied to specific techniques. |

---

## Live response tool documentation

- **[Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/)** — Autoruns, Process Explorer, TCPView, ProcMon, and others. Each tool has documentation on Microsoft's site. Autoruns is the most important for persistence hunting.
- **[LiME (Linux Memory Extractor)](https://github.com/504ensicsLabs/LiME)** — GitHub repository with build instructions and usage examples for Linux memory acquisition.
- **[GRR Rapid Response](https://github.com/google/grr)** — Google's open-source remote live response framework. Worth understanding at a conceptual level even if you don't deploy it.
