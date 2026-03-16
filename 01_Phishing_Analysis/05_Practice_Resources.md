# Practice Resources — Phishing Analysis

Hands-on practice is the only way to get fast at this. These are the platforms and sources worth your time.

---

## Labs and practice platforms

| Platform | What it offers |
| :--- | :--- |
| [LetsDefend](https://letsdefend.io/) | Phishing analysis module with simulated emails and virtual tools. One of the most directly relevant for BTL1 prep. Free and paid tiers. |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Sister platform to BTL1. Challenges frequently include phishing as part of broader DFIR investigations. |
| [CyberDefenders](https://cyberdefenders.org/) | CTF-style blue team challenges. Many start with a phishing email as the initial access vector. |
| [TryHackMe](https://tryhackme.com/) | SOC and analyst paths include phishing-related rooms alongside broader defensive content. |

---

## Sample email sources

> These contain real or realistic phishing URLs. Do not visit them directly. Use for hash lookups, reputation checks, and URLScan submissions only.

- [PhishTank](https://phishtank.org/) — Community-verified phishing URL database. Good for finding current active samples to analyze safely.
- [OpenPhish](https://openphish.com/) — Active phishing URL feeds. Handle with the same precautions as a live attachment.
- GitHub — Search `"phishing email samples" eml` for repositories with `.eml` files for offline analysis. Verify the source before downloading anything.

---

## Keeping up with current techniques

Phishing techniques evolve constantly. These are worth following:

- [Cofense Blog](https://cofense.com/blog/) — Dedicated phishing defense research
- [Proofpoint Threat Insight](https://www.proofpoint.com/us/blog/threat-insight) — Campaign analysis and emerging techniques
- [SANS ISC](https://isc.sans.edu/) — Daily threat reports, frequently includes email-based attack analysis
- Search for specific campaign analyses: `"Qakbot phishing analysis"`, `"HTML smuggling email"`, `"QR code phishing 2024"` — recent case studies from security researchers are some of the best learning material available

---

## Tool documentation worth reading

The tools in `02_Tools.md` all have solid documentation with examples that go beyond what any cheatsheet covers:

- VirusTotal API docs — useful once you want to automate lookups
- URLScan.io search syntax — lets you find related scans by domain, IP, or hash
- Any.Run knowledge base — explains how to read process trees and network artifacts in sandbox reports
