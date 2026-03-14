# Key Concepts — Threat Intelligence

Threat intelligence is what separates reacting to an alert from understanding an attack. The goal isn't just to identify that something bad happened — it's to understand who did it, how, and what they're likely to do next.

---

## Data vs intelligence

A list of malicious IPs is data. Intelligence is that list with context: which actor uses this infrastructure, what campaign it belongs to, what TTPs are associated with it, and what other indicators you should be looking for. Without context, you're just blocking individual data points.

---

## IOCs vs TTPs

**Indicators of Compromise (IOCs)** are the observable evidence of an attack — the artifacts left behind. They're the starting point for investigation but not the end goal.

| IOC type | Examples |
| :--- | :--- |
| IP addresses | C2 servers, phishing infrastructure, scanning sources |
| Domains | Malicious domains, DGA-generated names |
| File hashes | MD5 / SHA1 / SHA256 of malware samples |
| URLs | Phishing pages, malware download locations |
| Email addresses | Attacker sender addresses |
| Host artifacts | Registry keys, mutex names, specific file paths |

**Tactics, Techniques, and Procedures (TTPs)** describe attacker behavior — not what they left behind, but how they operated.

- **Tactic** — the objective: what goal is the attacker pursuing? (e.g., Persistence, Lateral Movement)
- **Technique** — the method: how are they achieving it? (e.g., T1053 Scheduled Task/Job)
- **Procedure** — the specific implementation: the exact tool or script used by a particular actor

IOCs are easy for attackers to change. TTPs are not. A compromised C2 IP gets replaced in hours; changing how an actor operates takes months.

---

## Types of intelligence

| Type | Focus | Useful for |
| :--- | :--- | :--- |
| Tactical | Specific IOCs for immediate blocking and detection | Alert triage, SIEM rules |
| Operational | Attacker TTPs and campaign infrastructure | Incident investigation, hunting |
| Strategic | Long-term threat landscape, actor motivations | Executive decisions, security planning |

In BTL1, you work mostly at the tactical and operational levels.

---

## The intelligence lifecycle

Even in a single investigation, you follow this pattern informally:

1. **Direction** — what do you need to know? (defined by the scenario)
2. **Collection** — gather data: logs, headers, sandbox output, tool results
3. **Processing** — extract and organize: pull IOCs, normalize formats
4. **Analysis** — interpret: what does this IP mean? Is this hash known malware? What TTPs are present?
5. **Dissemination** — report findings with context and evidence
6. **Feedback** — in real environments, this improves the next cycle

---

## The Pyramid of Pain

David Bianco's model shows which indicator types cause the most disruption to an attacker when defenders act on them.
```
