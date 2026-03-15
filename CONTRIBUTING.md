# Contributing to btl1-field-notes

Corrections, updated syntax, and new detection patterns are welcome. This is a living reference — if something is wrong, outdated, or missing, that's worth fixing.

---

## What's in scope

- Corrected CLI syntax, commands, or tool flags
- Updated tool references when projects rename, move, or change syntax
- New SPL detection patterns, Wireshark filters, or Volatility plugin examples
- Additional artifact locations for Windows or Linux forensics
- Fixed or updated external links
- Improved explanations where something is unclear or inaccurate

## What won't be accepted

- Exam questions, answers, or scenario-specific content from any BTL1 instance
- Lab infrastructure details from the BTL1 environment
- Tool names used exclusively in the BTL1 exam (NDA compliance)
- Generic content that could appear in any security study guide unchanged
- Content with emojis, academic phrasing, or slide-deck-style bullet lists

The writing standard is practitioner voice — direct, concrete, real commands. If a section reads like a textbook, it doesn't fit.

---

## How to contribute

1. **Open an issue first** for anything beyond a typo or broken link — describe what's wrong and what you'd change
2. Fork the repository
3. Create a branch with a descriptive name:
   ```
   fix/volatility-malfind-syntax
   add/linux-proc-artifacts
   docs/splunk-lateral-movement-query
   ```
4. Make your changes
5. Commit with a clear message:
   ```
   git commit -m "forensics: correct vol3 cmdline plugin syntax for Linux"
   git commit -m "siem: add failed login spray detection pattern"
   ```
6. Open a pull request with a one-line description of what changed and why

---

## Style rules

- No emojis anywhere
- No "this section covers", "it is important to", "comprehensive", "leveraging"
- Every code block must contain real, working syntax — no placeholder pseudocode
- Short sentences. Concrete examples. If you're explaining a concept, use a real scenario
- Tables for structured data, prose for explanations — not bullet lists for everything

---

## NDA compliance

This repository contains general technical documentation and references to publicly available tools only. Do not submit:

- Specific BTL1 exam questions or answers
- Lab infrastructure details from any BTL1 exam instance
- Tool names used exclusively in the BTL1 environment
- Scenario-specific findings from any exam session

Pull requests containing NDA-restricted material will be closed without review.
