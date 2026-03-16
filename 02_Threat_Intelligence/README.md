# Threat Intelligence

Threat intelligence in BTL1 is about giving context to what you're seeing. A malicious IP on its own tells you something happened — understanding what infrastructure it belongs to, what malware family it's associated with, and what TTPs the actor typically uses tells you what's actually going on and what to look for next.

This section covers the concepts, frameworks, and tools you need to enrich IOCs and map attacker behavior.

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [01_Key_Concepts.md](01_Key_Concepts.md) | IOCs vs TTPs, intelligence types, Pyramid of Pain, intelligence lifecycle |
| [02_MITRE_ATTACK.md](02_MITRE_ATTACK.md) | ATT&CK framework structure, how to use it during an investigation |
| [03_Tools.md](03_Tools.md) | Reputation platforms, DNS/WHOIS tools, passive DNS, device search engines |
| [04_IoC_Workflow.md](04_IoC_Workflow.md) | Step-by-step investigation workflows for IPs, domains, hashes, and URLs |
| [05_Practice_Resources.md](05_Practice_Resources.md) | Blogs, data platforms, training resources |

---

## Quick reference

**IOC types and where they come from:**

| IOC | Typical source |
| :--- | :--- |
| IP address | Email headers, SIEM logs, PCAP, memory analysis |
| Domain name | Phishing email, DNS logs, malware strings |
| File hash | Attachment, dropped file, memory carve |
| URL | Email body, proxy logs, sandbox report |
| Email address | `From:`, `Reply-To:`, `Return-Path:` |
| Registry key / mutex | Memory analysis, sandbox behavior report |

**Pyramid of Pain — what's worth blocking:**

| Indicator type | Pain for attacker | Notes |
| :--- | :--- | :--- |
| File hashes | Trivial | Recompile = new hash |
| IP addresses | Easy | Rotate infrastructure |
| Domain names | Moderate | Cost and effort to replace |
| Network artifacts | Annoying | User-agents, URI patterns |
| Host artifacts | Annoying | Filenames, registry keys |
| Tools | Significant | Takes time to replace or modify |
| TTPs | Painful | Forces fundamental behavior change |

**ATT&CK quick lookup:**
- Full matrix → [attack.mitre.org](https://attack.mitre.org/)
- Visual layer tool → [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)

---

> Start with `01_Key_Concepts.md` for the framework, then `04_IoC_Workflow.md` when you have indicators to investigate.
