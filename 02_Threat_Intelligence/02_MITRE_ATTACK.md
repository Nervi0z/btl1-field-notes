# MITRE ATT&CK

ATT&CK is a knowledge base of adversary behavior built from real-world attack observations. It's the standard framework for describing what attackers do and how defenders can detect or mitigate it.

ATT&CK = Adversarial Tactics, Techniques, and Common Knowledge.

---

## Structure

The Enterprise Matrix (the one relevant for BTL1) is organized into three layers:

**Tactics** — the *why*. The attacker's objective at each phase.

| ID | Tactic |
| :--- | :--- |
| TA0001 | Initial Access |
| TA0002 | Execution |
| TA0003 | Persistence |
| TA0004 | Privilege Escalation |
| TA0005 | Defense Evasion |
| TA0006 | Credential Access |
| TA0007 | Discovery |
| TA0008 | Lateral Movement |
| TA0009 | Collection |
| TA0011 | Command and Control |
| TA0010 | Exfiltration |
| TA0040 | Impact |

**Techniques** — the *how*. A specific method to achieve a tactic. Each has a unique ID.
- Example: `T1566` — Phishing (Initial Access)
- Example: `T1059` — Command and Scripting Interpreter (Execution)

**Sub-techniques** — a more specific variation of a technique.
- Example: `T1566.001` — Spearphishing Attachment
- Example: `T1566.002` — Spearphishing Link
- Example: `T1059.001` — PowerShell

**Procedures** — the real-world implementation. How a specific actor or malware sample used a technique, with references to public reporting.

---

## How to use ATT&CK during an investigation

The process is observation → lookup → context → documentation.

**Example — you find a suspicious scheduled task:**
```
1. Observe: A new scheduled task runs a PowerShell command at startup
2. Search ATT&CK: "scheduled task persistence windows"
3. Find: T1053.005 — Scheduled Task/Job: Scheduled Task
   Tactics: Persistence, Privilege Escalation
4. Read the page: understand how attackers use it, what artifacts it leaves
5. Document: "Evidence of T1053.005 — task name X runs Y command at Z time"
```

**Example — phishing email with malicious attachment:**
```
T1566.001 — Spearphishing Attachment (Initial Access)
T1204.002 — User Execution: Malicious File (Execution)
T1059.001 — PowerShell (Execution) — if macro drops PS script
T1547.001 — Registry Run Keys (Persistence) — if malware adds persistence
```

---

## ATT&CK Navigator

The Navigator lets you create layers on the matrix — color coding techniques you've observed, techniques you have detections for, or techniques associated with a specific actor group.

Useful for:
- Visualizing what TTPs appeared in an incident
- Comparing your detection coverage against an actor group's known TTPs
- Building investigation checklists for common attack patterns

→ [mitre-attack.github.io/attack-navigator](https://mitre-attack.github.io/attack-navigator/)

---

## In the BTL1 context

Mapping findings to ATT&CK is not always explicitly required, but doing it when you're confident in the identification strengthens your report. It shows you understand not just *what* happened but *why* that artifact is significant in the context of the attack.

Keep it honest — only map what you can actually support with evidence.
