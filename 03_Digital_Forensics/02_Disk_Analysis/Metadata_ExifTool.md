# Metadata Analysis with ExifTool

---

## What metadata is and where it lives

Metadata is information about a file embedded inside the file itself — not visible when you open the document normally, but readable with the right tool. It's distinct from filesystem metadata (the timestamps and permissions recorded by the OS) and often survives when filesystem metadata has been wiped or modified.

Common types you'll encounter:

- **EXIF** — Found in JPEG, TIFF, and RAW images. Camera model, capture date and time, exposure settings, and often GPS coordinates. Created automatically by the capturing device.
- **Document metadata** — Found in PDF, Word (.docx), Excel (.xlsx). Author name, organization, software version used to create the file, creation and modification dates, sometimes revision history.
- **XMP** — Adobe's extensible metadata format, embedded in PDFs and image files. Often overlaps with EXIF but with richer editing history.
- **ID3 tags** — Audio metadata in MP3 files: artist, album, title, encoding software.

---

## Forensic value

**Attribution**: Author field in a Word document might show the real username or full name of whoever created it — even if the document was later modified by someone else. This has broken OPSEC for threat actors who forgot to sanitize documents before use in phishing campaigns.

**Timeline verification**: A photo's EXIF `DateTimeOriginal` reflects when the shutter was pressed (based on the camera's clock). If it contradicts the `File Modified` timestamp in the filesystem, that's an inconsistency worth documenting.

**Location data**: GPS coordinates in photo EXIF data can place a device at a specific location at a specific time. Even when coordinates aren't embedded, camera make/model can narrow attribution.

**Software fingerprinting**: The `Software` or `CreatorTool` field shows what application created or last modified the file, including version numbers. Useful for linking documents to a specific workstation or toolchain.

**Inconsistencies**: A PDF claiming to be created on a date before the software version listed in its metadata existed, or an image EXIF showing a different timezone than the filesystem — these are flags worth noting in a report.

---

## ExifTool commands

```bash
# view all metadata for a file
exiftool suspicious_document.pdf

# view specific fields
exiftool -Author -Creator -CreationDate suspicious_document.pdf

# view GPS coordinates if present
exiftool -GPSLatitude -GPSLongitude -GPSPosition photo.jpg

# process all files in a directory
exiftool /path/to/directory/

# recursive — process all files in directory and subdirectories
exiftool -r /path/to/directory/

# output as tab-delimited (useful for importing into spreadsheets)
exiftool -T -FileName -Author -CreationDate -Software *.pdf

# save output to a text file
exiftool suspicious_document.pdf > metadata_output.txt

# save metadata for every file in a directory (creates a .txt per file)
exiftool -w txt /path/to/directory/

# output all fields in CSV format
exiftool -csv *.jpg > metadata.csv

# check ExifTool version
exiftool -ver
```

**Removing metadata** (only on copies — never on original evidence):
```bash
# remove all metadata from a file
exiftool -all= document.pdf

# remove GPS only
exiftool -GPS:all= photo.jpg

# remove metadata from all files in a directory
exiftool -all= /path/to/directory/
```

---

## What to look for

**Author clues:**
- `Author`, `Creator`, `Last Modified By` — real usernames or full names
- Compare against known usernames in the investigation
- Non-ASCII characters or locale-specific names can indicate geographic origin

**Timeline clues:**
- `Create Date`, `Modify Date`, `DateTimeOriginal` — compare against filesystem timestamps
- Timezone offsets in EXIF can contradict claimed location
- Modification timestamps after the claimed "creation" date indicate editing

**GPS and location:**
- `GPS Latitude`, `GPS Longitude`, `GPS Position` — convert to decimal degrees for mapping
- `GPS Altitude`, `GPS Date/Time` — full location picture
- `Make`, `Model` — device that captured the file

**Inconsistencies to flag:**
- Creation date earlier than the software version that supposedly created the file
- Author field showing a different username than the one who allegedly sent it
- GPS coordinates placing a device in a location inconsistent with claimed facts
- `ColorSpace`, `ResolutionUnit` combinations that don't match the claimed camera model
- Metadata timestamps in a different timezone than the system timezone
