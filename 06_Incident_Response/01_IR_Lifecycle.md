# Incident Response Lifecycle

Six phases. In practice they overlap — containment often starts before analysis is complete, and lessons learned happen throughout rather than just at the end.

---

## Phase 1 — Preparation

Getting ready before an incident happens: building playbooks, configuring logging, deploying detection tooling, establishing communication channels, and training the team.

**What BTL1 tests**: Preparation shows in how you approach the lab — having your note-taking system ready, knowing which tool to use for which task, and not wasting time searching for basic command syntax you should already know.

**Common failure**: Spending the first two hours of the exam configuring your workspace and learning tools rather than investigating.

---

## Phase 2 — Identification

Detecting that an incident has occurred, determining its scope, and confirming it's a real incident rather than a false positive.

**What BTL1 tests**: Reading the scenario carefully, identifying the initial indicators, and scoping the investigation before running tools. Understanding which hosts and accounts are in scope.

**Common failure**: Starting tool runs before reading the full scenario and understanding what you're looking for. Finding evidence of one thing and missing that the scenario requires you to find something adjacent.

---

## Phase 3 — Containment

Stopping the attacker from continuing to cause damage — without destroying evidence needed for analysis.

Two types of containment:
- **Short-term**: Immediate isolation to stop the bleeding. Network isolation of affected hosts, disabling compromised accounts.
- **Long-term**: More measured steps that don't destroy evidence — restricting access, monitoring for attacker return, blocking known IOCs.

**What BTL1 tests**: Understanding that containment comes *after* evidence collection on affected systems, not before. Knowing the commands to isolate a host on Windows and Linux.

**Common failure**: Recommending a system shutdown before collecting volatile data (memory dump, running processes, network connections). Shutdown destroys everything in RAM.

> Before isolating a system: collect a memory dump, document running processes and network connections, capture relevant log files. Then isolate.

---

## Phase 4 — Eradication

Removing the attacker's presence: deleting malware, closing access paths, removing persistence mechanisms.

**What BTL1 tests**: Identifying specific persistence mechanisms found during analysis (registry Run keys, scheduled tasks, malicious services, cron jobs) and knowing how to remove them. Verifying the attacker hasn't left secondary persistence.

**Common failure**: Removing the obvious piece of malware without checking for secondary persistence. Attackers routinely plant backup access points.

---

## Phase 5 — Recovery

Restoring systems to normal operation — verifying they're clean, restoring from known-good backups if needed, and monitoring for attacker return.

**What BTL1 tests**: This phase is mostly outside the exam scope, but understanding it conceptually helps frame findings in your report — what would need to happen next.

**Common failure**: Recovering before fully confirming eradication. Systems restored too early get reinfected via residual persistence that wasn't caught.

---

## Phase 6 — Lessons Learned

Documenting what happened, what worked, what didn't, and what changes would prevent recurrence.

**What BTL1 tests**: The incident report is the primary deliverable — it's the written version of lessons learned. A clear timeline, identified root cause, recommendations for remediation and detection improvement.

**Common failure**: Writing findings as a log of what you did rather than a narrative of what the attacker did. The report should tell the attacker's story from initial access to detection, supported by evidence from your investigation.
