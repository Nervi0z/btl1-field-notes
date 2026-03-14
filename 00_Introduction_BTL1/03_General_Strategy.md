# General Strategy

Twenty-four hours is both more and less time than it sounds. Here's how to use it without losing the thread.

---

## Before you start the clock

Get these sorted before day one — not during.

**Tools**: Be comfortable with Volatility, Splunk, Wireshark, Autopsy, and KAPE before you sit the exam. The lab is not the time to learn a new tool. You should be able to open any of them and start working within 30 seconds.

**Note-taking setup**: Pick your tool — CherryTree, Obsidian, Notion, a structured text file — and have it ready. You'll be pasting commands, IPs, hashes, timestamps, and file paths constantly. An organized system at the start saves hours later.

**Workspace**: Good connection, minimal distractions, water and food nearby. These sound obvious until you're at hour 18.

---

## The first two hours

Don't start running tools. Start reading.

Read the scenario document more than once. Understand:
- What type of incident you're dealing with
- What the report requires — specific questions, format, evidence standards
- What the scope is — which systems, which timeframes

Then take a quick pass through the environment to understand what you have access to. Sketch a rough investigation plan. It will change, but having one stops you from drifting.

---

## Time management

There's no perfect split — every scenario is different — but some principles hold:

**Don't disappear into rabbit holes.** If you've been investigating something for a long time with no clear progress, mark it, document where you stopped, and move on. You can come back. What you can't do is spend four hours on one artifact and run out of time for everything else.

**Reserve time for the report.** Many candidates underestimate this. Writing a clear, evidence-backed report takes longer than writing notes for yourself. Build that time in from the start — or write sections of the report as you confirm findings, which is even better.

**Prioritize what the scenario explicitly asks for.** If there are specific questions, answer them directly. Don't assume good analysis will speak for itself — make sure everything the report asks for is addressed.

---

## Note-taking during the investigation

This is where most time is either saved or lost.

Document everything as you go:
- The exact commands you run
- What they returned — full output where relevant, not just summaries
- IOCs: IPs, hashes, usernames, file paths, timestamps
- Dead ends — what you checked and ruled out
- Hypotheses you're working with and whether the evidence supports them

The goal is a set of notes that could be turned into a report without having to go back and re-examine anything. If you have to reopen a tool to recover something you already found, your notes weren't complete enough.

---

## Writing the report

Write sections as you go when possible. It's far easier to document a finding when you've just confirmed it than to reconstruct it from notes four hours later.

When you write:
- Back every claim with evidence — exact data, not paraphrases
- Separate the executive summary from the technical detail
- Answer every question the scenario poses, explicitly
- Be precise: timestamps, hashes, paths, event IDs

---

## During the exam

Some things that sound obvious but get forgotten under pressure:

Take short breaks. Get up, move around, rest your eyes. At hour 12 without a break, analysis quality drops visibly. Five minutes away from the screen is not wasted time.

If you're blocked, change your approach before you change your mood. Try a different data source, a different tool, a different hypothesis. Persistence is useful; stubbornness isn't.

Stay methodical. When things feel chaotic, slow down and go back to the evidence.

---

<details>
<summary>A note on sleep</summary>

It's a 24-hour window, not a 24-hour sprint. If exhaustion is genuinely blocking your thinking, a short sleep is more productive than continuing to work badly. Most people don't need this, but it's worth knowing it's an option.

</details>
