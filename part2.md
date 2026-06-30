## 5. Network Forensics — Deep Dive

### Capturing Traffic Properly for Evidentiary Use

```bash
# Full packet capture, not just headers — you want payloads for evidence
sudo tcpdump -i eth0 -s 0 -w capture_$(date +%Y%m%d_%H%M%S).pcap
# -s 0 = capture full packet (snaplen), not truncated

# Hash the capture immediately after stopping it, same as a disk image:
sha256sum capture_*.pcap

# For high-volume/long-term capture, rotate files to avoid one giant
# unwieldy file and disk exhaustion:
sudo tcpdump -i eth0 -s 0 -w capture_%Y%m%d_%H%M%S.pcap -G 3600
# -G 3600 = new file every hour

# Capturing on a SPAN/mirror port (passive, doesn't touch traffic) is
# the forensically cleanest method — no risk of dropped packets from
# an inline device, and no chance of the capture host injecting traffic.
```

### Wireshark — Forensic Filter Cheatsheet (Expanded)

```
# CREDENTIAL HARVESTING
http.request.method == "POST" && http contains "password"
ftp.request.command == "PASS"
pop.request.command == "PASS"
imap.request contains "LOGIN"

# DATA EXFILTRATION INDICATORS
http.request.method == "POST" && http.content_length > 1000000
dns.qry.name.len > 50                    # abnormally long DNS queries
                                           # (DNS tunneling indicator)
frame.len > 1000 && udp.port == 53        # oversized DNS packets

# LATERAL MOVEMENT / SMB
smb2.cmd == 3                             # SMB tree connect (share access)
smb2.cmd == 5 && smb2.flags.response == 0 # SMB file create requests

# COMMAND & CONTROL BEACONING
# Beaconing often shows REGULAR INTERVALS to the same destination.
# Use Statistics > Conversations, sort by duration/packets, look for
# suspiciously regular timing — a real human doesn't poll a server
# every exactly 60.000 seconds for 8 hours straight.

# TLS/SSL INSPECTION (can't see encrypted content, but CAN see metadata)
tls.handshake.type == 1                   # ClientHello — reveals SNI
                                           # (target domain) even when
                                           # the rest of the session is
                                           # encrypted
tls.handshake.extensions_server_name contains "suspicious-domain"

# ARP-BASED MITM DETECTION
arp.duplicate-address-detected
# Wireshark flags this when it sees conflicting ARP replies for the
# same IP — classic ARP spoofing/MITM indicator

# ICMP TUNNELING
icmp && data.len > 64                     # normal pings are ~32-64
                                           # bytes of data; larger ICMP
                                           # payloads can indicate
                                           # ICMP-based C2 or exfil
```

### Zeek (formerly Bro) — Automated Traffic Analysis at Scale

```bash
# Zeek doesn't just capture — it generates structured, queryable logs
# for every connection, protocol, and file transfer it sees, which
# scales far better than manually scrolling Wireshark for large captures.

zeek -r capture.pcap
ls *.log
#   conn.log     → every connection (5-tuple, duration, bytes, state)
#   http.log     → every HTTP request/response with metadata
#   dns.log      → every DNS query/response
#   ssl.log      → every TLS handshake (including JA3 fingerprints!)
#   files.log    → every file Zeek saw transferred, with hashes
#   notice.log   → Zeek's own built-in anomaly detections

# JA3 / JA3S fingerprinting (in ssl.log) identifies the CLIENT
# APPLICATION generating TLS traffic based on its handshake
# characteristics, even through encryption — a known JA3 hash for
# Cobalt Strike or Metasploit's TLS stub is a strong C2 indicator
# even if you can't decrypt the payload.

# Query conn.log for top talkers:
cat conn.log | zeek-cut id.orig_h id.resp_h | sort | uniq -c | sort -rn | head

# Find all files Zeek extracted and carved automatically:
ls extract_files/
```

### NetworkMiner — Passive Reconstruction

```
NetworkMiner reconstructs, from a pcap, without active scanning:
  - Files transferred (extracted and saved automatically)
  - Images viewed
  - Credentials sent in cleartext protocols
  - Hostnames, OS fingerprints of hosts observed
  - Sessions, organized per-host rather than per-packet

This "per-host" view is often more useful for an investigator than
Wireshark's packet-by-packet view — it answers "what did THIS machine
do across the whole capture" directly.

Run via: Windows native, or under Wine/in a Windows VM on Linux.
```

