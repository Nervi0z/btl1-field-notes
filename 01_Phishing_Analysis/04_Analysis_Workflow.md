# Analysis Workflow — Phishing Triage

A structured sequence for working through a suspicious email from receipt to documented verdict. Adapt this to what the scenario gives you — not every case will have attachments, and some will have nothing but a suspicious link.

> Work inside an isolated VM. Treat the email as live until you've confirmed otherwise.

---

## Step 1 — Get the email into a safe format

If possible, work with the raw `.eml` or `.msg` file rather than viewing it in a client. This preserves the complete headers and lets you examine everything without triggering anything.
```bash
# view raw content
cat suspicious.eml
```

---

## Step 2 — Initial read-through (no clicking)

Before touching any tools, read the email with your eyes:

- Does the sender address match the display name?
- Is there urgency, threat language, or an unusual request?
- Hover over links — does the destination match the link text?
- Are there attachments you weren't expecting?

Document your initial observations. They'll inform your hypothesis for the rest of the analysis.

---

## Step 3 — Header analysis

Extract the full headers and run them through a parser (MxToolbox or Google Messageheader).

Work through this checklist:
```
[ ] Trace the Received: chain bottom to top — identify the actual source IP
[ ] Check SPF result — pass or fail?
[ ] Check DKIM result — pass or fail?
[ ] Check DMARC result — pass or fail?
[ ] Compare From: vs Reply-To: vs Return-Path: — any mismatches?
[ ] Look up the source IP in AbuseIPDB and VirusTotal
[ ] Note any suspicious sending domains
```

**Any DMARC fail combined with a From: domain impersonating a known brand is a strong spoofing indicator.**

---

## Step 4 — Body analysis

Read for social engineering technique and extract anything actionable:
```
[ ] What's the pretext? (invoice, account alert, delivery notification, HR communication)
[ ] Does it request credentials, payment, or personal data?
[ ] List every URL in the body — don't click, just collect
[ ] Note any phone numbers or secondary contact details
[ ] Check if the HTML source contains hidden links different from visible text
```

---

## Step 5 — URL analysis

For each URL collected in Step 4:
```bash
# defang first
echo "http://suspicious-domain.com/login" | sed 's/http/hxxp/g; s/\./[.]/g'
```

Then for each defanged URL:
```
[ ] Look up in VirusTotal — check domain age, detections, related files
[ ] Submit to URLScan.io — get a screenshot and contacted domain list
[ ] If shortened URL, expand it first and analyze the destination
[ ] Check the domain in OTX for threat intel context
```

---

## Step 6 — Attachment analysis

**Do not open directly.**
```bash
# identify real file type
file suspicious_attachment.pdf

# hash it
sha256sum suspicious_attachment.pdf
md5sum suspicious_attachment.pdf
```
```
[ ] Look up both hashes in VirusTotal
[ ] If unknown hash — submit to sandbox (Any.Run, Hybrid Analysis, Triage)
[ ] Run strings and grep for embedded URLs or IPs
[ ] Check for double extension (invoice.pdf.exe)
[ ] If Office file — check for macro indicators (VBA strings, AutoOpen, Document_Open)
```

---

## Step 7 — Consolidate IOCs

Collect everything into a clean list before writing the verdict:

| IOC type | Value | Source |
| :--- | :--- | :--- |
| Sender IP | `x.x.x.x` | `Received:` header |
| Sending domain | `evil-domain.com` | `From:` / `Return-Path:` |
| URL | `hxxp://phish[.]com/login` | Email body |
| File hash (SHA256) | `abc123...` | Attachment |
| Reply-To address | `attacker@gmail.com` | `Reply-To:` header |

---

## Step 8 — Verdict and documentation
```
Verdict options: Malicious / Suspicious / Legitimate / Inconclusive

For each finding:
- What did you observe?
- What tool confirmed it?
- What does it mean for the verdict?
```

Write a brief summary — one paragraph is enough — explaining the verdict and the two or three key pieces of evidence that drove it. That summary becomes the basis for your report section or escalation note.
