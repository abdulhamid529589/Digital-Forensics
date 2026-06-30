## 8. Mobile Forensics — Android / iOS Deep Dive

### Acquisition Types — Know Which You're Getting

```
LOGICAL ACQUISITION
  What it gets: data accessible through normal OS APIs — contacts,
  messages, call logs, app data the OS exposes, photos.
  Easiest, fastest, works on locked phones with debugging enabled, but
  MISSES deleted data and anything outside the API surface.

FILE SYSTEM ACQUISITION
  What it gets: the full accessible file system (requires root/jailbreak
  or specific exploits). Sees app databases directly, SQLite files
  including deleted-but-not-vacuumed rows, more complete than logical.

PHYSICAL ACQUISITION (the gold standard, increasingly hard to get)
  What it gets: a bit-for-bit copy of the flash storage, including
  unallocated space — true equivalent of a disk image, can carve
  deleted data. Modern Android/iOS encryption-at-rest and locked
  bootloaders make this difficult/impossible without exploits, often
  requiring commercial tools (Cellebrite UFED, Magnet GrayKey/AXIOM)
  that exploit specific chipset/OS version vulnerabilities — which
  change constantly as vendors patch them.

CLOUD ACQUISITION
  What it gets: data backed up to iCloud/Google account — can
  sometimes recover MORE than what's on the device (older deleted
  messages still in a backup), or be the ONLY option if the physical
  device acquisition fails. Requires legal process directed at Apple/
  Google (subpoena/warrant), not a tool you run yourself.
```

### Android — Detailed Workflow

```bash
# ── PRE-ACQUISITION: Preserve device state ───────────────────────
# If device is unlocked and you have legal authority to interact:
#   - Enable airplane mode (prevents remote wipe commands, but be
#     aware some wipe mechanisms work even without connectivity)
#   - DO NOT let the screen lock if you don't have the passcode —
#     getting back in may be impossible
#   - Photograph the screen state before any interaction

adb devices                              # confirm device sees you
adb shell getprop ro.build.version.release   # Android version —
                                               # determines what's possible

# ── LOGICAL BACKUP (works without root on many devices) ──────────
adb backup -apk -shared -all -f full_backup.ab
# Note: many modern apps set android:allowBackup="false" specifically
# to block this — expect partial results on hardened/recent devices.

# Convert .ab to a readable tar (older Android versions, format varies):
( printf "\x1f\x8b\x08\x00\x00\x00\x00\x00" ; tail -c +25 full_backup.ab ) | tar xfvz -

# ── IF ROOTED / FILE-SYSTEM ACCESS AVAILABLE ─────────────────────
adb shell "su -c 'tar -czf /sdcard/full_dump.tar.gz /data/data/'"
adb pull /sdcard/full_dump.tar.gz ./

# ── KEY ANDROID ARTIFACT LOCATIONS (expanded) ─────────────────────
# /data/data/[package]/databases/      → SQLite app data
# /data/data/[package]/shared_prefs/   → XML preference files (often
#                                         contain auth tokens, settings)
# /data/data/[package]/cache/           → cached data, sometimes
#                                          includes downloaded media
#                                          the user never explicitly saved
# /data/system/users/0/settings_*.xml   → device-wide settings
# /data/misc/wifi/WifiConfigStore.xml   → saved WiFi SSIDs + PSKs
#                                          (often recoverable even when
#                                          encrypted at rest, format
#                                          varies by Android version)
# /data/system/usagestats/              → per-app usage time stats —
#                                          can show app was opened at a
#                                          specific time even with no
#                                          other evidence
# /data/system/recent_tasks/             → recently used apps with
#                                           thumbnails (visual evidence
#                                           of what was on screen!)

# ── SQLITE FORENSICS — RECOVERING DELETED ROWS ────────────────────
# SQLite doesn't immediately erase deleted rows — they often remain in
# unallocated pages within the .db file until VACUUM is run or the
# page is reused. Tools that read raw pages (not just SQL queries)
# can recover this:

pip3 install --break-system-packages sqlite-dissect
python3 -m sqlite_dissect -o ./output/ msgstore.db
# Parses the SQLite file format directly — including the WAL
# (Write-Ahead Log) and rollback journal files if present alongside
# the main .db file, which often contain recently committed OR
# recently rolled-back transactions not yet in the main file:
ls msgstore.db-wal msgstore.db-journal 2>/dev/null
# ALWAYS collect these companion files alongside the .db — examining
# the .db alone without its -wal file can mean missing the most recent
# activity, since WAL-mode SQLite may not have merged recent writes
# into the main file yet.

# Standard query interface for allocated (non-deleted) data:
sqlite3 msgstore.db ".tables"
sqlite3 msgstore.db "SELECT * FROM messages ORDER BY timestamp DESC LIMIT 50;"
```