### DNS Forensics — Reconstructing Browsing History from Network Logs

```bash
# Even when HTTPS hides content, DNS queries reveal EVERY domain looked
# up — often the single richest source of "what did this device visit."

tshark -r capture.pcap -Y "dns.flags.response == 0" \
  -T fields -e frame.time -e ip.src -e dns.qry.name | sort -u

# Passive DNS replication concept: many enterprises log all DNS queries
# centrally (via DNS server logs or a tool like PassiveDNS) precisely
# so they have months of "what domain did this IP resolve" history
# even after the device is gone — ask if such logs exist before
# assuming the pcap is your only DNS evidence source.
```

### Building a Network Forensics Timeline Across Multiple Sources

```
The strongest network forensic case correlates MULTIPLE independent
log sources for the same event, because no single source is 100%
reliable or complete:

  Firewall/perimeter logs    → connection allowed/denied, timestamps
  IDS/IPS alerts (Suricata/Snort) → signature-based detections
  DNS server query logs       → what was resolved, by whom, when
  DHCP lease logs               → which device had which IP at that time
                                    (CRITICAL — IP addresses get reused!)
  Proxy logs                    → full URL, user-agent, sometimes user identity
  NetFlow/sFlow records         → connection metadata even without full pcap
  Endpoint logs (the host itself) → process that made the connection

EXAMPLE: "IP 10.0.5.23 contacted a known C2 server at 14:32" is
meaningless on its own in a DHCP environment — you MUST pull the DHCP
lease log to prove WHICH physical machine/user had that IP at 14:32
before you can attribute the activity to a specific suspect.
```

### Suricata — Signature-Based Detection on Captured Traffic (Offline)

```bash
sudo apt install -y suricata
# Run Suricata against an already-captured pcap (not just live traffic)
# to flag known-malicious signatures retroactively:
suricata -r capture.pcap -c /etc/suricata/suricata.yaml -l ./suricata_output/
cat ./suricata_output/fast.log
cat ./suricata_output/eve.json | jq '.alert'
```

---

## 6. File System Forensics — NTFS / EXT4 / APFS Internals

### NTFS Internals — Deep Dive

```
NTFS is built entirely around the $MFT (Master File Table) — literally
EVERY file and directory, including system metadata files themselves,
has an entry (1024 bytes each) in the $MFT.

KEY METADATA FILES (all start with $, all hidden):
  $MFT           → the table itself
  $MFTMirr       → backup copy of first 4 MFT entries (disaster recovery)
  $LogFile       → transaction log (NTFS journaling) — records FS
                    operations for crash recovery, ALSO a forensic
                    goldmine for recently deleted/renamed/modified files
  $Volume        → volume label, NTFS version, dirty flag
  $AttrDef       → attribute definitions
  $Bitmap        → tracks which clusters are allocated/free
  $Boot          → boot sector
  $BadClus       → bad cluster list
  $Secure        → security descriptors (permissions)
  $UpCase        → uppercase conversion table
  $Extend\$UsnJrnl  → USN Change Journal — logs EVERY file system
                       change (create/modify/delete/rename) with a
                       reason code and timestamp; even more granular
                       than $LogFile for reconstructing user activity

EACH MFT ENTRY contains ATTRIBUTES, the two most forensically critical:
  $STANDARD_INFORMATION (SI) → the "normal" MAC(B) timestamps shown by
                                most tools and easily modified by
                                attacker tools (timestomping utilities
                                target THIS attribute)
  $FILE_NAME (FN)            → a SECOND, independent set of MAC(B)
                                timestamps, set by the FILESYSTEM
                                itself and much harder for userland
                                attacker tools to modify

THE KEY ANTI-TIMESTOMPING TECHNIQUE:
  Compare SI timestamps against FN timestamps using `istat` or
  MFTECmd. If SI shows "created 2019" but FN shows "created last
  Tuesday" — someone timestomped the file. This single check has
  cracked countless cases where an attacker tried to make a malicious
  file blend in with old system files.

  istat case001_disk1.E01 4521
  # Compare $STANDARD_INFORMATION vs $FILE_NAME blocks in the output
```

