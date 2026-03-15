# Evidence Acquisition

Acquisition is the process of creating a forensic copy of original evidence — disk or memory — in a way that preserves the original state. Everything downstream depends on doing this right. A corrupted or improperly acquired image can invalidate an entire investigation.

In BTL1 scenarios, images are typically provided. Understanding formats and integrity verification still matters — you'll reference hash values when documenting your chain of custody.

---

## Image formats

| Format | Extension | Notes |
| :--- | :--- | :--- |
| RAW/DD | `.dd`, `.img`, `.bin`, `.001` | Straight bit-for-bit copy. No metadata, no compression. Splits into segments with numbered extensions. Compatible with almost every tool. |
| EnCase | `.E01`, `.Ex01` | Includes case metadata, optional compression, and internal hashing per block. De facto standard in many professional environments. |
| AFF4 | `.aff4` | Open forensic format with advanced features. Less common but gaining traction. |
| Memory dump (raw) | `.raw`, `.mem`, `.bin` | Raw memory acquisition output from most tools. |
| Windows crash dump | `.dmp` | Windows memory dump format — readable by Volatility. |
| VMware memory | `.vmem` | VMware guest memory — readable by Volatility with associated `.vmss` file. |

---

## Physical vs logical — when each matters

**Physical image** copies the entire device sector by sector — allocated space, unallocated space, slack space, everything. Use this for investigations where you might need to recover deleted files or examine unallocated areas for carved artifacts.

**Logical image** copies only the active files and directory structure. Faster and smaller, but you lose anything that's been deleted and anything outside the active file system. Use when speed matters more than completeness, or when the investigation scope is limited to active files.

For BTL1: assume physical unless told otherwise.

---

## Disk acquisition

> Always connect the source disk through a write blocker before acquisition. Hardware write blockers are preferred. Software write blockers (like those built into FTK Imager) work when hardware isn't available.

```bash
# dd — basic bit-for-bit copy
# WARNING: double-check if= and of= before running — wrong direction destroys evidence
sudo dd if=/dev/sdb of=/mnt/evidence/disk.dd bs=4M status=progress

# dd with split output (2GB segments)
sudo dd if=/dev/sdb bs=4M status=progress | split -b 2G - /mnt/evidence/disk.

# dcfldd — dd with on-the-fly hashing and better progress output
sudo dcfldd if=/dev/sdb of=/mnt/evidence/disk.dd bs=4M \
  hash=sha256 sha256log=/mnt/evidence/disk.sha256 \
  hashwindow=1G \
  status=on

# verify the source disk block device before running
lsblk
fdisk -l
```

**FTK Imager** (Windows GUI): File → Create Disk Image → Physical Drive → select source → output as E01 with MD5+SHA1 hashing enabled.

---

## Memory acquisition

Memory acquisition must happen on the live running system — before shutdown. Once the system is off, the RAM contents are gone.

**Windows:**
```
# DumpIt (Magnet Forensics) — run as administrator
DumpIt.exe /O C:\evidence\memory.raw

# using FTK Imager: File → Capture Memory → set destination path
```

**Linux (LiME — Linux Memory Extractor):**
```bash
# LiME requires a kernel module compiled for the target kernel version
# Load the module and dump to a file
sudo insmod lime-<kernel_version>.ko "path=/mnt/evidence/memory.lime format=lime"

# dump over network (avoids writing to target disk)
sudo insmod lime-<kernel_version>.ko "path=tcp:4444 format=lime"
# on acquisition host:
nc <target_ip> 4444 > memory.lime
```

---

## Hashing and integrity verification

Hash values are how you prove the copy matches the original and that neither has been modified since acquisition.

**Acquisition integrity workflow:**

1. Hash the source disk before acquisition (or verify the hash is recorded by the acquisition tool)
2. Acquire the image
3. Hash the resulting image file
4. Compare — if they match, the copy is verified
5. Store the hash values with the case documentation
6. Re-verify the image hash before each analysis session

```bash
# Linux — hash a disk image
sha256sum disk.dd
sha256sum disk.dd > disk.dd.sha256

# verify against stored hash
sha256sum -c disk.dd.sha256

# MD5 (faster but weaker — use SHA256 as primary)
md5sum disk.dd

# hash multiple segments at once
sha256sum disk.00* > disk_segments.sha256
```

```powershell
# PowerShell — hash a file
Get-FileHash -Algorithm SHA256 .\disk.E01

# hash and save to file
Get-FileHash -Algorithm SHA256 .\disk.E01 | Export-Csv -Path .\disk_hash.csv
```

> SHA256 is the preferred algorithm. MD5 is still commonly used alongside it for compatibility, but MD5 alone is insufficient for evidence integrity — known collision attacks exist. If the tool only supports MD5, document that limitation.
