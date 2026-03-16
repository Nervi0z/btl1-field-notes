# How BTL1 Thinks About Assessment

Most certifications test whether you know something. BTL1 tests whether you can do something under pressure, with incomplete information, and a clock running.

That distinction changes how you need to prepare.

---

## It's hypothesis-driven, not checklist-driven

The lab doesn't hand you a list of tasks in order. You get a scenario, some initial indicators, and access to the evidence. What you do with that is up to you.

The expected approach is something like:

1. **Form a hypothesis** based on what you're seeing — *"if this process was injected, there should be a suspicious network connection tied to its PID"*
2. **Test it** using the available toolset
3. **Pivot** when the evidence doesn't support it, without losing time
4. **Correlate across sources** — disk, memory, network, and logs don't tell separate stories; they tell one story from different angles

Running tools without a clear reason for running them is how you waste hours. The lab rewards analysts who think before they type.

---

## The report is half the score

Technical findings that never make it into the report might as well not exist. The final incident report is the primary deliverable — it's what gets graded.

That means:

- Every claim needs evidence: exact timestamps, log snippets, hashes, screenshots
- The structure matters: executive summary separate from the technical detail
- Nothing can be left unanswered: if the scenario asks a specific question, it needs a specific answer

A common failure mode is spending 20 hours on excellent analysis and two hours on a rushed report. The analysis doesn't show through unless the writing does.

---

## Tool proficiency goes beyond syntax

Knowing what a tool is called is not the same as knowing how to use it. BTL1 expects you to:

- Know which tool fits which problem — Volatility for volatile memory, Autopsy for disk artifacts, Splunk for log correlation
- Write queries and filters that cut through noise — optimized SPL, precise Wireshark display filters, targeted Volatility modules
- Interpret raw output — hex, Sysmon events, PCAP streams — and translate it into a clear timeline

The environment gives you the tools. What you do with them is the assessment.

---

## The 24-hour window is a constraint, not a buffer

Twenty-four hours sounds like a lot. It isn't, once you factor in:

- Reading and understanding the scenario (don't skip this)
- Working across five domains with overlapping evidence
- Writing a detailed report with supporting screenshots and timestamps
- Any time lost to dead ends or tool issues

The time pressure is intentional. It simulates the operational tempo of a real SOC incident. Analysts who pass are the ones who stay methodical when the clock is running.

---

> The assessment isn't designed to catch you out — it's designed to see how you work. Methodical, evidence-backed, clearly documented analysis is what it's looking for.