### Parsing the MFT Properly

```bash
# Eric Zimmerman's MFTECmd (run via mono on Linux, or native on Windows)
# is the standard tool — parses the entire $MFT into a CSV with EVERY
# file ever recorded, including deleted entries whose MFT slot hasn't
# been reused yet.

mono MFTECmd.exe -f mft.raw --csv ./output/ --csvf mft_parsed.csv

# Once in CSV, load into a spreadsheet or pandas and filter/sort by
# timestamp to build activity timelines, or grep for specific filenames
# the suspect might have tried to delete.
```

### USN Journal — Reconstructing "Was This File Ever Here, Even If Fully Gone"

```bash
# Unlike $MFT entries (which get reused and overwritten), $UsnJrnl
# records a ROLLING LOG of every change — so even after an MFT slot
# is reused, the journal may still show "file X was created at time Y
# and deleted at time Z" as a historical record, until the journal
# itself rolls over (it has a configurable max size, often a few hundred MB).

# Extract $UsnJrnl:$J data stream (the actual journal data):
icat case001_disk1.E01 <inode_of_UsnJrnl> > usnjrnl_j.raw

# Parse with Eric Zimmerman's tooling or:
python3 -m pip install --break-system-packages libusn  # community parser, verify reliability
# Reason codes in the journal tell you EXACTLY what happened:
#   USN_REASON_FILE_CREATE, USN_REASON_FILE_DELETE,
#   USN_REASON_DATA_OVERWRITE, USN_REASON_RENAME_OLD_NAME, etc.
```

### Volume Shadow Copies — Free Historical Snapshots

```
Windows VSS (Volume Shadow Copy Service) periodically snapshots the
filesystem for System Restore / backup purposes. These snapshots can
contain COMPLETE COPIES of files exactly as they were at snapshot time
— including files later deleted or modified by the suspect, sometimes
giving you a full "time machine" view without any special carving.

# On a mounted/live system, list shadow copies:
vssadmin list shadows

# Mount a specific shadow copy to browse it like a normal drive:
mklink /D C:\shadow_mount \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\

# From a forensic image, tools like Arsenal Image Mounter or
# ShadowExplorer can mount VSS snapshots embedded in the image for
# direct browsing — check these BEFORE concluding a file is
# unrecoverable, attackers frequently forget shadow copies exist.
```

### EXT4 (Linux) Internals

```
EXT4 KEY STRUCTURES:
  Superblock        → filesystem-wide metadata (size, block size,
                        last mount time, state)
  Inode table        → like NTFS's MFT conceptually — one inode per
                        file/directory, containing metadata + pointers
                        to data blocks
  Journal (jbd2)      → EXT4's journaling layer — similar forensic value
                          to NTFS's $LogFile, can reveal recent operations

  EXT4 only has 3 standard timestamps shown by `stat`:
    atime (access), mtime (modify), ctime (metadata change)
  PLUS, if the filesystem was created with extended attributes
  support: crtime (creation time) — check with `stat` (newer util-linux
  versions show this as "Birth:").

  stat suspicious_file
  debugfs -R "stat <inode_number>" /dev/sdb1   # deep inode inspection,
                                                 # works on a mounted
                                                 # IMAGE via loop device

# Deleted file recovery on EXT4 is harder than NTFS because EXT4 zeroes
# out the inode's block pointers on deletion by default (unlike NTFS
# which mostly just marks space free). extundelete and ext4magic exist
# but success rates are lower — emphasizes why CARVING (foremost/
# scalpel/photorec on raw unallocated space) is often more reliable
# on Linux targets than inode-based recovery:

sudo apt install extundelete
extundelete /dev/sdb1 --restore-all -o ./recovered/
```

### APFS (macOS/iOS) — Brief Orientation

