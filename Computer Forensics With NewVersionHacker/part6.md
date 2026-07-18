# Digital Forensics — Part 6

### Storage Interfaces, Data Recovery & Hash Verification

**Course:** Cyber Crime Investigation & Digital Forensics
**Topics:** HDD/SSD Interfaces, Data Recovery Methods, Forensic Hash Calculation
**For:** University Cybersecurity / Digital Forensics Students

---

## Table of Contents

1. [Why Interface Knowledge Matters for Forensics](#1-why-interface-knowledge-matters-for-forensics)
2. [Complete Storage Interface Reference](#2-complete-storage-interface-reference)
3. [Interface Evolution Timeline](#3-interface-evolution-timeline)
4. [Data Recovery — Core Concepts](#4-data-recovery--core-concepts)
5. [How Deleted Files Are Recovered](#5-how-deleted-files-are-recovered)
6. [Hash Verification in Forensics](#6-hash-verification-in-forensics)
7. [Forensic Data Recovery Tools](#7-forensic-data-recovery-tools)
8. [Comprehensive Lab Exercises](#8-comprehensive-lab-exercises)
9. [Exam Q&A](#9-exam-qa)
10. [Quick Reference Cheatsheet](#10-quick-reference-cheatsheet)

---

## 1. Why Interface Knowledge Matters for Forensics

When an investigator arrives at a crime scene, they encounter storage devices. Before imaging any device, they must:

```
Physical storage device found
        |
        v
Identify the interface type
        |
        v
Select correct write blocker for that interface
        |
        v
Select correct forensic imaging hardware/cables
        |
        v
Image the device without contaminating evidence
```

**Using the wrong write blocker or cable = cannot image the drive = evidence lost.**

Real cases involve old hardware — PATA, SCSI, and older interfaces are common in criminal investigations because criminals often use old, cheap, or discarded equipment. You must know all of them.

---

## 2. Complete Storage Interface Reference

### Category 1 — ATA Family (Older Interfaces)

**ATA = Advanced Technology Attachment** — the umbrella standard for connecting storage to computers.

#### PATA — Parallel ATA

```
Full name:  Parallel Advanced Technology Attachment
Also called: IDE (Integrated Drive Electronics)
             EIDE (Enhanced IDE)
Connector:  Wide ribbon cable — 40 or 80 pins
Width:      ~5cm wide flat ribbon cable
Color:      Usually gray or colored ribbon

Key specs:
├── 40-pin version (older IDE):  Max ~33 MB/s
├── 80-pin version (EIDE):       Max ~133 MB/s
├── Supports 2 drives per cable (Master / Slave)
├── Used in: Desktop PCs, older laptops, CD/DVD drives

Forensic relevance:
├── Very common in old crime scene computers
├── Requires PATA write blocker (NOT SATA)
├── Master/Slave jumper setting affects which drive you image
└── Still found in industrial systems and old servers
```

**Master/Slave Configuration (important for forensics):**

```
One PATA cable supports two drives:
├── Drive at END of cable   = Master (primary)
└── Drive at MIDDLE of cable = Slave (secondary)

Jumper pins on back of drive set which it is.
In forensics: Document jumper settings before touching anything.
```

#### EIDE — Enhanced Integrated Drive Electronics

```
Full name:  Enhanced Integrated Drive Electronics
Connector:  80-conductor ribbon cable (looks like PATA but more wires)
Speed:      Up to 133 MB/s (ATA-6 / ATA-7)
Advantage:  Faster than original 40-pin PATA
Disadvantage: Still parallel = signal interference at high speeds
Found in:   Late 1990s to early 2000s computers
```

---

### Category 2 — SATA (Current Standard)

```
Full name:  Serial Advanced Technology Attachment
Also:       Serial ATA, SATA I/II/III
Connector:  Small 7-pin data connector + 15-pin power
Cable:      Thin ~7mm wide flat cable (not ribbon)
Colors:     Usually red (data cable), black connector

Versions:
├── SATA I   (SATA 1.5Gb/s) → 150 MB/s   — 2003
├── SATA II  (SATA 3Gb/s)   → 300 MB/s   — 2004
└── SATA III (SATA 6Gb/s)   → 600 MB/s   — 2008 (current)

Compatible with: HDDs, 2.5" SSDs, laptop drives, optical drives

Forensic relevance:
├── MOST COMMON interface in current cases
├── SATA write blockers widely available
├── One device per cable (simpler than PATA)
├── Hot-swappable (can connect while power on — BUT DON'T in forensics)
└── Used in both HDDs and SSDs
```

---

### Category 3 — SCSI Family

```
Full name:  Small Computer System Interface
Pronounced: "Scuzzy"
Purpose:    High-performance interface for servers and workstations

Versions (many — speed increases with version):
├── SCSI-1       → 5 MB/s,   8-bit
├── SCSI-2       → 10 MB/s,  8-bit
├── Wide SCSI    → 20 MB/s,  16-bit
├── Ultra SCSI   → 20 MB/s
├── Ultra2 SCSI  → 80 MB/s
├── Ultra3 SCSI  → 160 MB/s
└── Ultra-640    → 640 MB/s

Key features:
├── Supports up to 15 devices on one cable (SCSI chain)
├── Each device has a SCSI ID (0–15)
├── Terminator required at end of SCSI chain
├── Common in servers, workstations, old NAS devices

SAS — Serial Attached SCSI (modern version):
├── Replaced parallel SCSI
├── Speed: 3, 6, 12 Gb/s
├── Used in enterprise servers
└── Backward compatible with SATA drives

Forensic relevance:
├── Found in server crime scenes
├── Requires SCSI write blocker
├── SCSI ID configuration important — document before touching
└── Termination issues can cause access problems
```

---

### Category 4 — PCIe / NVMe (Fastest Current)

```
Full name:  Peripheral Component Interconnect Express
Interface:  Connects directly to motherboard via PCIe lanes

PCIe versions:
├── PCIe 3.0 → ~3.9 GB/s per lane
├── PCIe 4.0 → ~7.9 GB/s per lane
└── PCIe 5.0 → ~15.7 GB/s per lane

NVMe (Non-Volatile Memory Express):
├── Protocol designed specifically for flash storage over PCIe
├── Replaces AHCI protocol used with SATA
├── Dramatically lower latency than SATA
├── M.2 NVMe SSDs: up to 7000+ MB/s (PCIe 4.0)

Form factors using PCIe/NVMe:
├── M.2 (2242, 2260, 2280 lengths) — most common laptop SSD
├── Add-in card (AIC) — desktop expansion card SSD
└── U.2 — enterprise NVMe drive

Forensic relevance:
├── Increasingly common in new cases
├── Requires NVMe write blocker (different from SATA)
├── Tableau TX1, Wiebetech NVMe are common forensic write blockers
├── M.2 drives can be SATA or NVMe — must identify before connecting
└── M.2 key type (M-key vs B-key vs B+M key) affects compatibility
```

---

### Category 5 — M.2 Interface (Form Factor)

M.2 is a **form factor** (physical size/shape), not an interface itself. An M.2 slot can support either SATA or NVMe protocol.

```
M.2 Key Types:
├── B-key  → Supports SATA and some PCIe x2
├── M-key  → Supports PCIe x4 NVMe (fastest)
└── B+M key → Fits both slots, usually SATA

Identifying M.2 type:
├── Check motherboard manual or slot label
├── Check SSD label (will say NVMe or SATA)
└── Windows Device Manager shows interface type

Forensic importance:
├── Do NOT assume all M.2 drives are NVMe
├── SATA M.2 needs SATA write blocker
├── NVMe M.2 needs NVMe write blocker
└── Using wrong write blocker = unable to image device
```

---

### Interface Comparison Table

| Interface     | Speed     | Connector     | Devices       | Era       | Common Use    |
| ------------- | --------- | ------------- | ------------- | --------- | ------------- |
| PATA (40-pin) | 33 MB/s   | Wide ribbon   | 2 per cable   | 1986–2000 | Old desktops  |
| EIDE (80-pin) | 133 MB/s  | Wide ribbon   | 2 per cable   | 1994–2006 | Late desktops |
| SATA I        | 150 MB/s  | Thin 7-pin    | 1 per cable   | 2003–     | HDDs, SSDs    |
| SATA II       | 300 MB/s  | Thin 7-pin    | 1 per cable   | 2004–     | HDDs, SSDs    |
| SATA III      | 600 MB/s  | Thin 7-pin    | 1 per cable   | 2008–     | HDDs, SSDs    |
| SCSI-1        | 5 MB/s    | 50-pin wide   | 8 per chain   | 1986–     | Servers       |
| SAS           | 12 Gb/s   | SAS connector | 15+ per chain | 2004–     | Servers       |
| PCIe 3.0 NVMe | 3500 MB/s | M.2 / PCIe    | 1 per slot    | 2013–     | Modern SSDs   |
| PCIe 4.0 NVMe | 7000 MB/s | M.2 / PCIe    | 1 per slot    | 2019–     | Fast SSDs     |

---

## 3. Interface Evolution Timeline

```
1986 ──── SCSI-1 (servers)
          ATA / IDE (desktops)

1994 ──── EIDE / Ultra-ATA (80-pin, faster desktops)

1996 ──── USB 1.1 (external storage, 12 Mb/s)

2000 ──── USB 2.0 (480 Mb/s)
          SCSI Ultra3 (servers, 160 MB/s)

2003 ──── SATA I  (replaces PATA in new builds)

2004 ──── SATA II, SAS (Serial Attached SCSI)

2008 ──── SATA III (600 MB/s, still current standard)
          USB 3.0 (5 Gb/s)

2013 ──── M.2 form factor introduced
          NVMe protocol (PCIe 3.0)

2019 ──── PCIe 4.0 NVMe (7000+ MB/s)
          USB 3.2 Gen 2x2 (20 Gb/s)

2022 ──── PCIe 5.0 NVMe (14000+ MB/s)
          USB4 (40 Gb/s)

Forensic reality:
Old crimes → old hardware → old interfaces (PATA, SCSI)
New crimes → new hardware → SATA, NVMe
You will encounter EVERYTHING — know it all.
```

---

## 4. Data Recovery — Core Concepts

### What Happens When You Delete a File?

```
User presses Delete:

FAT/NTFS File System Response:
├── File content remains in sectors on disk
├── Only the directory entry is marked "deleted"
├── First character of filename changed to special marker (0xE5 in FAT)
├── Sectors marked as "available" in allocation table
└── Data physically still present — until overwritten

Result:
├── File invisible to user through normal OS
├── File completely recoverable with forensic tools
├── Recovery window: hours to months (until sectors reused)
```

### Sector-by-Sector Scanning

Forensic recovery tools scan every sector of the drive looking for file signatures:

```
Every file type starts with a known signature (magic bytes):

File Type   | Header Signature (Hex)     | Footer
────────────|───────────────────────────|──────────────
JPEG image  | FF D8 FF                  | FF D9
PNG image   | 89 50 4E 47 0D 0A 1A 0A   | 49 45 4E 44 AE 42 60 82
PDF         | 25 50 44 46 2D            | 25 25 45 4F 46
ZIP file    | 50 4B 03 04               | 50 4B 05 06
EXE file    | 4D 5A                     | (variable)
MP4 video   | 00 00 00 xx 66 74 79 70   | (variable)
Word DOCX   | 50 4B 03 04 (ZIP-based)   | -
SQLite DB   | 53 51 4C 69 74 65 20 33   | -

Recovery tools scan RAW sectors for these signatures
→ Find header → read until footer → extract file
→ Works even when file system is damaged or deleted
→ Called "file carving"
```

### Recovery Possibilities by Scenario

| Scenario                               | Recovery Chance | Notes                                  |
| -------------------------------------- | --------------- | -------------------------------------- |
| File deleted, drive not written since  | Very High       | Sectors untouched                      |
| File deleted, drive heavily used since | Medium          | Some sectors overwritten               |
| Drive formatted (Quick Format)         | High            | Data still present, just tables reset  |
| Drive formatted (Full Format)          | Lower           | Sectors zeroed but carving may recover |
| File deleted on SSD with TRIM          | Very Low        | Sectors immediately erased             |
| Secure wipe (7-pass DoD)               | Near Zero       | Data overwritten multiple times        |
| Drive physically destroyed             | Zero            |                                        |

---

## 5. How Deleted Files Are Recovered

### The File Carving Process

```
Forensic image of drive
        |
        v
Tool reads sector 0, 1, 2, 3... (every sector)
        |
        v
For each sector: check if it matches any known file header
        |
        v
Header found (e.g., FF D8 FF = JPEG)?
        |
        v
Read forward until matching footer found
        |
        v
Extract those sectors as a file
        |
        v
Save to output folder
        |
        v
Continue scanning next sector
```

### File System Metadata Recovery

Beyond file carving, tools also analyze file system structures:

```
NTFS File System:
├── MFT (Master File Table) — database of all files
│   ├── Each file has an MFT record
│   ├── Record marked "not in use" when deleted
│   ├── Record content remains readable
│   └── Contains: filename, timestamps, size, data location
│
├── $MFT → parse deleted entries → recover filename, dates
├── $LogFile → transaction journal → see recent operations
└── $USNJrnl → change journal → history of file operations

FAT File System:
├── Directory entry first byte set to 0xE5 (deleted marker)
├── Remaining bytes intact → filename, size, cluster chain
└── FAT table entry zeroed → but data clusters untouched
```

---

## 6. Hash Verification in Forensics

### What is a Hash?

A hash is a **fixed-length fingerprint** generated from any data by a mathematical function. The same input always produces the same output. Even a single bit change produces a completely different hash.

```
Input: "Hello World"
MD5:    b10a8db164e0754105b7a99be72e3fe5
SHA1:   0a4d55a8d778e5022fab701977c5d840bbc486d0
SHA256: a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e

Change one character: "Hello world" (lowercase w)
MD5:    e59ff97941044f85df5297e1c302d260
SHA256: 64ec88ca00b268e5ba1a35678a1b5316d212f4f366b2477232534a8aeca37f3c

Completely different — even tiny changes produce entirely different hashes.
```

### Why Hash Verification is Critical in Forensics

```
Chain of Evidence:

1. Drive found at crime scene
        |
2. BEFORE imaging: Hash the original drive
   SHA256: abc123def456...
        |
3. Create forensic image
        |
4. Hash the image
   SHA256: abc123def456...  ← MUST MATCH original
        |
5. Work on image (never the original)
        |
6. At any time: re-hash original to prove no tampering
   SHA256: abc123def456...  ← Still matches? Original untouched
        |
7. Court: present matching hashes as proof of integrity

If hashes don't match at any step → evidence potentially contaminated
→ Chain of custody broken → evidence may be inadmissible
```

### Hash Algorithms Used in Forensics

| Algorithm | Output Length     | Current Status           | Use in Forensics                                 |
| --------- | ----------------- | ------------------------ | ------------------------------------------------ |
| MD5       | 128-bit (32 hex)  | Cryptographically broken | Still used for verification (not authentication) |
| SHA-1     | 160-bit (40 hex)  | Deprecated               | Phasing out                                      |
| SHA-256   | 256-bit (64 hex)  | Current standard         | Primary forensic hash                            |
| SHA-512   | 512-bit (128 hex) | Strong                   | High-security cases                              |

**In practice:** Generate BOTH MD5 and SHA-256. Courts accept this combination.

### Hash Verification Commands

```bash
# Generate SHA-256 hash of a drive (before imaging)
sudo sha256sum /dev/sdb > original_drive_hash.txt

# Generate SHA-256 hash of the forensic image
sha256sum evidence.img > image_hash.txt

# Compare — must be identical
diff original_drive_hash.txt image_hash.txt
# No output = identical = evidence integrity confirmed

# Generate MD5 also (belt-and-suspenders approach)
sudo md5sum /dev/sdb > original_md5.txt
md5sum evidence.img > image_md5.txt

# Verify a single file
sha256sum recovered_file.jpg

# Hash an entire folder of recovered files
sha256sum /output/recovered/* > all_recovered_hashes.txt

# Check specific hash value
echo "a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e  evidence.img" | sha256sum -c
# Output: evidence.img: OK
```

---

## 7. Forensic Data Recovery Tools

### Free / Open Source Tools

| Tool               | Platform       | Type | Best For                                      |
| ------------------ | -------------- | ---- | --------------------------------------------- |
| **Autopsy**        | Win/Linux/Mac  | GUI  | Complete forensic analysis, file recovery     |
| **TestDisk**       | Cross-platform | CLI  | Partition recovery, deleted file recovery     |
| **PhotoRec**       | Cross-platform | CLI  | File carving, deleted photo/document recovery |
| **Foremost**       | Linux          | CLI  | File carving by header/footer                 |
| **Scalpel**        | Linux          | CLI  | Advanced file carving                         |
| **Bulk Extractor** | Cross-platform | CLI  | Extract email, URLs, credit cards from images |
| **Recuva**         | Windows        | GUI  | Consumer-friendly file recovery               |
| **Volatility**     | Cross-platform | CLI  | RAM dump analysis                             |
| **dc3dd**          | Linux          | CLI  | Forensic imaging with hashing                 |
| **Guymager**       | Linux          | GUI  | Forensic disk imaging                         |

### Commercial Tools (Industry Standard)

| Tool                       | Company          | Notes                                |
| -------------------------- | ---------------- | ------------------------------------ |
| **FTK (Forensic Toolkit)** | AccessData       | Most widely used in law enforcement  |
| **EnCase**                 | OpenText         | Court-accepted, enterprise forensics |
| **Magnet AXIOM**           | Magnet Forensics | Excellent for mobile + computer      |
| **X-Ways Forensics**       | X-Ways           | Fast, efficient, less expensive      |
| **Cellebrite UFED**        | Cellebrite       | Mobile device forensics leader       |
| **Oxygen Forensic**        | Oxygen           | Mobile and cloud forensics           |
| **R-Studio**               | R-Tools          | Excellent data recovery              |
| **GetDataBack**            | Runtime Software | Consumer/professional recovery       |

### SSD-Specific Considerations

```
Standard tools work on SSD images but:

1. TRIM-deleted data → Not recoverable through software
2. Wear leveling → Physical location ≠ logical address
3. Chip-off required for:
   ├── Severely damaged SSDs
   ├── Hardware-encrypted drives (after secure erase)
   └── Locked/password-protected controllers

Chip-off tools:
├── PC-3000 Flash (top commercial tool)
├── NAND Flash programmer + custom FTL reversal
└── Requires specialist hardware + training
```

---

## 8. Comprehensive Lab Exercises

### Lab 1 — Identify All Storage Interfaces on Your System

```bash
# Linux — identify all storage devices and their interfaces
sudo lshw -class disk
# Shows: logical name, product, description, interface

# More detailed
sudo lshw -class disk -class storage

# Check specific drive interface
sudo hdparm -I /dev/sda | head -30
# Look for: "Transport:" line (SATA, ATA, etc.)

# For NVMe
sudo nvme list
sudo nvme id-ctrl /dev/nvme0

# USB drives
lsusb
# or
udevadm info --query=all --name=/dev/sdb | grep -i "id\|speed\|version"

# Check all block devices with interface info
lsblk -o NAME,TYPE,TRAN,SIZE,MODEL,VENDOR
# TRAN column: sata, nvme, usb, ata

# Windows equivalent (PowerShell)
# Get-PhysicalDisk | Select FriendlyName, MediaType, BusType, Size
```

**Document for each drive found:**

- Device name (/dev/sda, /dev/nvme0, etc.)
- Interface type (SATA, NVMe, USB)
- Drive model
- Capacity
- Rotational (HDD) or not (SSD)

---

### Lab 2 — Forensic Imaging with Hash Verification

```bash
# SETUP: Use a spare USB drive as your "evidence" drive
# DO NOT use your system drive

# Step 1: Identify your test USB drive
lsblk
# Note: sdb = your test USB, DO NOT mix up sda (system) and sdb (USB)

# Step 2: Hash the drive BEFORE imaging (baseline)
sudo sha256sum /dev/sdb | tee baseline_hash.txt
sudo md5sum /dev/sdb | tee baseline_md5.txt
echo "Baseline hash recorded at $(date)" >> baseline_hash.txt

# Step 3: Create forensic image with dc3dd
sudo dc3dd if=/dev/sdb \
  hof=/tmp/evidence.img \
  hash=sha256 \
  log=/tmp/acquisition_log.txt \
  bs=4096

# Step 4: Verify image hash matches original
sha256sum /tmp/evidence.img | tee image_hash.txt

# Step 5: Compare baseline vs image hash
# Extract just the hash values for comparison
BASELINE=$(cat baseline_hash.txt | awk '{print $1}')
IMAGE=$(cat image_hash.txt | awk '{print $1}')

if [ "$BASELINE" = "$IMAGE" ]; then
  echo "VERIFIED: Image matches original drive"
  echo "Chain of custody: INTACT"
else
  echo "ERROR: Hash mismatch — image does not match original"
  echo "Chain of custody: BROKEN"
fi

# Step 6: Document everything
cat > acquisition_report.txt << EOF
=== Forensic Acquisition Report ===
Date/Time: $(date)
Examiner: Your Name
Case: Lab Exercise
Evidence: /dev/sdb (USB Drive)
Destination: /tmp/evidence.img

Original Drive Hash (SHA-256):
$(cat baseline_hash.txt)

Forensic Image Hash (SHA-256):
$(cat image_hash.txt)

Hash Verification: $([ "$BASELINE" = "$IMAGE" ] && echo "PASSED" || echo "FAILED")

Acquisition Log:
$(cat /tmp/acquisition_log.txt)
EOF

cat acquisition_report.txt
```

---

### Lab 3 — Deleted File Recovery with PhotoRec

```bash
# Step 1: Create test data on your USB drive
sudo mkdir -p /mnt/test_usb
sudo mount /dev/sdb1 /mnt/test_usb

# Create test files
echo "Secret document content for recovery test" > /mnt/test_usb/test_document.txt
cp /usr/share/pixmaps/*.png /mnt/test_usb/ 2>/dev/null || true

# Step 2: Delete the files
rm /mnt/test_usb/test_document.txt
rm /mnt/test_usb/*.png 2>/dev/null || true
sync
sudo umount /mnt/test_usb

# Step 3: Verify files are "gone" from normal access
sudo mount /dev/sdb1 /mnt/test_usb
ls /mnt/test_usb/  # Files should NOT appear
sudo umount /mnt/test_usb

# Step 4: Recover with PhotoRec
sudo apt install testdisk  # PhotoRec comes with testdisk package

sudo mkdir -p /tmp/recovered
sudo photorec /dev/sdb  # Interactive tool
# In menu: Select partition → File system type → Choose output folder /tmp/recovered
# Or use non-interactive mode:

# Non-interactive PhotoRec
sudo photorec /d /tmp/recovered/ /dev/sdb

# Step 5: Check recovered files
ls -la /tmp/recovered/recup_dir*/
find /tmp/recovered -name "*.txt" -exec cat {} \;
find /tmp/recovered -name "*.png" | head -5

echo "Files recovered: $(find /tmp/recovered -type f | wc -l)"
```

---

### Lab 4 — File Carving with Foremost

```bash
# Install foremost
sudo apt install foremost

# Create a forensic image first
sudo dd if=/dev/sdb of=/tmp/usb_image.img bs=4M status=progress

# Carve for specific file types
sudo mkdir -p /tmp/foremost_output

# Carve for JPEGs and PNGs only
sudo foremost -t jpg,png -i /tmp/usb_image.img -o /tmp/foremost_output/

# Carve for ALL supported types
sudo foremost -t all -i /tmp/usb_image.img -o /tmp/foremost_output_all/

# Check the audit log
cat /tmp/foremost_output/audit.txt

# View recovered files
ls -la /tmp/foremost_output/jpg/
ls -la /tmp/foremost_output/png/

# Count total recovered
find /tmp/foremost_output -type f -not -name "audit.txt" | wc -l
```

---

### Lab 5 — Hash Calculation on Recovered Files

```bash
# After recovering files, hash everything for evidence integrity

# Hash a single file
sha256sum /tmp/recovered/file.jpg

# Hash all recovered files and save to evidence manifest
find /tmp/recovered -type f | while read file; do
  sha256sum "$file"
done > /tmp/evidence_manifest.txt

echo "Evidence Manifest created: $(wc -l < /tmp/evidence_manifest.txt) files"
cat /tmp/evidence_manifest.txt | head -20

# Verify a specific file later hasn't changed
sha256sum -c /tmp/evidence_manifest.txt 2>&1 | grep -v "OK$"
# Any output here means a file changed — integrity violated

# Create MD5 manifest too
find /tmp/recovered -type f | xargs md5sum > /tmp/evidence_manifest_md5.txt

# Hash the manifests themselves
sha256sum /tmp/evidence_manifest.txt /tmp/evidence_manifest_md5.txt
```

---

### Lab 6 — Autopsy (Full Forensic Analysis)

Autopsy is the standard GUI forensic tool — free, open source, used by law enforcement.

```bash
# Install Autopsy (Linux)
sudo apt install autopsy

# OR download from: https://www.sleuthkit.org/autopsy/
# (Better to use the Windows version for GUI)

# Start Autopsy (Linux text-based web interface)
sudo autopsy
# Open browser: http://localhost:9999/autopsy

# CREATE A CASE:
# 1. New Case
# 2. Enter case name, investigator name, description
# 3. Add image (your forensic image .img file)
# 4. Select analysis modules:
#    ✓ Recent Activity
#    ✓ Hash Lookup
#    ✓ File Type Identification
#    ✓ Extension Mismatch Detector
#    ✓ Embedded File Extractor
#    ✓ Keyword Search
#    ✓ Email Parser
#    ✓ Interesting Files Identifier
#    ✓ PhotoRec Carver
# 5. Run ingest

# ANALYZE RESULTS:
# - Views → File Types → Images (find all photos)
# - Views → Deleted Files (all deleted files found)
# - Results → Keyword Hits (if keywords configured)
# - Data Artifacts → Web History, Bookmarks, Downloads
# - Data Artifacts → Recent Documents
# - Timeline (visual timeline of all file activity)
```

---

### Lab 7 — String Searching in Disk Image

```bash
# Search for specific text in a disk image (without mounting)
# Useful for finding evidence when you know a keyword

# Search for email addresses
strings /tmp/usb_image.img | grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"

# Search for URLs
strings /tmp/usb_image.img | grep -E "https?://"

# Search for specific keyword
strings /tmp/usb_image.img | grep -i "password" | head -20

# Search for credit card patterns
strings /tmp/usb_image.img | grep -E "[0-9]{4}[-\s]?[0-9]{4}[-\s]?[0-9]{4}[-\s]?[0-9]{4}"

# Search for IP addresses
strings /tmp/usb_image.img | grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"

# Extract all readable strings and save
strings /tmp/usb_image.img > /tmp/all_strings.txt
wc -l /tmp/all_strings.txt

# Search the strings file for multiple keywords
grep -i "confidential\|secret\|password\|private" /tmp/all_strings.txt
```

---

### Lab 8 — Bulk Extractor (Automated Evidence Extraction)

```bash
# Install bulk_extractor
sudo apt install bulk-extractor

# Run bulk_extractor on forensic image
sudo mkdir -p /tmp/bulk_output
sudo bulk_extractor -o /tmp/bulk_output /tmp/usb_image.img

# View what was automatically extracted
ls /tmp/bulk_output/
# email.txt      → All email addresses found
# url.txt        → All URLs found
# domain.txt     → All domain names found
# telephone.txt  → Phone numbers
# ccn.txt        → Credit card numbers
# bitcoin.txt    → Bitcoin addresses
# histogram files → Frequency analysis

# Review emails found
cat /tmp/bulk_output/email.txt | head -20

# Review URLs
cat /tmp/bulk_output/url.txt | head -20

# Wordlist from image (useful for password attacks in authorized pentests)
cat /tmp/bulk_output/wordlist.txt | head -20
```

---

### Lab 9 — Complete Forensic Workflow (End-to-End)

```bash
#!/bin/bash
# Complete forensic workflow script
# Replace /dev/sdb with your test drive

EVIDENCE_DRIVE="/dev/sdb"
CASE_DIR="/tmp/forensic_case_$(date +%Y%m%d_%H%M%S)"
EXAMINER="Your Name"
CASE_NUMBER="LAB-001"

mkdir -p "$CASE_DIR"/{images,recovered,hashes,reports,bulk}

echo "=== FORENSIC CASE: $CASE_NUMBER ===" | tee "$CASE_DIR/case_log.txt"
echo "Examiner: $EXAMINER" | tee -a "$CASE_DIR/case_log.txt"
echo "Date: $(date)" | tee -a "$CASE_DIR/case_log.txt"
echo "Evidence: $EVIDENCE_DRIVE" | tee -a "$CASE_DIR/case_log.txt"
echo "" | tee -a "$CASE_DIR/case_log.txt"

# STEP 1: Document the evidence drive
echo "[STEP 1] Documenting evidence drive..." | tee -a "$CASE_DIR/case_log.txt"
sudo hdparm -I "$EVIDENCE_DRIVE" > "$CASE_DIR/reports/drive_info.txt" 2>/dev/null
sudo smartctl -a "$EVIDENCE_DRIVE" > "$CASE_DIR/reports/smart_data.txt" 2>/dev/null
lsblk -o NAME,TYPE,SIZE,FSTYPE,LABEL "$EVIDENCE_DRIVE" >> "$CASE_DIR/case_log.txt"

# STEP 2: Hash original evidence
echo "[STEP 2] Hashing original evidence..." | tee -a "$CASE_DIR/case_log.txt"
sudo sha256sum "$EVIDENCE_DRIVE" | tee "$CASE_DIR/hashes/original_sha256.txt"
sudo md5sum "$EVIDENCE_DRIVE" | tee "$CASE_DIR/hashes/original_md5.txt"

# STEP 3: Create forensic image
echo "[STEP 3] Creating forensic image..." | tee -a "$CASE_DIR/case_log.txt"
sudo dc3dd if="$EVIDENCE_DRIVE" \
  hof="$CASE_DIR/images/evidence.img" \
  hash=sha256 \
  log="$CASE_DIR/reports/acquisition_log.txt" \
  bs=4096 2>&1 | tee -a "$CASE_DIR/case_log.txt"

# STEP 4: Verify image hash
echo "[STEP 4] Verifying image integrity..." | tee -a "$CASE_DIR/case_log.txt"
sha256sum "$CASE_DIR/images/evidence.img" | tee "$CASE_DIR/hashes/image_sha256.txt"
ORIG=$(awk '{print $1}' "$CASE_DIR/hashes/original_sha256.txt")
IMG=$(awk '{print $1}' "$CASE_DIR/hashes/image_sha256.txt")
if [ "$ORIG" = "$IMG" ]; then
  echo "HASH VERIFICATION: PASSED" | tee -a "$CASE_DIR/case_log.txt"
else
  echo "HASH VERIFICATION: FAILED - STOP INVESTIGATION" | tee -a "$CASE_DIR/case_log.txt"
  exit 1
fi

# STEP 5: File carving on image
echo "[STEP 5] Carving deleted files..." | tee -a "$CASE_DIR/case_log.txt"
sudo foremost -t all -i "$CASE_DIR/images/evidence.img" -o "$CASE_DIR/recovered/" 2>/dev/null

# STEP 6: Automated evidence extraction
echo "[STEP 6] Running bulk_extractor..." | tee -a "$CASE_DIR/case_log.txt"
sudo bulk_extractor -o "$CASE_DIR/bulk/" "$CASE_DIR/images/evidence.img" 2>/dev/null

# STEP 7: Hash all recovered files
echo "[STEP 7] Hashing recovered evidence..." | tee -a "$CASE_DIR/case_log.txt"
find "$CASE_DIR/recovered" -type f | while read f; do
  sha256sum "$f"
done > "$CASE_DIR/hashes/recovered_files_manifest.txt"

# STEP 8: Summary report
echo "" | tee -a "$CASE_DIR/case_log.txt"
echo "=== CASE SUMMARY ===" | tee -a "$CASE_DIR/case_log.txt"
echo "Image integrity: VERIFIED" | tee -a "$CASE_DIR/case_log.txt"
echo "Recovered files: $(find "$CASE_DIR/recovered" -type f | wc -l)" | tee -a "$CASE_DIR/case_log.txt"
echo "Emails found: $(wc -l < "$CASE_DIR/bulk/email.txt" 2>/dev/null || echo 0)" | tee -a "$CASE_DIR/case_log.txt"
echo "URLs found: $(wc -l < "$CASE_DIR/bulk/url.txt" 2>/dev/null || echo 0)" | tee -a "$CASE_DIR/case_log.txt"
echo "Case directory: $CASE_DIR" | tee -a "$CASE_DIR/case_log.txt"
echo "Completed: $(date)" | tee -a "$CASE_DIR/case_log.txt"
```

---

## 9. Exam Q&A

### Q1: What is the difference between PATA and SATA?

**PATA (Parallel ATA):** Uses a wide 40 or 80-pin ribbon cable, connects up to 2 drives per cable (master/slave), max speed ~133 MB/s, old technology found in pre-2006 computers.
**SATA (Serial ATA):** Uses a thin 7-pin data cable, connects one drive per cable, max speed 600 MB/s (SATA III), current standard in HDDs and 2.5" SSDs. SATA is serial transmission (one bit at a time in a stream) vs PATA's parallel transmission (multiple bits simultaneously on parallel wires).

### Q2: What is the full form of SCSI and where is it found forensically?

SCSI = Small Computer System Interface. It is found in older servers and workstations. It supports a chain of up to 15 devices with each device having a unique SCSI ID. Forensically, SCSI is important because server-related crimes (data theft, hacking infrastructure) often involve SCSI drives. SAS (Serial Attached SCSI) is its modern successor used in current enterprise servers.

### Q3: What is PCIe NVMe and why is it forensically different from SATA?

PCIe NVMe (Peripheral Component Interconnect Express / Non-Volatile Memory Express) is the fastest current SSD interface, connecting directly to the motherboard via PCIe lanes. It is forensically different because: it requires NVMe-specific write blockers (SATA write blockers don't work), it uses the NVMe protocol over PCIe (not AHCI), it is dramatically faster, and M.2 form factor drives may be either SATA or NVMe requiring identification before connecting.

### Q4: What happens to a file when it is deleted from an HDD?

When deleted from an HDD: the file's directory entry is marked with a deletion marker (0xE5 in FAT), the file's clusters are marked as "available" in the file allocation table, but the actual data in those sectors remains physically unchanged on the disk. The file is invisible to the OS but recoverable with forensic tools until the sectors are overwritten by new data.

### Q5: What is file carving and how does it work?

File carving is the recovery of files based on their content structure, without relying on file system metadata. Tools scan every sector of a disk image looking for known file signatures (magic bytes) — for example, JPEGs start with FF D8 FF. When a header is found, the tool reads forward until it finds the matching footer, then extracts those bytes as a file. File carving works even when the file system is damaged, formatted, or deleted.

### Q6: What is hash verification and why is it essential in digital forensics?

Hash verification is the process of generating a cryptographic hash (MD5, SHA-256) of evidence before and after any operation to prove the data has not been modified. It is essential because: courts require proof that evidence has not been tampered with, it confirms forensic images are exact copies of original drives, and any change to the data (even one bit) produces a completely different hash, making tampering detectable. Without hash verification, evidence may be inadmissible in court.

### Q7: Why must a forensic investigator never work directly on original evidence?

Working on the original evidence risks modifying it — even mounting a drive causes the OS to write timestamps, temp files, and metadata. Any modification breaks the chain of custody and can make evidence inadmissible. The correct procedure is: hash the original, create a forensic image (verified by hash), then perform all analysis on the image. The original is sealed and stored untouched.

### Q8: What is the Master/Slave configuration in PATA drives?

PATA supports two drives on one ribbon cable. The "Master" drive is connected at the end of the cable and the "Slave" at the middle connector. Small jumper pins on the back of each drive set which role it plays. In forensics, documenting and photographing the jumper settings before any intervention is important because misidentifying which drive is which could lead to imaging the wrong device.

### Q9: Name five forensic data recovery tools and classify them.

Free/Open Source: Autopsy (GUI, complete forensic suite), TestDisk/PhotoRec (CLI, partition and file recovery), Foremost (CLI, file carving), Scalpel (CLI, advanced carving), Bulk Extractor (CLI, automated evidence extraction).
Commercial: FTK Forensic Toolkit (law enforcement standard), EnCase (court-accepted), Magnet AXIOM (computer + mobile), X-Ways Forensics (efficient, professional), R-Studio (data recovery specialist).

### Q10: What is the significance of a "sector-by-sector" scan in data recovery?

A sector-by-sector scan examines every logical sector of a storage device sequentially, regardless of what the file system says is used or free. This is essential because: deleted files have their file system entries marked as available but data remains in sectors, formatted drives still have data in sectors, and file carving can only work if every sector is examined. Skipping sectors based on file system allocation tables would miss evidence.

---

## 10. Quick Reference Cheatsheet

### Interface Identification

```bash
sudo lshw -class disk                    # All disk hardware info
lsblk -o NAME,TYPE,TRAN,SIZE,MODEL       # TRAN = interface type
sudo hdparm -I /dev/sda | grep Transport # Interface type of sda
sudo nvme list                           # List NVMe drives
udevadm info /dev/sdb | grep ID_BUS     # USB bus type
```

### Disk Imaging

```bash
sudo dd if=/dev/sdb of=image.img bs=4M status=progress
sudo dc3dd if=/dev/sdb hof=image.img hash=sha256 log=acq.log bs=4096
sudo guymager                             # GUI imaging
```

### Hash Verification

```bash
sha256sum file_or_drive                  # Generate SHA-256
md5sum file_or_drive                     # Generate MD5
sha256sum -c hashes.txt                  # Verify all files in manifest
diff hash_before.txt hash_after.txt      # Compare two hash files
```

### File Recovery

```bash
sudo photorec /dev/sdb                   # Interactive recovery
sudo foremost -t all -i img.img -o out/  # File carving all types
sudo foremost -t jpg,pdf -i img.img -o out/  # Specific types
sudo scalpel img.img -o out/             # Advanced carving
strings img.img | grep "keyword"         # Text search in image
```

### Evidence Extraction

```bash
sudo bulk_extractor -o output/ image.img # Auto extract email, URLs, etc.
cat output/email.txt                      # Found email addresses
cat output/url.txt                        # Found URLs
cat output/ccn.txt                        # Credit card numbers found
```

### Hash Manifest

```bash
# Create manifest of all recovered files
find /recovered -type f | xargs sha256sum > manifest.txt

# Verify manifest
sha256sum -c manifest.txt
# Lines ending in OK = unchanged
# Any FAILED lines = file modified
```

---

## Interface Quick Reference Card

```
Interface     Full Name                              Speed          Era
──────────────────────────────────────────────────────────────────────────
PATA (IDE)    Parallel Advanced Technology Attachment  133 MB/s    1986–2006
EIDE          Enhanced Integrated Drive Electronics    133 MB/s    1994–2006
SATA I        Serial Advanced Technology Attachment    150 MB/s    2003–
SATA II       Serial Advanced Technology Attachment    300 MB/s    2004–
SATA III      Serial Advanced Technology Attachment    600 MB/s    2008–
SCSI-1        Small Computer System Interface          5 MB/s      1986–
SAS           Serial Attached SCSI                     12 Gb/s     2004–
PCIe 3 NVMe   Peripheral Component Interconnect Exp.  3500 MB/s   2013–
PCIe 4 NVMe   Peripheral Component Interconnect Exp.  7000 MB/s   2019–
USB 2.0       Universal Serial Bus                     60 MB/s     2000–
USB 3.0       Universal Serial Bus                     625 MB/s    2008–
```

---

_Digital Forensics Part 6 | Next: Part 7 — Hash Calculation Deep Dive, Chain of Custody, and Court Reporting_
_Practice: Autopsy tutorial | TryHackMe: Digital Forensics room | Sleuth Kit documentation_
