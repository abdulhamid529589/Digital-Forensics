# 🔍 Digital Forensics — Comprehensive Practical Lab Guide

### University Cybersecurity Course | Practical Series Part 1

> **⚠️ Legal & Ethical Notice:** Digital forensics work must only be performed on devices/systems you own, have explicit written authorization to examine, or on dedicated lab environments. Unauthorized access to digital evidence is illegal and can compromise real criminal investigations. Always follow proper chain-of-custody procedures.

---

## 📋 Table of Contents

1. [Introduction to Digital Forensics](#1-introduction-to-digital-forensics)
2. [Lab Environment Setup](#2-lab-environment-setup)
3. [Data Recovery](#3-data-recovery)
4. [Hash Values & File Integrity](#4-hash-values--file-integrity)
5. [Forensic Disk Imaging](#5-forensic-disk-imaging)
6. [Case Investigation with Autopsy / FTK](#6-case-investigation-with-autopsy--ftk)
7. [Metadata Analysis](#7-metadata-analysis)
8. [Malware Identification via Hash Lookup](#8-malware-identification-via-hash-lookup)
9. [Chain of Custody & Evidence Handling](#9-chain-of-custody--evidence-handling)
10. [Forensics Career Roadmap](#10-forensics-career-roadmap)

---

## 1. Introduction to Digital Forensics

### 1.1 What Is Digital Forensics?

Digital forensics is the science of **collecting, preserving, analyzing, and presenting digital evidence** in a way that is legally admissible in court. It is used in:

- Cybercrime investigations
- Corporate incident response
- Civil litigation
- Law enforcement investigations
- Internal security audits

### 1.2 Core Principles

```
┌─────────────────────────────────────────────────────────┐
│              Digital Forensics Principles               │
├─────────────────┬───────────────────────────────────────┤
│ Integrity       │ Evidence must not be altered          │
│ Authenticity    │ Evidence must be provably original    │
│ Chain of Custody│ Every handler must be documented      │
│ Repeatability   │ Results must be reproducible          │
│ Non-repudiation │ Actions cannot be denied              │
└─────────────────┴───────────────────────────────────────┘
```

### 1.3 The Forensics Investigation Process

```
1. Identification   → Recognize potential digital evidence
2. Preservation     → Protect evidence from alteration (imaging)
3. Collection       → Gather evidence systematically
4. Examination      → Extract relevant data
5. Analysis         → Interpret the data
6. Presentation     → Report findings for court/stakeholders
```

### 1.4 Key Rule: Never Investigate Original Evidence

> You **always** work on a **forensic image (copy)** of the original evidence. Working on the original can:
>
> - Alter timestamps and metadata
> - Overwrite deleted data
> - Make evidence inadmissible in court
> - Destroy the chain of custody

---

## 2. Lab Environment Setup

### 2.1 Recommended Tools

| Tool             | Purpose                          | Cost             |
| ---------------- | -------------------------------- | ---------------- |
| SS Data Recovery | File/data recovery from drives   | Free demo / Paid |
| Recuva           | File recovery (Windows)          | Free             |
| Hash Calculator  | MD5/SHA hash computation         | Free             |
| VirusTotal       | Online hash-based malware lookup | Free             |
| R-Drive Image    | Forensic disk imaging            | Trial / Paid     |
| Autopsy          | Full case investigation suite    | Free             |
| FTK Imager       | Forensic imaging + preview       | Free             |
| Wireshark        | Network forensics                | Free             |
| Volatility       | RAM/memory forensics             | Free             |

### 2.2 Lab Hardware

- A **dedicated forensics PC or VM** (not your personal machine)
- USB write blocker (hardware) — prevents modifying evidence during examination
- A test USB drive / pen drive to practice on
- External hard drive for storing forensic images

### 2.3 Safe Lab Setup

```
                ┌─────────────────────────┐
                │    FORENSICS WORKSTATION │
                │    (Isolated / VM)       │
                └────────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
       ┌──────┴──────┐ ┌─────┴─────┐ ┌────┴─────┐
       │ Evidence    │ │ Forensic  │ │ Output   │
       │ (Original)  │ │ Image     │ │ Reports  │
       │ READ-ONLY   │ │ (.E01)    │ │ + Hashes │
       └─────────────┘ └───────────┘ └──────────┘
```

**Never connect evidence drives to an unprotected system.**

---

## 3. Data Recovery

### 3.1 Why Data Recovery Matters in Forensics

When criminals delete files to hide evidence, the data is often still recoverable because:

- Deleting a file only removes the **directory entry** (pointer)
- The actual data remains on disk until **overwritten** by new data
- Tools can scan for these "orphaned" data blocks and reconstruct files

### 3.2 How Deletion Works

```
Normal Delete (Recycle Bin):
File → Recycle Bin → File reference still exists, data intact

Shift+Delete (Permanent Delete):
File → Directory entry removed → Data still on disk (unallocated space)

Secure Wipe (DOD 7-pass):
File → Data overwritten multiple times → Very difficult to recover
```

### 3.3 Lab: Data Recovery with SS Data Recovery

#### Preparation — Create Test Evidence

```
1. Insert a test USB drive (use one you own)
2. Open the USB in File Explorer
3. Create a new text document: "Secret_Notes.txt"
4. Write some content inside it (e.g., "Hello Cyber Warriors - Test File")
5. Save and close the file
6. Delete it using Shift+Delete (permanent delete)
7. Also empty the Recycle Bin to confirm deletion
8. Verify: open the drive — it should appear empty
```

#### Running the Recovery Tool

```
Step 1: Open SS Data Recovery (or Recuva / TestDisk)
Step 2: Select the drive/partition to scan
         → Choose your USB drive (e.g., G: drive)
         → Do NOT choose your system drive (C:)

Step 3: Run a deep scan
         → Wait for 100% completion
         → Note: recovered data often EXCEEDS actual drive size
           (because old overwritten data is also found)

Step 4: Review recovered files
         → Deleted files section
         → Lost files section
         → Raw/unformatted file section

Step 5: Recover target file
         → Select "Secret_Notes.txt" (or your deleted file)
         → IMPORTANT: Save to a DIFFERENT drive, not the source drive
           (e.g., source = G: → save recovered file to D: or E:)

Step 6: Verify recovery
         → Open the recovered file
         → Confirm original content is intact
```

#### Why More Data Than Expected?

When you scan a 10 GB drive, you may find 80–110 GB of recoverable data. This is because:

- Files are stored in **sectors** that persist through multiple uses
- Old data from previous owners may still be in unused sectors
- The tool reads all raw sectors, not just the active file system
- Files from years or even decades ago can appear (2001, 2004, 2015, etc.)

### 3.4 Recovery Rules for Investigators

```
✅ Always recover to a separate evidence storage drive
✅ Record the names, sizes, and dates of all recovered files
✅ Calculate and record hash values of recovered files immediately
✅ If one tool fails, try multiple tools (each tool has different capabilities)
✅ Document every step taken
✅ Photograph the drive before and after connection
```

### 3.5 Common Recovery Tools Compared

| Tool             | Best For                           | Speed  | Depth     |
| ---------------- | ---------------------------------- | ------ | --------- |
| Recuva           | Simple recovery, photos, documents | Fast   | Medium    |
| SS Data Recovery | Deep scan, multiple file types     | Medium | High      |
| TestDisk (free)  | Partition recovery, Linux          | Medium | High      |
| R-Studio         | Professional, RAID recovery        | Slow   | Very High |
| Autopsy          | Full forensic case + recovery      | Slow   | Very High |

---

## 4. Hash Values & File Integrity

### 4.1 What Is a Hash Value?

A hash value is a **unique fixed-length digital fingerprint** of a file. It is generated by a mathematical algorithm called a **hash function**.

Think of it like an **Aadhaar card number for files**:

- Every file gets a unique number
- If even one character inside the file changes, the hash changes completely
- The file name, location, or extension does NOT affect the hash
- Only internal data changes affect the hash

### 4.2 Why Hash Values Are Critical in Forensics

```
Use Case 1: Proving Integrity
   → Hash the evidence file immediately after recovery
   → Hash it again later before presenting to court
   → If hashes match = no tampering occurred

Use Case 2: Identifying Malware
   → Calculate hash of suspicious file
   → Look it up on VirusTotal
   → Database identifies known malware by hash

Use Case 3: Deduplication
   → Find duplicate files in large evidence sets
   → Two files with same hash = identical files

Use Case 4: Verifying Forensic Images
   → Hash the source drive before imaging
   → Hash the forensic image after creation
   → Hashes must match = perfect copy
```

### 4.3 Common Hash Algorithms

| Algorithm | Output Length           | Status                       | Use Case               |
| --------- | ----------------------- | ---------------------------- | ---------------------- |
| MD5       | 128-bit (32 hex chars)  | Old, but common in forensics | Quick integrity checks |
| SHA-1     | 160-bit (40 hex chars)  | Deprecated for security      | Legacy systems         |
| SHA-256   | 256-bit (64 hex chars)  | Current standard             | General forensics      |
| SHA-512   | 512-bit (128 hex chars) | High security                | Sensitive evidence     |

### 4.4 Lab: Calculating Hash Values

#### Using Hash Calculator Tool (GUI)

```
Step 1: Open Hash Calculator
Step 2: Browse and select your file (e.g., Secret_Notes.txt)
Step 3: Select algorithms: MD5, SHA-256, SHA-512
Step 4: Click Calculate
Step 5: Copy and record all hash values in your evidence log

Example output:
   File: Secret_Notes.txt
   MD5:    5f4dcc3b5aa765d61d8327deb882cf99
   SHA256: ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f
```

#### Using Command Line (Windows)

```powershell
# MD5
Get-FileHash "C:\Evidence\Secret_Notes.txt" -Algorithm MD5

# SHA256
Get-FileHash "C:\Evidence\Secret_Notes.txt" -Algorithm SHA256

# SHA512
Get-FileHash "C:\Evidence\Secret_Notes.txt" -Algorithm SHA512
```

#### Using Command Line (Linux/Kali)

```bash
md5sum    Secret_Notes.txt
sha256sum Secret_Notes.txt
sha512sum Secret_Notes.txt

# Hash multiple files at once
sha256sum /path/to/evidence/* > evidence_hashes.txt
```

### 4.5 Lab: Demonstrating Hash Sensitivity

```
Test 1: Original file
   Content: "Hello Cyber Warriors Please Subscribe This Channel"
   MD5: [record value A]

Test 2: Add a period (.)
   Content: "Hello Cyber Warriors Please Subscribe This Channel."
   MD5: [record value B]  ← Completely different from A

Test 3: Remove the period
   Content: "Hello Cyber Warriors Please Subscribe This Channel"
   MD5: [record value C]  ← Should equal A (matches original)

Test 4: Add extra space
   Content: "Hello Cyber Warriors Please Subscribe This Channel "
   MD5: [record value D]  ← Different again due to trailing space

Test 5: Rename the file
   File name changed, but content unchanged
   MD5: [record value E]  ← Same as A (name doesn't affect hash)
```

**Key findings from this test:**

- Any internal change, no matter how small, changes the hash
- File name changes do NOT change the hash
- This makes hashes reliable for integrity verification

### 4.6 HMAC — Hash Message Authentication Code

HMAC adds a **secret key** to the hash calculation. This means:

- Only someone with the correct key can generate or verify the hash
- Provides authentication in addition to integrity

```
Normal Hash:   SHA256(data)         = fixed result
HMAC:          SHA256(key + data)   = result tied to your secret key

Use case: When you need to prove BOTH that:
  1. The data has not changed (integrity)
  2. You are the one who created the hash (authentication)
```

```bash
# HMAC on Linux
echo -n "Hello Cyber Warriors" | openssl dgst -sha256 -hmac "mysecretkey"

# In Python
import hmac, hashlib
result = hmac.new(b'mysecretkey', b'Hello Cyber Warriors', hashlib.sha256).hexdigest()
print(result)
```

> **Note for court presentation:** When presenting evidence to court, generally use standard hashes (MD5/SHA256) without HMAC, so the process is simpler for judges to understand. HMAC is useful for internal authentication systems.

### 4.7 Evidence Log Template

```
═══════════════════════════════════════════════════════
          DIGITAL EVIDENCE HASH LOG
═══════════════════════════════════════════════════════
Case Number    : [CASE-2025-001]
Examiner       : [Your Name]
Date/Time      : [2025-08-21 14:30:00]
Evidence Item  : [USB Drive - 16GB SanDisk]

File Name      : Secret_Notes.txt
File Size      : 52 bytes
Date Created   : 2025-08-21 14:15:33
Date Modified  : 2025-08-21 14:16:02
Date Recovered : 2025-08-21 14:25:18

MD5            : [paste MD5 hash here]
SHA256         : [paste SHA256 hash here]
SHA512         : [paste SHA512 hash here]

Notes          : File recovered from G: partition of USB drive.
                 Shift+Delete was used by suspect. Recycle Bin
                 also emptied. File recovered using SS Data Recovery.

Examiner Sign  : ___________________
═══════════════════════════════════════════════════════
```

---

## 5. Forensic Disk Imaging

### 5.1 What Is a Forensic Image?

A forensic image is a **sector-by-sector, bit-for-bit copy** of an entire drive or partition, including:

- All active files
- Deleted files
- Unallocated space (where deleted data lives)
- File system structures
- Slack space

It is different from a regular file copy (which only copies visible files).

### 5.2 Image File Formats

| Format       | Extension  | Tool           | Notes                                                               |
| ------------ | ---------- | -------------- | ------------------------------------------------------------------- |
| EnCase Image | .E01       | EnCase, FTK    | Most common in legal/court settings; supports compression + hashing |
| Raw Image    | .dd / .img | dd, FTK Imager | Simple bit-copy; no built-in compression                            |
| AFF          | .aff       | AFFLIB         | Open source; metadata support                                       |
| SMART        | .s01       | SMART          | Linux-focused                                                       |

**Recommended format for court use: .E01 (EnCase format)**

### 5.3 Lab: Creating a Forensic Image with FTK Imager (Free)

#### Installation

```
Download: https://www.exterro.com/ftk-imager
Install on your forensics workstation
```

#### Creating a Drive Image

```
Step 1: Insert evidence drive (using a hardware write blocker if available)

Step 2: Open FTK Imager → File → Create Disk Image

Step 3: Select Image Source
   → Physical Drive (entire drive including all partitions)
   → Logical Drive (single partition like G: or F:)
   → For full forensics: choose Physical Drive

Step 4: Select the source drive
   → Find your evidence USB/drive in the list
   → Do NOT select your system drive

Step 5: Add Image Destination
   → Click "Add"
   → Choose image type: E01 (recommended)
   → Fill in evidence details:
      - Case Number
      - Evidence Number
      - Unique Description
      - Examiner Name
      - Notes

Step 6: Choose destination folder
   → Select a DIFFERENT drive (not source drive)
   → Name your image file (e.g., "Evidence_USB_2025-08-21")
   → Set image fragment size (leave default or 0 for single file)

Step 7: Start imaging
   → Click Finish → Start
   → Tool will create image AND calculate MD5/SHA1 automatically
   → This may take minutes to hours depending on drive size

Step 8: Verify completion
   → FTK Imager shows verification hashes on completion
   → Source hash vs. image hash must match exactly
   → Save the verification report
```

#### Creating an Image with R-Drive Image

```
Open R-Drive Image → Create Image

Select source:
   → Choose the correct partition or full drive
   → For a 16GB USB with 2 partitions:
      Option A: Select the full physical drive (both partitions together)
      Option B: Select only the partition you need (e.g., G: only)

Configure options:
   → Check "Verify image after creation"
   → Compression: Faster Speed or Smaller Size (your choice)
   → Backup actual data only vs. sector-by-sector (choose sector-by-sector for forensics)
   → Password: optional, adds encryption to image

Select destination:
   → Different drive from source
   → NEVER save image to the same drive you are imaging

Start and verify:
   → Monitor progress
   → Confirm "Created successfully"
   → Record the image file location and hash
```

### 5.4 Using dd for Imaging (Linux/Kali)

```bash
# Identify drives
lsblk
fdisk -l

# Image entire drive (physical)
# if = input file (source), of = output file (destination)
dd if=/dev/sdb of=/mnt/external/evidence.dd bs=4096 status=progress

# Image with hash verification using dc3dd
dc3dd if=/dev/sdb of=/mnt/external/evidence.dd hash=sha256 log=/mnt/external/hash.log

# Image with compression
dd if=/dev/sdb bs=4096 | gzip > /mnt/external/evidence.dd.gz

# Verify the image
md5sum /dev/sdb
md5sum /mnt/external/evidence.dd
# Both hashes must match
```

### 5.5 Mounting a Forensic Image (Read-Only)

```bash
# Mount E01 image (using ewf-tools)
apt install ewf-tools
ewfmount evidence.E01 /mnt/ewf/

# Mount the raw image inside (read-only!)
mount -o ro,loop /mnt/ewf/ewf1 /mnt/evidence/

# Now investigate /mnt/evidence/ safely
ls /mnt/evidence/
```

---

## 6. Case Investigation with Autopsy / FTK

### 6.1 Introduction to Autopsy (Free & Open Source)

Autopsy is a digital forensics platform used by law enforcement worldwide. It can:

- Analyze forensic images
- Recover deleted files
- Extract metadata from files
- Analyze browser history, emails, call logs
- Perform keyword searches across entire drives
- Generate professional reports

**Download:** https://www.autopsy.com/download/

### 6.2 Lab: Creating a New Case in Autopsy

```
Step 1: Open Autopsy → New Case

Step 2: Fill in case details
   Case Name        : Case_2025_001_USB_Investigation
   Base Directory   : D:\Forensics\Cases\    (external drive)
   Case Number      : 2025-001
   Examiner Name    : [Your Name]
   Examiner Phone   : [Optional]
   Examiner Email   : [Optional]
   Organization     : [University / Lab Name]

Step 3: Add Data Source
   → Click "Add Data Source"
   → Type: Disk Image or VM File
   → Browse to your .E01 or .dd image file
   → Select timezone

Step 4: Configure Ingest Modules
   These modules automatically run analysis:
   ✅ Recent Activity         → browser history, recent docs
   ✅ Hash Lookup             → check hashes against known databases
   ✅ File Type Identification → identify files by content, not extension
   ✅ Keyword Search          → search for specific words/phrases
   ✅ Email Parser            → extract emails
   ✅ EXIF Parser             → extract photo metadata (GPS, camera, date)
   ✅ Extension Mismatch      → find files with wrong extensions (hiding)

Step 5: Run analysis
   → Click Finish
   → Wait for ingestion to complete (may take time)
   → Progress shown in bottom right
```

### 6.3 Navigating the Autopsy Interface

```
Left Panel (Tree View):
├── Data Sources
│   └── [Your Image]
│       ├── $OrphanFiles     → Recovered deleted files
│       ├── $Unalloc         → Raw unallocated space
│       └── [Partitions]
│           ├── Documents
│           ├── Downloads
│           └── ...
│
├── Views
│   ├── File Types           → Images, Videos, Documents, etc.
│   ├── Deleted Files        → All deleted items
│   └── File Size            → Sort by size
│
└── Results
    ├── Extracted Content
    │   ├── Recent Documents
    │   ├── Web History
    │   ├── Installed Programs
    │   └── Email Messages
    └── Keyword Hits         → Results of your keyword search

Right Panel: File preview, hex view, metadata
Bottom Panel: Ingest progress, notifications
```

### 6.4 Key Investigation Tasks in Autopsy

```
Task 1: Find all deleted files
   → Tree View → Deleted Files
   → Right-click any file → Extract File → Save to evidence folder

Task 2: Search for keywords
   → Tools → Keyword Search
   → Type: "password", "secret", "account number", etc.
   → Results highlight the files containing those words

Task 3: View browser history
   → Results → Extracted Content → Web History
   → See: URLs visited, dates, times, browser used

Task 4: Analyze images (EXIF)
   → Results → Extracted Content → EXIF Data
   → Photos may contain: GPS coordinates, camera model, exact date/time

Task 5: Find files with wrong extension
   → Results → Extracted Content → Extension Mismatch Detected
   → E.g., a .jpg file that is actually an .exe (malware hiding as image)

Task 6: Generate report
   → Tools → Generate Report
   → Choose: HTML Report or Excel Report
   → Report includes all findings, timestamps, hashes
```

---

## 7. Metadata Analysis

### 7.1 What Is Metadata?

Metadata is **data about data** — hidden information embedded in files. It can reveal:

| File Type      | Metadata Examples                                       |
| -------------- | ------------------------------------------------------- |
| Photos (JPEG)  | GPS location, camera model, date/time taken, software   |
| Word Documents | Author name, edit history, revision count, company name |
| PDFs           | Creator software, creation date, author                 |
| Videos         | Device used, creation date, GPS (if enabled)            |
| Emails         | Sender IP, server path, timestamps                      |

### 7.2 Lab: Extracting Metadata

#### From Images — ExifTool (Free)

```bash
# Install
apt install exiftool       # Linux
# or download from: https://exiftool.org  (Windows)

# Basic metadata extraction
exiftool photo.jpg

# Extract GPS coordinates specifically
exiftool -GPSLatitude -GPSLongitude photo.jpg

# Extract metadata from all files in folder
exiftool /path/to/evidence/photos/

# Export to CSV for reporting
exiftool -csv /path/to/evidence/ > metadata_report.csv

# Remove metadata (for privacy, not forensics)
exiftool -all= photo.jpg
```

#### From Word Documents

```bash
# Extract metadata from .docx
exiftool document.docx

# Or open with Python
from docx import Document
doc = Document('document.docx')
props = doc.core_properties
print(f"Author:   {props.author}")
print(f"Created:  {props.created}")
print(f"Modified: {props.modified}")
print(f"Revision: {props.revision}")
```

### 7.3 GPS Coordinates in Photos

```
Example EXIF data from a crime scene photo:
   GPS Latitude  : 28° 36' 36.00" N
   GPS Longitude : 77° 13' 48.00" E

Convert to decimal:
   Latitude  = 28 + (36/60) + (36/3600) = 28.61°
   Longitude = 77 + (13/60) + (48/3600) = 77.23°

Paste into Google Maps → Exact location found!
```

---

## 8. Malware Identification via Hash Lookup

### 8.1 How Hash-Based Malware Detection Works

```
Known malware → calculated hash → stored in threat databases
Your suspicious file → calculate its hash → compare to database
Match found → file is identified as known malware
```

### 8.2 Lab: Using VirusTotal for Hash Lookup

**Website:** https://www.virustotal.com

#### Method 1: Upload the File

```
Step 1: Go to virustotal.com
Step 2: Click "Choose File" → upload suspicious file
Step 3: VirusTotal:
   - Calculates hash of the file
   - Checks against 70+ antivirus engines
   - Returns results: clean / suspicious / malicious

Result example:
   Detection: 48/72 engines flagged as malicious
   File name: suspicious.exe
   MD5: 44d88612fea8a8f36de82e1278abb02f
   Type: Trojan.GenericKD.XXXXX
```

#### Method 2: Search by Hash (Without Uploading File)

```
Step 1: Calculate hash of your file locally:
   sha256sum suspicious.exe  (Linux)
   Get-FileHash suspicious.exe -Algorithm SHA256  (Windows)

Step 2: Copy the hash value
Step 3: Go to virustotal.com
Step 4: Click "Search" tab
Step 5: Paste the hash value and search

This method is safer because:
   → You don't upload the actual file (keeps evidence intact)
   → Works for previously analyzed files
   → Useful for known malware families
```

#### Method 3: Using VirusTotal API (Programmatic)

```python
import requests

API_KEY = "your_virustotal_api_key"  # free API key from virustotal.com

def check_file_hash(file_hash):
    url = f"https://www.virustotal.com/api/v3/files/{file_hash}"
    headers = {"x-apikey": API_KEY}
    response = requests.get(url, headers=headers)
    data = response.json()

    if response.status_code == 200:
        stats = data['data']['attributes']['last_analysis_stats']
        print(f"Malicious: {stats['malicious']}")
        print(f"Suspicious: {stats['suspicious']}")
        print(f"Clean: {stats['undetected']}")
    else:
        print("Hash not found in database")

check_file_hash("44d88612fea8a8f36de82e1278abb02f")
```

### 8.3 Useful Threat Intelligence Databases

| Platform        | URL                 | Use                         |
| --------------- | ------------------- | --------------------------- |
| VirusTotal      | virustotal.com      | Hash + file scanning        |
| MalwareBazaar   | bazaar.abuse.ch     | Known malware samples       |
| Hybrid Analysis | hybrid-analysis.com | Sandbox + hash lookup       |
| ANY.RUN         | any.run             | Interactive sandbox         |
| MISP            | misp-project.org    | Threat intelligence sharing |

---

## 9. Chain of Custody & Evidence Handling

### 9.1 What Is Chain of Custody?

Chain of custody is the **documented, unbroken record** of who handled evidence, when, where, and why. Without it, evidence can be challenged and dismissed in court.

### 9.2 Chain of Custody Form Template

```
══════════════════════════════════════════════════════════════
               CHAIN OF CUSTODY FORM
══════════════════════════════════════════════════════════════
Case Number    : ___________________________
Case Name      : ___________________________

EVIDENCE DESCRIPTION:
   Item #       : ___________________________
   Description  : ___________________________
   Serial #     : ___________________________
   Make/Model   : ___________________________

INITIAL COLLECTION:
   Date/Time    : ___________________________
   Location     : ___________________________
   Collected By : ___________________________
   Signature    : ___________________________

HASH VALUES (calculated immediately upon collection):
   MD5          : ___________________________
   SHA256       : ___________________________

TRANSFERS:
┌────────────────┬──────────────┬──────────────┬────────────┐
│ Released By    │ Received By  │ Date/Time    │ Purpose    │
├────────────────┼──────────────┼──────────────┼────────────┤
│                │              │              │            │
│                │              │              │            │
│                │              │              │            │
└────────────────┴──────────────┴──────────────┴────────────┘

STORAGE:
   Location     : ___________________________
   Container    : ___________________________
   Seal #       : ___________________________

══════════════════════════════════════════════════════════════
```

### 9.3 Best Practices

```
✅ Photograph evidence in situ before touching anything
✅ Use anti-static bags for electronic devices
✅ Use hardware write blockers before connecting drives
✅ Calculate hashes immediately after imaging
✅ Store evidence in a locked, access-controlled location
✅ Document every action taken on the evidence
✅ Use tamper-evident seals on evidence containers
✅ Never work on original evidence — always use forensic image
✅ Keep backups of forensic images (at least 2 copies)
✅ Date and sign every step
```

---

## 10. Forensics Career Roadmap

### 10.1 Specializations in Digital Forensics

```
Computer Forensics      → File systems, OS artifacts, deleted data
Mobile Forensics        → Smartphones, call logs, app data, GPS
Network Forensics       → Packet analysis, logs, intrusion traces
Memory Forensics        → RAM analysis (running processes, keys in memory)
Malware Forensics       → Reverse engineering malicious software
Cloud Forensics         → AWS/Azure/GCP logs and artifacts
IoT Forensics           → Smart devices, cameras, industrial systems
```

### 10.2 Recommended Learning Path

```
Month 1-2: Foundations
├── Operating systems internals (Windows & Linux)
├── File systems: NTFS, FAT32, ext4
├── Networking basics: TCP/IP, DNS, HTTP
└── Theory: evidence handling, legal requirements

Month 3-4: Core Tools
├── Autopsy — case investigation
├── FTK Imager — imaging and preview
├── Wireshark — network analysis
├── Volatility — memory forensics
└── ExifTool — metadata extraction

Month 5-6: Hands-On Practice
├── Build your own lab (VMs with evidence scenarios)
├── Practice on CyberDefenders.org challenges
├── BlueTeamLabs.online
└── DFIR.training exercises

Month 7-12: Advanced Topics
├── Memory forensics with Volatility
├── Mobile forensics (Cellebrite concepts)
├── Malware analysis basics
├── Log analysis and SIEM tools
└── Report writing for court

Year 2+: Professional
├── Certifications (see below)
├── Internship / junior role
├── Specialize in one area
└── Contribute to open source forensics tools
```

### 10.3 Key Certifications

| Certification                                 | Provider   | Level        |
| --------------------------------------------- | ---------- | ------------ |
| CompTIA Security+                             | CompTIA    | Entry        |
| CCE (Certified Computer Examiner)             | ISFCE      | Intermediate |
| GCFE (GIAC Certified Forensic Examiner)       | GIAC       | Intermediate |
| GCFA (GIAC Certified Forensic Analyst)        | GIAC       | Advanced     |
| EnCE (EnCase Certified Examiner)              | OpenText   | Advanced     |
| CHFI (Computer Hacking Forensic Investigator) | EC-Council | Intermediate |

### 10.4 Free Practice Resources

```
CyberDefenders        → cyberdefenders.org     (blue team/forensics challenges)
Blue Team Labs        → blueteamlabs.online     (SOC/forensics scenarios)
DFIR.training         → dfir.training           (curated forensics learning)
13Cubed (YouTube)     → youtube.com/c/13Cubed   (forensics tutorials)
SANS Reading Room     → sans.org/reading-room   (research papers)
VulnHub               → vulnhub.com             (practice VMs)
```

---

## 📚 Quick Reference — Commands Cheat Sheet

```bash
# ── Hash Calculation ──────────────────────────────────────
md5sum file.txt
sha256sum file.txt
sha512sum file.txt
Get-FileHash file.txt -Algorithm SHA256      # Windows PowerShell

# ── Disk Imaging with dd ─────────────────────────────────
lsblk                                        # list drives
dd if=/dev/sdb of=/mnt/external/image.dd bs=4096 status=progress
dc3dd if=/dev/sdb of=/mnt/external/image.dd hash=sha256 log=hash.log

# ── Mount Forensic Images ─────────────────────────────────
ewfmount image.E01 /mnt/ewf/
mount -o ro,loop /mnt/ewf/ewf1 /mnt/evidence/

# ── Metadata Extraction ───────────────────────────────────
exiftool photo.jpg
exiftool -csv /evidence/photos/ > metadata.csv

# ── File Recovery (Linux) ─────────────────────────────────
photorec /dev/sdb                            # recover by file type
testdisk /dev/sdb                            # partition recovery

# ── Strings in Binary Files ──────────────────────────────
strings suspicious.exe | grep -i "http\|password\|admin"

# ── File Type Identification ─────────────────────────────
file unknown_file                            # identify by magic bytes
```

---

_This guide is intended for university cybersecurity and digital forensics coursework. Always follow legal and ethical guidelines, obtain proper authorization before examining any device, and maintain strict chain-of-custody procedures._