### WhatsApp-Specific Forensics

```bash
# WhatsApp encrypts its local database backups (msgstore.db.crypt12/14/15)
# The encryption key, on Android, is derived and can sometimes be
# recovered from:
#   /data/data/com.whatsapp/files/key   (raw key file, if accessible
#                                         via root)

# Decrypt with a known tool (wa-crypt-tools or similar, verify current
# maintained fork before relying on it for a real case):
pip3 install --break-system-packages wa-crypt-tools
wadecrypt key msgstore.db.crypt15 msgstore_decrypted.db

sqlite3 msgstore_decrypted.db "SELECT key_remote_jid, data, timestamp FROM messages ORDER BY timestamp DESC;"
```

### iOS Forensics — Key Differences

```
iOS is dramatically more locked-down than Android by design:
  - Full disk encryption tied to the Secure Enclave hardware — without
    the passcode (or a chip-level exploit), physical/file-system
    acquisition is generally not possible with open-source tools.
  - iTunes/Finder backups (if unencrypted-backup mode is off, password-
    protected) are often your MOST PRACTICAL acquisition path if you
    can get the device unlocked or already have an existing backup.

# Working with an iTunes/Finder backup (if you have access to one,
# e.g. from a seized computer that synced with the phone):
pip3 install --break-system-packages iphone_backup_decrypt
# (verify currently maintained library before relying on it)

# Backup structure (unencrypted backup):
# Manifest.db (SQLite) maps obfuscated file names back to real paths —
# parse this FIRST to know what you're looking at:
sqlite3 Manifest.db "SELECT fileID, domain, relativePath FROM Files WHERE relativePath LIKE '%sms%';"

# Key iOS artifact locations (post Manifest.db lookup):
#   HomeDomain/Library/SMS/sms.db        → iMessage/SMS database
#   HomeDomain/Library/CallHistoryDB/    → call history
#   CameraRollDomain/Media/DCIM/          → photos
#   HomeDomain/Library/Safari/History.db  → Safari browsing history

# Without a backup and without exploit-based tooling, your realistic
# options narrow to: (a) logical sync data if device is unlocked,
# (b) iCloud data via legal process to Apple, (c) commercial forensic
# tools that license specific jailbreak/exploit chains for the exact
# iOS version in question. This is a genuine, current limitation worth
# being upfront about with investigators rather than overpromising.
```

### Detecting Spyware / Stalkerware (MVT — Mobile Verification Toolkit)

```bash
pip3 install --break-system-packages mvt

# Designed originally to detect Pegasus-style spyware, but useful
# generally for identifying indicators of compromise on a device:
mvt-android check-androidqf --output ./mvt_results/ ./androidqf_dump/
mvt-ios check-backup --output ./mvt_results/ ./ios_backup/

# MVT checks against a database of known malicious indicators
# (domains, process names, file hashes) maintained by Amnesty
# International's Security Lab — useful in cases involving suspected
# targeted surveillance, e.g. stalkerware in domestic violence cases,
# which is an area where police digital forensics support is
# increasingly needed.
```

---

## 9. Malware Forensics — Static & Dynamic (Deep Dive)

### Safe Handling — Before You Touch Anything

```
RULE 1: Never execute on your analysis host. Ever. Even "just to see."
RULE 2: Work in an isolated VM with NO shared folders, NO clipboard
        sharing, networking either fully OFF or routed through a
        controlled fake-internet (INetSim/FakeNet) so the malware
        thinks it has internet but never reaches anything real.
RULE 3: Snapshot the VM BEFORE every execution. Restore after every run.
RULE 4: Never email/upload/share the live binary outside controlled
        channels — rename with a non-executable extension and/or
        password-protect zip (password: infected) when transferring,
        a long-standing community convention to prevent accidental
        execution or AV auto-deletion in transit.
```

### Static Analysis — Going Deeper

