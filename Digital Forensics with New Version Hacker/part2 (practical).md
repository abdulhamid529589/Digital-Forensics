# 🗂️ Windows File System Forensics — Comprehensive Lab Guide

### Digital Forensics Practical Series | Part 2: File Systems, NTFS & Image Formats

> **⚠️ Legal Notice:** All forensic investigation techniques must only be performed on systems you own or have explicit written authorization to examine, or on dedicated lab environments. Never examine original evidence — always work on forensic images.

---

## 📋 Table of Contents

1. [Windows File System Overview](#1-windows-file-system-overview)
2. [The Sleuth Kit — Command Line Forensics](#2-the-sleuth-kit--command-line-forensics)
3. [Master File Table (MFT) Deep Dive](#3-master-file-table-mft-deep-dive)
4. [Forensic Image Formats](#4-forensic-image-formats)
5. [Creating Forensic Images — Practical Lab](#5-creating-forensic-images--practical-lab)
6. [File Recovery via Command Line](#6-file-recovery-via-command-line)
7. [WinHex — Advanced Data Recovery](#7-winhex--advanced-data-recovery)
8. [Memory Forensics Basics](#8-memory-forensics-basics)
9. [Cheat Sheet — All Commands](#9-cheat-sheet--all-commands)

---

## 1. Windows File System Overview

### 1.1 What Is a File System?

A file system is the method an operating system uses to **organize, store, and retrieve data** on a disk. Without a file system, data would be an unorganized pile of bits.

| OS               | Primary File System | Notes                                          |
| ---------------- | ------------------- | ---------------------------------------------- |
| Windows (modern) | NTFS                | Most common; supports large files, permissions |
| Windows (old)    | FAT32               | USB drives, older systems; simpler structure   |
| Linux            | ext4                | Default Linux file system                      |
| macOS            | APFS                | Apple File System (modern Macs)                |
| All (USB)        | exFAT               | Cross-platform compatible                      |

### 1.2 Disk Structure Hierarchy

```
Physical Disk (Hard Drive / SSD / USB)
│
├── Partition Table (MBR or GPT)
│   ├── Partition 1 (e.g., C: drive — NTFS)
│   ├── Partition 2 (e.g., D: drive — NTFS)
│   ├── Partition 3 (e.g., Recovery — NTFS)
│   └── Unallocated Space
│
└── Each Partition Contains:
    ├── Boot Sector
    ├── Master File Table (NTFS)
    └── File Data Areas
```

### 1.3 Allocated vs. Unallocated Space

```
Allocated Space:
   → Disk space that has been assigned to a file system partition
   → Visible and accessible to the OS
   → Files are stored here

Unallocated Space:
   → Disk space not yet assigned to any partition
   → OS cannot use it until allocated
   → May contain remnants of old data (forensically valuable!)
   → Visible in Disk Management as "Unallocated"
```

**Viewing partitions graphically:**

```
Windows → Right-click "This PC" → Manage → Disk Management
```

### 1.4 Sectors and Clusters

```
Sector:    Smallest physical unit of disk storage = 512 bytes (traditional)
                                                   or 4096 bytes (modern Advanced Format)
Cluster:   Logical grouping of sectors (defined by file system)
           Typically 4KB to 64KB in size

File occupies one or more clusters.
If a file is smaller than a cluster, the leftover space is called "slack space"
Slack space can contain fragments of previously deleted files — valuable for forensics!
```

---

## 2. The Sleuth Kit — Command Line Forensics

### 2.1 What Is The Sleuth Kit (TSK)?

The Sleuth Kit (TSK) is a **free, open-source command-line digital forensics toolkit** used by law enforcement and security professionals worldwide. It is the engine behind Autopsy (GUI).

**Download:** https://www.sleuthkit.org/sleuthkit/

**Key tools inside TSK:**

| Command       | Full Name             | Purpose                             |
| ------------- | --------------------- | ----------------------------------- |
| `mmls`        | Media Management List | Show partition table                |
| `fsstat`      | File System Status    | Show file system details            |
| `img_stat`    | Image Status          | Show image file details             |
| `istat`       | Inode Status          | Show file metadata (MFT entry)      |
| `fls`         | File List             | List files and directories          |
| `tsk_recover` | TSK Recover           | Recover all files from image        |
| `blkls`       | Block List            | List data blocks                    |
| `ils`         | Inode List            | List all inodes (including deleted) |

### 2.2 Opening TSK via Command Line

```
Step 1: Navigate to TSK installation folder
        e.g., C:\sleuthkit\bin\

Step 2: Click on the address bar in File Explorer
        Type: cmd
        Press Enter

→ CMD opens directly inside the TSK bin folder
→ All TSK commands are now available
```

### 2.3 Lab: View Partition Table with mmls

**mmls = Media Management List**
Lists all partitions on a disk image or physical device.

```bash
# Syntax
mmls [options] <image_file>

# Basic usage — view partition table
mmls "C:\forensics\evidence\windows_image.E01"

# With space between command and path:
mmls "C:\forensics\images\evidence.dd"
```

**Sample output:**

```
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000206847   0000204800   NTFS / exFAT (0x07)
003:  000:001   0000206848   0000411647   0000204800   NTFS / exFAT (0x07)
004:  -------   0000411648   0000419429   0000007782   Unallocated
```

**Reading the output:**

- `Start` / `End` = sector numbers where partition begins and ends
- `Length` = size of partition in sectors
- `NTFS / exFAT` = file system type
- `Unallocated` = space not assigned to any partition

**Getting help for any TSK command:**

```bash
mmls       # shows help and all available options
mmls -h    # explicit help flag
```

### 2.4 Lab: Check File System Status with fsstat

**fsstat = File System Status**
Reveals file system details: type, volume name, cluster size, MFT location, etc.

```bash
# Syntax — must specify file system type (-f) and image file
fsstat -f ntfs "C:\forensics\images\windows.dd"

# For a specific partition offset (from mmls output):
# If partition starts at sector 2048, sector size = 512 bytes:
# Offset = 2048 * 512 = 1048576 bytes
fsstat -f ntfs -o 2048 "C:\forensics\images\windows.dd"
```

**Key information in fsstat output:**

```
File System Type: NTFS
Volume Serial Number: XXXXXXXXXXXXXXXX
OEM Name: NTFS
Volume Name: [Drive Label]
File System Type: NTFS
Cluster Size: 4096 bytes
Total Cluster Range: 0 - 15728639
Total Sector Range: 0 - 125829119

$MFT Entry:     Starting at Cluster XXXX
$MFTMirr Entry: Starting at Cluster XXXX

Metadata Information:
Root Directory: 5
```

### 2.5 Lab: Check Image Status with img_stat

**img_stat = Image Statistics**
Shows image file format, size, and sector information.

```bash
img_stat "C:\forensics\images\windows.dd"
```

**Sample output:**

```
IMAGE FILE INFORMATION
--------------------------------------------
Image Type: raw

Size in bytes: 64424509440
Sector size: 512
```

Image types that may appear: `raw`, `ewf` (E01), `aff`

---

## 3. Master File Table (MFT) Deep Dive

### 3.1 What Is the MFT?

The **Master File Table (MFT)** is the heart of NTFS. It is a special database that stores metadata for **every file and directory** on an NTFS volume.

Think of it as the **index of a book** — it tells you where everything is and details about it.

```
MFT Structure:
┌─────────────────────────────────────────────┐
│              Master File Table               │
├──────┬───────────────────────────────────────┤
│ #0   │ $MFT itself                           │
│ #1   │ $MFTMirr — Mirror/backup copy of MFT  │
│ #2   │ $LogFile — Transaction log            │
│ #3   │ $Volume — Volume information          │
│ #4   │ $AttrDef — Attribute definitions      │
│ #5   │ . (root directory)                    │
│ #6   │ $Bitmap — Cluster allocation map      │
│ #7   │ $Boot — Boot sector                   │
│ #8   │ $BadClus — Bad cluster list           │
│ #9   │ $Secure — Security descriptors        │
│ #10  │ $UpCase — Uppercase filename table    │
│ #11  │ $Extend — Extended metadata           │
│ #12-15│ Reserved for future use             │
│ #16  │ User files and directories begin here │
│ ...  │ ...                                   │
└──────┴───────────────────────────────────────┘
```

### 3.2 MFT Entry Attributes

Each MFT entry (file record) contains these **attributes**:

```
┌─────────────────────────┬────────────────────────────────────────────────┐
│ Attribute               │ Contents                                       │
├─────────────────────────┼────────────────────────────────────────────────┤
│ $STANDARD_INFORMATION   │ Timestamps (created, modified, accessed,       │
│                         │ MFT-changed), file flags, owner ID             │
├─────────────────────────┼────────────────────────────────────────────────┤
│ $FILE_NAME              │ File name, parent directory reference,         │
│                         │ file size (allocated + actual), timestamps     │
├─────────────────────────┼────────────────────────────────────────────────┤
│ $DATA                   │ The actual file content (for small files,      │
│                         │ stored directly in MFT; large files point      │
│                         │ to clusters on disk)                           │
├─────────────────────────┼────────────────────────────────────────────────┤
│ $BITMAP                 │ Tracks which clusters are in use for           │
│                         │ a directory or the volume itself               │
├─────────────────────────┼────────────────────────────────────────────────┤
│ $SECURITY_DESCRIPTOR    │ Access control: who can read/write/execute,    │
│                         │ file ownership, permissions                    │
├─────────────────────────┼────────────────────────────────────────────────┤
│ $INDEX_ROOT             │ Directory index (lists files in a folder)      │
│ $INDEX_ALLOCATION       │ Used for large directories                     │
└─────────────────────────┴────────────────────────────────────────────────┘
```

### 3.3 Lab: Extracting MFT Entry Metadata with istat

**istat = Inode/MFT entry Status**
Displays all metadata stored in a specific MFT entry number.

```bash
# Syntax
istat -f ntfs "image_file" <entry_number>

# View MFT entry #0 ($MFT itself)
istat -f ntfs "C:\forensics\images\windows.dd" 0

# View MFT entry #1 ($MFTMirr — mirror file)
istat -f ntfs "C:\forensics\images\windows.dd" 1

# View MFT entry #2 ($LogFile)
istat -f ntfs "C:\forensics\images\windows.dd" 2

# View MFT entry #3 ($Volume)
istat -f ntfs "C:\forensics\images\windows.dd" 3

# View root directory (#5)
istat -f ntfs "C:\forensics\images\windows.dd" 5
```

**Sample output for entry #0:**

```
MFT Entry Header Values:
Entry: 0        Sequence: 1
$LogFile Sequence Number: XXXXXXXXXX
Allocated File

$STANDARD_INFORMATION Attribute Values:
Created:   2019-07-22 11:37:31 (IST)
File Modified: 2019-07-22 11:37:31 (IST)
MFT Modified:  2019-07-22 11:37:31 (IST)
Accessed:  2019-07-22 11:37:31 (IST)

$FILE_NAME Attribute Values:
Parent MFT Entry: 5   Seq: 1
Name: $MFT
Allocated Size: 0   Actual Size: 0

$DATA Attribute Values:
Size: [actual MFT size in bytes]
```

### 3.4 The Four Timestamps (MACB)

NTFS tracks four timestamps per file, critical for timeline analysis:

```
M — Modified:  When was the FILE CONTENT last changed?
A — Accessed:  When was the file last opened/read?
C — Changed:   When was the MFT entry last changed? (metadata change)
B — Born:      When was the file CREATED?

These are stored in $STANDARD_INFORMATION and $FILE_NAME attributes.

⚠️ Forensic Note: $STANDARD_INFORMATION timestamps can be modified by
   tools (anti-forensics). $FILE_NAME timestamps are harder to fake.
   Always compare both sets!
```

### 3.5 MFT Entry Numbers — Reference Table

| Entry # | System File | Contents                                 |
| ------- | ----------- | ---------------------------------------- |
| 0       | $MFT        | Master File Table itself                 |
| 1       | $MFTMirr    | Backup copy of first 4 MFT entries       |
| 2       | $LogFile    | Transaction log for recovery             |
| 3       | $Volume     | Volume name, version, flags              |
| 4       | $AttrDef    | Attribute type definitions               |
| 5       | . (dot)     | Root directory                           |
| 6       | $Bitmap     | Cluster allocation map                   |
| 7       | $Boot       | Boot sector / BPB                        |
| 8       | $BadClus    | Bad cluster list                         |
| 9       | $Secure     | Security descriptors                     |
| 10      | $UpCase     | Uppercase conversion table               |
| 11      | $Extend     | Extended metadata directory              |
| 12–15   | (Reserved)  | Future use                               |
| 16+     | User files  | Your actual files and folders begin here |

### 3.6 Lab: List All Files with fls

**fls = File List**
Lists all files and directories in a file system, including deleted ones.

```bash
# List all files (including deleted — marked with *)
fls -f ntfs "C:\forensics\images\windows.dd"

# List files in a specific directory (entry number from fls output)
fls -f ntfs "C:\forensics\images\windows.dd" 5

# Recursive listing of all files
fls -r -f ntfs "C:\forensics\images\windows.dd"

# Show deleted files only
fls -d -f ntfs "C:\forensics\images\windows.dd"

# Full path output (useful for timeline)
fls -r -p -f ntfs "C:\forensics\images\windows.dd"
```

**Reading fls output:**

```
r/r 48-128-1: secret_document.docx         ← regular file, allocated
d/d 5:        .                             ← directory (root)
r/r * 93-128-1: deleted_file.txt           ← * = deleted file!
```

---

## 4. Forensic Image Formats

### 4.1 Overview

When creating a forensic image, the format you choose determines what metadata is stored, whether compression is applied, and which tools can open it.

```
┌──────────┬────────────────────────────────────────────────────────────────┐
│ Format   │ Full Name                                                      │
├──────────┼────────────────────────────────────────────────────────────────┤
│ DD / RAW │ Raw disk image (no format overhead)                            │
│ SMART    │ Simple Management Artifact Recovery Technology                 │
│ E01      │ Expert Witness Format (by Guidance Software / EnCase)          │
│ AFF      │ Advanced Forensic Format                                       │
│ MEM      │ Memory Image (for RAM dumps)                                   │
└──────────┴────────────────────────────────────────────────────────────────┘
```

### 4.2 Detailed Format Comparison

```
┌─────────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ Feature     │ DD/RAW   │ SMART    │ E01      │ AFF      │ MEM      │
├─────────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ Extension   │ .dd/.img │ .s01     │ .E01     │ .aff     │ .mem/.dmp│
│ Compression │ ❌ No    │ ✅ Yes   │ ✅ Yes   │ ✅ Yes   │ ❌ No    │
│ Metadata    │ ❌ No    │ ✅ Yes   │ ✅ Yes   │ ✅ Yes   │ N/A      │
│ Encryption  │ ❌ No    │ ❌ No    │ ✅ Yes   │ ✅ Yes   │ No       │
│ Checksums   │ ❌ No    │ ✅ Yes   │ ✅ Yes   │ ✅ Yes   │ Varies   │
│ Segmented   │ ❌ No    │ Optional │ ✅ Yes   │ Optional │ No       │
│ Open Source │ ✅ Yes   │ ❌ No    │ ❌ No    │ ✅ Yes   │ ✅ Yes   │
│ Tool Support│ Wide     │ Limited  │ Wide     │ Moderate │ Wide     │
│ File Size   │ = Source │ Smaller  │ Smaller  │ Smaller  │ = RAM    │
└─────────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

### 4.3 Format Details

#### DD / RAW Format

```
Extension:   .dd  or  .img  or  .raw
Description: Exact bit-for-bit copy of a drive
             No headers, no metadata, no compression

Advantages:
  ✅ Universal compatibility — works with almost every tool
  ✅ Simple format — no proprietary overhead
  ✅ Can be created with built-in OS tools (dd command)

Disadvantages:
  ❌ Same size as source (no compression = large files)
  ❌ No metadata storage (no case number, examiner, etc.)
  ❌ No built-in integrity checking

Best for:
  → Quick imaging when tool compatibility is critical
  → Linux-based investigations
  → When simplicity is preferred over features
```

#### E01 Format (Expert Witness Format)

```
Extension:   .E01, .E02, .E03, ... (segmented)
Developed by: Guidance Software (now OpenText)
Description: Most widely used forensic image format in legal proceedings

Advantages:
  ✅ Stores full case metadata (case #, examiner, date, notes)
  ✅ Built-in MD5 and SHA-1 checksums per segment
  ✅ Compression reduces storage requirements
  ✅ Automatically splits into 2GB segments
  ✅ Supported by: EnCase, FTK, Autopsy, X-Ways, Magnet Axiom

Disadvantages:
  ❌ Proprietary format (not open standard)
  ❌ Not supported by every tool

Best for:
  → Court evidence (most recognized format)
  → When metadata storage is required
  → When storage space is a concern
  → Professional investigations

Segment naming:
  evidence.E01  ← First segment (main file)
  evidence.E02  ← Second segment
  evidence.E03  ← Third segment
  (Open E01 and all segments load automatically)
```

#### AFF Format (Advanced Forensic Format)

```
Extension:   .aff
Developed by: Simson Garfinkel (open source)
Description: Open-source forensic format designed for long-term evidence preservation

Advantages:
  ✅ Fully open source — no vendor lock-in
  ✅ Works with any compatible tool
  ✅ Supports compression, metadata, AND encryption
  ✅ Designed specifically for forensic evidence

Disadvantages:
  ❌ Less widely supported than E01
  ❌ Moderate tool adoption

Best for:
  → When vendor neutrality is important
  → Long-term evidence archiving
  → When encryption of evidence is required
```

#### SMART Format

```
Extension:   .s01
Full Name:   Simple Management Artifact Recovery Technology
Description: Proprietary to SMART Forensic software

Advantages:
  ✅ Supports compression
  ✅ Stores metadata and timestamps

Disadvantages:
  ❌ Only works with SMART forensic tool
  ❌ Very limited tool support

Best for:
  → Use only when working specifically with SMART software
```

#### MEM / Memory Image Format

```
Extension:   .mem, .dmp, .raw
Description: Image of volatile memory (RAM)
             NOT used for hard drives — only for RAM dumps

Why RAM matters forensically:
  → Running processes captured at time of imaging
  → Encryption keys may be in memory
  → Network connections visible
  → User activity (typed passwords, clipboard)
  → Malware that only exists in RAM (fileless malware)

Key rule: Image RAM BEFORE shutting down a running system
          (RAM is cleared on shutdown!)
```

### 4.4 Format Selection Guide

```
Question: What do you need?

Need maximum compatibility?          → Use DD/RAW
Need court-ready evidence?           → Use E01
Need vendor independence?            → Use AFF
Need to capture running PC's RAM?    → Use MEM/DMP
Working only with SMART software?    → Use SMART

For most professional investigations: E01 is the recommended standard
```

---

## 5. Creating Forensic Images — Practical Lab

### 5.1 Creating E01 Format with FTK Imager (GUI)

FTK Imager is free and produces court-ready E01 images.

**Download:** https://www.exterro.com/ftk-imager

```
Step 1: Open FTK Imager → File → Create Disk Image

Step 2: Select source type:
   Physical Drive  → Entire drive (all partitions)
   Logical Drive   → Single partition (C:, D:, etc.)
   Image File      → Image of an existing image (for conversion)

Step 3: Select your target drive from the list
        (use a hardware write blocker!)

Step 4: Click "Add" to add destination
   Select image type: E01 (EnCase Image File Format)

Step 5: Fill in evidence metadata
   Case Number:        CASE-2025-001
   Evidence Number:    E001
   Unique Description: USB Drive 16GB SanDisk
   Examiner:           [Your Name]
   Notes:              Seized from suspect on 2025-08-21

Step 6: Set destination path
        → Select a DIFFERENT drive (NOT the source drive)
        → Name: evidence_usb_2025-08-21
        → Image fragment size: 0 (auto) or 2000 MB

Step 7: Options
   ✅ Verify images after they are created
   ✅ Create directory listings

Step 8: Start imaging → Finish
   Progress bar shows completion
   On finish: MD5 and SHA1 hash values shown

Step 9: Save verification report
   Shows: Source hash vs. Destination hash (must match!)
```

**After imaging, you will have:**

```
evidence_usb_2025-08-21.E01       ← First segment
evidence_usb_2025-08-21.E02       ← Second segment (if >2GB)
evidence_usb_2025-08-21.txt       ← Hash and metadata report
```

### 5.2 Creating DD Format with Command Line (Windows)

**Using the dd tool for Windows:**

```powershell
# Step 1: List all disks to find target
wmic diskdrive list brief /format:list

# Output example:
# DeviceID = \\.\PHYSICALDRIVE0   ← Internal C: drive
# DeviceID = \\.\PHYSICALDRIVE1   ← External / USB drive
# Caption = Generic-       Mass Storage USB Device

# Step 2: Navigate to dd tool folder
cd C:\tools\dd\

# Step 3: Create DD image
# if = input file (source drive)
# of = output file (destination — must be different drive!)
# bs = block size (512 or 64k for speed)
# --progress = show progress

dd if=\\.\PHYSICALDRIVE1 of=E:\evidence\drive_image.dd bs=512k --progress

# Creating image of a specific partition (logical drive):
dd if=\\.\G: of=E:\evidence\partition_g.dd bs=512k --progress
```

**DD image characteristics:**

```
Size:     Exactly same as source (no compression)
Format:   Raw binary — no headers or metadata
Metadata: None embedded — document separately in case notes!
Hash:     Calculate manually after creation:
```

```powershell
# Calculate and record hashes after dd imaging
Get-FileHash E:\evidence\drive_image.dd -Algorithm MD5
Get-FileHash E:\evidence\drive_image.dd -Algorithm SHA256
```

### 5.3 Creating DD Format with dd on Linux/Kali

```bash
# List drives
lsblk
fdisk -l

# Basic dd imaging
dd if=/dev/sdb of=/mnt/external/evidence.dd bs=4M status=progress

# With hash generation during imaging (dc3dd)
apt install dc3dd
dc3dd if=/dev/sdb of=/mnt/external/evidence.dd hash=sha256 log=/mnt/external/hash.log

# Verify image matches source
sha256sum /dev/sdb
sha256sum /mnt/external/evidence.dd
# Must be identical!

# Creating E01 on Linux (using ewfacquire)
apt install libewf-dev ewf-tools
ewfacquire /dev/sdb
# Prompts for case info → creates evidence.E01
```

### 5.4 Verifying Image Integrity

After imaging, **always verify** the hash of source matches the image:

```bash
# Linux
md5sum /dev/sdb                          # Source hash
md5sum /mnt/external/evidence.dd         # Image hash
# Must match!

# Windows (PowerShell)
Get-FileHash \\.\PHYSICALDRIVE1 -Algorithm MD5   # Source (may need admin)
Get-FileHash evidence.dd -Algorithm MD5          # Image hash
```

---

## 6. File Recovery via Command Line

### 6.1 Lab: Recovering All Files with tsk_recover

**tsk_recover** extracts all allocated AND unallocated files from a forensic image.

```bash
# Syntax
tsk_recover [options] <image_file> <output_directory>

# Create output directory first
mkdir C:\forensics\recovered_files\

# Recover all files (allocated + unallocated)
tsk_recover -f ntfs -e "C:\forensics\images\windows.dd" "C:\forensics\recovered_files\"

# Options explained:
# -f ntfs    → specify file system type
# -e         → recover both allocated AND unallocated files
# (without -e, only allocated files are recovered)

# For a specific partition offset (from mmls):
tsk_recover -f ntfs -o 2048 -e "C:\forensics\images\windows.dd" "C:\forensics\recovered_files\"
```

**After recovery you will find:**

```
C:\forensics\recovered_files\
├── orphan\                    ← Files with no directory entry (truly deleted)
│   ├── file001.jpg
│   ├── file002.docx
│   └── ...
├── Users\
│   └── [username]\
│       ├── Documents\
│       ├── Downloads\
│       └── Desktop\
├── Windows\
├── $Recycle.Bin\              ← Recycle bin contents
└── ...
```

**Note on errors:**
Errors during tsk_recover are normal and expected. Ignore `File not found` errors unless you see command syntax errors. The final count of recovered files is what matters.

### 6.2 Lab: Targeted File Recovery with icat

**icat** extracts the content of a specific file by its MFT entry number.

```bash
# First find the file's MFT entry number using fls
fls -r -f ntfs "windows.dd" | grep "secret"
# Output: r/r 1234-128-1:  secret_document.docx

# Extract the file using its entry number (1234)
icat -f ntfs "windows.dd" 1234 > "C:\forensics\recovered\secret_document.docx"

# Verify the recovered file's hash
sha256sum "C:\forensics\recovered\secret_document.docx"
```

### 6.3 Building a File Timeline

```bash
# Create a body file (timeline data for all files)
fls -r -m / -f ntfs "windows.dd" > bodyfile.txt

# Convert to timeline (mactime)
mactime -b bodyfile.txt -d > timeline.csv

# Open timeline.csv in Excel for analysis
# Sort by date to see sequence of events
```

---

## 7. WinHex — Advanced Data Recovery

### 7.1 What Is WinHex?

WinHex is a professional hex editor and forensic tool that operates at the **raw byte level**, allowing you to:

- View and edit raw disk sectors
- Recover deleted files by file type (file carving)
- Analyze disk structures manually
- Perform memory analysis
- Search for specific byte patterns

**Download:** https://www.x-ways.net/winhex/

### 7.2 Lab: File Recovery by Type (File Carving)

File carving recovers files based on their **magic bytes** (file signature) regardless of file system damage.

```
Every file type starts with a unique byte signature:

File Type    Magic Bytes (Hex)       ASCII
JPEG         FF D8 FF                ÿØÿ
PNG          89 50 4E 47             .PNG
PDF          25 50 44 46             %PDF
ZIP          50 4B 03 04             PK..
EXE/DLL      4D 5A                   MZ
DOCX         50 4B 03 04             PK.. (ZIP-based)
MP4          00 00 00 ?? 66 74 79 70 ....ftyp
```

**Using WinHex for file carving:**

```
Step 1: Open WinHex as Administrator
Step 2: File → Open (select your forensic image .dd or .E01)
Step 3: Tools → Disk Tools → File Recovery by Type
Step 4: Select file types to recover:
        ✅ Photos (JPG, PNG, GIF, BMP)
        ✅ Documents (DOC, DOCX, PDF, XLS)
        ✅ Videos (MP4, AVI, MOV)
        ✅ Archives (ZIP, RAR)
        ✅ Email (PST, EML)
Step 5: Set output folder (different drive from source!)
Step 6: Click OK → Start Recovery
Step 7: Review recovery log
        Shows: X files found, Y complete, Z incomplete/corrupt
```

**WinHex recovery output:**

```
FileRecovery_Type_Log.txt — summary of all recovered files
Recovered folders:
├── JPG\        → recovered images
├── PDF\        → recovered documents
├── ZIP\        → recovered archives
└── MP4\        → recovered videos

Note: Incomplete/corrupt files are partially recoverable
      Open them anyway — may contain useful partial data
```

### 7.3 Viewing Raw Hex Data

```
WinHex shows data in three columns:
┌──────────┬──────────────────────────────────────┬──────────────────┐
│ Offset   │ Hex Values (raw bytes)               │ ASCII Text       │
├──────────┼──────────────────────────────────────┼──────────────────┤
│ 00000000 │ FF D8 FF E0 00 10 4A 46 49 46 00 01  │ ÿØÿà..JFIF..    │
│ 00000010 │ 01 00 00 01 00 01 00 00 FF E1 16 24  │ ........ÿá..$   │
└──────────┴──────────────────────────────────────┴──────────────────┘

Reading this:
FF D8 FF = JPEG magic bytes → this is the start of a JPEG file
The ASCII column shows readable text embedded in binary files
```

---

## 8. Memory Forensics Basics

### 8.1 Why RAM Forensics Matters

RAM (Random Access Memory) is **volatile** — it is erased when the computer is shut down. It can contain:

```
✦ Running processes at time of seizure
✦ Open network connections
✦ Encryption keys (TrueCrypt, BitLocker keys!)
✦ Recently typed passwords
✦ Clipboard contents
✦ Browser history (not yet written to disk)
✦ Fileless malware (exists only in memory)
✦ Decrypted versions of encrypted files
✦ Injected malicious code in legitimate processes
```

**The golden rule: If the computer is ON, image RAM FIRST before shutting it down.**

### 8.2 Creating a RAM Dump

**Using WinPMEM (free):**

```powershell
# Download winpmem from GitHub
# Run as Administrator
winpmem.exe C:\forensics\ram_dump.mem

# The .mem file is your RAM image
```

**Using FTK Imager:**

```
FTK Imager → File → Capture Memory
Output file: C:\forensics\ram_dump.mem
Include pagefile: ✅ Yes (pagefile contains overflow from RAM)
```

### 8.3 Analyzing RAM with Volatility (Free)

Volatility is the industry-standard RAM forensics framework.

**Download:** https://www.volatilityfoundation.org/

```bash
# Install Volatility 3
pip install volatility3

# Identify the OS profile
vol -f ram_dump.mem windows.info

# List running processes
vol -f ram_dump.mem windows.pslist

# Find hidden/injected processes
vol -f ram_dump.mem windows.pstree
vol -f ram_dump.mem windows.malfind

# Network connections at time of capture
vol -f ram_dump.mem windows.netstat

# List all DLLs loaded by each process
vol -f ram_dump.mem windows.dlllist

# Extract command line history
vol -f ram_dump.mem windows.cmdline

# Find files referenced in memory
vol -f ram_dump.mem windows.filescan

# Dump a specific process's memory
vol -f ram_dump.mem windows.memmap --pid 1234 --dump

# Extract registry hives from memory
vol -f ram_dump.mem windows.registry.hivelist
```

**Forensic value of RAM analysis:**

```
Scenario: Suspect encrypted files with VeraCrypt, then minimized it
          (encryption software still running in RAM)

RAM analysis reveals:
  → VeraCrypt.exe running (process list)
  → Encryption master key in memory (malfind / memdump)
  → Mounted encrypted volume path
  → Files recently accessed within encrypted container

→ This is impossible to find on disk after shutdown!
```

---

## 9. Cheat Sheet — All Commands

### TSK (The Sleuth Kit) Commands

```bash
# ── Partition Analysis ────────────────────────────────────
mmls image.dd                          # List partition table
mmls -t gpt image.dd                   # GPT partition table

# ── File System Analysis ──────────────────────────────────
fsstat -f ntfs image.dd                # NTFS file system status
fsstat -f fat32 image.dd               # FAT32 file system status
img_stat image.dd                      # Image file information

# ── MFT / Metadata Analysis ───────────────────────────────
istat -f ntfs image.dd 0               # MFT entry #0
istat -f ntfs image.dd 2               # LogFile entry
istat -f ntfs image.dd 5               # Root directory
istat -f ntfs image.dd 1234            # Specific file entry

# ── File Listing ─────────────────────────────────────────
fls -f ntfs image.dd                   # List all files
fls -r -f ntfs image.dd                # Recursive list
fls -d -f ntfs image.dd                # Deleted files only
fls -r -p -f ntfs image.dd            # Full paths

# ── File Recovery ─────────────────────────────────────────
tsk_recover -f ntfs -e image.dd /output/          # Recover all files
icat -f ntfs image.dd 1234 > output.docx          # Extract specific file

# ── Timeline Creation ─────────────────────────────────────
fls -r -m / -f ntfs image.dd > body.txt           # Create body file
mactime -b body.txt -d > timeline.csv             # Generate timeline
```

### Imaging Commands

```bash
# ── Linux/Kali Imaging ───────────────────────────────────
dd if=/dev/sdb of=/mnt/ext/evidence.dd bs=4M status=progress
dc3dd if=/dev/sdb of=/mnt/ext/evidence.dd hash=sha256 log=hashes.log
ewfacquire /dev/sdb                    # Create E01 format

# ── Hash Verification ─────────────────────────────────────
md5sum evidence.dd
sha256sum evidence.dd
sha512sum evidence.dd

# ── Mount Image (Read-Only) ───────────────────────────────
ewfmount evidence.E01 /mnt/ewf/
mount -o ro,loop /mnt/ewf/ewf1 /mnt/evidence/
```

### Windows PowerShell

```powershell
# List all physical disks
wmic diskdrive list brief /format:list

# Calculate hashes
Get-FileHash evidence.dd -Algorithm MD5
Get-FileHash evidence.dd -Algorithm SHA256
Get-FileHash evidence.dd -Algorithm SHA512

# Compare two hashes (verification)
(Get-FileHash source.dd -Algorithm SHA256).Hash -eq `
(Get-FileHash image.dd -Algorithm SHA256).Hash
```

### Volatility Memory Forensics

```bash
# Basic analysis
vol -f ram.mem windows.info            # OS information
vol -f ram.mem windows.pslist          # Running processes
vol -f ram.mem windows.pstree          # Process tree
vol -f ram.mem windows.netstat         # Network connections
vol -f ram.mem windows.cmdline         # Command line args
vol -f ram.mem windows.malfind         # Hidden/injected code
vol -f ram.mem windows.filescan        # File handles in memory
vol -f ram.mem windows.hashdump        # Password hashes from memory
```

---

## 📚 Key Concepts Summary

```
File System     → How OS organizes data on disk (NTFS for Windows)
Partition Table → Map of all partitions on a disk (MBR or GPT)
MFT             → Master File Table — index of all files in NTFS
MFT Entry       → Record for each file containing attributes
$STANDARD_INFO  → File timestamps (MACB times)
$FILE_NAME      → File name and parent directory
$DATA           → Actual file content
Slack Space     → Unused space at end of a cluster (contains old data)
Unallocated     → Disk space not part of any active file system
File Carving    → Recovery of files by their magic bytes, ignoring FS
Timeline        → Chronological list of file system events (key for investigations)
Write Blocker   → Hardware/software that prevents writes to evidence drive
Volatile Memory → RAM — cleared on shutdown; image first!
```

---

_This guide covers The Sleuth Kit command-line forensics, NTFS MFT structure, all forensic image formats, practical imaging with FTK Imager and dd, file carving with WinHex, and memory forensics with Volatility — all for educational use in authorized forensic lab environments._
