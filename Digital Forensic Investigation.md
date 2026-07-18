# Digital Forensic Investigation — Complete Practical Study Notes

### Cyber Crime Investigation Course — University Cybersecurity Department

> **Lab Scope:** All forensic exercises are performed on:
>
> - Intentionally created evidence drives (your own USB/hard drives with test data)
> - Virtual machine disk images you generate yourself
> - Metasploitable2 and your own isolated lab systems
>
> Never perform forensic investigation on systems, drives, or data you do not own or lack explicit written authorization to examine.

---

## 📋 Table of Contents

1. [The 13 Steps of Cyber Crime Primary Investigation](#1-the-13-steps-of-cyber-crime-primary-investigation)
2. [Lab Scenario](#2-lab-scenario)
3. [Lab 1 — Data Recovery (Deleted File Recovery)](#3-lab-1--data-recovery-deleted-file-recovery)
   - [Tools Overview](#31-tools-overview)
   - [Practical with S Data Recovery](#32-practical-with-s-data-recovery)
4. [Lab 2 — Hash Calculation & Integrity Verification](#4-lab-2--hash-calculation--integrity-verification)
   - [What Is a Hash Value?](#41-what-is-a-hash-value)
   - [Hash Calculation with HashCalc](#42-hash-calculation-with-hashcalc)
   - [HMAC — Keyed Hash Authentication](#43-hmac--keyed-hash-authentication)
   - [Malware Detection via Hash — VirusTotal](#44-malware-detection-via-hash--virustotal)
5. [Lab 3 — Hash Comparison](#5-lab-3--hash-comparison)
   - [Manual Comparison with HashCalc](#51-manual-comparison-with-hashcalc)
   - [Effect of Modifications on Hash](#52-effect-of-modifications-on-hash)
   - [MD5 Calculator Comparison Tool](#53-md5-calculator-comparison-tool)
6. [Lab 4 — File Identification & Extension Analysis](#6-lab-4--file-identification--extension-analysis)
   - [File Extension Mismatch](#61-file-extension-mismatch)
   - [FileViewer / FileAlyzer Tools](#62-fileviewer--filealyzer-tools)
7. [Lab 5 — Forensic Imaging (Copy vs. Clone vs. Forensic Image)](#7-lab-5--forensic-imaging-copy-vs-clone-vs-forensic-image)
   - [Three Types of Duplication Explained](#71-three-types-of-duplication-explained)
   - [Physical Drive vs. Logical Drive](#72-physical-drive-vs-logical-drive)
   - [Creating a Forensic Image with R-Drive Image](#73-creating-a-forensic-image-with-r-drive-image)
8. [Lab 6 — Evidence Analysis with Autopsy](#8-lab-6--evidence-analysis-with-autopsy)
   - [Creating a New Case](#81-creating-a-new-case)
   - [Adding Evidence Source](#82-adding-evidence-source)
   - [Analyzing Results](#83-analyzing-results)
   - [Timeline Analysis](#84-timeline-analysis)
   - [Extension Mismatch Detection](#85-extension-mismatch-detection)
9. [Lab 7 — File System Analysis with The Sleuth Kit (Command Line)](#9-lab-7--file-system-analysis-with-the-sleuth-kit-command-line)
   - [Key Sleuth Kit Commands](#91-key-sleuth-kit-commands)
   - [Master File Table (MFT) Entries](#92-master-file-table-mft-entries)
   - [File & Directory Listing](#93-file--directory-listing)
10. [Lab 8 — Linux File System Analysis in Autopsy](#10-lab-8--linux-file-system-analysis-in-autopsy)
    - [Important Linux Directory Structure](#101-important-linux-directory-structure)
11. [Lab 9 — Forensic Report (65B Report)](#11-lab-9--forensic-report-65b-report)
12. [Forensic Evidence Bags & Chain of Custody](#12-forensic-evidence-bags--chain-of-custody)
13. [Comprehensive Lab Exercises](#13-comprehensive-lab-exercises)
    - [Lab A: Create Evidence Drive & Simulate Crime](#lab-a-create-evidence-drive--simulate-crime)
    - [Lab B: Full Forensic Image Creation](#lab-b-full-forensic-image-creation)
    - [Lab C: Hash Generation & Integrity Testing](#lab-c-hash-generation--integrity-testing)
    - [Lab D: Data Recovery on Your Own Drive](#lab-d-data-recovery-on-your-own-drive)
    - [Lab E: Autopsy Full Analysis Walkthrough](#lab-e-autopsy-full-analysis-walkthrough)
    - [Lab F: Sleuth Kit Command-Line Investigation](#lab-f-sleuth-kit-command-line-investigation)
    - [Lab G: VirusTotal Hash Investigation](#lab-g-virustotal-hash-investigation)
14. [Glossary](#14-glossary)
15. [Tools Reference & Certifications](#15-tools-reference--certifications)

---

## 1. The 13 Steps of Cyber Crime Primary Investigation

These steps are the standard operating procedure for any cyber crime investigation, whether private or through law enforcement.

```
13 STEPS OF CYBER CRIME INVESTIGATION
══════════════════════════════════════════════════════════════════════

  Step 1  — IDENTIFICATION
            Determine that a cyber crime has occurred.
            Types: Civil, Financial, Administrative.
            Obtain authorization (court order / higher authority).

  Step 2  — OBTAIN PERMISSION
            Police cases: court order already obtained.
            Private cases: client's lawyer brings court order,
            or investigator obtains it independently.
            Either way — written legal authorization is MANDATORY.

  Step 3  — USE FIRST RESPONDER TOOLS
            First Responder Toolkit = complete set of tools
            used at the crime scene BEFORE touching the device.

            ┌─────────────────────────────────────────────────┐
            │  IF COMPUTER IS ON (POWERED):                   │
            │  → Capture volatile data FIRST (lost on shutdown)│
            │    - RAM dump (memory contents)                 │
            │    - Running processes                          │
            │    - Network connections (who was connected)    │
            │    - Cookies, session tokens                    │
            │    - Clipboard contents                         │
            │  → THEN power down                              │
            ├─────────────────────────────────────────────────┤
            │  IF COMPUTER IS OFF (POWERED DOWN):             │
            │  → Volatile data already gone                   │
            │  → Remove hard disk for lab investigation        │
            │  → Laptop: bring entire unit to lab, remove     │
            │    hard disk there                              │
            └─────────────────────────────────────────────────┘

            For phones: seize entire phone (do not disassemble)

  Step 4  — SEIZE THE EVIDENCE
            Computer:
              → Remove hard disk
              → Place in ANTI-STATIC BAG (protects from
                static charge, which can damage magnetic media)
              → Wear GLOVES at all times
              → Do not touch chips/boards bare-handed

            Phone / Tablet:
              → Place in FARADAY BAG (blocks all signals)
              → Prevents remote wiping, remote hacking,
                location tracking, or any remote command

            Removable media (USB, SD cards):
              → Anti-static bag

            Non-removable components (soldered to board):
              → Faraday bag for the entire device

  Step 5  — SEAL THE EVIDENCE
            After placing in bags, seal officially.
            Label with case number, date, time, officer details.
            Tamper-evident seals must be used.

  Step 6  — CREATE TWO FORENSIC COPIES (Bit-by-Bit)
            → Forensic Image ≠ normal copy/paste
            → Forensic image copies EVERY bit including empty space
            → Creates exact duplicate of original evidence
            → Make TWO copies:
                Copy 1 → Kept by investigator for analysis
                Copy 2 → Given to the person/company whose
                         device was seized (legal requirement)
            → Original evidence → stored in secure location

  Step 7  — GENERATE HASH VALUES
            → Calculate hash of original, copy 1, copy 2
            → All three hashes MUST match (proves no tampering)
            → Algorithms: SHA-256 (standard), MD5 (legacy/reference)
            → Any tampering changes hash completely
            → Document all hash values in the case file

  Step 8  — MAINTAIN CHAIN OF CUSTODY
            → Complete paper trail of who touched what, when
            → Every transfer of evidence is documented:
                - Date and time
                - Name and ID of officer handing over
                - Name and ID of officer receiving
                - Location of transfer
                - Condition of evidence
            → This document is critical for court admissibility

  Step 9  — TRANSPORT EVIDENCE TO LAB
            → Follow all previous steps before moving evidence
            → Use proper transport containers
            → Maintain chain of custody during transport
            → Log arrival at lab

  Step 10 — STORE ORIGINAL EVIDENCE IN SECURE LOCATION
            → Access-controlled secure room
            → CCTV coverage
            → Authorized personnel only (Evidence Manager)
            → Proper temperature/humidity control
            → No one accesses original without documentation

  Step 11 — ANALYZE FORENSIC COPY
            → NEVER analyze the original evidence
            → Investigate the forensic copy (Copy 1)
            → Original is preserved for court presentation
            → If analysis copy is damaged → regenerate from original

  Step 12 — GENERATE FORENSIC REPORT (65B Report)
            → Under Electronic Evidence Act (Section 65B)
            → Contents:
                - Full case details
                - Chain of custody record
                - Hash values of all evidence
                - Tools used for investigation (with versions)
                - MAC address of investigation workstation
                - Step-by-step methodology
                - Findings and conclusions
            → Only authorized/certified forensic examiners
            → Unapproved tools → report rejected by court

  Step 13 — COURT TESTIMONY (if required)
            → If judges require demonstration
            → Perform live in court exactly what was done in lab
            → Use original evidence (from secure storage)
            → All steps re-demonstrated in front of judge
            → Validates the forensic report
            → If all legal procedures followed → report validated

══════════════════════════════════════════════════════════════════════
```

**Critical legal principle:** The investigator is personally liable for any mistakes. Errors in process can invalidate the entire investigation and expose the investigator to legal liability.

---

## 2. Lab Scenario

**Scenario:** An employee of a company is suspected of:

1. Transferring company funds to their personal account (financial fraud)
2. Deleting important company data using `Shift + Delete` (permanent deletion, bypasses Recycle Bin)
3. Manipulating data records

**Your role:** Forensic investigator hired by the company.

**Legal basis:** Company (victim) provides authorization. In real cases, a court order is also required.

**What `Shift + Delete` does:**

```
Normal Delete:     File → Recycle Bin → recoverable easily
Shift + Delete:    File → GONE from file system index
                   BUT: data remains on disk in "unallocated space"
                        until overwritten by new data
                        → RECOVERABLE with forensic tools
```

---

## 3. Lab 1 — Data Recovery (Deleted File Recovery)

### 3.1 Tools Overview

| Tool                          | Type                           | Notes                       |
| ----------------------------- | ------------------------------ | --------------------------- |
| **R-Studio**                  | Paid (trial available)         | Professional-grade recovery |
| **Stellar Data Recovery**     | Paid (trial available)         | Widely used in forensics    |
| **MiniTool Partition Wizard** | Paid/Free                      | Partition + recovery        |
| **Recuva**                    | Free (basic) / Paid (advanced) | Good beginner tool          |
| **Tenorshare 4DDiG**          | Paid                           | Phone and PC recovery       |
| **EaseUS Data Recovery**      | Paid (trial available)         | Used in this course         |

> **Key insight:** Paid tools recover more files and go deeper. Free tools are good for learning but miss many files. In real forensic work, approved commercial tools are mandatory.

### 3.2 Practical with EaseUS Data Recovery

**Setup:** Attach the evidence drive (external USB or removed hard disk via USB adapter) to your investigation workstation.

**Connection steps:**

```
1. Attach the evidence hard disk to a USB-to-SATA/IDE adapter
2. Connect adapter to your investigation workstation (Kali VM or Windows)
3. If using a VM: VM → Removable Devices → attach the USB device to VM
4. Verify connection: device appears in file manager / disk management
```

**Recovery procedure:**

**Step 1: Open S Data Recovery → Select the evidence drive**

```
Interface shows all detected drives:
  - OS drives (C, D, etc. — your workstation)
  - Evidence drive (will show as USB or by drive letter)
  → SELECT the evidence drive
```

**Step 2: Scan modes explained**

```
QUICK SCAN
  └── Fast scan → finds recently deleted files
  └── Misses many files (especially old deletions)
  └── Good for: recently deleted files

ADVANCED SCAN (Deep Scan)
  └── Scans every sector of the drive
  └── Finds files deleted long ago (years!)
  └── Takes longer but finds dramatically more
  └── Example from course: Quick Scan = 1 file | Advanced Scan = 9,916 files
  └── ALWAYS use Advanced Scan in forensic work
```

**Step 3: Understanding scan results**

```
SCAN RESULTS PANEL
══════════════════════════════════════════════════════════════

  Volume (your evidence drive)
  ├── Existing Files (still present, not deleted)
  ├── Deleted Files ← PRIMARY FORENSIC INTEREST
  │   ├── Documents (.docx, .xlsx, .pdf)
  │   ├── Images (.jpg, .png)
  │   ├── Videos (.mp4, .avi)
  │   ├── Audio (.mp3)
  │   └── Archives (.zip, .rar)
  └── Lost Files (deeper recovery — more damaged)

  File listing shows:
  - File name
  - File size
  - File format/type
  - Original location (path)
  - Date deleted (when available)
  - Preview (images, documents — shows content without opening)

══════════════════════════════════════════════════════════════
```

**Step 4: Recovery process**

```
1. Browse deleted files → identify relevant evidence
   - Ask client: "What files were deleted? What were their names?"
   - If unknown: open each file type and check relevance
   - Use preview window to check content without extracting

2. Tick-mark files to recover:
   - Individual files: tick individual items
   - Entire folder: tick the folder checkbox
   - All deleted: select all

3. Click "Recover"

4. CRITICAL — Choose destination:
   ╔══════════════════════════════════════════════════════════╗
   ║  NEVER save recovered files BACK to the source drive!   ║
   ║  Always save to a DIFFERENT drive.                      ║
   ║  Saving to same drive = overwrites other recoverable    ║
   ║  data = evidence destruction!                           ║
   ╚══════════════════════════════════════════════════════════╝
   → Choose D: drive or any other drive EXCEPT the evidence drive

5. Recovered files → saved in folder named by recovery tool
   Example: "Lost Files" folder on D: drive
```

**Recovery date relevance in forensics:**

- Files show metadata including deletion date and creation date
- Photos show EXIF data → original capture date/time/location
- A photo from 2017 on a 2024 drive is still fully recoverable
- Recovery quality depends on: drive quality, tool quality, how much new data was written after deletion

---

## 4. Lab 2 — Hash Calculation & Integrity Verification

### 4.1 What Is a Hash Value?

```
HASH VALUE — CONCEPT
══════════════════════════════════════════════════════════════

  A hash is a FIXED-LENGTH DIGITAL FINGERPRINT of any data.
  Think of it as the Aadhaar (National ID) number of a file.

  Properties:
  ✓ Same input ALWAYS produces same hash (deterministic)
  ✓ ANY change to input → COMPLETELY different hash
  ✓ Even adding a single SPACE changes the hash entirely
  ✓ Cannot reverse a hash back to original data (one-way)
  ✓ Two different files CANNOT have same hash (collision-resistant)

  Uses in Digital Forensics:
  1. INTEGRITY VERIFICATION
     Evidence hash before investigation = hash after investigation
     → No tampering occurred
     Hashes differ → evidence has been modified

  2. MALWARE DETECTION
     Known malware files have known hash values (databases)
     Match hash → identify malicious file without opening it

  3. EVIDENCE AUTHENTICATION
     Original + Copy 1 + Copy 2 → all three hashes must match

══════════════════════════════════════════════════════════════
```

**Hash algorithms used in forensics:**

| Algorithm   | Output Size             | Status                                                        |
| ----------- | ----------------------- | ------------------------------------------------------------- |
| MD5         | 128-bit (32 hex chars)  | Deprecated (collisions possible) but still used for reference |
| SHA-1       | 160-bit (40 hex chars)  | Deprecated for security                                       |
| **SHA-256** | 256-bit (64 hex chars)  | **Current standard**                                          |
| SHA-512     | 512-bit (128 hex chars) | High security                                                 |
| SHA-3       | Variable                | Newest standard                                               |

**Example SHA-256 hash:**

```
File: evidence.pdf
SHA-256: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

### 4.2 Hash Calculation with HashCalc

**Tool:** HashCalc (free, Windows)

**Interface overview:**

```
HashCalc Interface
══════════════════════════════════════════════
  Data Format:    ○ File  ○ Text  ○ Hex
  [Browse button] → select your file

  HMAC:  ☐ (check to enable key-based hashing)
  Key:   [enter key here if HMAC enabled]

  Algorithms:
  ☑ MD5         [result shows here]
  ☑ SHA1        [result shows here]
  ☑ SHA256      [result shows here]
  ☑ SHA512      [result shows here]
  ☑ CRC32       [result shows here]
  ... (many more)

  [Calculate]
══════════════════════════════════════════════
```

**Steps:**

```
1. Open HashCalc
2. Data Format → select "File"
3. Browse → select the file (recovered PDF, document, etc.)
   (Full path shows in the field automatically)
4. Select algorithms: at minimum SHA-256, MD5
5. Click "Calculate"
6. ALL hash values appear simultaneously
7. SCREENSHOT or COPY the results → paste into your case notes
```

**Sample output format for your case notes:**

```
HASH VALUES — Evidence File: admin_presentation.pptx
Date/Time: 2025-01-10 14:32:00
Investigator: [Your Name]
Workstation MAC: AA:BB:CC:DD:EE:FF

MD5:    ba8d49e8a54e6f2c29f7abd1a6c20b7e
SHA-1:  5f4dcc3b5aa765d61d8327deb882cf99b7e2cd82
SHA-256: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
SHA-512: [value]
```

### 4.3 HMAC — Keyed Hash Authentication

**HMAC = Hash-Based Message Authentication Code**

- Normal hash: identifies the file's content
- HMAC: hash + secret key = even if someone knows the file content, they cannot regenerate the same hash without the key
- Provides both integrity AND authentication

```
NORMAL HASH vs. HMAC
══════════════════════════════════════════════════════════════

  NORMAL HASH:
  File → SHA-256 → 2cf24dba5fb0a30e26...
  Anyone can verify this.

  HMAC:
  File + Secret Key → HMAC-SHA-256 → a9b7c4d2e1f3...
  Only someone with the secret key can verify this.
  More secure for court evidence in sensitive cases.

══════════════════════════════════════════════════════════════
```

**In HashCalc:**

```
1. Check ☑ HMAC
2. Enter your secret key in the Key field
   (e.g., "CaseID_2025_SecretKey")
3. Calculate
4. Hash values are NOW different from the non-HMAC version
5. Document both: normal hash AND HMAC hash
```

> **Note:** Applying HMAC changes ALL hash values. This is proof that the key-based authentication is working. Side-by-side comparison will show every value is different.

### 4.4 Malware Detection via Hash — VirusTotal

**VirusTotal** (virustotal.com) is a free online service that:

- Checks files/hashes against databases from 70+ antivirus companies worldwide
- Detects malware by hash comparison
- Analyzes entropy (compression level of a file) to detect suspicious packing
- Allows checking URLs, IP addresses, domains, and file hashes

**Methods:**

**Method 1: Upload file**

```
1. Go to virustotal.com
2. Click "Choose file" → select suspicious file
3. Wait for scan
4. Results show:
   - Detection ratio: X/72 engines detected threat
   - File hashes (MD5, SHA-1, SHA-256) shown in Details tab
   - "Clean" result = 0 detections
   - Any detection = malicious
```

**Method 2: Submit hash (no upload needed)**

```
1. Calculate SHA-256 of file using HashCalc
2. Copy the SHA-256 hash
3. Go to virustotal.com → Search tab
4. Paste the hash → press Enter
5. If in VirusTotal database: shows full scan results
6. If not: "Not found" → upload file for first analysis
```

**Method 3: Submit URL/IP**

```
1. VirusTotal → URL tab
2. Paste suspicious URL or IP
3. Results: phishing status, malware hosting, reputation
```

**Understanding VirusTotal results:**

```
VIRUSTOTAL RESULTS EXPLAINED
══════════════════════════════════════════════════════════════

  Detection: 0/72    → Clean file (safe)
  Detection: 5/72    → Possibly malicious (false positive possible)
  Detection: 40/72   → Almost certainly malicious
  Detection: 72/72   → Confirmed malware

  Entropy Analysis:
  Normal entropy:  → file is not unusually compressed
  High entropy:    → file is highly compressed or encrypted
                     → common in malware (packed/obfuscated)

  Details Tab shows:
  - MD5, SHA-1, SHA-256 of the file
  - File type
  - File size
  - First submission date
  - Number of times submitted

══════════════════════════════════════════════════════════════
```

---

## 5. Lab 3 — Hash Comparison

### 5.1 Manual Comparison with HashCalc

**Goal:** Prove that a file has or has not been tampered with.

**Procedure:**

```
Step 1: Calculate hash of file in ORIGINAL STATE
        → Copy hash value → save to a text file
        (This is your "before" reference hash)

Step 2: If file is tampered with (or you suspect tampering):
        → Calculate hash again

Step 3: Compare both values
        Match → file unchanged
        Different → file has been modified
```

### 5.2 Effect of Modifications on Hash

The following experiments from the course demonstrate hash sensitivity:

```python
# These are conceptual demonstrations — run in your lab:

FILE CONTENT: "DDD"
SHA-256: [value A]

MODIFICATION 1: Add a period → "DDD."
SHA-256: [COMPLETELY DIFFERENT from value A]
Conclusion: Even 1 character = full hash change

MODIFICATION 2: Remove period → back to "DDD"
SHA-256: [MATCHES value A again]
Conclusion: Hash is deterministic — same content = same hash

MODIFICATION 3: Add a space → "DDD "
SHA-256: [COMPLETELY DIFFERENT from value A]
Conclusion: Even a SPACE changes the hash

MODIFICATION 4: Remove space → back to "DDD"
SHA-256: [MATCHES value A]

MODIFICATION 5: Rename file extension DDD.txt → DDD.pdf (by hand)
SHA-256: [SAME AS value A]
Conclusion: Renaming extension does NOT change hash
            (file CONTENT unchanged = hash unchanged)

MODIFICATION 6: Convert file using an online tool TXT→PDF
SHA-256: [DIFFERENT from value A]
Conclusion: Tool conversion changes metadata
            (upload, processing, re-download = metadata changes)

KEY TAKEAWAY:
  What changes hash: content, metadata
  What does NOT change hash: file extension rename (manual)
  What DOES change hash in practice: tool-based conversion
```

### 5.3 MD5 Calculator Comparison Tool

A dedicated comparison tool for side-by-side hash matching:

```
MD5 Calculator Tool
══════════════════════════════════════════════════════════════

  1. Open MD5 Calculator
  2. Load your evidence file → hash auto-calculated and displayed
  3. Use "Compare To" field → paste the ORIGINAL hash value
  4. Result:
       == → Hashes match (file unchanged)
       ≠  → Hashes differ (file modified/tampered)

  Workflow for forensic comparison:
  a) Calculate hash of recovered evidence → copy
  b) Calculate hash of working copy → copy
  c) Compare both → should be identical
  d) If different → investigation protocol violation occurred

══════════════════════════════════════════════════════════════
```

---

## 6. Lab 4 — File Identification & Extension Analysis

### 6.1 File Extension Mismatch

**What is extension mismatch?**

- A file's extension (`.pdf`, `.jpg`, `.txt`) is the label
- The ACTUAL content (file signature / magic bytes) is what the file really is
- Criminals change extensions to hide file type
- `malware.exe` renamed to `report.pdf` → extension mismatch

**Why forensic investigators must detect this:**

```
Attacker hides data → renames file_x.zip as image.jpg
Investigator blindly looks at .jpg files → misses the hidden zip
File mismatch detector → "image.jpg is actually a ZIP file"
```

**How files are identified (magic bytes / file signatures):**

```
Every file type has a unique signature at the start of the file:

File Type    Extension    Magic Bytes (Hex)    ASCII
──────────   ─────────    ─────────────────    ──────────
JPEG image   .jpg         FF D8 FF              ÿØÿ
PNG image    .png         89 50 4E 47           .PNG
PDF          .pdf         25 50 44 46           %PDF
ZIP archive  .zip         50 4B 03 04           PK..
EXE/DLL      .exe         4D 5A                 MZ
Office (new) .docx        50 4B 03 04           PK.. (it's actually a ZIP)
MP4 video    .mp4         00 00 00 1C 66 74 79  ....fty

Tools read these bytes → identify REAL type regardless of extension
```

### 6.2 FileViewer / FileAlyzer Tools

**FileViewer Express (free) and FileViewer Pro (paid):**

```
Open tool → Browse drive → Select file
→ Tool shows:
  - Actual file type (from magic bytes)
  - File properties
  - If extension matches actual type: NORMAL
  - If extension doesn't match: MISMATCH FLAGGED

FileAlyzer (free tool):
  Select file → tool shows internal structure
  If opening a "PDF" reveals binary data inconsistent with PDF:
  → File is NOT actually a PDF → extension was changed
  → Real format identifiable from magic bytes
```

**Detecting mismatch via Autopsy (covered in Lab 6):**

```
Autopsy → Results → Extension Mismatch Detected
→ Lists ALL files where stated extension ≠ actual file type
Example: "report.txt" → actual type: JPEG image
```

**Manual method — checking file behavior:**

```
Attempt to open "filename.mp4" in a video player
→ If error: "Cannot render this stream" or "No suitable codec"
→ File is NOT actually a video
→ Extension has been changed
→ Try opening with hex editor to read magic bytes
```

---

## 7. Lab 5 — Forensic Imaging (Copy vs. Clone vs. Forensic Image)

### 7.1 Three Types of Duplication Explained

```
THREE TYPES OF DATA DUPLICATION
══════════════════════════════════════════════════════════════

  EXAMPLE SCENARIO:
  Evidence pen drive: 8 GB total
  Data present:       1 GB
  Empty space:        7 GB

  ┌────────────────────────────────────────────────────────┐
  │  TYPE 1: NORMAL COPY (Copy-Paste)                      │
  │                                                        │
  │  Original: [1GB Data] [7GB Empty]                      │
  │                  ↓ copy                                │
  │  Copy:     [1GB Data]                                  │
  │                                                        │
  │  Empty space NOT copied. Only data files.              │
  │  NOT suitable for forensics.                           │
  └────────────────────────────────────────────────────────┘

  ┌────────────────────────────────────────────────────────┐
  │  TYPE 2: CLONE                                         │
  │                                                        │
  │  Original: [1GB Data] [7GB Empty]                      │
  │                  ↓ clone                               │
  │  Clone:    [1GB Data]                                  │
  │                                                        │
  │  Only DATA is cloned. Not empty space.                 │
  │  Clone size = data size (1 GB in this example).        │
  │  NOT suitable for forensics.                           │
  └────────────────────────────────────────────────────────┘

  ┌────────────────────────────────────────────────────────┐
  │  TYPE 3: FORENSIC IMAGE (Bit-by-Bit)                   │
  │                                                        │
  │  Original: [1GB Data] [7GB Empty]                      │
  │                  ↓ forensic image                      │
  │  Image:    [1GB Data] [7GB Empty Space]                │
  │                                                        │
  │  EVERY BIT is copied — data AND empty space.           │
  │  Image size = FULL DRIVE SIZE (8 GB).                  │
  │                                                        │
  │  WHY empty space matters:                              │
  │  Deleted files remain in "empty" space until           │
  │  overwritten. Forensic image captures those remnants.  │
  │                                                        │
  │  Cost: More expensive software, more storage needed.   │
  │  Legal: Required for court-admissible evidence.        │
  └────────────────────────────────────────────────────────┘

══════════════════════════════════════════════════════════════
```

### 7.2 Physical Drive vs. Logical Drive

```
DRIVE TERMINOLOGY
══════════════════════════════════════════════════════════════

  PHYSICAL DRIVE:
  → The actual hardware (the hard disk itself)
  → One physical disk = one physical drive
  → Identified as: Disk 0, Disk 1, /dev/sda, /dev/sdb

  LOGICAL DRIVE (Partition):
  → A virtual section created by dividing a physical drive
  → One physical disk can have many logical drives
  → Example: 1TB physical disk → partitioned into:
      C: (100GB) → system/OS
      D: (400GB) → data
      E: (500GB) → media
  → Each C/D/E is a "logical drive"
  → Logical = created by software, can be resized/deleted

  WHY IT MATTERS IN FORENSICS:
  → Forensic image of physical drive → captures ALL partitions
  → Forensic image of logical drive → captures ONE partition only
  → Always image the PHYSICAL drive when possible

══════════════════════════════════════════════════════════════
```

### 7.3 Creating a Forensic Image with R-Drive Image

**Setup:**

```
Attach evidence drive to workstation (USB adapter)
Install R-Drive Image (paid; trial for practice)
Run as Administrator
```

**Steps:**

```
1. Open R-Drive Image
2. Select: "Create an Image"
3. Select SOURCE:
   → Physical Drive (Disk 1) → images entire physical disk
   → Partition (D:) → images only that partition
   → Select the EVIDENCE drive (NOT your investigation drive)

4. Select DESTINATION:
   → Choose a DIFFERENT drive with enough free space
   → Rule: source drive ≠ destination drive
   → Image will be larger than actual data (includes empty space)

5. Configure options:
   → Check image upon creation: ☑ YES (verify integrity immediately)
   → Sector-by-sector copy: ☑ YES (bit-by-bit = forensic quality)
   → Compression: optional (reduces size but takes longer)
   → Password protect: optional (add for sensitive cases)
   → Estimated image size shown before starting

6. Start imaging
   → Shows: partition progress, total progress
   → Do NOT interrupt (corrupts image)

7. Completion:
   → Image file saved with .rdr extension
   → Verify hash of the image immediately after creation
   → Document: image file name, location, creation date/time
```

**Image file formats in forensics:**

| Format   | Extension             | Notes                                           |
| -------- | --------------------- | ----------------------------------------------- |
| RAW / DD | `.dd`, `.raw`, `.img` | Simple bit-by-bit dump; no metadata             |
| EnCase   | `.E01`, `.E02`        | Industry standard; includes case metadata, hash |
| FTK      | `.AD1`                | AccessData format                               |
| AFF      | `.aff`                | Open forensic format                            |
| VMDK     | `.vmdk`               | VMware virtual disk (for VM forensics)          |

---

## 8. Lab 6 — Evidence Analysis with Autopsy

**Autopsy** is a free, open-source digital forensics GUI built on The Sleuth Kit. It is accepted in courts worldwide.

Download: sleuthkit.org/autopsy

### 8.1 Creating a New Case

```
Autopsy Launch Screen:
  → New Case
  OR
  → Open Case (to resume previous investigation)
  OR
  → Recent Cases (quick access)

New Case Setup:
  1. Case Name: "Ramnagar_Corp_Fraud_2025" (descriptive)
  2. Base Directory: where case files are saved
  3. Case Number: e.g., "CASE-2025-003"
  4. Examiner Name: your name
  5. Examiner Phone & Email
  6. Organization: your company/department
  7. Finish → case folder created
```

### 8.2 Adding Evidence Source

```
After creating case → "Add Data Source" dialog

Select evidence type:
  ○ Disk Image or VM File  ← use this for forensic images
  ○ Local Disk             ← for live analysis of attached drive
  ○ Logical Files          ← for individual files/folders

For Disk Image:
  1. Browse → select your .E01 / .dd / .vmdk / .img file
  2. Optional: enter MD5/SHA-256 hash → Autopsy verifies integrity
  3. Time Zone: set correctly (affects timestamp analysis)

Configure Ingest Modules (what Autopsy should analyze):
  ☑ Recent Activity (browser history, recently opened files)
  ☑ Hash Lookup (checks against NSRL + custom bad hashes)
  ☑ File Type Identification (magic bytes analysis)
  ☑ Extension Mismatch Detector ← critical
  ☑ Exif Parser (photo metadata)
  ☑ Keyword Search
  ☑ Email Parser
  ☑ Encryption Detection
  ☑ Interesting Files (flags suspicious files)

→ Finish → Autopsy begins ingesting and analyzing
```

### 8.3 Analyzing Results

**Left panel — Data Sources tree:**

```
Data Sources
└── [Evidence Drive Name]
    └── [Image File]
        ├── Volume Shadow Copies
        ├── $OrphanFiles (unallocated/deleted file fragments)
        └── [Partitions]
            └── [File System]
                ├── $RECYCLE.BIN     ← deleted files in bin
                ├── Documents
                ├── Downloads
                ├── Desktop
                └── ... (all directories)
```

**Left panel — Results tree (ingest module findings):**

```
Results
├── Extracted Content
│   ├── Accounts (user accounts found)
│   ├── Cookies
│   ├── Devices Attached (external drives ever connected)
│   ├── Encryption Detected (password-protected files)
│   ├── EXIF Metadata (photo location, camera, date)
│   ├── Extension Mismatch Detected ← FILES WITH WRONG EXTENSION
│   ├── Installed Programs
│   ├── Operating System Info
│   ├── Recent Documents
│   ├── Shell Bags (folder access history)
│   ├── Web Bookmarks
│   ├── Web Downloads
│   ├── Web History
│   └── Web Search (search terms used)
│
├── Interesting Items
│   └── Files flagged as suspicious
│
└── Keyword Hits (if keyword search configured)
```

**Key analysis areas:**

**Viewing deleted files:**

```
File Views → Deleted Files
OR
File system tree → files marked with ✗ (red X) = deleted

Right-click deleted file → Extract File → save to your analysis folder
```

**File metadata view:**

```
Click any file → bottom panel shows:
  - File name
  - Parent directory (original location)
  - Modified date (last change)
  - Accessed date (last opened)
  - Created date
  - Changed date (metadata change)
  - File size
  - MD5 hash (calculated by Autopsy)
  - File type (from magic bytes)
  - MIME type
```

**Tagging important files:**

```
Right-click file → Add File Tag
Options:
  - Notable Item (worth investigating)
  - Follow Up
  - Bookmark
  - Custom tag

Tagged files appear in: Results → Tags
Useful for: marking evidence you want in your final report
```

**Thumbnail view for images:**

```
Views → File Types → Images
→ All images shown as thumbnails
→ Click any to see metadata + preview
→ Deleted images marked with ✗
```

**Email analysis:**

```
Results → Extracted Content → E-mail Messages
→ Shows email headers, sender, recipient, subject, date
→ Attachments can be extracted
```

### 8.4 Timeline Analysis

```
Autopsy → Timeline (top menu or Views → Timeline)

Granularity options:
  Year      → Overview of all years of activity
  Month     → Monthly breakdown
  Day       → Daily activity
  Hour      → Hourly breakdown
  Minute    → Minute-by-minute
  Second    → Second-by-second (very detailed)

Reading the timeline:
  X-axis = Time
  Y-axis = Number of file system events

Click any bar → shows what files were created/modified/accessed
                at that exact time

Use case: "What was happening on the date of the suspected fraud?"
→ Zoom to that date → see every file activity at that moment
```

### 8.5 Extension Mismatch Detection

```
Results → Extension Mismatch Detected

Shows files where:
  Stated extension ≠ actual file type (from magic bytes)

Example:
  File: "quarterly_report.pdf"
  Actual type: ZIP archive
  → Someone hid a ZIP as a PDF

Action:
  1. Right-click → Extract File
  2. Open in hex editor → view magic bytes
  3. Rename with correct extension
  4. Open to see contents
  5. Document in report
```

---

## 9. Lab 7 — File System Analysis with The Sleuth Kit (Command Line)

The Sleuth Kit (TSK) is the command-line engine beneath Autopsy. Useful when you need specific low-level analysis.

**Open terminal in the TSK bin directory:**

```cmd
# Navigate to Sleuth Kit installation
cd "C:\Program Files\SleuthKit\bin"
# OR open CMD directly from the folder:
# Type "cmd" in the address bar of the folder → press Enter
```

### 9.1 Key Sleuth Kit Commands

```bash
# ─────────────────────────────────────────────────────────────
# PARTITION/VOLUME LAYOUT
# ─────────────────────────────────────────────────────────────

# List partitions in an image
mmls "C:\path\to\evidence.dd"

# Output example:
# GUID Partition Table (EFI)
# Offset Sector: 0
# Units are in 512-byte sectors
#      Slot      Start        End          Length       Description
# 000:  Meta      0000000000   0000000000   0000000001   Safety Table
# 001:  -------   0000000000   0000002047   0000002048   Unallocated
# 002:  000       0000002048   0000001050623  0001048576  NTFS (0x07)
# 003:  001       0001050624   0001953525167  0952474544  NTFS (0x07)

# ─────────────────────────────────────────────────────────────
# FILE SYSTEM DETAILS
# ─────────────────────────────────────────────────────────────

# Get file system information (Windows NTFS)
fsstat -f ntfs "C:\path\to\evidence.dd" -o 0

# Get file system information (using image directly)
fsstat "C:\evidence.dd"

# Output includes:
# - File System Type: NTFS
# - Volume Name
# - Number of sectors
# - Cluster size
# - Root directory entry
# - MFT (Master File Table) location

# ─────────────────────────────────────────────────────────────
# IMAGE INFORMATION
# ─────────────────────────────────────────────────────────────

# Get image metadata
img_stat "C:\path\to\evidence.dd"

# Output:
# IMAGE FILE INFORMATION
# Raw Image
# Sector Size: 512
# Image Format: raw
# 2097152 sectors (1073741824 bytes = 1 GB)

# Convert to GB: 1073741824 / 1024 / 1024 / 1024 = 1 GB
```

### 9.2 Master File Table (MFT) Entries

NTFS stores all file information in the Master File Table. Each file and directory has a numbered entry.

```bash
# Get metadata of MFT entry (file/folder)
istat -f ntfs "C:\evidence.dd" 0       # Entry 0: MFT itself
istat -f ntfs "C:\evidence.dd" 1       # Entry 1: MFT Mirror (backup)
istat -f ntfs "C:\evidence.dd" 5       # Entry 5: Root directory
istat -f ntfs "C:\evidence.dd" 6       # Entry 6: Bitmap
istat -f ntfs "C:\evidence.dd" 7       # Entry 7: Boot file
istat -f ntfs "C:\evidence.dd" 8       # Entry 8: Bad cluster file

# Useful entries:
# 0  = $MFT (Master File Table)
# 1  = $MFTMirr (MFT backup copy)
# 2  = $LogFile (NTFS transaction log)
# 3  = $Volume (volume information)
# 4  = $AttrDef (attribute definitions)
# 5  = . (root directory)
# 6  = $Bitmap (cluster allocation status)
# 7  = $Boot (boot sector)
# 8  = $BadClus (bad clusters)

# Get volume metadata (entry 3 = $Volume)
istat -f ntfs "C:\evidence.dd" 3

# Output: File system type, volume name, NTFS version, etc.

# Get bad cluster list (entry 8)
istat -f ntfs "C:\evidence.dd" 8
# If output is empty → no bad clusters
```

### 9.3 File & Directory Listing

```bash
# List files and directories
fls -f ntfs "C:\evidence.dd"

# Output format:
# d/d 5-128-1:     .                    (directory)
# d/d 11-128-4:    $OrphanFiles
# r/r 25-128-1:    readme.txt           (regular file)
# r/r 26-128-1:    report.pdf
# d/d 27-128-1:    Documents/

# r/r = regular file
# d/d = directory
# -/r = DELETED file (the dash indicates deletion)

# Recursive listing
fls -r -f ntfs "C:\evidence.dd"

# Show deleted files only
fls -r -d -f ntfs "C:\evidence.dd"

# List from a specific MFT entry (e.g., entry 5 = root)
fls -f ntfs "C:\evidence.dd" 5
```

**Recovering files with TSK:**

```bash
# Extract a specific file using its MFT entry number
# First find the entry number from fls output
# Then use icat to extract

icat -f ntfs "C:\evidence.dd" MFT_ENTRY_NUMBER > output_file.pdf

# Example: extract file at MFT entry 100
icat -f ntfs "C:\evidence.dd" 100 > recovered_document.pdf

# Recover ALL deleted files to a directory
tsk_recover "C:\evidence.dd" "C:\recovery_folder\"
```

**Additional useful commands:**

```bash
# Calculate MD5/SHA1 of image for integrity
md5sum evidence.dd     # Linux
# On Windows: use HashCalc tool or PowerShell:
Get-FileHash evidence.dd -Algorithm SHA256

# Search for keywords in unallocated space
srch_strings "C:\evidence.dd" | findstr "password"
srch_strings "C:\evidence.dd" | findstr "confidential"

# Timeline creation
mactime -b bodyfile.txt -d > timeline.csv
fls -r -m "/" "C:\evidence.dd" > bodyfile.txt
mactime -b bodyfile.txt -d > timeline.csv
# Opens in Excel/LibreOffice for timeline review
```

---

## 10. Lab 8 — Linux File System Analysis in Autopsy

### 10.1 Important Linux Directory Structure

When analyzing a Linux forensic image, understanding the directory structure is essential:

```
LINUX FILE SYSTEM STRUCTURE FOR FORENSIC ANALYSIS
══════════════════════════════════════════════════════════════

  /          ← Root directory (everything starts here)
  ├── /bin   ← Essential command binaries (anyone can run)
  │          Commands: ls, cp, mv, cat, echo, etc.
  │
  ├── /sbin  ← System binaries (root/admin only)
  │          Commands: fdisk, ifconfig, mount, etc.
  │
  ├── /boot  ← Boot files (bootloader, kernel)
  │          Evidence: kernel version, boot configuration
  │
  ├── /etc   ← CRITICAL FORENSIC DIRECTORY
  │   ├── /etc/passwd      ← USER ACCOUNTS
  │   │   Format: username:x:UID:GID:comment:home:shell
  │   │   Shows: all users, their IDs, home directories
  │   ├── /etc/shadow      ← HASHED PASSWORDS
  │   │   Format: username:$hash$salt$value:last_changed:...
  │   │   Contains hashed passwords (not plaintext)
  │   ├── /etc/group       ← Group memberships
  │   ├── /etc/hosts       ← Static DNS mappings
  │   ├── /etc/crontab     ← Scheduled tasks
  │   └── /etc/ssh/        ← SSH configuration
  │
  ├── /home  ← USER DATA DIRECTORIES
  │   ├── /home/alice/     ← User Alice's files
  │   │   ├── .bash_history  ← COMMANDS HISTORY ← KEY EVIDENCE
  │   │   ├── .bashrc        ← Shell configuration
  │   │   ├── Desktop/
  │   │   ├── Documents/
  │   │   ├── Downloads/
  │   │   └── .ssh/          ← SSH keys (if any)
  │   └── /home/bob/
  │
  ├── /lib   ← Shared libraries
  ├── /media ← Mount point for removable drives
  ├── /mnt   ← Manual mount points
  │          Evidence: other drives that were mounted
  │
  ├── /opt   ← Optional/third-party software
  ├── /proc  ← Process information (RAM data — not in disk image)
  ├── /root  ← Root user's home directory
  │   ├── .bash_history ← ROOT'S COMMAND HISTORY ← KEY EVIDENCE
  │   └── ...
  │
  ├── /snap  ← Snap packages
  ├── /srv   ← Server data
  ├── /sys   ← System kernel data
  ├── /tmp   ← Temporary files (evidence: recently used files)
  │
  ├── /usr   ← User programs and data
  │   ├── /usr/bin/   ← User commands
  │   └── /usr/share/ ← Shared data/docs
  │
  └── /var   ← CRITICAL FORENSIC DIRECTORY
      ├── /var/log/         ← SYSTEM LOGS ← KEY EVIDENCE
      │   ├── auth.log      ← Login attempts, sudo usage
      │   ├── syslog        ← System events
      │   ├── kern.log      ← Kernel events
      │   ├── apache2/      ← Web server logs (if Apache)
      │   └── nginx/        ← Web server logs (if Nginx)
      ├── /var/mail/        ← Local email
      └── /var/spool/cron/  ← Cron jobs per user

══════════════════════════════════════════════════════════════
```

**High-value forensic artifacts in Linux:**

| Location                          | What it reveals                                |
| --------------------------------- | ---------------------------------------------- |
| `/etc/passwd`                     | All user accounts, UIDs, home directories      |
| `/etc/shadow`                     | Password hashes, password change dates         |
| `/home/user/.bash_history`        | Every command the user typed                   |
| `/root/.bash_history`             | Every root command executed                    |
| `/var/log/auth.log`               | All login attempts (success/failure)           |
| `/var/log/syslog`                 | System events timeline                         |
| `/home/user/.ssh/known_hosts`     | Servers this user has connected to             |
| `/home/user/.ssh/authorized_keys` | SSH keys with remote access                    |
| `/tmp/`                           | Temporary files (may contain attack artifacts) |
| `/var/log/apache2/access.log`     | Web server requests (if web server present)    |
| Cron files                        | Scheduled tasks (persistence mechanisms)       |

**In Autopsy — Linux image analysis:**

```
Add Data Source → select .img file of Linux system
→ Autopsy identifies Linux ext4/ext3 file system
→ Browse /etc/passwd → shows all user accounts
→ Browse /home/username/.bash_history → command history
→ Browse /var/log/ → system logs
→ Deleted files → appears with ✗ mark
→ Results → OS Account Information → shows accounts
→ Results → E-mail Messages → local mail
```

---

## 11. Lab 9 — Forensic Report (65B Report)

**Section 65B** of the Indian Evidence Act / Electronic Evidence Act governs electronic evidence in court. The format is widely recognized internationally.

**Report structure:**

```
FORENSIC INVESTIGATION REPORT — SECTION 65B
══════════════════════════════════════════════════════════════

CASE INFORMATION
  Case Number:        CASE-2025-003
  Case Title:         Employee Data Fraud — [Company Name]
  Date of Report:     January 10, 2025
  Investigator:       [Full Name]
  Certification:      [Cert Name + Number]
  Organization:       [Your Forensic Firm]
  Contact:            [Phone, Email]

AUTHORIZATION
  Authorized by:      [Company / Court / Police]
  Authorization Type: [Court Order / Written Consent]
  Order Number:       [Court Order Number if applicable]

EVIDENCE DETAILS
  Evidence Item 1:
    Description:      Western Digital 500GB HDD
    Serial Number:    WD-XXXXXXXX
    Make/Model:       WD Blue 500GB
    Date Seized:      January 8, 2025, 14:30 hrs
    Location Seized:  Ramnagar Corp, Server Room
    Seized by:        [Officer Name + ID]
    Condition:        Good condition, no physical damage

HASH VALUES
  Evidence Item 1 (Original HDD):
    MD5:    [value]
    SHA-256: [value]
  Forensic Copy 1 (Investigator's Copy):
    MD5:    [SAME as original]
    SHA-256: [SAME as original]
  Forensic Copy 2 (Returned to Owner):
    MD5:    [SAME as original]
    SHA-256: [SAME as original]

INVESTIGATION WORKSTATION DETAILS
  Computer Make/Model:  [e.g., Dell Latitude 7490]
  Operating System:     Windows 11 Pro v22H2
  MAC Address:          AA:BB:CC:DD:EE:FF
  IP Address:           192.168.1.50
  Investigation Date:   January 9, 2025

TOOLS USED
  Tool 1: Autopsy 4.21.0 (open source, court-approved)
  Tool 2: HashCalc 2.02
  Tool 3: R-Drive Image 7.0 (licensed)
  Tool 4: S Data Recovery 6.4
  Tool 5: The Sleuth Kit 4.11.1
  [Note: Only court-approved tools listed]

METHODOLOGY
  Step 1: Received evidence with chain of custody documentation
  Step 2: Verified hash of original evidence
  Step 3: Created forensic image using R-Drive Image (sector-by-sector)
  Step 4: Verified image hash matches original
  Step 5: Analyzed forensic image copy using Autopsy
  Step 6: Recovered deleted files using S Data Recovery
  Step 7: Analyzed file system using The Sleuth Kit
  Step 8: Generated report

FINDINGS
  1. [Finding 1]: 47 files deleted via Shift+Delete on January 5, 2025
     - File types: Excel (.xlsx), PDF (.pdf), Word (.docx)
     - Files successfully recovered (attached as Exhibit A)

  2. [Finding 2]: Financial transfer records found in browser history
     - Timestamp: January 4, 2025, 11:45 AM
     - Evidence: Firefox browser history cache (Exhibit B)

  3. [Finding 3]: Extension mismatch detected
     - "report.jpg" is actually a ZIP archive
     - ZIP contained: [encrypted files]

CHAIN OF CUSTODY LOG
  [Table of every transfer — see Section 12]

CONCLUSION
  Based on the investigation, evidence of data deletion and
  financial fraud was recovered from the seized device.
  The recovered evidence is provided in the exhibits attached.

DECLARATION
  I, [Name], hereby certify that this report is accurate
  and the investigation was conducted in accordance with
  accepted forensic standards and legal requirements.

Signature: _______________    Date: _______________

══════════════════════════════════════════════════════════════
```

---

## 12. Forensic Evidence Bags & Chain of Custody

### Evidence Bag Types

```
EVIDENCE BAG SELECTION GUIDE
══════════════════════════════════════════════════════════════

  ANTI-STATIC BAG (gray/pink metallic bag)
  Use for:
  ✓ Hard disk drives (HDDs/SSDs)
  ✓ RAM modules
  ✓ USB drives
  ✓ SD cards
  ✓ Any circuit board component
  ✓ Motherboard components

  Why: Static charge from human body or environment
       can damage magnetic media and circuits.
       Anti-static bag dissipates static safely.
  ─────────────────────────────────────────────────────────

  FARADAY BAG (silver, signal-blocking bag)
  Use for:
  ✓ Mobile phones
  ✓ Tablets
  ✓ Smart watches
  ✓ GPS devices
  ✓ Any wireless-capable device

  Why: Blocks ALL signals (WiFi, GSM/4G/5G, Bluetooth,
       NFC, GPS). Prevents:
       - Remote wipe commands
       - Remote lock commands
       - New messages/calls modifying data
       - Location tracking
       - Any remote hacking
  ─────────────────────────────────────────────────────────

  ADDITIONAL REQUIREMENTS:
  ✓ Wear NITRILE GLOVES when handling all evidence
    (static prevention + fingerprint preservation)
  ✓ Tamper-evident seal on all bags
  ✓ Label with: case number, item number, date, time,
    seizing officer, location of seizure

══════════════════════════════════════════════════════════════
```

### Chain of Custody Form

```
CHAIN OF CUSTODY DOCUMENT
═══════════════════════════════════════════════════════════════════

CASE NUMBER: CASE-2025-003
EVIDENCE ITEM: #001 — WD 500GB HDD (Serial: WD-XXXXX)

┌─────────────────────────────────────────────────────────────────┐
│ Date/Time  │ Released By         │ Received By         │ Purpose │
├────────────┼─────────────────────┼─────────────────────┼─────────┤
│ 08-Jan-25  │ [Crime Scene]       │ Inv. John Doe       │ Seizure │
│ 14:30      │ N/A                 │ ID: INV-001         │         │
│            │                     │ Ph: +91-XXXXXXXXXX  │         │
├────────────┼─────────────────────┼─────────────────────┼─────────┤
│ 08-Jan-25  │ Inv. John Doe       │ Lab Tech: Jane Smith│ Lab     │
│ 16:00      │ ID: INV-001         │ ID: LAB-003         │ Storage │
│            │                     │ Ph: +91-XXXXXXXXXX  │         │
├────────────┼─────────────────────┼─────────────────────┼─────────┤
│ 09-Jan-25  │ Lab Tech: Jane Smith│ Inv. John Doe       │Analysis │
│ 09:00      │ ID: LAB-003         │ ID: INV-001         │         │
│            │                     │                     │         │
├────────────┼─────────────────────┼─────────────────────┼─────────┤
│ 09-Jan-25  │ Inv. John Doe       │ Lab Tech: Jane Smith│Secure   │
│ 18:30      │ ID: INV-001         │ ID: LAB-003         │Storage  │
└────────────┴─────────────────────┴─────────────────────┴─────────┘

STORAGE LOCATION: Evidence Room B, Cabinet 3, Slot 7
ACCESS CONTROL: Authorized personnel only
CCTV Coverage: Yes (Camera ID: CAM-04)
Temperature: 20°C, Humidity: 45%

═══════════════════════════════════════════════════════════════════
```

---

## 13. Comprehensive Lab Exercises

> ⚠️ **All labs use YOUR OWN test drives, VMs, and data. Never perform forensic analysis on systems or data you do not own or have written authorization to examine.**

---

### Lab A: Create Evidence Drive & Simulate Crime

**Objective:** Set up a realistic forensic investigation scenario.

```bash
# PART 1: Create test evidence on a USB drive (your own USB)

# Format a USB drive (WARNING: this erases everything on it)
# In Kali Linux:
lsblk                    # identify your USB (e.g., /dev/sdb)
sudo mkfs.ext4 /dev/sdb1 # format with ext4
sudo mount /dev/sdb1 /mnt/evidence

# Create normal files (will be the "existing" files)
echo "Company Financial Report Q4 2024" > /mnt/evidence/finance_report.txt
echo "Employee ID: 1042, Transfer Amount: $50,000" >> /mnt/evidence/finance_report.txt
mkdir /mnt/evidence/Projects
echo "Project Alpha - Confidential" > /mnt/evidence/Projects/project_alpha.txt

# Create files that the "attacker" will delete
echo "Evidence of fraud - transaction log" > /mnt/evidence/transaction_log.txt
echo "Password: admin123" > /mnt/evidence/credentials.txt
echo "Bank account: 1234567890" >> /mnt/evidence/credentials.txt

# "Delete" the evidence (simulate Shift+Delete / rm)
rm /mnt/evidence/transaction_log.txt
rm /mnt/evidence/credentials.txt

# Add a file with wrong extension (extension mismatch)
cp /mnt/evidence/finance_report.txt /mnt/evidence/photo_vacation.jpg
# This is actually a text file saved as .jpg

sudo umount /mnt/evidence
# USB now has: some files present, some deleted, one extension mismatch
```

```powershell
# Windows version — create test evidence on USB:

# Open USB in File Explorer
# Create files:
New-Item -Path "E:\finance_report.txt" -ItemType File
Set-Content -Path "E:\finance_report.txt" -Value "Financial Report Q4"
New-Item -Path "E:\Projects" -ItemType Directory
Set-Content -Path "E:\transaction_log.txt" -Value "Fraud evidence log"
Set-Content -Path "E:\credentials.txt" -Value "Password: admin123"

# "Delete" files (Shift+Delete simulation — bypass Recycle Bin):
Remove-Item -Path "E:\transaction_log.txt" -Force
Remove-Item -Path "E:\credentials.txt" -Force

# Create extension mismatch
Copy-Item "E:\finance_report.txt" "E:\photo_vacation.jpg"
```

---

### Lab B: Full Forensic Image Creation

**Objective:** Create a court-admissible forensic image of your test USB drive.

```bash
# Method 1: dd (Linux — free, powerful)
# Identify source device
lsblk
# /dev/sdb = your USB evidence drive

# Calculate hash BEFORE imaging (baseline)
sha256sum /dev/sdb > /root/evidence_original_hash.txt
cat /root/evidence_original_hash.txt

# Create forensic image with dd
sudo dd if=/dev/sdb of=/root/evidence.dd bs=512 status=progress conv=noerror,sync

# Parameters:
# if= input file (source)
# of= output file (destination image)
# bs= block size (512 bytes = one sector)
# status=progress = show progress
# conv=noerror,sync = continue on read errors; pad bad sectors with zeros

# Hash the image
sha256sum /root/evidence.dd > /root/image_hash.txt

# Compare hashes (must match)
cat /root/evidence_original_hash.txt
cat /root/image_hash.txt
# If SHA-256 values match → image is perfect forensic copy

# Method 2: dc3dd (forensic-focused dd variant)
sudo apt install dc3dd -y
sudo dc3dd if=/dev/sdb of=/root/evidence_dc3.dd hash=sha256 \
  hlog=/root/dc3dd_hash.log progress=on

# dc3dd automatically logs hash values during imaging

# Method 3: guymager (GUI forensic imager on Kali)
# Start → guymager
# Select device → right-click → Acquire image
# Format: EWF (E01) or Linux DD
# Fill in case details
# Start → image created with automatic hash verification
```

**Verify image integrity:**

```bash
# Mount image as read-only to verify content
sudo mkdir /mnt/image_verify
sudo mount -o ro,loop /root/evidence.dd /mnt/image_verify
ls /mnt/image_verify    # Should show same files as original USB
sudo umount /mnt/image_verify
```

---

### Lab C: Hash Generation & Integrity Testing

**Objective:** Prove hash values change with any modification.

```bash
# Create a test file
echo "This is the original content" > /root/test_integrity.txt

# Calculate hash
sha256sum /root/test_integrity.txt
# Record this value: [HASH_A]

# Modification 1: Add a period
echo "This is the original content." > /root/test_integrity.txt
sha256sum /root/test_integrity.txt
# Compare with [HASH_A] → completely different

# Modification 2: Remove period (restore original)
echo "This is the original content" > /root/test_integrity.txt
sha256sum /root/test_integrity.txt
# Should MATCH [HASH_A] exactly

# Modification 3: Add a space at end
printf "This is the original content " > /root/test_integrity.txt
sha256sum /root/test_integrity.txt
# Different from [HASH_A]

# Python script for batch hash comparison:
```

```python
#!/usr/bin/env python3
"""
Forensic Hash Comparison Tool
Run on your own test files only
"""
import hashlib
import os

def calculate_hash(filepath, algorithm='sha256'):
    h = hashlib.new(algorithm)
    with open(filepath, 'rb') as f:
        for chunk in iter(lambda: f.read(65536), b''):
            h.update(chunk)
    return h.hexdigest()

def compare_hashes(file1, file2):
    hash1 = calculate_hash(file1)
    hash2 = calculate_hash(file2)

    print(f"File 1: {file1}")
    print(f"SHA-256: {hash1}")
    print(f"\nFile 2: {file2}")
    print(f"SHA-256: {hash2}")
    print(f"\nMatch: {'✓ IDENTICAL' if hash1 == hash2 else '✗ DIFFERENT — POSSIBLE TAMPERING'}")
    return hash1 == hash2

# Usage examples:
# compare_hashes('/root/original_evidence.pdf', '/root/copy1.pdf')
# compare_hashes('/root/evidence.dd', '/root/evidence_copy.dd')

# Generate hashes for all files in a directory (evidence drive)
def hash_directory(path):
    results = {}
    for root, dirs, files in os.walk(path):
        for filename in files:
            filepath = os.path.join(root, filename)
            try:
                results[filepath] = {
                    'md5': calculate_hash(filepath, 'md5'),
                    'sha256': calculate_hash(filepath, 'sha256')
                }
            except PermissionError:
                results[filepath] = {'error': 'Permission denied'}
    return results

# Run:
# hashes = hash_directory('/mnt/evidence')
# for path, values in hashes.items():
#     print(f"{path}: SHA256={values.get('sha256', 'N/A')}")
```

---

### Lab D: Data Recovery on Your Own Drive

**Objective:** Recover deleted test files using command-line tools.

```bash
# Using PhotoRec (free, included in testdisk package)
sudo apt install testdisk -y

# Run PhotoRec on your image file (not the original!)
photorec /root/evidence.dd

# Interactive menu:
# Select disk → /root/evidence.dd
# Partition → select the correct partition
# File system → Other (for Linux ext) or Intel (for FAT/NTFS)
# Choose files to recover → All (or specific types)
# Destination → /root/recovered_files/

# PhotoRec recovers files based on magic bytes, NOT file names
# (file names are lost — recovered as fileXXXXXX.pdf, etc.)

# Using Foremost (carving tool — recovers by file signature)
sudo apt install foremost -y
foremost -t all -i /root/evidence.dd -o /root/foremost_output/

# Output organized by file type:
# /root/foremost_output/pdf/ → recovered PDFs
# /root/foremost_output/jpg/ → recovered JPGs
# /root/foremost_output/docx/ → recovered Word files

# Using Scalpel (similar to foremost, configurable)
sudo apt install scalpel -y
# Edit config: nano /etc/scalpel/scalpel.conf
# Uncomment file types you want to recover
scalpel -o /root/scalpel_output/ /root/evidence.dd
```

---

### Lab E: Autopsy Full Analysis Walkthrough

**Objective:** Complete Autopsy investigation workflow on your test image.

```
1. Create your forensic image (Lab B)

2. Open Autopsy
   New Case → fill all fields carefully
   → Case name, number, examiner info

3. Add Data Source
   → Disk Image or VM File
   → Select your .dd image
   → Enter SHA-256 hash from Lab B for verification
   → Enable ALL ingest modules

4. Wait for ingest to complete (progress bar)

5. Investigate systematically:

   a) Deleted Files
      File Views → Deleted Files (✗ icon)
      → How many deleted files found?
      → What types? (documents, images, etc.)
      → Right-click → Extract File → save to analysis folder

   b) Extension Mismatch
      Results → Extension Mismatch Detected
      → List all mismatches
      → Extract each file
      → Use hex editor to identify true type

   c) Browser History (if Windows image)
      Results → Extracted Content → Web History
      → Look for suspicious sites, financial sites

   d) Timeline
      Timeline → zoom to date of suspected incident
      → What happened in those hours?

   e) User Accounts
      Results → Extracted Content → OS Accounts
      → Who had accounts on this system?
      → When did accounts last log in?

   f) Devices Attached
      Results → Extracted Content → Devices Attached
      → What USB devices were ever connected?
      → When?

6. Tag important evidence files

7. Generate Report
   Generate Report → HTML or Excel format
   → Include: all findings, tagged items, timeline
```

---

### Lab F: Sleuth Kit Command-Line Investigation

**Objective:** Perform investigation using only command-line Sleuth Kit tools.

```bash
# On Kali Linux, TSK is pre-installed
# If not: sudo apt install sleuthkit -y

IMAGE="/root/evidence.dd"

# Step 1: Get image info
img_stat "$IMAGE"

# Step 2: Find partitions
mmls "$IMAGE"
# Note the offset of the partition you want (in sectors)
# Example output: partition starts at sector 2048

OFFSET=2048  # adjust to your partition offset

# Step 3: File system info
fsstat -o $OFFSET "$IMAGE"

# Step 4: List all files (including deleted)
fls -r -o $OFFSET "$IMAGE" > /root/file_listing.txt
cat /root/file_listing.txt | grep "^\-" | head -20  # deleted files

# Step 5: Investigate specific file by MFT entry
# Find entry number from fls output, e.g., entry 27
istat -o $OFFSET "$IMAGE" 27

# Step 6: Extract specific file
icat -o $OFFSET "$IMAGE" 27 > /root/recovered_27.pdf

# Step 7: List deleted files only
fls -r -d -o $OFFSET "$IMAGE"

# Step 8: Search for strings in unallocated space
blkls -o $OFFSET "$IMAGE" | strings | grep -i "password\|account\|transfer"

# Step 9: Create timeline
fls -r -m "/" -o $OFFSET "$IMAGE" > /root/bodyfile.txt
mactime -b /root/bodyfile.txt -d 2024-01-01 2025-12-31 > /root/timeline.csv
# Open timeline.csv in LibreOffice Calc for analysis
```

---

### Lab G: VirusTotal Hash Investigation

**Objective:** Use hash-based malware detection for forensic purposes.

```bash
# Generate hashes of suspicious files from your test evidence
sha256sum /root/test_suspicious.exe
sha256sum /root/test_document.pdf

# Submit to VirusTotal via command line (requires API key)
# Free API key at: virustotal.com/gui/join-us

pip3 install vt-py   # Official VirusTotal Python library

python3 << 'EOF'
import vt

API_KEY = "YOUR_FREE_API_KEY_HERE"

with vt.Client(API_KEY) as client:
    # Check a hash
    file_hash = "sha256_hash_of_your_test_file_here"

    try:
        file_info = client.get_object(f"/files/{file_hash}")
        stats = file_info.last_analysis_stats
        print(f"Malicious:  {stats['malicious']}")
        print(f"Suspicious: {stats['suspicious']}")
        print(f"Undetected: {stats['undetected']}")
        print(f"File type:  {file_info.type_description}")
    except vt.error.APIError as e:
        if e.code == "NotFoundError":
            print("Hash not in VirusTotal database - upload file for first scan")
        else:
            raise
EOF

# For your MERN/PERN web apps - check npm packages for known malicious hashes:
# npm audit  (checks package vulnerabilities)
# Then cross-check suspicious package hashes with VirusTotal
```

---

## 14. Glossary

| Term                        | Definition                                                                                                                               |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **65B Report**              | Forensic investigation report format under Section 65B of the Electronic Evidence Act; required for court-admissible electronic evidence |
| **Anti-Static Bag**         | Bag that dissipates static charge; used for storing HDD, RAM, circuit boards during evidence collection                                  |
| **Autopsy**                 | Free, open-source digital forensics GUI built on The Sleuth Kit; widely accepted in courts                                               |
| **Bit-by-bit Copy**         | Copying every single bit of a storage device including empty space; the foundation of forensic imaging                                   |
| **Chain of Custody**        | Documented paper trail recording who handled evidence, when, where, and why; essential for court admissibility                           |
| **Clone**                   | Copy of only the DATA on a drive, not the empty space; NOT suitable for forensics                                                        |
| **DD**                      | Unix command and file format for raw disk imaging (bit-by-bit); produces `.dd` or `.raw` files                                           |
| **E01**                     | EnCase forensic image format; includes metadata, case information, and built-in integrity checking                                       |
| **Entropy**                 | Measure of randomness/compression in a file; high entropy in executable files may indicate malware packing                               |
| **Evidence Manager**        | Person responsible for storing and tracking physical evidence in a forensic lab                                                          |
| **Extension Mismatch**      | When a file's extension (`.pdf`) does not match its actual file type (ZIP, image, etc.)                                                  |
| **Faraday Bag**             | Signal-blocking bag for phones and wireless devices; prevents remote wipe, tracking, and access                                          |
| **File Carving**            | Recovering files from raw data based on file signatures (magic bytes) without relying on file system                                     |
| **First Responder**         | First investigator at the crime scene; responsible for volatile data capture and evidence preservation                                   |
| **Forensic Image**          | Exact bit-for-bit duplicate of an entire storage device including all free/unallocated space                                             |
| **Hash Value**              | Fixed-length digital fingerprint of data; any modification changes the hash completely                                                   |
| **HMAC**                    | Hash-based Message Authentication Code; hash combined with a secret key for authenticated integrity verification                         |
| **Ingest Module**           | Autopsy plugin that analyzes specific aspects of evidence (browser history, email, hashes, etc.)                                         |
| **Magic Bytes**             | First few bytes of a file that identify its true type regardless of file extension                                                       |
| **Master File Table (MFT)** | NTFS database that records information about every file and directory on a Windows volume                                                |
| **MD5**                     | Message Digest 5; 128-bit hash algorithm; deprecated for security but still used in forensics as reference                               |
| **Metadata**                | Data about data; for files: creation date, modification date, author, camera model, GPS location, etc.                                   |
| **PhotoRec**                | Free file carving tool that recovers files based on magic bytes; ignores file system                                                     |
| **RAM Dump**                | Capture of all data in RAM at a specific moment; captures running processes, encryption keys, etc.                                       |
| **Scalpel**                 | Configurable file carving tool; recovers files by signature; more configurable than foremost                                             |
| **SHA-256**                 | Secure Hash Algorithm 256-bit; current standard for forensic evidence integrity verification                                             |
| **Shift+Delete**            | Windows keyboard shortcut that permanently deletes files bypassing the Recycle Bin                                                       |
| **The Sleuth Kit (TSK)**    | Open-source command-line digital forensics toolkit; provides `fsstat`, `fls`, `istat`, `icat`, etc.                                      |
| **Timeline**                | Chronological view of all file system events; shows what happened when on a device                                                       |
| **Unallocated Space**       | Disk space not assigned to any file; contains remnants of deleted files until overwritten                                                |
| **Volatile Data**           | Data that exists only in RAM and is lost on shutdown; processes, network connections, logged-in users                                    |
| **VirusTotal**              | Free online service for checking files/URLs/hashes against 70+ antivirus databases                                                       |
| **Write Blocker**           | Hardware or software that prevents any write operations to the evidence drive during investigation                                       |

---

## 15. Tools Reference & Certifications

### Investigation Tools

| Category                  | Tool                   | Platform      | Cost            | Notes                            |
| ------------------------- | ---------------------- | ------------- | --------------- | -------------------------------- |
| **Forensic Imaging**      | dd / dc3dd             | Linux         | Free            | Raw bit-for-bit imaging          |
| **Forensic Imaging**      | Guymager               | Linux         | Free            | GUI imager; E01 format           |
| **Forensic Imaging**      | R-Drive Image          | Windows       | Paid            | Good for training                |
| **Forensic Imaging**      | FTK Imager             | Windows       | Free            | Industry standard imager         |
| **Analysis (GUI)**        | **Autopsy**            | Win/Linux/Mac | Free            | Primary tool — court accepted    |
| **Analysis (CLI)**        | **The Sleuth Kit**     | Win/Linux     | Free            | Underlying engine for Autopsy    |
| **Analysis (Commercial)** | EnCase                 | Windows       | $$$$            | Enterprise standard              |
| **Analysis (Commercial)** | Paraben Device Seizure | Windows       | $$$$            | Mobile + PC forensics            |
| **Data Recovery**         | Recuva                 | Windows       | Free/Paid       | Good beginner tool               |
| **Data Recovery**         | PhotoRec               | Win/Linux     | Free            | File carving by signature        |
| **Data Recovery**         | Scalpel                | Linux         | Free            | Configurable file carving        |
| **Data Recovery**         | Stellar Recovery       | Windows       | Paid            | Professional grade               |
| **Hash Tool**             | HashCalc               | Windows       | Free            | Multiple algorithms; HMAC        |
| **Hash Tool**             | md5sum / sha256sum     | Linux         | Free (built-in) | Terminal-based                   |
| **Hash Tool**             | CertUtil               | Windows       | Free (built-in) | `certutil -hashfile file SHA256` |
| **Malware Check**         | **VirusTotal**         | Web           | Free            | 70+ AV engines                   |
| **File ID**               | FileViewer             | Windows       | Free/Paid       | Extension mismatch               |
| **File ID**               | TrID                   | Win/Linux     | Free            | File type identification         |
| **Memory**                | Volatility             | Win/Linux     | Free            | RAM dump analysis                |
| **Memory**                | Rekall                 | Win/Linux     | Free            | RAM analysis                     |
| **Mobile**                | Cellebrite             | Windows       | $$$$            | Phone forensics (professional)   |
| **Mobile**                | Oxygen Forensics       | Windows       | $$$$            | Phone + cloud forensics          |
| **Network**               | Wireshark              | Win/Linux     | Free            | Packet capture analysis          |
| **Hex Editor**            | HxD                    | Windows       | Free            | View file raw bytes              |
| **Hex Editor**            | xxd / hexdump          | Linux         | Free (built-in) | Terminal hex viewer              |

### Windows Built-in Hash (no tools needed):

```powershell
# PowerShell — no extra tools required
Get-FileHash C:\evidence.pdf -Algorithm SHA256
Get-FileHash C:\evidence.pdf -Algorithm MD5

# CertUtil
certutil -hashfile C:\evidence.pdf SHA256
certutil -hashfile C:\evidence.pdf MD5
```

### Linux Built-in Hash (no tools needed):

```bash
sha256sum /path/to/evidence.pdf
md5sum /path/to/evidence.pdf
sha1sum /path/to/evidence.pdf
sha512sum /path/to/evidence.pdf
```

### Certifications in Digital Forensics

| Certification         | Provider        | Level        | Focus                                  |
| --------------------- | --------------- | ------------ | -------------------------------------- |
| **CompTIA Security+** | CompTIA         | Beginner     | Broad security including forensics     |
| **CompTIA CySA+**     | CompTIA         | Intermediate | Threat hunting, analysis               |
| **CEH**               | EC-Council      | Intermediate | Ethical hacking + forensics overview   |
| **CHFI**              | EC-Council      | Intermediate | Computer Hacking Forensic Investigator |
| **GCFE**              | SANS/GIAC       | Intermediate | Windows forensics                      |
| **GCFA**              | SANS/GIAC       | Advanced     | Advanced forensics                     |
| **GREM**              | SANS/GIAC       | Advanced     | Malware analysis                       |
| **EnCE**              | OpenText/EnCase | Advanced     | EnCase product forensics               |
| **CCE**               | ISFCE           | Advanced     | Certified Computer Examiner            |
| **CFCE**              | IACIS           | Advanced     | Computer forensics                     |

### Practice Platforms

| Platform                  | What to practice                                  |
| ------------------------- | ------------------------------------------------- |
| **CyberDefenders**        | Free forensic challenges with real evidence files |
| **Blue Team Labs Online** | Forensic labs, SIEM, incident response            |
| **CTFtime.org**           | CTF competitions often include forensics          |
| **DigitalCorpora.org**    | Free forensic disk images for practice            |
| **NIST CFReDS**           | Reference data sets for forensic tool testing     |
| **TryHackMe**             | Forensics rooms (search: "forensics", "autopsy")  |

### Key Reference URLs

```
Autopsy:         sleuthkit.org/autopsy
The Sleuth Kit:  sleuthkit.org
VirusTotal:      virustotal.com
PhotoRec:        cgsecurity.org/wiki/PhotoRec
Volatility:      volatilityfoundation.org
NIST NSRL:       nsrl.nist.gov  (known good file hash database)
CyberDefenders:  cyberdefenders.org
DigitalCorpora:  digitalcorpora.org
```

---

_Notes compiled from Digital Forensic Investigation Course video transcripts_
_University Cybersecurity Department — Student Reference Document_
_All lab work conducted in authorized environments using test drives and self-created evidence_
_Last updated: 2025_