```bash
# ── HASHING AND THREAT INTEL LOOKUP ───────────────────────────────
sha256sum malware.exe
# Check the hash (NOT the file itself, to avoid uploading sensitive/
# confidential samples publicly) against:
#   VirusTotal API (hash search, not file upload, if confidentiality matters)
#   MalwareBazaar (abuse.ch)
#   Hybrid-Analysis

# ── PE FILE STRUCTURE ANALYSIS ─────────────────────────────────────
sudo apt install -y pev
peheader malware.exe              # PE header overview
pesec malware.exe                  # security features (ASLR/DEP/etc enabled?)
pestr malware.exe                  # extract printable strings from PE sections specifically

pip3 install --break-system-packages pefile
python3 -c "
import pefile
pe = pefile.PE('malware.exe')
print('Compile timestamp:', pe.FILE_HEADER.TimeDateStamp)
print('Sections:')
for s in pe.sections:
    print(f'  {s.Name.decode().strip()}: entropy={s.get_entropy():.2f}')
print('Imports:')
for entry in pe.DIRECTORY_ENTRY_IMPORT:
    print(f'  {entry.dll.decode()}')
    for imp in entry.imports:
        if imp.name:
            print(f'    {imp.name.decode()}')
"
# A section with entropy > 7.0 is almost certainly packed/encrypted —
# legitimate code sections typically sit in the 5.5-6.5 range.
# The .text section having abnormally high entropy while file size is
# small is a classic packer signature.

# ── DANGEROUS API IMPORTS — KNOW WHAT TO LOOK FOR ─────────────────
# Process injection:    CreateRemoteThread, VirtualAllocEx,
#                        WriteProcessMemory, NtMapViewOfSection
# Keylogging:            SetWindowsHookEx, GetAsyncKeyState
# Persistence:            RegSetValueEx, CreateService
# Anti-debugging:          IsDebuggerPresent, CheckRemoteDebuggerPresent,
#                          OutputDebugString (timing-based detection)
# Anti-VM:                 various CPUID checks, looking for VM-specific
#                          registry keys/MAC address OUI prefixes
# Network:                  WinINet/WinHTTP functions, raw Winsock
#                          (connect/send/recv), or higher-level
#                          InternetOpenUrl

# ── YARA RULES — PATTERN-MATCHING AT SCALE ─────────────────────────
sudo apt install -y yara
# YARA lets you write/use rules matching byte patterns, strings, or
# structural conditions across many files at once — essential for
# "does this match a known malware family" or "scan this whole disk
# image for anything matching known IOCs":

cat > rule.yar << 'EOF'
rule Suspicious_PowerShell_Encoded
{
    strings:
        $a = "-enc" nocase
        $b = "-EncodedCommand" nocase
        $c = "FromBase64String"
    condition:
        any of them
}
EOF
yara rule.yar malware.exe
yara -r rule.yar /mnt/evidence/   # recursive scan across mounted evidence

# Public rule sets worth knowing (verify license/use terms):
#   YARA-Rules project (github.com/Yara-Rules/rules)
#   Florian Roth's signature-base (used by THOR/Loki scanners)

# ── DISASSEMBLY / DECOMPILATION ────────────────────────────────────
# Ghidra (free, NSA-released, full decompiler) — the standard free tool
# for serious static reverse engineering:
#   Import binary → Auto-analyze → browse decompiled C-like pseudocode
#   per function, cross-reference strings to find where suspicious
#   API calls are actually used in logic, not just listed in imports.

# radare2 / Cutter (GUI front-end for r2) — scriptable, fast, good for
# quick triage before committing to a full Ghidra session:
r2 -A malware.exe
[0x00401000]> afl              # list all detected functions
[0x00401000]> pdf @ main       # disassemble the main function
```

### Document/Macro Malware (Common in Phishing Cases)

```bash
# Office documents (.docm, .xlsm) carrying VBA macro malware are
# extremely common in real-world phishing — and don't require full
# binary reverse engineering, just macro extraction:

pip3 install --break-system-packages oletools
olevba suspicious.docm
# Extracts and DEOBFUSCATES VBA macro code automatically where
# possible, flags suspicious keywords (Shell, CreateObject,
# AutoOpen, AutoExec — auto-run triggers) with risk ratings.

mraptor suspicious.docm          # quick macro-malware likelihood triage
oleid suspicious.docm            # general document risk indicators

# PDF malware (embedded JS, exploit-laden PDFs):
pip3 install --break-system-packages peepdf
peepdf -i suspicious.pdf
# Interactive shell to inspect objects, extract embedded JavaScript:
#   PDF> object 12
#   PDF> js_analyse 12
```

### Dynamic Analysis — Controlled Detonation