```
APFS introduced major changes from the old HFS+ format that change
forensic approach:
  - Native snapshots (similar concept to VSS) — check for these first
  - Native encryption (FileVault) — without the key/password, you get
    nothing; key recovery often depends on memory forensics (the
    volume key is in RAM while mounted) or the user's iCloud recovery key
  - Copy-on-write — deleted/overwritten data may persist longer in
    "free" space than expected because of how COW allocates new blocks
    rather than overwriting in place

Tooling: Autopsy/Sleuth Kit have growing APFS support; for serious
macOS/iOS work, commercial tools (Cellebrite, Magnet AXIOM) currently
have meaningfully better APFS and iOS-specific support than open-source
options — worth knowing as a limitation when scoping what's achievable
with your free toolkit alone.
```

---

## 7. Log Analysis — Windows / Linux / Cloud (Expanded)

### Full Windows Event ID Reference (Beyond the Basics)

```
AUTHENTICATION
  4624  Successful logon       (check Logon Type! see below)
  4625  Failed logon
  4634  Logoff
  4647  User-initiated logoff
  4648  Logon with explicit credentials (runas) — lateral movement signal
  4672  Special privileges assigned to new logon (admin-equivalent token)
  4768  Kerberos TGT requested
  4769  Kerberos service ticket requested  — "Kerberoasting" attacks
        generate MASSIVE volumes of these for many different SPNs in
        a short window, a detectable anomaly
  4771  Kerberos pre-auth failed
  4776  NTLM authentication (credential validation)

LOGON TYPE CODES (critical context for event 4624/4625):
  2  = Interactive (physical console / RDP-equivalent in some configs)
  3  = Network (e.g. accessing a share — most common, also used by
       pass-the-hash attacks)
  4  = Batch (scheduled task)
  5  = Service (service account startup)
  7  = Unlock (workstation unlock)
  8  = NetworkCleartext (Basic Auth — password sent in clear, rare/bad)
  9  = NewCredentials (RunAs /netonly)
  10 = RemoteInteractive (RDP!)
  11 = CachedInteractive (logon using cached domain creds, no DC contact)

  A 4624 with Logon Type 3 from an unfamiliar internal IP, immediately
  followed by file access on a share, is a textbook lateral movement
  pattern.

ACCOUNT MANAGEMENT
  4720  New user account created
  4722  Account enabled
  4724  Password reset attempt
  4725  Account disabled
  4728  User added to a global security group
  4732  User added to a LOCAL group (esp. local Administrators!)
  4738  User account changed
  4756  User added to a UNIVERSAL group

SYSTEM / PERSISTENCE / TAMPERING
  7045  New SERVICE installed — classic persistence mechanism, ALWAYS
        review unexpected new services
  4697  Service installed (logged in Security log too, different source)
  1102  Security audit log CLEARED — extremely high-value alert; an
        attacker covering tracks paradoxically generates the loudest
        possible signal, because clearing the log itself is logged
  4719  System audit policy changed (attacker weakening future logging)
  4732  (see above)
  104   System log cleared (System log, not Security log)

POWERSHELL-SPECIFIC (Microsoft-Windows-PowerShell/Operational log)
  4103  Module logging — records cmdlet invocations and parameters
  4104  Script Block Logging — records the ACTUAL CODE of executed
        scripts, including DE-OBFUSCATED content for many obfuscation
        techniques. This is one of the single best sources for
        catching "powershell -enc ..." attacks if it was enabled —
        check whether it WAS enabled before assuming there's no
        PowerShell evidence.
```

### Sysmon — If It Was Installed, This Is Gold

```
Sysmon (System Monitor, a free Microsoft Sysinternals tool) logs FAR
more detail than default Windows auditing, IF it was installed and
configured before the incident:

  Event ID 1   Process creation (with FULL command line + hashes!)
  Event ID 3   Network connection (process-level, not just netstat)
  Event ID 5   Process terminated
  Event ID 7   Image/DLL loaded
  Event ID 8   CreateRemoteThread (process injection indicator)
  Event ID 10  ProcessAccess (e.g. something accessing lsass.exe —
               classic credential-dumping precursor)
  Event ID 11  File created
  Event ID 12/13/14  Registry events (create/modify/delete/rename)
  Event ID 22  DNS query (per-process!)

  Always check first: "Was Sysmon running on this machine, and what
  was its config (which Sysmon config XML)?" — if yes, this is often
  more useful than every other log source combined, because it directly
  links process → network → file → registry actions together.
```

