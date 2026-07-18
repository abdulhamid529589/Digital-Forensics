# Digital Forensics — Part 2: Investigation Procedures & Lab Setup

### Cybersecurity Department | Course Notes Series

> **Previous:** Part 1 — What is Digital Forensics, Cyber Crime Classification, Evidence Types
> **This Part:** 13-Step Investigation Procedure, Bit-Stream Imaging, First Responder Tools, Lab Setup, Investigation Phases, Challenges
> **Next:** Practical tool usage — Autopsy, FTK, Volatility, Write Blockers

---

## Table of Contents

1. [The 13-Step Investigation Procedure](#1-the-13-step-investigation-procedure)
2. [Bit-Stream Copy vs Regular Copy — Deep Dive](#2-bit-stream-copy-vs-regular-copy--deep-dive)
3. [First Responder Tools](#3-first-responder-tools)
4. [Four Phases of Investigation](#4-four-phases-of-investigation)
5. [Phase 1: Primary Investigation — Lab Setup](#5-phase-1-primary-investigation--lab-setup)
6. [Hardware Requirements for Forensic Lab](#6-hardware-requirements-for-forensic-lab)
7. [Software Requirements for Forensic Lab](#7-software-requirements-for-forensic-lab)
8. [Phase 2: First Response at Crime Scene](#8-phase-2-first-response-at-crime-scene)
9. [Phase 3: Investigation Phase](#9-phase-3-investigation-phase)
10. [Phase 4: Post-Investigation & Reporting](#10-phase-4-post-investigation--reporting)
11. [The Investigation Team — Roles](#11-the-investigation-team--roles)
12. [Challenges in Digital Forensics](#12-challenges-in-digital-forensics)
13. [65B Report — Court Admissibility (India)](#13-65b-report--court-admissibility-india)
14. [Practical Labs](#14-practical-labs)
15. [Attack Scenario → Full Investigation Walkthrough](#15-attack-scenario--full-investigation-walkthrough)
16. [Anti-Forensics vs Forensic Counter-Techniques](#16-anti-forensics-vs-forensic-counter-techniques)

---

## 1. The 13-Step Investigation Procedure

This is the standard operating procedure every digital forensics investigator must follow. Each step is non-negotiable — skipping any step can make evidence inadmissible in court.

```
┌─────────────────────────────────────────────────────────────────┐
│              13-STEP INVESTIGATION PROCEDURE                    │
├────┬────────────────────────────────────────────────────────────┤
│ 01 │ IDENTIFY the crime as a cyber crime                       │
│ 02 │ GATHER initial / primary evidence                         │
│ 03 │ SECURE AUTHORIZATION from court for seizure               │
│ 04 │ UTILIZE first responder tools at the scene                │
│ 05 │ SEIZE / CONFISCATE evidence at the scene                  │
│ 06 │ PRODUCE bit-stream duplicate copy of evidence             │
│ 07 │ GENERATE hash values on the duplicate                     │
│ 08 │ ENSURE continuity — maintain chain of custody             │
│ 09 │ TRANSFER evidence to forensic lab                         │
│ 10 │ SECURELY STORE the original evidence                      │
│ 11 │ ANALYZE the duplicate for evidence                        │
│ 12 │ COMPILE a forensic report (65B report in India)           │
│ 13 │ APPEAR IN COURT and provide expert testimony if needed    │
└────┴────────────────────────────────────────────────────────────┘
```

### Step-by-Step Breakdown with Security Context

---

#### Step 01: Identify the Crime

Determine that the incident is a **cyber crime** before any forensic process begins.

**Key questions at this stage:**

- Was a digital device involved in the commission of the crime?
- Is there evidence of unauthorized access, data theft, fraud, or system damage?
- Does this fall under the IT Act (India) or relevant jurisdiction's cyber law?

**Cyber crime indicators:**

```
✓ Suspicious login alerts from unknown IP
✓ Files encrypted / ransomware note found
✓ Defaced website
✓ Unauthorized bank transactions
✓ Phishing emails with known victim
✓ Device found at physical crime scene
✓ Company reports data breach
```

---

#### Step 02: Gather Primary Evidence

Collect the **minimum initial evidence** that _confirms_ the crime is digital in nature. This is not full collection yet — just enough to establish that a cyber crime occurred.

**Primary evidence examples:**

- A phone found at a murder scene
- A computer showing active ransomware screen
- A server log screenshot showing unauthorized access
- A screenshot of a threatening message or phishing email

**Rule:** Do NOT deeply interact with the device. Just confirm and photograph.

---

#### Step 03: Secure Authorization from Court

**Why this step exists:** Without legal authorization, any evidence collected may be challenged in court as illegally obtained. Different rules apply based on case type:

| Case Type                               | Authorization Required                                                                   |
| --------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Private / Civil case**                | Court order required before seizure                                                      |
| **Police / Criminal case**              | Police already have statutory powers; forensic team joins with existing authorization    |
| **Corporate internal investigation**    | Internal policy + HR authorization (no court needed unless criminal prosecution planned) |
| **Emergency (live attack in progress)** | Can act immediately to stop damage; court order obtained retroactively                   |

**Legal framework (India):**

- Section 79A of IT Act — designates authorized examiners for digital evidence
- Section 65B — admissibility of electronic records
- CrPC Section 91/93 — production of documents and search/seizure

---

#### Step 04: Utilize First Responder Tools

Deploy the appropriate tools based on what type of evidence you found. **First responder tools** are used immediately upon arrival at the scene, before any investigation proper begins.

See [Section 3](#3-first-responder-tools) for complete tool list.

---

#### Step 05: Seize / Confiscate Evidence

**How to properly seize digital evidence:**

```
If device is OFF:
  → Leave it off
  → Do NOT power it on (booting changes timestamps, logs, swap files)
  → Pack immediately in antistatic bag
  → Label: case number, date, time, location, found by whom

If device is ON:
  → Photograph the screen FIRST (volatile visible data)
  → Capture RAM dump BEFORE anything else
  → Note all open programs, windows, network connections
  → Then power off (either pull plug or controlled shutdown based on situation)
  → Pack in antistatic bag

For mobile phones:
  → Do NOT let it connect to any network
  → Enable airplane mode immediately (if possible without unlocking)
  → Place in Faraday bag
  → Never charge it via untrusted charger (juice jacking risk in reverse)
```

**Evidence labeling format:**

```
Case No:    [CASE-2026-001]
Item No:    [ITEM-001]
Description: [Black Samsung Galaxy S21, IMEI unknown]
Found by:   [Investigator Name, Badge/ID]
Found at:   [Location description]
Date/Time:  [17-Jul-2026, 14:30 IST]
Condition:  [Powered OFF, no visible damage]
Sealed by:  [Name]
Hash:       [Calculated after imaging]
```

---

#### Step 06: Produce Bit-Stream Duplicate Copy

Make an **exact bit-for-bit copy** of the original evidence before doing anything else. All investigation happens on this copy — never on the original.

See [Section 2](#2-bit-stream-copy-vs-regular-copy--deep-dive) for complete explanation.

---

#### Step 07: Generate Hash Values on the Duplicate

Hash values are the **digital fingerprint** of the evidence. Like your Aadhaar card number uniquely identifies you, a hash uniquely identifies the exact contents of a file or drive.

```
Hash Properties:
  1. Same input → ALWAYS same output hash
  2. Even one bit changes → completely different hash
  3. Cannot reverse-engineer original from hash (one-way function)
  4. Two different inputs → virtually impossible to have same hash (collision-resistant)
```

```bash
# Generate hash of forensic image (do this immediately after imaging):
md5sum    evidence_copy.img > evidence_md5.txt
sha256sum evidence_copy.img > evidence_sha256.txt

# Contents of evidence_sha256.txt:
# a3f5c9d2e1b8f7a4c6e3d9b2a1f8e5c7d4b3a2f1e9d8c7b6a5f4e3d2c1b0a  evidence_copy.img

# Later in court — verify nothing changed:
sha256sum -c evidence_sha256.txt
# evidence_copy.img: OK  → untampered
# evidence_copy.img: FAILED → tampered → inadmissible!
```

**Hash algorithms used in forensics:**

| Algorithm | Output Size             | Status                          | Use                               |
| --------- | ----------------------- | ------------------------------- | --------------------------------- |
| MD5       | 128-bit (32 hex chars)  | Vulnerable to collision attacks | Still used for speed in forensics |
| SHA-1     | 160-bit (40 hex chars)  | Deprecated                      | Avoid                             |
| SHA-256   | 256-bit (64 hex chars)  | **Recommended**                 | Standard in modern forensics      |
| SHA-512   | 512-bit (128 hex chars) | Most secure                     | Critical evidence                 |

---

#### Step 08: Maintain Chain of Custody

**Chain of Custody = a living document** recording every action taken with the evidence.

```
CHAIN OF CUSTODY FORM
═══════════════════════════════════════════════════════
Case Number: CASE-2026-001
Evidence Item: ITEM-001 (Samsung Galaxy S21)

ENTRY LOG:
─────────────────────────────────────────────────────
Date     | Time  | Action              | Person        | Signature
─────────────────────────────────────────────────────
17/07/26 | 14:30 | Seized at scene     | Inv. A. Sharma | _____
17/07/26 | 15:00 | Placed in Faraday   | Inv. A. Sharma | _____
17/07/26 | 16:45 | Transported to lab  | Inv. R. Verma  | _____
17/07/26 | 17:00 | Received at lab     | Tech. P. Singh | _____
17/07/26 | 17:15 | Imaging started     | Tech. P. Singh | _____
17/07/26 | 18:30 | Imaging complete    | Tech. P. Singh | _____
         |       | MD5:  [hash value]  |                |
         |       | SHA256: [hash value]|                |
17/07/26 | 18:45 | Original secured    | Lab Manager    | _____
18/07/26 | 09:00 | Analysis on copy    | Examiner S.K.  | _____
25/07/26 | 14:00 | Report submitted    | Examiner S.K.  | _____
28/07/26 | 10:00 | Presented to court  | Examiner S.K.  | _____
═══════════════════════════════════════════════════════
```

**Any gap in this chain = defense attorney challenges the evidence.**

---

#### Steps 09–10: Transfer to Lab & Secure Storage

- Evidence transported in **tamper-evident sealed packaging**
- Both the original AND the forensic copy go to the lab
- Original stored in **access-controlled evidence room**
  - Only lab administrator and authorized personnel can access
  - CCTV monitored
  - Temperature and humidity controlled (protects magnetic media)
  - Anti-static environment
- **Access log maintained** — every entry/exit recorded

---

#### Steps 11–13: Analyze, Report, Court

Covered in detail in Phases 3 and 4 below.

---

## 2. Bit-Stream Copy vs Regular Copy — Deep Dive

This is one of the most important concepts in forensics.

### Visual Explanation

```
ORIGINAL PEN DRIVE (10 GB total, 2 GB data):
┌─────────────────────────────────────────────────────┐
│ FILE A │ FILE B │ DELETED C │ SLACK │   FREE SPACE  │
│ 500MB  │ 1.5GB  │ (200MB)   │       │    7.8 GB     │
│ visible│ visible│ invisible!│ hidden│    "empty"    │
└─────────────────────────────────────────────────────┘
         Total: 10 GB

REGULAR COPY (cp / ctrl+c / drag-drop):
┌─────────────────────────────────────────────┐
│ FILE A │ FILE B │            EMPTY           │
│ 500MB  │ 1.5GB  │                            │
└─────────────────────────────────────────────┘
  Only 2GB visible files copied!
  ✗ DELETED FILE C — LOST (contains evidence!)
  ✗ SLACK SPACE — LOST (may contain fragments)
  ✗ ACCESS TIMESTAMPS MODIFIED (evidence tampering!)

BIT-STREAM FORENSIC IMAGE (dd / FTK Imager / dcfldd):
┌─────────────────────────────────────────────────────┐
│ FILE A │ FILE B │ DELETED C │ SLACK │   FREE SPACE  │
│ 500MB  │ 1.5GB  │ (200MB)   │       │    7.8 GB     │
│        │        │ RECOVERED!│ kept  │  may contain  │
│        │        │           │       │  fragments!   │
└─────────────────────────────────────────────────────┘
  ALL 10 GB copied, sector by sector!
  ✓ Deleted files recoverable (haven't been overwritten yet)
  ✓ Slack space preserved (file fragments)
  ✓ Unallocated space preserved
  ✓ Source drive NOT modified (read-only)
```

### Where Deleted Evidence Hides

```
File System Concepts (why bit-stream matters):

When you delete a file on most file systems:
  → File entry removed from directory listing (invisible)
  → Data NOT actually wiped from disk
  → Sectors marked as "available" but still contain original data
  → Data remains until overwritten by new files

Bit-stream copy captures the ENTIRE disk including:
  ┌─────────────────┬────────────────────────────────┐
  │ Allocated Space │ Active, visible files           │
  ├─────────────────┼────────────────────────────────┤
  │ Unallocated     │ Deleted files (not yet          │
  │ Space           │ overwritten) ← FORENSIC GOLD   │
  ├─────────────────┼────────────────────────────────┤
  │ Slack Space     │ Remnants at end of file         │
  │                 │ clusters (file fragments)       │
  ├─────────────────┼────────────────────────────────┤
  │ MFT / FAT       │ File system metadata            │
  │ (metadata)      │ timestamps, ownership, etc.     │
  └─────────────────┴────────────────────────────────┘
```

### Clone vs Bit-Stream — The Key Difference

| Feature             | Clone                        | Bit-Stream Forensic Image            |
| ------------------- | ---------------------------- | ------------------------------------ |
| What is copied      | Active data only             | Every single bit (sector by sector)  |
| Deleted files       | Lost                         | Preserved and recoverable            |
| Slack space         | Lost                         | Preserved                            |
| Free space          | Ignored                      | Copied (contains deleted data!)      |
| Source modification | May change timestamps        | Write blocker = zero changes         |
| Court admissible    | No (chain of custody broken) | Yes (with hash verification)         |
| Tool used           | OS copy function             | `dd`, `dcfldd`, FTK Imager, Guymager |
| Output format       | Files                        | `.img`, `.E01`, `.AFF` image file    |

### Forensic Image Formats

| Format                      | Extension             | Description                               | Used By      |
| --------------------------- | --------------------- | ----------------------------------------- | ------------ |
| **Raw**                     | `.img`, `.raw`, `.dd` | Simple bit-for-bit, no compression        | `dd`, dcfldd |
| **E01 (Expert Witness)**    | `.E01`                | Compressed, has built-in hash, metadata   | EnCase, FTK  |
| **AFF (Advanced Forensic)** | `.AFF`                | Open source, compressed                   | Older tools  |
| **AFF4**                    | `.AFF4`               | Modern open standard                      | AXIOM        |
| **L01**                     | `.L01`                | Logical image (files only, not full disk) | EnCase       |

---

## 3. First Responder Tools

Everything you need when you first arrive at a crime scene, before the full investigation begins.

### Complete First Responder Kit

```
FIRST RESPONDER KIT
═══════════════════════════════════════════════════════════

HARDWARE:
┌─────────────────────────────────────────────────────────┐
│ ✓ Write Blocker (hardware)                              │
│   → Prevents ANY write to evidence device               │
│   → Connects between evidence drive and forensic PC     │
│   → Examples: Tableau T35u, WiebeTech Forensic UltraDock│
│                                                         │
│ ✓ Faraday Bag / Signal Blocking Bag                     │
│   → Blocks all RF signals (WiFi, Bluetooth, 4G/5G, GPS) │
│   → Prevents remote wipe of phones                      │
│   → Prevents network-based evidence tampering           │
│   → Comes in phone size, laptop size, full-room size    │
│                                                         │
│ ✓ Anti-Static Bags                                      │
│   → Protects hard drives, SSDs from static discharge    │
│   → Human body carries static charge → damages magnetics│
│   → Use for any metallic/electronic component           │
│                                                         │
│ ✓ Drive Duplicator (hardware)                           │
│   → Standalone device — duplicates drives without a PC  │
│   → Makes bit-stream copy even if forensic PC unavailable│
│   → Has built-in hash generation                        │
│   → Example: Logicube Falcon, TD3 by CRU               │
│                                                         │
│ ✓ SIM Card Reader                                       │
│   → Reads SIM card contents directly                    │
│   → Extracts contacts, SMS stored on SIM                │
│   → ICCID, IMSI numbers for carrier subpoena            │
│                                                         │
│ ✓ Memory Card Reader                                    │
│   → SD, microSD, CompactFlash, etc.                     │
│   → Connect to forensic PC via write blocker            │
│                                                         │
│ ✓ Evidence Collecting Bags (Antistatic)                 │
│   → Multiple sizes                                      │
│   → Tamper-evident seal                                 │
│   → Printed label area for chain of custody info        │
│                                                         │
│ ✓ Camera / Phone (for scene documentation)              │
│   → Photograph everything before touching               │
│   → Video walk-through of scene                         │
└─────────────────────────────────────────────────────────┘

SOFTWARE (installed on forensic laptop):
┌─────────────────────────────────────────────────────────┐
│ ✓ Forensic Imaging Software                             │
│   → Creates bit-stream copies with hash verification    │
│   → Examples: FTK Imager (free), Guymager, dd/dcfldd   │
│                                                         │
│ ✓ Hash Calculator                                       │
│   → Generates MD5/SHA256 of evidence                    │
│   → Examples: HashCalc, md5sum/sha256sum (Linux)        │
│   → Built into FTK Imager and most forensic tools       │
│                                                         │
│ ✓ Write Blocker (software)                              │
│   → Software equivalent when hardware blocker unavailable│
│   → USB Write Blocker (Windows registry tweak or tool)  │
│   → Not as reliable as hardware — use hardware when able│
│                                                         │
│ ✓ Mobile Data Recovery Software                         │
│   → Dr. Fone, Oxygen Forensics, Cellebrite UFED        │
│   → Extracts call logs, messages, deleted data          │
│                                                         │
│ ✓ SIM Card Analysis Software                            │
│   → Reads SIM contents, carrier info                    │
│   → Extracts deleted SMS from SIM                       │
│                                                         │
│ ✓ RAM Capture Tool                                      │
│   → Winpmem (Windows), LiME (Linux)                    │
│   → Must be run FIRST on live systems                   │
└─────────────────────────────────────────────────────────┘
```

### Write Blocker — Critical Tool Explained

```
WITHOUT WRITE BLOCKER:
Evidence Drive → USB → Forensic PC
  OS automatically mounts drive
  Windows creates thumbnail cache (writes to drive!)
  Access timestamps update (modifies evidence!)
  Drive letter assigned (may alter partition tables)
  Result: EVIDENCE TAMPERED, INADMISSIBLE

WITH HARDWARE WRITE BLOCKER:
Evidence Drive → Write Blocker → USB → Forensic PC
  Write Blocker intercepts ALL write commands
  Read commands: allowed through
  Write commands: BLOCKED at hardware level
  Result: Evidence 100% unmodified, court admissible

Hardware Write Blocker Schematic:
  [Evidence Drive] ──▶ [Write Blocker]
                              │
                     Reads: ──┼──▶ [Forensic PC]
                              │
                     Writes: ─┴──✗ BLOCKED
```

### Faraday Bag — Why It Matters

```
Threat scenario:
  Criminal knows he's been arrested
  His accomplice sends remote wipe command to his phone
  → iCloud "Erase iPhone" triggered remotely
  → Google "Find My Device" → Erase triggered
  → MDM (Mobile Device Management) remote wipe
  → Result: All evidence DESTROYED

Faraday Bag solution:
  Phone placed in Faraday bag at arrest scene
  All signals blocked: WiFi, 4G/5G, Bluetooth, GPS, NFC
  Remote wipe command cannot reach the phone
  Evidence preserved

Signal Blocking Materials:
  ✓ Faraday bags (purpose-built, tested)
  ✓ Faraday cages (full rooms in lab environments)
  ✗ Microwave ovens (do NOT use — can damage device)
  ✗ Aluminum foil (not reliable, not admissible)
```

---

## 4. Four Phases of Investigation

```
┌────────────────┐   ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│    PHASE 1     │   │    PHASE 2     │   │    PHASE 3     │   │    PHASE 4     │
│    PRIMARY     │──▶│     FIRST      │──▶│  INVESTIGATION │──▶│     POST       │
│ INVESTIGATION  │   │   RESPONSE     │   │    PHASE       │   │ INVESTIGATION  │
│                │   │                │   │                │   │                │
│ Before scene   │   │ At the scene   │   │ In the lab     │   │ After analysis │
│ Lab setup,     │   │ Document,      │   │ Analyze copy,  │   │ Report, court  │
│ planning,      │   │ seize,         │   │ recover data,  │   │ testimony      │
│ budgeting      │   │ image, hash    │   │ find evidence  │   │                │
└────────────────┘   └────────────────┘   └────────────────┘   └────────────────┘
```

---

## 5. Phase 1: Primary Investigation — Lab Setup

Everything that happens **before** you go to a crime scene. Your lab must be ready.

### Planning & Budget

```
Budget considerations for a forensic lab:
  ┌─────────────────────────────────────────────────┐
  │ Personnel    │ Examiners, analysts, support      │
  │ Hardware     │ Workstations, write blockers,     │
  │              │ drive duplicators, storage        │
  │ Software     │ EnCase, FTK, Cellebrite licenses  │
  │ Lab space    │ Rent, physical security, utilities │
  │ Legal        │ Lab licensing, insurance           │
  │ Training     │ Certifications, courses            │
  │ Consumables  │ Evidence bags, labels, hard drives │
  └─────────────────────────────────────────────────┘
```

### Physical Lab Design Requirements

```
FORENSIC LAB ENVIRONMENT REQUIREMENTS:

Physical Security:
  ✓ Security guard at entrance
  ✓ CCTV cameras covering all areas (especially evidence room)
  ✓ Access control — key card / biometric entry
  ✓ Visitor log maintained at all times
  ✓ Located in secure area / building

Evidence Room (separate from work area):
  ✓ Access limited to authorized personnel only
  ✓ Temperature controlled (prevents media degradation)
  ✓ Low humidity (prevents corrosion)
  ✓ Anti-static flooring and surfaces
  ✓ Fire suppression system (NOT water — use gas/halon)
  ✓ Separate logs for every entry

Work Area:
  ✓ Proper lighting (examining small components)
  ✓ ESD (Electrostatic Discharge) workstations
  ✓ Anti-static mats and wrist straps
  ✓ Network connectivity (for tool updates, online resources)
  ✓ Air conditioning (keeps hardware cool during long imaging)
  ✓ Emergency power backup (UPS + generator)
    → Critical: If power fails mid-imaging = corrupted image
    → UPS must support ALL workstations simultaneously

Forensic Lab Licensing:
  ✓ Lab must be licensed to operate (varies by country/state)
  ✓ Without license: reports not accepted in court
  ✓ In India: Cyber forensics labs need MeitY recognition
  ✓ Regular audits and re-certification required
```

---

## 6. Hardware Requirements for Forensic Lab

```
MINIMUM HARDWARE SETUP FOR FORENSIC LAB:
══════════════════════════════════════════════════════

1. FORENSIC WORKSTATIONS (minimum 2)
   ─────────────────────────────────
   Why 2+? If one fails, investigation continues uninterrupted
   Specs (minimum recommended):
   • CPU: Intel i9 / AMD Ryzen 9 (high core count for parallel analysis)
   • RAM: 64 GB+ (Volatility/Autopsy are RAM-hungry)
   • GPU: Dedicated (hashcat for password cracking runs on GPU)
   • Storage: 4TB+ NVMe SSD (fast imaging speeds)
   • Multiple drive bays (SATA, SAS, NVMe, M.2)
   • USB 3.x + Thunderbolt ports
   • Isolated network interface (for safe malware analysis)

2. WRITE BLOCKERS (hardware)
   ─────────────────────────
   One per examiner workstation
   Supports: SATA, IDE, SAS, USB, Firewire, SD
   Examples: Tableau T35u, WiebeTech UltraDock, Logicube

3. DRIVE DUPLICATOR
   ─────────────────
   Standalone device — no PC needed
   Duplicates drive to drive or drive to image file
   Speed: 20-30 GB/min (much faster than software)
   Has LCD display showing progress and hash values

4. STORAGE INFRASTRUCTURE
   ──────────────────────
   • NAS (Network Attached Storage): 50TB+ for evidence storage
   • RAID configuration for redundancy
   • Backup system (3-2-1 rule: 3 copies, 2 media, 1 offsite)
   • Tape archive for long-term storage

5. CABLES & ADAPTERS (comprehensive set)
   ──────────────────────────────────────
   SATA to USB, IDE to USB, M.2 adapters,
   micro-USB, USB-C, Lightning (iOS devices),
   Firewire 400/800, Thunderbolt adapters

6. MEDIA STERILIZATION SYSTEM
   ───────────────────────────
   Sterilize new drives before using for imaging
   Ensures no previous data on the destination drive
   Tools: DBAN (Darik's Boot and Nuke), `dd if=/dev/zero`

7. PORTABLE FORENSIC SYSTEM (field kit)
   ──────────────────────────────────────
   Laptop + write blocker + portable drive duplicator
   For use at crime scene when lab is not reachable
   Examples: FRED-L (Field Forensic Recovery Evidence Device)

8. MOBILE DEVICE EXTRACTION HARDWARE
   ────────────────────────────────────
   Cellebrite UFED Touch 2 (industry standard)
   MSAB XRY (alternative)
   JTAG boxes (for deep extraction bypassing OS)
   Chip-off equipment (for destroyed devices)
```

---

## 7. Software Requirements for Forensic Lab

```
CATEGORY              TOOL                    PURPOSE
────────────────────────────────────────────────────────────────────
OS for Forensics      CAINE Linux             Purpose-built forensic OS (auto write-blocking)
                      Kali Linux              Broad tool support
                      SIFT Workstation        SANS forensic toolkit

Disk Imaging          FTK Imager              Free, industry-standard imaging
                      Guymager                Linux GUI imager
                      dd / dcfldd             Command-line imaging
                      EnCase Forensic         Enterprise platform

All-in-One Analysis   Autopsy + Sleuth Kit    Free, powerful, GUI-based
                      FTK (Forensic Toolkit)  Commercial, fast
                      EnCase                  Enterprise commercial
                      X-Ways Forensics        Lightweight, fast (commercial)
                      Axiom (Magnet)          User activity focus

Memory Analysis       Volatility 3            RAM dump analysis (free)
                      Rekall                  Alternative (less maintained)

Password Recovery     Hashcat                 GPU-accelerated cracking
                      John the Ripper         CPU-based cracking
                      Passware Kit            Commercial all-in-one

Data Recovery         Recuva                  Windows, free
                      PhotoRec                Cross-platform, open source
                      R-Studio                Commercial, powerful

Mobile Forensics      Cellebrite UFED         Industry standard (paid)
                      Oxygen Forensics        Comprehensive mobile analysis
                      Dr. Fone                Consumer tool, some forensic use
                      UFED Physical Analyzer  Physical extraction analysis

Network Analysis      Wireshark               Packet capture and analysis
                      NetworkMiner            File extraction from pcap
                      Zeek (Bro)              Network logging framework

Malware Analysis      Ghidra                  NSA's free reverse engineering tool
                      IDA Pro                 Industry standard (expensive)
                      Cuckoo Sandbox          Automated malware analysis
                      Any.run                 Online sandbox

Hash Calculation      HashCalc                Windows GUI hash tool
                      sha256sum / md5sum      Linux built-in

Report Writing        Magnet AXIOM Reports    Automated report generation
                      Autopsy HTML reports    Built-in report export
                      Microsoft Word          Manual report writing

Antivirus             Multiple vendors        Your forensic PC must be protected
                      (Isolated network)      Never analyze malware on internet-connected system
```

---

## 8. Phase 2: First Response at Crime Scene

### What to Do Step by Step When You Arrive

```
ARRIVAL AT CRIME SCENE — CHECKLIST:

□ 1. Do NOT touch any device immediately
□ 2. Photograph the entire scene from multiple angles
□ 3. Note: which devices are ON, which are OFF
□ 4. Note: network connections visible, cables, peripherals
□ 5. Photograph all device screens as-is (before anything changes)

If devices are ON:
□ 6. Capture RAM FIRST (volatile — disappears on power off)
□ 7. Note running processes, open applications
□ 8. Note network connections (active connections to C2 servers?)
□ 9. Note logged-in user accounts
□ 10. Screenshot / photo of everything on screen

Evidence collection:
□ 11. Mobile phones → Airplane mode or Faraday bag immediately
□ 12. Pack each item in separate labeled antistatic/evidence bag
□ 13. Document what was found where (sketch or photo of layout)
□ 14. Collect all cables, chargers, SIM cards, memory cards
□ 15. Check for hidden storage: USB ports, card readers, cloud

Documentation:
□ 16. Fill chain of custody form for each item
□ 17. Serial numbers of all devices
□ 18. MAC addresses of networking devices (if visible)
□ 19. Photo of each labeled evidence bag before sealing
□ 20. Seal bags with tamper-evident tape, sign across seal
```

### First Response Decision Tree

```
Device at scene
     │
     ├─ Is it ON? ──YES──▶ RAM dump first → screenshot → then decisions below
     │                         │
     │                         └─▶ Active ransomware encrypting?
     │                                 YES → Pull power (stop damage) → note exception
     │                                  NO → Capture RAM → controlled shutdown
     │
     └─ Is it OFF? ──▶ Leave OFF → pack in antistatic bag → transport to lab

Mobile phone
     │
     ├─ Is it ON? ──YES──▶ Enable airplane mode immediately
     │                         If can't unlock → Faraday bag
     │                         Don't let it sync to anything
     │
     └─ Is it OFF? ──▶ Faraday bag immediately
                           Do NOT power it on at scene
```

---

## 9. Phase 3: Investigation Phase

This happens **in the forensic lab**, working only on the **duplicate copy**.

### Investigation Workflow

```
Forensic Copy (bit-stream image)
         │
         ▼
┌─────────────────────────────────────────────────────┐
│  MOUNT IMAGE (read-only) in Autopsy / FTK           │
│  → File system parsed automatically                 │
│  → Deleted files indexed                            │
│  → Keyword search ready                             │
└──────────────────────────┬──────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  FILE        │   │   DELETED    │   │   METADATA   │
│  ANALYSIS    │   │   FILE       │   │   ANALYSIS   │
│              │   │   RECOVERY   │   │              │
│ Documents,   │   │ Recover from │   │ Timestamps,  │
│ images,      │   │ unallocated  │   │ EXIF data,   │
│ executables  │   │ space        │   │ author info  │
└──────────────┘   └──────────────┘   └──────────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
┌─────────────────────────────────────────────────────┐
│  TIMELINE CREATION                                  │
│  → Correlate all timestamps across all evidence     │
│  → When were files created, accessed, modified?     │
│  → When did attacker log in?                        │
│  → When were files deleted?                         │
└──────────────────────────┬──────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│  KEYWORD & PATTERN SEARCH                           │
│  → Search for victim name, attacker alias           │
│  → Search for malware signatures (YARA)             │
│  → Search for phone numbers, emails, IPs            │
│  → Search for financial info                        │
└──────────────────────────┬──────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│  ARTIFACT EXTRACTION                                │
│  → Browser history, cookies, downloads              │
│  → Email archives                                   │
│  → Chat logs (WhatsApp, Telegram backups)           │
│  → Recently accessed files list                     │
│  → Recycle Bin contents                             │
│  → Registry (Windows) — program execution, USB use  │
└─────────────────────────────────────────────────────┘
```

---

## 10. Phase 4: Post-Investigation & Reporting

### The Forensic Report Structure

```
FORENSIC REPORT TEMPLATE
═══════════════════════════════════════════════════════

SECTION 1: EXECUTIVE SUMMARY
  • Brief summary of findings (non-technical, for client/judge)
  • Conclusion: Was a crime committed? Who? When?

SECTION 2: CASE INFORMATION
  • Case number, client name, date assigned
  • Investigator name, qualifications, lab license number
  • Date of investigation

SECTION 3: EVIDENCE RECEIVED
  • List of all evidence items with:
    - Item number, description, condition on receipt
    - Hash values (MD5 + SHA256) at time of receipt
    - Chain of custody reference

SECTION 4: TOOLS USED
  • Every software tool: name, version, license
  • Every hardware tool: make, model, serial number
  • Why each tool was chosen (validated, court-accepted)

SECTION 5: METHODOLOGY
  • Step-by-step procedures followed
  • Why each step was taken
  • Any deviations from standard procedure and reasons

SECTION 6: FINDINGS
  • Detailed technical findings
  • Screenshots of evidence with captions
  • Timeline of events
  • Each finding numbered for reference in court

SECTION 7: CONCLUSION
  • Technical conclusion supporting or refuting allegations
  • Limitations of investigation
  • What could NOT be determined and why

SECTION 8: ANNEXURES
  • Hash verification logs
  • Tool output logs
  • Chain of custody forms
  • Lab certification documentation
```

### 65B Certificate (India-Specific)

In India, **Section 65B of the Indian Evidence Act** governs the admissibility of electronic records.

```
65B CERTIFICATE REQUIREMENTS:
  □ Must be signed by person in charge of the computer
    that produced the electronic record
  □ Must state:
    - The output was produced by the computer
    - The computer was in regular use during the relevant period
    - The computer was operating properly
    - The output reproduces the information stored
  □ Lab must issue this certificate along with forensic report
  □ Without 65B certificate → digital evidence not admissible
  □ 2021 amendment: person need not physically appear if
    certificate is provided (Arjun Panditrao case)
```

---

## 11. The Investigation Team — Roles

```
FORENSIC INVESTIGATION TEAM STRUCTURE:
════════════════════════════════════════

Role                  Responsibility
──────────────────────────────────────────────────────────────
PHOTOGRAPHER          Documents crime scene before any evidence
                      is touched. Photographs devices, cables,
                      room layout, everything as found.

INCIDENT RESPONDER    First technical person at scene.
                      Secures evidence, prevents tampering,
                      manages initial evidence seizure.

INCIDENT ANALYZER     Analyzes what type of incident occurred,
                      what damage was done, what type of crime.
                      Technical assessment at scene.

EVIDENCE EXAMINER     Performs detailed technical examination
                      of evidence in the lab. Runs forensic
                      tools, recovers data, finds artifacts.

EVIDENCE INVESTIGATOR Investigates significance of examined
                      evidence in context of the case.
                      Connects technical findings to the crime.

EVIDENCE DOCUMENTER   Maintains chain of custody documentation.
                      Records every action taken with evidence.

EVIDENCE MANAGER      Manages secure storage and access to
                      all evidence. Controls evidence room.

EXPERT WITNESS        Appears in court to explain technical
                      findings to judge and jury in simple terms.
                      Must withstand cross-examination.

ATTORNEY / LAWYER     Provides legal guidance on what evidence
                      is relevant, what questions to answer,
                      how to present findings legally.

NOTE: One person can hold multiple roles in smaller teams.
      In large government investigations, separate people per role.
```

---

## 12. Challenges in Digital Forensics

These are real obstacles you'll face as an investigator:

### 1. Speed

```
Challenge: Getting court permission takes time (government processes).
           Meanwhile, volatile evidence is being lost.

Reality:   A Meterpreter session will timeout.
           Malware may have a kill switch triggered by network change.
           Attacker may remotely wipe device.

Solution:  Emergency preservation orders in some jurisdictions.
           Document everything immediately on arrival.
           Prioritize volatile data capture before permissions are formally complete.
```

### 2. Anonymity (VPNs, Tor, Spoofing)

```
What attackers use:
  • VPN (hides real IP behind VPN server IP)
  • Tor (routes through multiple nodes, exit node visible)
  • Proxy chains
  • MAC address spoofing
  • Compromised third-party systems (attacking through victim's machine)
  • Burner phones, prepaid SIMs

How forensics fights back:
  • VPN providers keep logs (subpoena them)
  • Tor is not 100% anonymous (timing correlation attacks, exit node monitoring)
  • Malware command-and-control infrastructure has hosting providers
  • Behavioral analysis — attacker mistakes over time
  • OSINT — social media, leaked credentials link to real identity
  • Hardware fingerprinting — GPU fingerprinting, browser fingerprinting
```

### 3. Volatile Evidence

```
Evidence that DISAPPEARS when device is powered off:
  • RAM contents (all processes, decryption keys, network connections)
  • ARP cache (which IP = which MAC at time of attack)
  • Routing tables
  • Running process list
  • Open file handles
  • Clipboard contents
  • Mounted encrypted volumes (TrueCrypt/VeraCrypt containers open in RAM)

Mitigation:
  • ALWAYS capture RAM first on live systems
  • Use dedicated RAM capture tools (LiME, WinPmem)
  • Document network state before powering off
  • Never pull the plug without first running RAM dump
```

### 4. Evidence Size

```
Challenge: A company's servers may hold 100TB+ of data.
           Imaging 1TB = ~1-3 hours (depending on hardware)
           100TB = days of imaging + weeks of analysis

Solutions:
  • Targeted collection: identify relevant time periods and users first
  • Hash filtering: use hash databases (NSRL) to skip known clean OS files
  • Keyword filtering: only extract files matching relevant terms
  • Remote forensics: tools like F-Response mount evidence over network
  • Prioritize: what evidence is most likely to be dispositive?
```

### 5. Anti-Forensic Tools (covered in detail in Section 16)

### 6. Global Legal Complexity

```
Scenario: Attacker in Russia, C2 server in Netherlands,
          victim in India, payment in Cayman Islands.

Challenges:
  • Each country has different cyber laws
  • Mutual Legal Assistance Treaties (MLATs) required for cross-border data
  • MLAT requests take months to years
  • Some countries do not cooperate (Russia, North Korea)
  • GDPR (Europe) restricts data sharing — even for investigations

India-specific:
  • IT Act 2000 (amended 2008)
  • CERT-In coordination required for national-level incidents
  • Interpol for international cases
```

### 7. Encrypted Evidence

```
Strong encryption = forensic dead end (currently)
  • VeraCrypt containers with strong passphrase → uncrackable
  • Signal messages → end-to-end encrypted, Signal doesn't have keys
  • Monero cryptocurrency → designed to be untraceable
  • iOS full-disk encryption → Apple cannot unlock without passcode

Workarounds:
  • Keylogger installed before arrest → capture passphrase as typed
  • RAM forensics → encryption key may be in memory while volume is mounted
  • Legal compulsion → in some jurisdictions, refusing to decrypt = contempt of court
  • Metadata → even if content is encrypted, who communicated with whom, when,
    how often → still reveals patterns (traffic analysis)
  • Physical coercion (legal system) → court-ordered decryption
```

---

## 13. 65B Report — Court Admissibility (India)

```
SECTION 65B — WHAT IT COVERS:

65B(1): Any information contained in electronic record
        printed on paper or stored on optical/magnetic media
        is deemed a document if conditions in 65B(2) are met.

65B(2) Conditions:
  (a) Output was produced by a computer during regular use
  (b) Computer was regularly used in the activity being investigated
  (c) Computer was operating properly OR if not, didn't affect output
  (d) Information reproduced/derived from information fed to computer

65B(4) Certificate must state:
  • Identification of electronic record
  • Description of device that produced it
  • Details of the electronic record or a copy of it
  • Name of person responsible for the device
  • Signed by the responsible person

LANDMARK CASE: Arjun Panditrao Khotkar v. Kailash Kushanrao Gorantyal (2020)
  • Supreme Court: 65B certificate is MANDATORY for electronic evidence
  • Without it: evidence inadmissible
  • This applies to: WhatsApp messages, emails, CCTV footage, call records
```

---

## 14. Practical Labs

### Lab 1: Write Blocker — Software Method (Linux)

```bash
# Method 1: Mount evidence USB as read-only (basic protection)
# First identify the device (CRITICAL — wrong device = disaster):
lsblk
dmesg | tail -20    # See newly connected device

# Mount read-only (prevents accidental writes):
sudo mkdir /mnt/evidence
sudo mount -o ro,noexec,nosuid /dev/sdb1 /mnt/evidence
# ro = read-only
# noexec = cannot execute files from drive
# nosuid = cannot set user ID from drive

# Verify it's read-only:
touch /mnt/evidence/test.txt
# Should give: "Read-only file system" error

# Method 2: Block device write at kernel level (stronger):
sudo blockdev --setro /dev/sdb
# This makes the ENTIRE device read-only at block level
# Even dd cannot write to it now

# Verify:
blockdev --getro /dev/sdb    # Returns "1" = read-only, "0" = writable

# To restore writable:
sudo blockdev --setrw /dev/sdb
```

### Lab 2: Create Bit-Stream Image and Verify Hash

```bash
# 1. Prepare: identify evidence drive
lsblk
# Assume evidence is /dev/sdb (DOUBLE CHECK THIS!)

# 2. Block all writes to evidence drive:
sudo blockdev --setro /dev/sdb

# 3. Generate BEFORE hash (proves original state before imaging):
sudo sha256sum /dev/sdb | tee /evidence/before_imaging_hash.txt

# 4. Create forensic image:
sudo dcfldd if=/dev/sdb of=/evidence/case001_item001.img \
     hash=sha256 \
     hashlog=/evidence/case001_item001_hash.log \
     statusinterval=100

# 5. Verify image matches original:
sudo sha256sum /evidence/case001_item001.img | tee /evidence/after_imaging_hash.txt

# Compare:
cat /evidence/before_imaging_hash.txt
cat /evidence/after_imaging_hash.txt
# These MUST show matching hash values for the image to be valid!

# 6. Mount the image (read-only) for analysis:
sudo mkdir /mnt/forensic_image
sudo mount -o ro,loop,offset=1048576 /evidence/case001_item001.img /mnt/forensic_image
# offset=1048576 (512*2048) skips to first partition — adjust based on partitions

# Better: use Autopsy which handles this automatically
```

### Lab 3: Full Chain of Custody Practice

```bash
# Simulate a complete evidence collection process:

# 1. Create a "fake evidence" USB (using a file instead):
dd if=/dev/urandom of=/tmp/fake_evidence.img bs=1M count=100
# Create some "evidence" on it:
mkfs.ext4 /tmp/fake_evidence.img
mkdir /mnt/fake
mount -o loop /tmp/fake_evidence.img /mnt/fake
echo "This is evidence: attacker used IP 192.168.1.100" > /mnt/fake/notes.txt
echo "Password123" > /mnt/fake/password.txt
mkdir /mnt/fake/deleted
echo "I deleted this" > /mnt/fake/deleted/secret.txt
rm /mnt/fake/deleted/secret.txt  # "delete" it
umount /mnt/fake

# 2. Generate BEFORE hash (document original state):
sha256sum /tmp/fake_evidence.img > /evidence_lab/original_hash.txt
date >> /evidence_lab/original_hash.txt
echo "Investigator: $(whoami)" >> /evidence_lab/original_hash.txt

# 3. Create forensic bit-stream copy:
dcfldd if=/tmp/fake_evidence.img of=/evidence_lab/case_copy.img \
       hash=sha256 hashlog=/evidence_lab/hash_log.txt

# 4. Analyze the COPY in Autopsy (never the original!):
autopsy &
# Open browser to http://localhost:9999/autopsy
# New Case → Add image → select /evidence_lab/case_copy.img
# Run ingest modules → find all files including deleted secret.txt!

# 5. Verify original was not modified:
sha256sum /tmp/fake_evidence.img
# Must match hash in original_hash.txt
```

### Lab 4: Simulate Attack on Your Website → Forensic Investigation

```bash
# PHASE 1: ATTACK (on your own intentionally vulnerable PHP site)
# Enable logging first:
sudo a2enmod log_forensic
echo 'ForensicLog /var/log/apache2/forensic_log' >> /etc/apache2/apache2.conf
sudo service apache2 restart

# Use sqlmap against your own DVWA/vulnerable site:
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=your_session; security=low" \
       --technique=BEUSTQ --all --batch 2>&1 | tee /tmp/attack_log.txt

# PHASE 2: FORENSIC INVESTIGATION — What did the attack leave behind?

# 2a. Check Apache access log for sqlmap fingerprint:
grep -i "sqlmap\|User-Agent.*sqlmap" /var/log/apache2/access.log

# 2b. Look for SQL injection patterns:
grep -E "UNION|SELECT|information_schema|0x7[0-9a-f]|SLEEP|BENCHMARK" \
     /var/log/apache2/access.log | head -20

# 2c. Count requests per second (DoS-like pattern of sqlmap):
awk '{print $4}' /var/log/apache2/access.log | \
    tr -d '[' | cut -d: -f1,2,3 | sort | uniq -c | sort -rn | head

# 2d. Extract attacker IP:
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head

# 2e. Timeline reconstruction:
grep "127.0.0.1" /var/log/apache2/access.log | \
    awk '{print $4, $5, $6, $7}' | head -30

# 2f. MySQL query log (if enabled):
grep -i "UNION\|information_schema\|users" /var/log/mysql/mysql-general.log

# 2g. Write forensic report entry:
cat > /tmp/investigation_report.txt << EOF
Case: Web Application SQL Injection
Date: $(date)
Investigator: $(whoami)
Target: http://localhost/dvwa/

FINDINGS:
- Attack started at: $(grep "sqlmap" /var/log/apache2/access.log | head -1 | awk '{print $4}')
- Attack source IP: $(grep "sqlmap" /var/log/apache2/access.log | head -1 | awk '{print $1}')
- Tool identified: sqlmap (from User-Agent header)
- Attack type: SQL injection (UNION-based and time-based blind)
- Total requests: $(grep "127.0.0.1" /var/log/apache2/access.log | wc -l)
- Data accessed: information_schema, user tables (see attached log)

HASH VALUES:
Access log SHA256: $(sha256sum /var/log/apache2/access.log)
EOF
cat /tmp/investigation_report.txt
```

---

## 15. Attack Scenario → Full Investigation Walkthrough

### Scenario: Ransomware Attack on Small Company

```
SITUATION:
  Company discovers files are encrypted
  Ransom note: "Pay 1 BTC to decrypt"
  CEO calls you in as forensic investigator

STEP-BY-STEP RESPONSE:

[14:00] ARRIVE AT SCENE
  → Photograph everything as-is
  → Note: 3 workstations, 1 server, all appear to have encrypted files
  → Note: Server is still running (ransomware may still be active!)

[14:05] VOLATILE DATA CAPTURE (server is ON)
  → Rush: capture RAM before anything else
  → winpmem.exe -o \\192.168.1.50\evidence\server_ram.img
  → Note all running processes on screen (ransomware process visible?)
  → netstat -an → any active connections to C2 server?

[14:20] NETWORK ISOLATION
  → Disconnect all machines from network (prevent further spread)
  → But: capture netstat/arp first!
  → arp -a > network_state.txt (which IPs were talking to which MACs)

[14:30] CONTROLLED SHUTDOWN
  → Now power off machines systematically (most affected last)
  → Server last (still capturing its state)

[15:00] IMAGING
  → Write blockers on all machines
  → Begin bit-stream imaging (server first — most evidence there)
  → dcfldd for each drive → hash generated automatically

[17:00] TRANSPORT TO LAB
  → All devices in antistatic/evidence bags
  → Chain of custody forms filled for each item
  → Original evidence locked in secure storage

[NEXT DAY] LAB ANALYSIS
  RAM Analysis (Volatility):
  → vol -f server_ram.img windows.pslist | grep -v "System\|svchost\|explorer"
    → Find ransomware process (often random-named: xkjf83.exe)
  → vol -f server_ram.img windows.netscan
    → Find C2 IP address (ransomware calling home)
  → vol -f server_ram.img windows.cmdline
    → Find how ransomware was launched (phishing → document → macro?)

  Disk Analysis (Autopsy / FTK):
  → Timeline: when did encryption start?
  → First encrypted file timestamp = attack start time
  → Look backwards from that time: what ran?
    → Prefetch: vssadmin.exe (deleted shadow copies) ← smoking gun
    → Event logs: user logged in via RDP from unknown IP = entry point
    → Scheduled tasks: persistence mechanism?
    → Registry Run keys: auto-start entry?

  Email Investigation:
  → Outlook .PST file in disk image
  → Autopsy extracts emails automatically
  → Find phishing email with .docm attachment (macro-enabled Word)
  → Attachment had macro that downloaded ransomware

FINDINGS:
  Entry point: Phishing email → malicious Word macro → PowerShell → ransomware
  C2 server: 185.x.x.x (Russia-based VPS)
  Attacker dwell time: 3 days before encryption (exfiltrated data first!)
  Patient zero: finance department user opened attachment

REPORT:
  Timeline of attack, malware hash (YARA identification),
  C2 infrastructure, attacker TTPs, recommendations to prevent recurrence
```

---

## 16. Anti-Forensics vs Forensic Counter-Techniques

| Anti-Forensic Action                     | Attacker's Goal                    | Forensic Counter                                                                                  |
| ---------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------- |
| `wevtutil cl System`                     | Clear Windows Event Logs           | Check backup logs, remote syslog, event log metadata shows clearing event (Event ID 1102)         |
| `rm -rf /var/log/*`                      | Delete Linux logs                  | Remote syslog server, RAM dump of syslogd process, filesystem journal                             |
| `touch -t 202001010000 malware.exe`      | Change file timestamps             | `$MFT` entry timestamps often differ from `$STANDARD_INFORMATION`, journal entries, backup copies |
| `shred -uzn 35 file.txt`                 | Securely delete files              | RAM dump may contain file if recently accessed, backup/snapshot copies, network shares            |
| `history -c && unset HISTFILE`           | Clear bash history                 | RAM forensics of bash process (`vol linux.bash`), `.bash_history` backup in `/home/`              |
| File renamed to `svchost.exe`            | Disguise malware                   | Hash comparison with known-good NSRL database, PE header analysis                                 |
| VeraCrypt container                      | Encrypt evidence                   | Key capture from RAM (container mounted), legal compulsion in some jurisdictions                  |
| Tor / VPN routing                        | Hide origin IP                     | Timing analysis, exit node correlation, VPN provider subpoena, OSINT on accounts used             |
| Memory-only malware (fileless)           | Leave no disk artifacts            | RAM forensics is ONLY option, volatile data capture critical                                      |
| Steganography in images                  | Hide data in plain sight           | Stegdetect, statistical analysis, binwalk, file size anomalies                                    |
| VSS deletion (`vssadmin delete shadows`) | Remove recovery points             | This action itself is logged (Event ID 524), and is evidence of ransomware                        |
| `CCleaner` / artifact wipers             | Remove browser history, temp files | LNK files, Prefetch, UserAssist registry still records program execution                          |

```bash
# DETECTING ANTI-FORENSICS in your lab:

# 1. Check if event logs were cleared (Windows - in RAM dump or disk):
vol -f memory.dmp windows.evtlogs 2>/dev/null | grep "Security"
# Event ID 1102 = Security log cleared (requires admin rights = attacker had admin)
# Event ID 104  = System log cleared

# 2. Check for timestamp manipulation (Linux):
# Compare ctime (inode change) vs mtime (content modified):
stat /tmp/suspicious_file
# If mtime was changed artificially: ctime will be NEWER than mtime
# (you can't change ctime without root — it records when metadata changed)

# 3. Find files with mismatched extensions vs actual content:
file /tmp/*    # file command checks MAGIC BYTES, not extension
# Output: "/tmp/totally_safe.jpg: ELF 64-bit LSB executable" = renamed malware!

# 4. Find recently run programs even if deleted (Windows Prefetch):
# Autopsy → Extracted Content → Prefetch files
# Shows: program name, path, last run time, run count
# Even if attacker deleted the .exe, prefetch records its execution!

# 5. Check UserAssist registry (programs run by user):
# Even if file deleted, registry records execution
# Autopsy → OS Accounts → Recent Activity

# 6. Detect steganography:
apt install steghide stegdetect -y
stegdetect suspicious.jpg        # Statistical detection
steghide info suspicious.jpg     # Try to get info
binwalk suspicious.jpg           # Look for embedded files
exiftool suspicious.jpg          # Check metadata anomalies
```

---

## Quick Reference: Investigation Procedure at a Glance

```
CRIME SCENE ARRIVAL ORDER OF OPERATIONS:
─────────────────────────────────────────
1.  PHOTOGRAPH first — touch nothing yet
2.  Is any device ON?
    YES → RAM dump IMMEDIATELY (LiME / WinPmem)
    NO  → Skip to step 5
3.  Network state capture: netstat, arp, ifconfig/ipconfig
4.  Controlled power-off OR Faraday bag for mobile
5.  Evidence bags: antistatic for drives, Faraday for phones
6.  LABEL everything: case#, item#, who/when/where
7.  Fill chain of custody form for EACH item
8.  Transport to lab (signed off by both parties)
9.  Lab: bit-stream image + hash BEFORE analysis
10. NEVER touch original — work only on copy
11. Document every action taken
12. Report + 65B certificate + court appearance if needed

HASH = PROOF OF INTEGRITY
CHAIN OF CUSTODY = PROOF OF HANDLING
BIT-STREAM IMAGE = COMPLETE PRESERVATION
```

---

_Digital Forensics Part 2 — Investigation Procedures & Lab Setup_
_Cybersecurity Department Course Notes_
_Series: Architecture → Registers → Forensics Part 1 → Forensics Part 2 (this)_
_All techniques for authorized environments and personal lab use only_
