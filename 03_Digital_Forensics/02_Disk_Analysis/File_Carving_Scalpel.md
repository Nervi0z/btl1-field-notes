# File Carving with Scalpel

---

## What file carving is

File carving recovers files by searching raw disk data for known file signatures — without using the filesystem. When a file is deleted, the filesystem marks the space as available but rarely overwrites the data immediately. Carving scans that unallocated space byte by byte, looking for the header and footer patterns that mark the start and end of known file types.

**When to use it:**
- A disk image with a corrupt or missing filesystem
- Recovering deleted files that aren't showing up in filesystem analysis
- Searching unallocated space for files that were removed to cover tracks

**Limitations:**
- Recovered files have no original filenames or timestamps — those live in the filesystem, not the data
- Files can be fragmented — if the fragments aren't contiguous, the recovered file will be incomplete or corrupt
- High false positive rate — some binary data matches file headers by coincidence
- Large files are more likely to be fragmented and therefore incomplete after carving

---

## Magic bytes — file signatures

Carving works because file formats start with predictable byte sequences. These are the most common ones you'll configure Scalpel to look for:

| File type | Header (hex) | Footer (hex) | Notes |
| :--- | :--- | :--- | :--- |
| JPEG | `FF D8 FF` | `FF D9` | Most common image format |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | `49 45 4E 44 AE 42 60 82` | |
| PDF | `25 50 44 46` (%PDF) | `25 25 45 4F 46` (%%EOF) | Footer can vary |
| ZIP | `50 4B 03 04` | `50 4B 05 06` | Also covers .docx, .xlsx, .pptx, .jar |
| PE executable | `4D 5A` (MZ) | None defined | Windows binaries |
| GIF | `47 49 46 38` (GIF8) | `00 3B` | |
| MP4/MOV | `66 74 79 70` at offset 4 | — | |
| SQLite | `53 51 4C 69 74 65 20 33` | — | Browser databases, many app databases |

---

## Configuring scalpel.conf

Scalpel does nothing until you tell it what to look for. The config file `/etc/scalpel/scalpel.conf` has entries for dozens of file types, all commented out by default.

**Before (commented, ignored):**
```
#	jpg	y	200000000	\xff\xd8\xff\xe0\x00\x10	\xff\xd9
#	pdf	y	5000000		%PDF	%%EOF
```

**After (uncommented, active):**
```
	jpg	y	200000000	\xff\xd8\xff\xe0\x00\x10	\xff\xd9
	pdf	y	5000000		%PDF	%%EOF
```

**Column meaning:** `extension | case-sensitive | max_file_size | header | footer`

Make a copy of the config before editing:
```bash
cp /etc/scalpel/scalpel.conf ~/my_scalpel.conf
```

Edit your copy to uncomment only the file types you need. Uncommenting everything creates enormous output with many false positives.

---

## Running Scalpel

```bash
# basic carve — analyze entire image
scalpel -c ~/my_scalpel.conf -o /mnt/evidence/carve_output disk.dd

# carve only unallocated space — use -b flag
# recommended for disk images where the filesystem is intact
scalpel -b -c ~/my_scalpel.conf -o /mnt/evidence/carve_output disk.dd

# the output directory must be empty or not exist before running
mkdir -p /mnt/evidence/carve_output
```

> The `-b` flag tells Scalpel to skip allocated space and focus on unallocated areas. Use it when the filesystem is intact — it reduces output size and processing time significantly.

---

## After carving

Scalpel creates a subdirectory for each file type in your output folder:

```bash
ls /mnt/evidence/carve_output/
# jpg-1-0/    pdf-1-0/    audit.txt

# audit.txt contains a summary of what was found
cat /mnt/evidence/carve_output/audit.txt

# count recovered files per type
ls /mnt/evidence/carve_output/jpg-1-0/ | wc -l

# hash all recovered files for documentation
find /mnt/evidence/carve_output/ -type f -exec sha256sum {} \; > carved_hashes.txt
```

Recovered filenames will look like `0000012345.jpg` — the number is the byte offset in the image where Scalpel found the header. This offset can be used to locate the file within the image for additional analysis.

Look up hashes of interesting carved files in VirusTotal before opening them. A carved .exe from unallocated space is almost more suspicious than one found in the active filesystem.