### Parsing EVTX Files (Windows Event Logs) on Linux

```bash
pip3 install python-evtx --break-system-packages

# Convert EVTX to readable XML:
evtx_dump.py Security.evtx > security.xml

# Or use Eric Zimmerman's EvtxECmd (cross-platform via mono) which
# outputs clean CSV with built-in "MapInfo" enrichment (auto-resolves
# common event IDs into human-readable summaries):
mono EvtxECmd.exe -f Security.evtx --csv ./output/ --csvf security.csv

# Grep for specific event IDs quickly from XML:
grep -A 5 "EventID>4625" security.xml | grep "IpAddress\|TargetUserName"
```

### Linux Log Analysis — Beyond auth.log

```bash
# journalctl (systemd journal) — modern Linux distros centralize most
# logging here, even if /var/log/auth.log also exists in parallel:
journalctl --since "2026-06-28" --until "2026-06-30"
journalctl -u sshd                       # logs for a specific service
journalctl _PID=1234                     # logs from a specific PID
journalctl -k                            # kernel ring buffer messages
journalctl --list-boots                  # see all recorded boot sessions
                                          # (useful if suspect rebooted
                                          # to try to clear evidence —
                                          # the journal often persists
                                          # across reboots if persistent
                                          # logging is enabled)

# Audit daemon (auditd) — if configured, gives FAR more granular detail
# than auth.log alone, similar in spirit to Windows Sysmon:
ausearch -m USER_LOGIN                   # search audit log by event type
ausearch -ua <uid>                       # all events by a specific user
aureport --summary                       # high-level summary report
aureport --auth                          # authentication-specific report

# Shell history beyond .bash_history:
cat ~/.zsh_history
cat ~/.python_history
# If HISTFILE was unset or history was cleared, check:
#   - auditd logs (if execve auditing was configured)
#   - Sysmon-equivalent on Linux: auditd rules for execve syscalls
#   - The MEMORY capture (linux.bash.Bash plugin recovers history that
#     `history -c` does NOT remove from the live readline buffer)
```

### Cloud Forensics — Logs You Need to Know Exist

```
As more evidence lives in cloud services, you need to know WHERE
to ask providers/admins to pull logs from:

  AWS    → CloudTrail (API call history), VPC Flow Logs (network),
           S3 access logs, GuardDuty findings
  Azure  → Activity Log, Azure AD Sign-in logs, NSG Flow Logs,
           Microsoft Defender alerts
  GCP    → Cloud Audit Logs, VPC Flow Logs
  M365   → Unified Audit Log (covers Exchange, SharePoint, Teams,
           Azure AD combined) — accessible via Purview/Compliance
           portal or PowerShell (Search-UnifiedAuditLog)
  Google Workspace → Admin console Audit & investigation logs

These logs typically have RETENTION LIMITS (often 90 days by default
on lower-tier plans) — if cloud evidence is relevant, this is the
single most time-sensitive thing to request/preserve, often more
urgent than the local disk image, because cloud logs can roll off
and be unrecoverable while a local disk sits safely in evidence storage.
```

### Correlating Logs — A Worked Example

```
Scenario: suspected unauthorized access to a file server.

1. Firewall log    → external IP X connected to internal IP Y at 03:14
2. DHCP log        → internal IP Y was assigned to MAC address Z at
                      that time
3. Asset inventory → MAC address Z belongs to known workstation
                      "WKSTN-42", assigned to employee John Doe
4. Windows Security log on WKSTN-42 → Event 4624, Logon Type 3,
                      account "svc_backup" at 03:13 (one minute before
                      the firewall log)
5. File server Security log → Event 4663 (object access) for
                      \\FS01\confidential\salaries.xlsx at 03:15 by
                      "svc_backup"
6. Sysmon on WKSTN-42 → Event 1 shows powershell.exe launched at
                      03:12 by John Doe's interactive session, with a
                      command line referencing a credential-stuffing
                      script

CONCLUSION CHAIN: the workstation assigned to John Doe ran a PowerShell
script that authenticated as a service account NOT belonging to John,
accessed sensitive data, one minute before an external connection was
logged. Every link in this chain is independently sourced and
timestamped — this is what a court-ready correlation looks like, versus
a single log entry in isolation.
```