```bash
# ── FAKENET-NG — Simulate the internet so malware "succeeds" at its
#    callbacks without ever reaching a real attacker server ──────────
# (Run on the analysis VM, before executing the sample)
sudo fakenet-ng
# Responds to ANY DNS query with the local machine's IP, runs fake
# HTTP/HTTPS/FTP/DNS/SMTP listeners — captures EXACTLY what the
# malware tried to send and to which domains, without risking
# anything reaching the real internet or alerting the actual attacker
# infrastructure that their malware was caught.

# ── PROCMON — File/Registry/Process activity (Windows VM) ──────────
# Filter to just the malware's PID after launch, export to CSV,
# review for: files dropped, registry keys set (persistence), child
# processes spawned.

# ── REGSHOT — Before/after registry diff ────────────────────────────
# Take a shot before execution, run the malware, take a second shot
# after, generate a diff — fast way to spot every registry change
# without manually combing through Procmon's firehose.

# ── NOTING BEHAVIOR FOR YOUR REPORT ──────────────────────────────────
# Document explicitly, with timestamps and screenshots:
#   - Process tree created (parent/child relationships)
#   - Files dropped (path, name, hash of EACH dropped file — they may
#     be secondary payloads worth their own static analysis pass)
#   - Registry persistence mechanism established (exact key/value)
#   - Network destinations contacted (domain, IP, port, protocol)
#   - Any privilege escalation or process injection observed

# ── ONLINE SANDBOXES (when sharing the sample externally is acceptable) ──
# any.run        → interactive, lets you click through prompts the
#                   malware shows, good for human-interaction-gated
#                   payloads
# Hybrid-Analysis → automated, generates a detailed behavioral report
#                   and IOC list automatically
# Note: for an active police investigation, uploading a sample to a
# THIRD-PARTY PUBLIC SANDBOX may alert sophisticated attackers (they
# monitor sandbox submission feeds for samples matching their own
# campaigns) and could itself need legal/procedural sign-off — check
# your department's evidence-handling policy before uploading
# case-related malware externally, even to "just check a hash."
```

### Building an IOC List From Your Analysis

```
A proper malware forensics deliverable always ends with a structured
Indicator of Compromise list, so other systems/investigators can be
checked for the same compromise:

  File hashes (SHA256 of the original sample AND any dropped files)
  File paths used for persistence
  Registry keys/values set
  Domains/IPs contacted
  Mutex names created (malware often creates a named mutex to avoid
    re-infecting the same machine twice — a unique, searchable string)
  YARA rule (write one capturing this specific sample's unique strings/
    structure, so future disk/memory scans can find it elsewhere)

Format as STIX/MISP-compatible JSON if your department shares IOCs
with other agencies or threat-intel platforms — a plain table is fine
for a single-case report, but structured format lets the indicators be
mechanically ingested elsewhere.
```

---

## 10. Email Forensics — Headers, Spoofing, Phishing (Deep Dive)

### Full Header Trace — Reading Received Headers Correctly

```
Received headers are added by EACH mail server the message passes
through, and are stacked with the NEWEST (closest to recipient) at the
TOP of the header block. To trace the ORIGIN, read from the BOTTOM
Received header upward.

Example header (annotated):
  Received: from mail.recipient-domain.com (mail.recipient-domain.com [203.0.113.5])
        by mx.gmail.com with ESMTPS id ...;          ← hop 3 (last, closest to recipient)
        Tue, 30 Jun 2026 14:32:10 -0700
  Received: from relay.middledomain.com (relay.middledomain.com [198.51.100.9])
        by mail.recipient-domain.com with ESMTP id ...;   ← hop 2
        Tue, 30 Jun 2026 14:32:05 -0700
  Received: from [192.0.2.77] (unknown [192.0.2.77])      ← hop 1 (ORIGIN —
        by relay.middledomain.com with SMTP id ...;          this is the
        Tue, 30 Jun 2026 14:31:58 -0700                      sender's actual
                                                               originating IP)

KEY PRINCIPLE: the FIRST (bottommost) Received header showing an
EXTERNAL IP not belonging to any known mail infrastructure is your best
candidate for the true origin. Be aware: each hop can be FORGED by
whoever controls that hop, BUT once the message reaches a server you
DON'T control and don't trust the sender to control (e.g. it transited
through Gmail's own infrastructure), THAT header becomes reliable,
since Gmail's servers wrote it truthfully based on what they actually
observed.
```

### Authentication Results — SPF / DKIM / DMARC Explained

