# Phishing Analysis

Most incidents start here. A malicious email lands in someone's inbox, they click something they shouldn't, and you're the one reconstructing what happened. This section covers how to pull apart a suspicious email systematically — headers, body, URLs, attachments — and extract everything useful before escalating.

---

## What's in this section

| File | What it covers |
| :--- | :--- |
| [01_Key_Concepts.md](01_Key_Concepts.md) | Header fields, authentication checks, IOC types |
| [02_Tools.md](02_Tools.md) | Reputation services, sandboxes, header analyzers, URL inspection |
| [03_Commands_Cheatsheet.md](03_Commands_Cheatsheet.md) | CLI commands for hashing, file identification, string extraction |
| [04_Analysis_Workflow.md](04_Analysis_Workflow.md) | Step-by-step triage from receipt to documented verdict |
| [05_Practice_Resources.md](05_Practice_Resources.md) | Labs, sample emails, platforms, blogs |

---

## Quick reference

**Header fields to check first:**

| Field | What to look for |
| :--- | :--- |
| `From:` | Display name vs actual address — do they match? |
| `Reply-To:` | Different from `From:`? Common in BEC and credential phishing |
| `Return-Path:` | Where bounces go — often reveals true sending infrastructure |
| `Received:` | Full delivery chain — read bottom to top |
| `Authentication-Results:` | SPF, DKIM, DMARC verdicts in one place |
| `X-Originating-IP:` | Sending IP before hitting the MTA |

**Authentication verdict quick read:**

| Result | Meaning |
| :--- | :--- |
| SPF `fail` | Sending IP not authorized by the domain's DNS records |
| DKIM `fail` | Signature invalid — message altered or spoofed |
| DMARC `fail` | Neither SPF nor DKIM passed alignment — strong spoofing indicator |

**Defanging for safe documentation:**
```
http://malicious-site.com/login  →  hxxp://malicious-site[.]com/login
```

---

> Start with `04_Analysis_Workflow.md` for the full triage sequence. Jump to `03_Commands_Cheatsheet.md` if you need syntax fast.
