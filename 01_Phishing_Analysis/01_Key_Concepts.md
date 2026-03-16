# Key Concepts — Phishing Analysis

Phishing is the most common initial access vector across incident scenarios. The analysis goal is always the same: determine if the email is malicious, extract IOCs, and document everything that supports the verdict.

---

## The four things you examine

### 1. Headers

The delivery metadata — usually hidden in standard email clients. Headers tell you where the email actually came from, what path it took, and whether it passes authentication.

**Fields that matter most:**

| Field | Why it matters |
| :--- | :--- |
| `Received:` | Full server hop chain — read bottom to top |
| `From:` | Easily spoofed — compare display name vs actual address |
| `Reply-To:` | If different from `From:`, replies go somewhere else |
| `Return-Path:` | Actual bounce address — often exposes real sending domain |
| `Authentication-Results:` | SPF, DKIM, DMARC verdicts |
| `X-Originating-IP:` | Sending IP before the first MTA |

**SPF / DKIM / DMARC — what each checks:**

- **SPF** — Is the sending IP authorized to send mail for this domain?
- **DKIM** — Was the message signed by the domain it claims to be from, and is the signature intact?
- **DMARC** — If SPF or DKIM fail, what should the receiving server do? A `fail` here is a strong spoofing signal.

---

### 2. Body

Read for social engineering patterns before you touch anything:

- Urgency or threat language — *"your account will be suspended"*
- Generic greetings — *"Dear customer"* instead of your name
- Requests for credentials, wire transfers, or personal data
- Grammar and formatting inconsistencies
- Mismatched sender context — tone that doesn't fit previous communication from that entity

---

### 3. URLs

Never trust the link text. Always examine the actual destination.

Common patterns in malicious URLs:
- Lookalike domains — `paypaI.com` (capital i), `paypal.security-check.com`
- URL shorteners hiding the real destination
- Direct IP addresses instead of domain names
- Subdomains designed to look like the real domain in the left part of the URL

**Always defang before documenting:**
```
http://evil.com/steal  →  hxxp://evil[.]com/steal
```

---

### 4. Attachments

Never open directly. Treat every attachment as live until proven otherwise.

File types that warrant immediate suspicion:

| Type | Risk |
| :--- | :--- |
| `.exe`, `.scr`, `.bat`, `.ps1`, `.js`, `.vbs` | Direct execution |
| `.docm`, `.xlsm`, `.pptm` | Macro-enabled Office files |
| `.zip`, `.rar`, `.iso`, `.img` | Container for any of the above |
| `invoice.pdf.exe` | Double extension — real type hidden |
| Password-protected archives | Bypasses automated AV scanning |

---

## IOCs you extract from a phishing email

| IOC type | Where it comes from |
| :--- | :--- |
| Sending IP | `Received:` headers, `X-Originating-IP:` |
| Sender domains | `From:`, `Return-Path:`, `Reply-To:` |
| URLs | Body, HTML source |
| File hashes | Attachments (MD5, SHA1, SHA256) |
| Subject patterns | Useful for SIEM detection rules |
| Email addresses | `From:`, `Reply-To:` — pivot in threat intel |
