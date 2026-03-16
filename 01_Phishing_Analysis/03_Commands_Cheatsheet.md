# Commands Cheatsheet — Phishing Analysis

Fast reference for the CLI tasks that come up most during phishing triage. All of these run in Linux unless noted.

> Run everything in an isolated analysis VM. Never execute suspicious files on your main machine.

---

## File hashing

Hash first, look it up in VirusTotal before you do anything else with the file.
```bash
# MD5
md5sum suspicious_attachment.exe

# SHA256 (preferred)
sha256sum suspicious_attachment.exe

# Both at once
md5sum suspicious_attachment.exe && sha256sum suspicious_attachment.exe
```

**Windows (PowerShell):**
```powershell
Get-FileHash -Algorithm MD5 .\suspicious_attachment.exe
Get-FileHash -Algorithm SHA256 .\suspicious_attachment.exe
```

---

## File identification

Don't trust the extension. Check what the file actually is.
```bash
# identify file type by magic bytes, not extension
file suspicious_attachment.pdf

# common outputs to watch for:
# "PE32 executable" in something named .pdf → fake extension
# "Zip archive data" in something named .docx → might be fine (Office format) or hiding something
# "HTML document" in something named .exe → obfuscated dropper
```

---

## String extraction

Pull readable text out of a binary — often reveals embedded URLs, IPs, registry keys, or C2 addresses.
```bash
# basic string extraction
strings suspicious_attachment.exe

# filter for anything that looks like a URL
strings suspicious_attachment.exe | grep -i "http"

# filter for IP addresses
strings suspicious_attachment.exe | grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'

# save full output for review
strings suspicious_attachment.exe > strings_output.txt
```

---

## Downloading for analysis

Only in an isolated environment. Never execute what you download.
```bash
# download without executing
curl -O https://suspicious-url.com/payload.exe

# wget equivalent
wget https://suspicious-url.com/payload.exe

# download and immediately hash
curl -O https://suspicious-url.com/payload.exe && sha256sum payload.exe
```

---

## Working with email files
```bash
# view raw headers and body
cat email.eml

# extract just the headers
python3 -c "
import email
msg = email.message_from_file(open('email.eml'))
for key, val in msg.items():
    print(f'{key}: {val}')
"

# decode base64-encoded body or attachment
base64 -d encoded_content.b64 > decoded_output

# check MIME parts
python3 -c "
import email
msg = email.message_from_file(open('email.eml'))
for part in msg.walk():
    print(part.get_content_type(), '|', part.get_filename())
"
```

---

## Defanging for documentation
```bash
# manual pattern — replace in your notes before sharing
# http://evil.com/path  →  hxxp://evil[.]com/path

# sed one-liner for a URL in a variable
echo "http://evil.com/path" | sed 's/http/hxxp/g; s/\./[.]/g'
```