```
SPF (Sender Policy Framework)
  Domain owner publishes a DNS TXT record listing which mail servers
  are authorized to send mail "from" that domain. Receiving server
  checks if the connecting IP matches.
  Header: Received-SPF: pass / fail / softfail / neutral
  A "fail" or "softfail" on a message claiming to be from a trusted
  domain is a strong phishing indicator — but SPF only validates the
  ENVELOPE sender, which the visible "From:" header CAN differ from.

DKIM (DomainKeys Identified Mail)
  Sending server cryptographically SIGNS the message with a private
  key; receiving server verifies using a public key published in DNS.
  Header: DKIM-Signature: ... ; Authentication-Results: dkim=pass/fail
  A valid DKIM pass means the message body WAS NOT MODIFIED after
  leaving the signing server, and that server held the private key for
  that domain — strong authenticity evidence, much stronger than SPF
  alone, when present and valid.

DMARC (Domain-based Message Authentication, Reporting & Conformance)
  Domain owner's policy for what to do when SPF/DKIM fail (none/
  quarantine/reject), PLUS a reporting mechanism.
  Header: Authentication-Results: ... dmarc=pass/fail

PRACTICAL FORENSIC CHECK:
  grep -i "Authentication-Results" email.eml
  All three failing on a message impersonating a known domain is about
  as close to definitive spoofing evidence as header analysis alone
  provides.
```

### Reply-To / Return-Path Mismatch — Classic BEC Indicator

```
Business Email Compromise (BEC) and phishing frequently use a
legitimate-looking "From" display name with a DIFFERENT actual reply
address, banking on the victim not checking:

  From: "John Smith - CEO" <john.smith@realcompany.com>
  Reply-To: j.smith.ceo@gmail-secure-mail.com    ← red flag

  grep -i "^From:\|^Reply-To:\|^Return-Path:" email.eml

Also check the Return-Path (where bounces go) — a Return-Path domain
unrelated to both the From AND Reply-To domains is another inconsistency
worth flagging explicitly in a report.
```

### Extracting and Analyzing Attachments Safely

```bash
# Use the python email_forensics.py approach from the base guide, but
# ALWAYS treat extracted attachments as live malware until proven
# otherwise — hash them immediately and route through the malware
# forensics workflow in Section 9 above before opening with anything
# other than a hex editor / `file` / `exiftool`.

file attachment_invoice.pdf
exiftool attachment_invoice.pdf   # check creation tool, author metadata
                                    # — attacker-built PDFs often retain
                                    # metadata from the tool that
                                    # generated them, sometimes even a
                                    # username or machine name

# For Office docs with macros — see Section 9's olevba workflow.
# For PDFs with embedded JS/exploits — see Section 9's peepdf workflow.
```

### URL Analysis Without Clicking

```bash
# Extract all URLs from the raw email source without rendering/
# fetching anything:
grep -oE 'https?://[^\s">]+' email.eml | sort -u

# Decode common URL obfuscation techniques attackers use:
#   - URL shorteners (bit.ly, tinyurl) — resolve via a HEAD request
#     ONLY from an isolated/throwaway analysis environment, never your
#     real machine, since some shortened links execute drive-by exploits
#     on visit:
curl -sI --max-time 5 "https://bit.ly/suspiciouslink" | grep -i location

#   - Look-alike domains (typosquatting): paypa1.com, micros0ft.com,
#     rnicrosoft.com (rn rendered to look like m) — manually eyeball
#     character-by-character, automated tools can also flag Unicode
#     homoglyph attacks (Cyrillic 'а' instead of Latin 'a', visually
#     identical but a different domain entirely):
python3 -c "
url = 'http://аpple.com/login'  # contains a Cyrillic 'а'
print([hex(ord(c)) for c in url if ord(c) > 127])
"
```

### Email Forensics Report Checklist

```
For a phishing/BEC case, your write-up should explicitly cover:
  ☐ Full Received header chain with origin IP identified
  ☐ SPF/DKIM/DMARC results and what they mean for authenticity
  ☐ From/Reply-To/Return-Path consistency check
  ☐ All URLs extracted and individually assessed (shortened? typosquat?)
  ☐ All attachments hashed and individually analyzed
  ☐ Sender IP geolocation/ASN lookup (context, not proof — note this
    explicitly, geolocation by IP is approximate and easily routed
    through VPNs/proxies, don't overstate its certainty in a report)
  ☐ Whether the same sender infrastructure appears in any other
    known case/campaign (check IOC databases)
```
