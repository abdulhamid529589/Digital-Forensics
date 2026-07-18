# Digital Forensics — Part 5

### Storage Media Forensics: SSD Architecture & RAM Analysis

**Course:** Cyber Crime Investigation & Digital Forensics
**Topic:** Understanding SSDs, NAND Flash, RAM Forensics, Storage Architecture
**For:** University Cybersecurity / Digital Forensics Students

---

> **Note on Becoming a Digital Forensics Investigator:**
> A single video or a week of study cannot make you a forensic investigator. The field demands mastery of both deep theory AND practical lab work — from investigation procedure to evidence collection, chain of custody, analysis, and court-ready reporting. This guide is one part of a much larger learning journey. Commit to the process.

---

## Table of Contents

1. [Why Storage Media Knowledge is Essential for Forensics](#1-why-storage-media-knowledge-is-essential-for-forensics)
2. [SSD — Solid State Drive Overview](#2-ssd--solid-state-drive-overview)
3. [NAND Flash Memory — How Data is Actually Stored](#3-nand-flash-memory--how-data-is-actually-stored)
4. [Digital vs Analog Signals — The Foundation](#4-digital-vs-analog-signals--the-foundation)
5. [Capacitors, Transistors & Data Storage Physics](#5-capacitors-transistors--data-storage-physics)
6. [SSD Internal Architecture](#6-ssd-internal-architecture)
7. [RAM — Volatile Memory Forensics](#7-ram--volatile-memory-forensics)
8. [Hard Disk vs SSD — Forensic Comparison](#8-hard-disk-vs-ssd--forensic-comparison)
9. [Forensic Implications of SSD vs HDD](#9-forensic-implications-of-ssd-vs-hdd)
10. [Comprehensive Lab Exercises](#10-comprehensive-lab-exercises)
11. [Exam Q&A](#11-exam-qa)
12. [Quick Reference Cheatsheet](#12-quick-reference-cheatsheet)

---

## 1. Why Storage Media Knowledge is Essential for Forensics

A digital forensics investigator **must deeply understand storage media** because:

```
Crime committed using a computer
        |
        v
Evidence stored in:
    ├── Hard Disk (HDD) — magnetic platters
    ├── SSD (Solid State Drive) — NAND flash chips
    ├── RAM — volatile, running processes
    ├── USB drives — NAND flash
    └── Memory cards — NAND flash

        |
        v
Investigator must know:
    ├── How data is physically stored on each medium
    ├── How data is deleted (and can it be recovered?)
    ├── What happens when power is cut (HDD vs SSD vs RAM)
    ├── How to image each medium correctly
    └── What tools to use for each type
```

**Without this foundation:** You cannot correctly collect evidence, cannot explain your findings in court, and cannot understand what the forensic tools are actually doing.

---

## 2. SSD — Solid State Drive Overview

### Definition

A **Solid State Drive (SSD)** is a storage device that uses semiconductor-based flash memory (NAND Flash chips) instead of rotating magnetic platters like a Hard Disk Drive (HDD).

### Two Types of SSD

| Type       | Name                 | Key Property                               | Example                            |
| ---------- | -------------------- | ------------------------------------------ | ---------------------------------- |
| **Type 1** | NAND Flash-based SSD | Non-volatile — data persists without power | Laptop SSD, USB drive, Memory card |
| **Type 2** | Volatile RAM-based   | Volatile — data lost when power removed    | RAM (DRAM/SDRAM)                   |

### NAND Flash SSD Key Properties

```
Non-volatile       → Data persists even when unplugged (unlike RAM)
Non-movable parts  → No spinning motor, no read/write arm
                   → More reliable if dropped
                   → Less mechanical failure risk
5–10× faster      → Than traditional Hard Disk Drive
Flash memory chip → NAND Flash chip stores the data
```

### Common Form Factors

```
2.5" SATA SSD    → Same size as laptop HDD, common in older laptops
M.2 SSD          → Much smaller card format, modern laptops
NVMe (PCIe)      → Fastest type, uses PCIe lanes directly
mSATA            → Mini-SATA, older small form factor
USB Flash Drive  → NAND flash in USB form
MicroSD/SD Card  → NAND flash in card form
eMMC             → Embedded NAND flash (soldered onto motherboard)
```

---

## 3. NAND Flash Memory — How Data is Actually Stored

### What is NAND?

NAND is a type of **logic gate** used in digital electronics. In storage context, NAND flash memory is a type of non-volatile storage that uses NAND gate logic for storing data.

### Why NAND (not AND, OR, NOR)?

NAND gates are used because:

- They are the most efficient for data storage
- They use less power than other gate types
- They allow data to be stored without continuous power
- They can be packed very densely (millions per chip)

### NAND Flash Memory Hierarchy

```
SSD
└── Flash Chips (multiple on PCB)
    └── Dies (1–4 per chip)
        └── Planes (2–4 per die)
            └── Blocks (hundreds per plane)
                └── Pages (64–256 per block, typically 4KB each)
                    └── Cells (millions per page)
                        └── Each cell = 1 or more bits stored as charge
```

### Types of NAND Cells

| Type    | Full Name         | Bits per Cell | Density | Speed   | Endurance             | Use Case                |
| ------- | ----------------- | ------------- | ------- | ------- | --------------------- | ----------------------- |
| **SLC** | Single Level Cell | 1 bit         | Low     | Fastest | Highest (100K cycles) | Enterprise, industrial  |
| **MLC** | Multi Level Cell  | 2 bits        | Medium  | Fast    | High (10K cycles)     | Performance SSDs        |
| **TLC** | Triple Level Cell | 3 bits        | High    | Medium  | Medium (3K cycles)    | Consumer SSDs           |
| **QLC** | Quad Level Cell   | 4 bits        | Highest | Slowest | Lower (1K cycles)     | High-capacity, low-cost |

**Forensic relevance:** QLC drives have lower endurance — they wear out faster. In long-term investigations, the wear level can indicate how heavily a device was used.

---

## 4. Digital vs Analog Signals — The Foundation

Understanding how computers communicate requires understanding signal types.

### Analog Signal

```
Amplitude
   |    /\      /\      /\
   |   /  \    /  \    /  \
   |  /    \  /    \  /    \
   | /      \/      \/      \
   |__________________________ Time

Properties:
- Continuously varying
- No fixed value — always changing
- Used in: radio, audio, older telecommunications
- Not suitable for precise digital storage
```

### Digital Signal

```
Voltage
   |___    ___    ___
   |   |  |   |  |   |
   |   |  |   |  |   |
   |   |__|   |__|   |___
   |__________________________ Time

Properties:
- Only two states: HIGH (1) or LOW (0)
- Binary — 0 and 1 only
- Used in: computers, digital storage, networking
- Reliable, noise-resistant
- Everything stored as binary: 0101001110...
```

### Why This Matters for Storage

All data in an SSD is stored as binary (0s and 1s). Each NAND cell is either:

- **Charged** = stores `1`
- **Discharged** = stores `0`

This is how all files — images, documents, code, anything — become sequences of 0s and 1s stored in millions of tiny cells.

---

## 5. Capacitors, Transistors & Data Storage Physics

### The Capacitor Analogy

A capacitor is an electronic component that stores electrical charge.

```
Capacitor state → Binary value
─────────────────────────────
Charged          →    1
Discharged       →    0

Key property: Holds charge even when disconnected
(just like how a charged capacitor in a fan still
 sparks when touched years later)
```

### How NAND Flash Uses This Principle

Inside every NAND flash chip, there are millions of **Floating Gate Transistors** (a special type of capacitor-transistor hybrid):

```
Each Floating Gate Transistor:

    Control Gate
         |
    ─────────────   ← Control gate (top)
    |  Insulator |
    ─────────────   ← Floating gate (stores the charge)
    |  Insulator |
    ─────────────
    Source    Drain   ← Current flows between these

How it works:
    Charged floating gate  →  blocks current flow  →  reads as 1
    Discharged floating gate → allows current flow →  reads as 0
```

### Data Storage Cycle

```
Writing data:
1. Voltage applied to control gate
2. Electrons tunnel through insulator into floating gate
3. Floating gate becomes charged
4. State locked — remains charged without power
5. Reads as binary 1

Erasing data:
1. Higher voltage applied in reverse
2. Electrons tunnel OUT of floating gate
3. Floating gate becomes discharged
4. Reads as binary 0
```

### Critical Forensic Point — NAND Flash Erase Behavior

**Unlike HDDs, NAND flash cannot overwrite directly.**

```
HDD:     [Old Data] → [Write New Data directly over it]

NAND:    [Old Data] → [Must ERASE entire block first] → [Write New Data]

This means:
- Erase operations happen at BLOCK level (large chunks)
- Write operations happen at PAGE level (small chunks)
- This creates a "garbage collection" need
- This is why SSDs have TRIM command
- This is why SSD forensics is more complex than HDD forensics
```

---

## 6. SSD Internal Architecture

### Physical Components

```
SSD (SATA 2.5")
┌─────────────────────────────────────────────┐
│                                             │
│  [Flash Chip] [Flash Chip] [Flash Chip]     │
│  [Flash Chip] [Flash Chip] [Flash Chip]     │
│                                             │
│  [DRAM Cache]    [Controller]               │
│                                             │
│  [Power Port]    [Host Interface Port]      │
│                                             │
│  [PCB — Printed Circuit Board underneath]  │
└─────────────────────────────────────────────┘
```

### Each Component Explained

#### 1. NAND Flash Chips

- Main storage — where all data lives
- Contains millions of floating gate transistors
- Multiple chips per SSD (increases capacity)
- Data stored as electrical charge in each cell

#### 2. Controller

```
Function: The "CPU" of the SSD

Responsibilities:
├── Wear leveling      → Distributes writes evenly across all cells
│                        (prevents early failure of frequently-written areas)
├── Error correction   → ECC (Error Correcting Code) fixes bit errors
├── Garbage collection → Cleans up invalid data blocks
├── Bad block management → Marks and avoids failing cells
├── Encryption        → Hardware AES encryption (on supported drives)
├── TRIM handling     → Processes OS requests to erase unused blocks
└── Read/write        → Manages all data flow in and out
```

#### 3. DRAM Cache

```
Function: Temporary high-speed buffer

Stores:
├── Flash Translation Layer (FTL) mapping table
│   → Maps logical addresses (what OS sees) to physical NAND locations
├── Recently accessed data (for fast re-read)
└── Write buffer (accumulates small writes before committing to NAND)

Forensic relevance:
→ DRAM cache is VOLATILE — lost when powered off
→ If SSD is found powered on at crime scene → image the DRAM cache first
→ May contain recently accessed file data and FTL mapping
```

#### 4. PCB (Printed Circuit Board)

- Green board that all components are mounted on
- Contains printed copper wiring connecting all parts
- Provides electrical pathways between controller, NAND chips, ports

#### 5. Host Interface

```
The port connecting SSD to the computer

Types:
├── SATA    → 6 Gb/s max, most common, backwards compatible
├── PCIe    → 32+ Gb/s, used with NVMe SSDs
├── NVMe    → Protocol over PCIe, fastest consumer SSDs
└── USB     → For external SSDs and flash drives

Forensic relevance:
→ Determines which write blocker you need
→ SATA write blockers ≠ NVMe write blockers
→ Never connect an evidence SSD without a write blocker
```

---

## 7. RAM — Volatile Memory Forensics

### What is RAM?

**RAM (Random Access Memory)** is the working memory of a computer. Unlike SSD/HDD, it is **volatile** — all data disappears when power is removed.

```
RAM = Short-term, fast, volatile working memory
SSD/HDD = Long-term, slower, non-volatile storage

Analogy:
SSD/HDD = Filing cabinet (permanent storage)
RAM = Your desk surface (working area, cleared when you go home)
```

### How RAM Works in Practice

```
Computer workflow:
                     ┌──────────────────┐
                     │    SSD/HDD       │
                     │ (all files live  │
                     │  here long-term) │
                     └────────┬─────────┘
                              │ When app opens:
                              │ copy of file loaded to RAM
                              ▼
                     ┌──────────────────┐
                     │      RAM         │
                     │ (active working  │
                     │  copies of files)│
                     └────────┬─────────┘
                              │ Processor reads/writes
                              ▼
                     ┌──────────────────┐
                     │    CPU           │
                     │ (processes data) │
                     └──────────────────┘

When computer shuts down:
→ RAM is completely cleared (volatile)
→ SSD/HDD retains all data (non-volatile)
```

### What Lives in RAM During Operation?

```
RAM contents (all lost at shutdown):
├── Running processes and their state
├── Open browser tabs and their content
├── Logged-in session credentials (cached tokens)
├── Encryption keys (if disk encryption active)
├── Network connections and their data
├── Clipboard contents
├── Currently typed text (before saving)
├── Decrypted versions of encrypted files (while open)
├── Chat messages in transit
└── Malware running in memory (may not touch disk)
```

### Critical Forensic Principle — RAM Dump

> **If you find a computer TURNED ON at a crime scene, the FIRST thing you do is acquire the RAM dump — before anything else.**

```
Why:
The moment the computer is shut down → all RAM content is gone FOREVER
RAM contains evidence that exists NOWHERE else

What RAM dump reveals:
├── What websites were open (URLs, content)
├── What documents were being edited
├── What the user was doing at the time
├── Encryption keys (can unlock encrypted drives)
├── Passwords typed but not yet saved
├── Malware running only in memory ("fileless malware")
└── Network activity at time of seizure
```

### DRAM — Dynamic RAM

**DRAM (Dynamic RAM)** is the standard type of RAM in computers.

```
"Dynamic" = data changes constantly
         = capacitors in RAM cells leak charge
         = must be refreshed thousands of times per second
         = data lost immediately when power removed

vs. NAND Flash:
         = designed to HOLD charge for years
         = no constant refresh needed
```

---

## 8. Hard Disk vs SSD — Forensic Comparison

| Feature                        | HDD (Hard Disk Drive)                    | SSD (Solid State Drive)               |
| ------------------------------ | ---------------------------------------- | ------------------------------------- |
| **Storage mechanism**          | Magnetic platters + read/write arm       | NAND Flash chips                      |
| **Moving parts**               | Yes (motor, platters, arm)               | No                                    |
| **Speed**                      | ~100-150 MB/s                            | 500MB/s – 7,000 MB/s                  |
| **Impact resistance**          | Low — drops cause damage                 | High — no moving parts                |
| **Power needed for data**      | No (magnetic)                            | No (charge in cells)                  |
| **Data deletion**              | Overwrite directly possible              | Must erase block first                |
| **Data recovery after delete** | Often possible (data remains in sectors) | Much harder (TRIM erases immediately) |
| **Wear out**                   | Mechanical wear                          | Cell write cycles (TLC: ~3000 cycles) |
| **Forensic imaging**           | Straightforward                          | More complex (TRIM, encryption)       |
| **Write blocker needed?**      | YES                                      | YES                                   |
| **Price per GB**               | Cheaper                                  | More expensive                        |

---

## 9. Forensic Implications of SSD vs HDD

### The TRIM Problem

**TRIM** is a command where the OS tells the SSD: "these blocks are now free, you can erase them."

```
User deletes file:
    ↓
OS marks space as available
    ↓ (on HDD: nothing else happens — data still there)
    ↓ (on SSD: TRIM command sent)
    ↓
SSD immediately erases those NAND blocks
    ↓
Data is PERMANENTLY gone — unrecoverable
```

**Forensic impact:**

- On HDD: deleted files often recoverable for months
- On SSD with TRIM enabled: deleted files gone almost immediately
- TRIM is enabled by default on Windows 7+, macOS, modern Linux

```bash
# Check if TRIM is enabled (Windows)
fsutil behavior query DisableDeleteNotify
# 0 = TRIM enabled (bad for recovery, good for privacy)
# 1 = TRIM disabled (good for recovery)

# Check TRIM on Linux
lsblk --discard
# Non-zero DISC-GRAN = TRIM supported and active
```

### Wear Leveling & Forensics

The SSD controller **distributes writes** to prevent any one area from wearing out faster. This means:

```
You write File A to sector 100:
→ SSD actually writes to physical block 547 (controller decides)
→ Logical address 100 → Physical block 547 (stored in FTL table)
→ You read sector 100 → controller translates to block 547

Forensic implication:
→ You cannot predict WHERE data physically sits
→ Logical file system structure ≠ physical NAND layout
→ Direct NAND chip reading ("chip-off forensics") required
   for physical-level analysis
```

### Hardware Encryption on SSDs

Many modern SSDs have **hardware AES encryption** (Self-Encrypting Drives — SED):

```
Data written → Controller encrypts → Stored in NAND
Data read → Retrieved from NAND → Controller decrypts → Sent to OS

Encryption key stored in controller
If "Secure Erase" performed → key destroyed → ALL data unrecoverable
```

**Forensic implication:** If a suspect performs Secure Erase on a self-encrypting SSD → evidence is gone. Cannot be recovered.

### Chip-Off Forensics

When normal software forensics fails (damaged drive, encryption), physical chip removal is used:

```
Chip-Off Process:
1. Disassemble SSD
2. Physically remove NAND flash chips from PCB
3. Read chips using specialized NAND reader hardware
4. Reconstruct data from raw NAND dumps
5. Reverse-engineer wear leveling translation
6. Recover data

Tools:
- PC-3000 Flash (Russia) — professional chip-off tool
- NAND Flash programmer hardware
- Custom scripts to reverse FTL mapping

Requires:
- Specialist hardware
- Deep technical knowledge
- Is destructive to original evidence (chips removed)
```

---

## 10. Comprehensive Lab Exercises

### Lab 1 — Identify Storage Media on Your System

**Objective:** Learn to identify what type of storage is in a device.

```bash
# Linux/Kali — Check storage devices
lsblk
# Shows all block devices, their type, size, mount points

# Detailed disk info
sudo fdisk -l

# SSD vs HDD check
cat /sys/block/sda/queue/rotational
# 0 = SSD (non-rotating)
# 1 = HDD (rotating)

# Check for multiple drives
lsblk -d -o NAME,ROTA,TYPE,SIZE,MODEL
# ROTA=0 → SSD, ROTA=1 → HDD

# Check NVMe specifically
ls /dev/nvme*
nvme list  # (install nvme-cli first: sudo apt install nvme-cli)

# Windows equivalent
wmic diskdrive get Caption,InterfaceType,Size,MediaType
```

**Document:**

- Type of storage (HDD/SSD/NVMe)
- Model number
- Capacity
- Interface type

---

### Lab 2 — SSD Health & SMART Data Analysis

**Objective:** Read SMART data from your drive to understand wear and health status.

```bash
# Install smartmontools
sudo apt install smartmontools

# Check basic SMART status
sudo smartctl -a /dev/sda
# or for NVMe:
sudo smartctl -a /dev/nvme0

# Key values to look for in SSD SMART output:
# 177 Wear_Leveling_Count  → How much the SSD has been used
# 179 Used_Rsvd_Blk_Cnt_Tot → Reserved blocks used (indicates wear)
# 231 SSD_Life_Left        → % of life remaining
# 233 Media_Wearout_Indicator → Wear indicator
# 241 Total_LBAs_Written   → Total data written to SSD

# For NVMe drives
sudo nvme smart-log /dev/nvme0
# Look for: percentage_used, data_units_written

# Run extended self-test
sudo smartctl -t long /dev/sda
sudo smartctl -l selftest /dev/sda  # View results

# Create a SMART report
sudo smartctl -a /dev/sda > smart_report.txt
```

**Forensic Application:** In investigations, SMART data shows:

- How much data was written (helps establish usage patterns)
- Drive health (is it about to fail? Evidence may be at risk)
- Power-on hours (usage timeline)

---

### Lab 3 — Create Forensic Disk Image (Write-Blocked)

**Objective:** Practice creating a forensically sound image of a storage device.

> **Use a USB drive or spare drive you own for this lab — never image an evidence drive without proper write blockers.**

```bash
# Method 1: dd (basic, always available)
# WARNING: Always verify destination (/dev/sdb) is your TARGET, not your source
# Using wrong destination destroys evidence

# Identify your drives FIRST
lsblk
# Example output:
# sda    → your system drive (DO NOT IMAGE TO HERE)
# sdb    → your USB test drive (source to image)
# sdc    → your destination drive (image goes here)

# Create image with dd
sudo dd if=/dev/sdb of=/path/to/destination/evidence.img bs=4M status=progress

# BETTER: Use dcfldd (forensic version of dd with hashing)
sudo apt install dcfldd
sudo dcfldd if=/dev/sdb of=/path/to/evidence.img hash=md5,sha256 hashlog=hashes.txt bs=4M

# BEST: Use dc3dd (most forensically sound)
sudo apt install dc3dd
sudo dc3dd if=/dev/sdb hof=/path/to/evidence.img hash=sha256 log=acquisition.log bs=4M

# PROFESSIONAL: Use Guymager (GUI tool)
sudo apt install guymager
sudo guymager
# Select source drive
# Set hash algorithm (MD5 + SHA1 recommended)
# Set output path and format (EWF/E01 recommended for cases)
# Verify image after creation (critical!)
```

**Verify image integrity:**

```bash
# Generate hash of original drive
sudo sha256sum /dev/sdb > original_hash.txt

# Generate hash of image file
sha256sum evidence.img > image_hash.txt

# Compare — must be IDENTICAL
diff original_hash.txt image_hash.txt
# No output = match = forensically sound image
```

---

### Lab 4 — RAM Dump Acquisition

**Objective:** Acquire and analyze volatile memory (RAM) from a live system.

```bash
# Method 1: LiME (Linux Memory Extractor)
# Most reliable Linux RAM acquisition tool

# Install build dependencies
sudo apt install build-essential linux-headers-$(uname -r)

# Get LiME
git clone https://github.com/504ensicsLabs/LiME
cd LiME/src
make

# Load the LiME kernel module and dump RAM to file
sudo insmod lime-$(uname -r).ko "path=/tmp/ram_dump.lime format=lime"

# Unload when done
sudo rmmod lime

# Method 2: Using /dev/mem (limited, not full RAM)
sudo dd if=/dev/mem of=/tmp/memdump.bin bs=1M count=512
# Only reads first 512MB on many modern kernels

# Method 3: /proc/kcore (kernel memory)
sudo dd if=/proc/kcore of=/tmp/kcore_dump bs=4M

# For Windows systems (at crime scene):
# Use: WinPmem (free), Magnet RAM Capture (free), FTK Imager (free)
# winpmem.exe -o ram_dump.aff4
```

---

### Lab 5 — Analyze RAM Dump with Volatility

**Volatility** is the standard tool for RAM dump analysis.

```bash
# Install Volatility 3 (modern version)
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
pip3 install -r requirements.txt

# Or install Volatility 2 (legacy but widely used in courses)
sudo apt install volatility

# ─── Volatility 3 Commands ───

# Identify OS profile of the RAM dump
python3 vol.py -f /path/to/ram_dump.lime banners.Banners

# List running processes at time of capture
python3 vol.py -f ram_dump.lime linux.pslist.PsList

# List processes with parent-child relationships
python3 vol.py -f ram_dump.lime linux.pstree.PsTree

# List network connections at time of capture
python3 vol.py -f ram_dump.lime linux.netstat.NetStat

# Find open files
python3 vol.py -f ram_dump.lime linux.lsof.Lsof

# Scan for malware indicators
python3 vol.py -f ram_dump.lime linux.malfind.Malfind

# Dump a specific process memory
python3 vol.py -f ram_dump.lime linux.memmap.Memmap --pid 1234 --dump

# For Windows RAM dumps:
python3 vol.py -f windows_ram.raw windows.pslist.PsList
python3 vol.py -f windows_ram.raw windows.netscan.NetScan
python3 vol.py -f windows_ram.raw windows.cmdline.CmdLine
python3 vol.py -f windows_ram.raw windows.filescan.FileScan
python3 vol.py -f windows_ram.raw windows.hashdump.Hashdump
python3 vol.py -f windows_ram.raw windows.clipboard.Clipboard
python3 vol.py -f windows_ram.raw windows.sessions.Sessions
```

**What to look for in RAM analysis:**

```
✓ Processes that shouldn't be running
✓ Processes with suspicious names (misspelled system processes)
✓ Unusual network connections (unknown external IPs)
✓ Command-line history (what commands were run)
✓ Browser history in memory (URLs visited)
✓ Encryption keys (look for recognizable key patterns)
✓ Injected code in legitimate processes
✓ Hidden processes (rootkit indicator)
```

---

### Lab 6 — File Recovery After Deletion

**Objective:** Understand how deletion works differently on HDD vs SSD.

```bash
# Install recovery tools
sudo apt install testdisk photorec foremost scalpel

# PhotoRec — recover deleted files by file signature
sudo photorec /dev/sdb
# Interactive menu — select partition, file types to recover
# Works on both HDD and SSD (but less effective on SSD with TRIM)

# Foremost — file carving by header/footer signatures
sudo foremost -i evidence.img -o /output/recovered/
# -t all    → recover all file types
# -t jpg,pdf,doc → specific types

# Scalpel — advanced file carver
sudo scalpel evidence.img -o /output/scalpel/

# TestDisk — recover deleted partitions
sudo testdisk /dev/sdb

# Check if TRIM has run (SSD only)
sudo hdparm -I /dev/sda | grep TRIM
# If TRIM is enabled → deleted SSD data likely unrecoverable
```

**Experiment:**

```bash
# On a test USB drive (ext4 filesystem):
# 1. Create test file
echo "Secret evidence text" > /mnt/usb/test_file.txt

# 2. Unmount and image BEFORE deletion
sudo umount /dev/sdb1
sudo dd if=/dev/sdb of=before_delete.img bs=4M

# 3. Delete the file
sudo mount /dev/sdb1 /mnt/usb
rm /mnt/usb/test_file.txt
sudo umount /dev/sdb1

# 4. Image AFTER deletion
sudo dd if=/dev/sdb of=after_delete.img bs=4M

# 5. Compare — on HDD/USB the data is still there!
strings after_delete.img | grep "Secret evidence"
# Should still see the text on HDD/USB

# On SSD with TRIM:
# Data would already be erased — strings won't find it
```

---

### Lab 7 — SMART Data Forensic Analysis

```bash
# Extract complete SMART log from evidence drive
sudo smartctl -x /dev/sda > complete_smart.txt

# Parse key forensic values
sudo smartctl -A /dev/sda | grep -E "Power_On_Hours|Power_Cycle_Count|Reallocated|Wear"

# For NVMe
sudo nvme smart-log /dev/nvme0 | grep -E "power_on_hours|unsafe_shutdowns|data_units"

# Create forensic SMART report
echo "=== Forensic SMART Report ===" > smart_forensic.txt
echo "Date: $(date)" >> smart_forensic.txt
echo "Drive: $(sudo smartctl -i /dev/sda | grep 'Model Family')" >> smart_forensic.txt
echo "" >> smart_forensic.txt
sudo smartctl -a /dev/sda >> smart_forensic.txt

# Key values to document:
# 9  Power_On_Hours     → Total time drive was powered on
# 12 Power_Cycle_Count  → Number of power cycles (shutdowns/startups)
# 177 Wear_Leveling_Count → SSD wear level
# 231 SSD_Life_Left     → Remaining life percentage
```

---

### Lab 8 — Write Blocker Verification

**Objective:** Verify that a write blocker is working correctly before imaging evidence.

```bash
# Software write blocking (for lab practice)
# In production, always use hardware write blocker

# Method 1: Mount as read-only
sudo mount -o ro /dev/sdb /mnt/evidence
# Try to write — should fail
sudo touch /mnt/evidence/test.txt  # Should return: Read-only file system

# Method 2: Use blockdev to set read-only
sudo blockdev --setro /dev/sdb
# Verify
sudo blockdev --getro /dev/sdb
# Should output: 1 (read-only = true)

# Method 3: Use hdparm
sudo hdparm -r1 /dev/sdb  # Set read-only

# Verify write blocking worked
# Hash the drive BEFORE attempted write
sudo sha256sum /dev/sdb > hash_before.txt

# Attempt to write (should fail or not change drive)
sudo dd if=/dev/zero of=/dev/sdb bs=512 count=1 2>&1
# Should show: Operation not permitted

# Hash AFTER attempted write
sudo sha256sum /dev/sdb > hash_after.txt

# Compare — must be identical
diff hash_before.txt hash_after.txt
# No output = write blocker working correctly
```

---

### Lab 9 — Metasploitable 2 Memory Analysis (Practice)

```bash
# Start Metasploitable 2 in VirtualBox
# Exploit a service to get a session
# Then perform memory analysis

# In Metasploit on Kali:
msfconsole
use exploit/multi/samba/usermap_script
set RHOST 192.168.56.101
exploit

# Once you have a shell, dump process list
# (demonstrates what's running in target's memory)
ps aux

# Upload LiME to target and acquire RAM
# (demonstrates live acquisition from compromised system)
upload /path/to/lime.ko /tmp/lime.ko

# On target (via shell):
insmod /tmp/lime.ko "path=/tmp/ram.lime format=lime"

# Download the RAM dump
download /tmp/ram.lime /forensics/target_ram.lime

# Analyze with Volatility
cd /path/to/volatility3
python3 vol.py -f /forensics/target_ram.lime banners.Banners
python3 vol.py -f /forensics/target_ram.lime linux.pslist.PsList
python3 vol.py -f /forensics/target_ram.lime linux.netstat.NetStat
```

---

## 11. Exam Q&A

### Q1: What are the two types of SSD?

**Type 1:** NAND Flash-based SSD — non-volatile, data persists without power. Examples: laptop SSD, USB drive, memory card.
**Type 2:** Volatile RAM-based — volatile, data is lost when power is removed. Example: DRAM used as RAM in computers.

### Q2: What is NAND Flash memory and why is it used in SSDs?

NAND Flash is a type of non-volatile semiconductor memory based on NAND logic gate circuitry. It stores data as electrical charge in floating gate transistors. It is used in SSDs because it is non-volatile (data persists without power), has no moving parts (more reliable), is 5–10× faster than HDDs, and can be packed into very small chips at high density.

### Q3: What is a RAM dump and why is it critical in digital forensics?

A RAM dump is a forensic copy of the entire contents of a computer's RAM at a specific point in time. It is critical because RAM is volatile — all its data is permanently lost when the computer is shut down. RAM contains evidence that exists nowhere else: running processes, open browser tabs, login session data, encryption keys, clipboard contents, and malware running only in memory. If a computer is found ON at a crime scene, the RAM dump must be acquired FIRST before any other action.

### Q4: What is the TRIM command and how does it affect SSD forensics?

TRIM is a command by which the operating system tells an SSD that certain blocks of data are no longer needed. The SSD immediately erases those blocks to prepare them for future writes. Forensic impact: on HDDs, deleted files often remain recoverable for a long time because the data is not immediately overwritten. On SSDs with TRIM enabled (default on modern systems), deleted files are erased almost immediately and become unrecoverable through standard forensic methods.

### Q5: What is the difference between volatile and non-volatile memory?

**Volatile memory** (e.g., RAM): Data is lost immediately when power is removed. It holds temporary data like running processes and open applications.
**Non-volatile memory** (e.g., SSD, HDD, USB drive): Data persists indefinitely without power. It holds permanent data like files and the operating system.

### Q6: What is wear leveling in an SSD and why does it matter forensically?

Wear leveling is a technique used by the SSD controller to distribute write operations evenly across all NAND cells, preventing any single area from wearing out prematurely. Forensically, this means the logical address of a file (what the OS sees) does not correspond directly to its physical location on the NAND chips. This complicates forensic analysis because you cannot predict where data physically resides, and chip-off forensics requires reversing the Flash Translation Layer (FTL) mapping.

### Q7: What components are inside an SSD?

An SSD contains: NAND Flash chips (where data is permanently stored), a Controller (manages all read/write operations, wear leveling, ECC, garbage collection), DRAM cache (temporary high-speed buffer for the FTL mapping table), PCB (Printed Circuit Board connecting all components), Power port (provides power to all components), and Host Interface port (connects SSD to the computer — SATA, NVMe/PCIe).

### Q8: What is DRAM and why is it dynamic?

DRAM (Dynamic Random Access Memory) is "dynamic" because its storage cells (capacitors) constantly leak charge and must be refreshed thousands of times per second to maintain data. This makes it volatile — the moment power is cut, the constant refresh stops, capacitors discharge, and all data is lost. It is called dynamic to distinguish it from SRAM (Static RAM), which holds data without refresh but is more expensive.

### Q9: What is chip-off forensics?

Chip-off forensics is a destructive physical technique where NAND flash chips are physically removed from a damaged or encrypted SSD, then read directly using specialized hardware. It is used when the drive is too damaged for normal connection, or when hardware encryption prevents software access. It requires specialized equipment (NAND readers), technical expertise to reverse the FTL mapping, and is destructive to the original evidence.

### Q10: Why must an investigator never connect evidence to a computer without a write blocker?

Without a write blocker, connecting a storage device to a computer causes the OS to automatically write to it — updating access timestamps, writing swap/temp files, modifying MFT entries, or mounting and indexing. This modifies the evidence, breaks chain of custody, makes the evidence potentially inadmissible in court, and invalidates any hash comparisons made before and after. A write blocker (hardware or software) prevents ALL write operations to the evidence drive while allowing reads.

---

## 12. Quick Reference Cheatsheet

### Storage Identification

```bash
lsblk                          # List all block devices
lsblk -d -o NAME,ROTA,SIZE    # ROTA=0 → SSD, ROTA=1 → HDD
cat /sys/block/sda/queue/rotational  # 0=SSD, 1=HDD
sudo fdisk -l                  # Detailed partition info
sudo smartctl -i /dev/sda      # Drive model and type
```

### SMART Analysis

```bash
sudo smartctl -a /dev/sda           # All SMART data
sudo smartctl -H /dev/sda           # Health status only
sudo smartctl -A /dev/sda           # Attributes only
sudo nvme smart-log /dev/nvme0      # NVMe SMART data
sudo nvme list                      # List NVMe drives
```

### Disk Imaging

```bash
# dd (basic)
sudo dd if=/dev/sdb of=image.img bs=4M status=progress

# dcfldd (with hashing)
sudo dcfldd if=/dev/sdb of=image.img hash=sha256 hashlog=hash.txt

# dc3dd (forensic)
sudo dc3dd if=/dev/sdb hof=image.img hash=sha256 log=acq.log

# Verify image
sha256sum /dev/sdb > original.hash
sha256sum image.img > image.hash
diff original.hash image.hash
```

### RAM Acquisition

```bash
# LiME (Linux)
sudo insmod lime.ko "path=/tmp/ram.lime format=lime"

# Check RAM size
free -h
cat /proc/meminfo | grep MemTotal
```

### Volatility 3 (RAM Analysis)

```bash
python3 vol.py -f ram.lime banners.Banners    # OS identification
python3 vol.py -f ram.lime linux.pslist       # Process list
python3 vol.py -f ram.lime linux.pstree       # Process tree
python3 vol.py -f ram.lime linux.netstat      # Network connections
python3 vol.py -f ram.lime linux.lsof         # Open files
python3 vol.py -f ram.lime linux.malfind      # Malware scan

# Windows
python3 vol.py -f win.raw windows.pslist      # Process list
python3 vol.py -f win.raw windows.netscan     # Network
python3 vol.py -f win.raw windows.cmdline     # Command history
python3 vol.py -f win.raw windows.hashdump    # Password hashes
```

### File Recovery

```bash
sudo photorec /dev/sdb                              # Recover by file type
sudo foremost -i image.img -o /output/             # File carving
sudo scalpel image.img -o /output/                 # Advanced carving
strings image.img | grep "keyword"                 # Search for text
```

### Write Blocker (Software)

```bash
sudo mount -o ro /dev/sdb /mnt/evidence            # Mount read-only
sudo blockdev --setro /dev/sdb                     # Block-level read-only
sudo blockdev --getro /dev/sdb                     # Verify (1=RO)
```

### TRIM Check

```bash
sudo hdparm -I /dev/sda | grep TRIM               # HDD/SSD TRIM status
lsblk --discard                                    # TRIM capability
sudo fstrim -v /mount/point                        # Run TRIM manually
```

---

## Summary: SSD vs HDD vs RAM at a Glance

```
Storage Type   Volatile?   Moving Parts?   Speed    Recovery After Delete   Forensic Difficulty
─────────────────────────────────────────────────────────────────────────────────────────────
HDD            No          YES             Slow     Often possible          Moderate
SSD (NAND)     No          No              Fast     Very difficult (TRIM)   High
USB Drive      No          No              Medium   Very difficult (TRIM)   High
RAM (DRAM)     YES         No              Fastest  Impossible after off    Very High (time)
```

---

_Digital Forensics Part 5 | Next: Part 6 — File Systems, MFT, and Evidence Collection Procedures_
_Practice: TryHackMe — Digital Forensics rooms | Volatility documentation: volatility3.readthedocs.io_
