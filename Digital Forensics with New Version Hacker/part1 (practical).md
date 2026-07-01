# Digital Forensics — Practical Lab Notes (Part 1)

**Course:** Digital Forensics
**Type:** Practical / Hands-on Lab
**Prerequisite:** Theory module (6-hour theory video) — recommended to review before this practical, as this session assumes familiarity with core forensic concepts.

---

## Table of Contents

1. [Introduction & Lab Overview](#1-introduction--lab-overview)
2. [Practical 1: Data Recovery](#2-practical-1-data-recovery)
3. [Practical 2: File Integrity Verification using Hashing](#3-practical-2-file-integrity-verification-using-hashing)
4. [Practical 3: Hash-Based Malware Identification](#4-practical-3-hash-based-malware-identification)
5. [Practical 4: Forensic Disk Imaging](#5-practical-4-forensic-disk-imaging)
6. [Practical 5: Investigating a Forensic Image](#6-practical-5-investigating-a-forensic-image)
7. [Key Takeaways](#7-key-takeaways)
8. [Tools Used in This Session](#8-tools-used-in-this-session)

---

## 1. Introduction & Lab Overview

This session is the **practical companion** to a prior 6-hour theory video on digital forensics. The theory video covered concepts but did not include hands-on demonstrations, so this session focuses **purely on practical execution** of forensic techniques used in real cybercrime investigations.

**Topics covered in this part:**

- Data recovery from storage media
- Hash calculation and integrity verification
- Hash-based malware/file-type identification
- Creation of forensic disk images
- Initial steps of investigating a forensic image using case-management tools

---

## 2. Practical 1: Data Recovery

### 2.1 Concept

Data recovery is one of the most well-known branches of digital forensics. Criminals often **delete files** to hide evidence of a crime. Forensic investigators use data recovery techniques to retrieve this deleted data and use it as evidence.

### 2.2 Tool Used

- **SS Data Recovery** (demo version used for practical)
- Note: Many alternative data recovery tools exist online. Different tools have different recovery capabilities — if one tool fails to recover certain data, try another.

### 2.3 Lab Setup

1. A USB pen drive (~16 GB) was connected to the system as a test/demo storage device — representing evidence media in a real investigation.
2. The drive was confirmed to be empty (no visible data) before the demonstration.

### 2.4 Step-by-Step Procedure

**Step 1 — Create test evidence**

- A text file named `Secret.txt` was created on the drive.
- Sample content written: _"Hello Cyber Warriors, Please Subscribe This Channel."_
- File was saved using `Ctrl + S`.

**Step 2 — Delete the evidence**

- File was deleted using **Shift + Delete** (permanent deletion — bypasses the Recycle Bin).
- Recycle Bin was also checked and emptied to confirm no trace remained through normal means.
- Drive appeared completely empty afterward.

**Step 3 — Select the drive for recovery**

- In SS Data Recovery, all connected drives were listed (OS drive, internal storage, and the two partitions of the USB pen drive — labeled F and G).
- The **target partition (G)** — the one the test file was deleted from — was selected for scanning.

**Step 4 — Scan the drive**

- A **full scan** was run on partition G.
- As scanning progressed, the tool recovered far more data than the drive's actual current capacity suggested — a key forensic insight:

> **Why recovered data can exceed the visible/current partition size:**
> Deleted data isn't immediately erased from the physical media; it remains until overwritten. A tool scanning at a deep (raw/sector) level can recover old, previously deleted files left behind by _previous_ uses of the drive — not just the most recent deletion. This is why a 1–2 GB partition yielded 4+ GB, then later 86–110+ GB of recoverable data on a different partition. (Note: This figure includes overlapping/duplicate recoverable fragments and "lost" raw file remnants, not necessarily all uniquely usable files.)

- Recovered items were categorized into groups such as:
  - **Deleted Files** — items matching known deletion records (e.g., the `Secret.txt` test file appeared here).
  - **Lost Files** — raw/unstructured file fragments (RAW/SHOCKWAVE/MOV/etc.) without clear filesystem metadata.

**Step 5 — Recover the target file**

- The deleted `Secret.txt` file was located under "Deleted Files," selected, and recovered.

> ⚠️ **Critical Forensic Rule:** Never recover/restore data onto the _same_ drive or partition it was recovered from. This risks overwriting un-recovered evidence and compromises the integrity of the source media.
> In this lab, recovered data was restored to a **separate destination drive (D:)**.

**Step 6 — Verify recovery**

- The recovered `Secret.txt` file was opened and confirmed to contain the original text exactly as written before deletion.
- File timestamps were also examined — recovered files showed dates ranging from very recent to extremely old (some appearing from years prior), demonstrating that deep scans can recover historical data left by previous drive usage, not just the most recent deletion event.

### 2.5 Second Partition Demonstration

- The tool was run again on the **second partition (F)** of the same USB drive.
- This partition's "available" reported size was small, but the deep scan recovered a disproportionately large volume of historical data (word documents, Excel files, photos, music, font files, Recycle Bin remnants, etc.), some with timestamps over a decade old.
- This reinforced the lesson: **recovery scope depends on the tool's capability and the physical media's history**, not just its current logical capacity.

### 2.6 Key Lessons from Data Recovery Practical

- Data recovery is foundational to retrieving evidence criminals attempt to destroy.
- Different recovery tools have different depth/capability — use multiple tools if one is insufficient.
- Always recover evidence to a **separate destination**, never back onto the source.
- Recovered data must later be **verified for authenticity** (covered next, via hashing) before it can be presented in court.

---

## 3. Practical 2: File Integrity Verification using Hashing

### 3.1 Concept

When evidence (such as a recovered file) is presented in court, the investigator must **prove it is the exact same, unaltered file** that was originally recovered — not a modified or substituted version.

**Analogy used:** Just as an Aadhar card (a national ID) uniquely identifies a person, a **hash value** acts as a unique "fingerprint" or ID number for a digital file.

### 3.2 Tool Used

- **Hash Calculator** (a general-purpose hash calculation tool; multiple similar tools exist online)

### 3.3 Hashing Algorithms Discussed

| Algorithm   | Notes                                                                  |
| ----------- | ---------------------------------------------------------------------- |
| **MD5**     | Older algorithm, historically common, considered weaker today          |
| **SHA-256** | Most commonly used algorithm currently                                 |
| **SHA-512** | Stronger / more commonly used than SHA-256 in higher-security contexts |

> Different algorithms are essentially different mathematical "formulas" for generating a fixed-length fingerprint of a file's contents. A tool can compute several algorithms on the same file simultaneously for comparison.

### 3.4 Step-by-Step Demonstration

**Step 1 — Baseline hash**

- The recovered `Secret.txt` file was located (note: investigators must carefully track _exactly_ where recovered evidence is stored — in this demo, some time was spent locating the correct destination folder, illustrating the importance of maintaining a clear **chain of custody log**).
- The original hash value (MD5 used as the reference/base) was calculated and saved to a notepad file labeled "Original" for comparison.

**Step 2 — Test: Modify content**

- A single **period (`.`)** was added to the file's text content.
- The file was saved and the hash recalculated.
- **Result:** The hash value changed completely — confirming that even a single-character change alters the entire hash output. This is the core principle of **hash-based integrity verification**.

**Step 3 — Test: Reverting the change**

- The added period was removed, but an **extra space character** was accidentally left behind.
- Hash recalculated — **still did not match** the original, because the space itself constituted a content change.
- Only after the space was also removed did the hash value return to match the original exactly.

**Conclusion:** _Any_ modification to file content — no matter how small (a character, a space) — produces a completely different hash value.

**Step 4 — Test: Renaming the file**

- The file was renamed (e.g., from `Secret.txt`/`Secret TXT` to simply `Secret`).
- Hash recalculated — **hash value remained identical** to the original.
- **Conclusion:** Hash values are based purely on file _content_, not on the filename or metadata like the name. Renaming a file does **not** affect its hash/integrity value.

### 3.5 HMAC (Hash-based Message Authentication Code)

- HMAC extends basic hashing by incorporating a **secret key** into the hash computation.
- When a key is set, the resulting hash will only match if:
  1. The file content is unchanged, **and**
  2. The same key is used during verification.
- Demonstrated behavior:
  - Calculating HMAC with key `"test"` produced one hash value.
  - Changing the key to `"NVH"` (with the same file content) produced a **different** hash value.
- **Use case:** HMAC adds an additional layer of authentication (proving the verifier also knows a shared secret), but for standard courtroom evidence presentation, **plain hash values (without HMAC) are typically used** for simplicity, since judges and court officials need an easily explainable, straightforward verification method.

### 3.6 Why This Matters for Court Evidence

- A hash value calculated **at the time of recovery** is recorded and preserved.
- At any later point (e.g., in court), the same file can be re-hashed.
- If the hash values match → the file's integrity is proven intact (unaltered).
- If they don't match → the file has been tampered with since recovery.
- Investigators should present this process to judges in **simple, clear terms**, since court officials generally are not technical experts — the explanation should focus on demonstrable, repeatable verification rather than deep technical jargon.

---

## 4. Practical 3: Hash-Based Malware Identification

### 4.1 Concept

Beyond proving file integrity, hash values also serve a **second major purpose**: identifying whether a file is **malicious** (e.g., a virus, Trojan, or other malware) by comparing its hash against databases of known-malicious file signatures.

### 4.2 Tool Used

- **VirusTotal** (a well-known public malware-scanning/aggregation website)
- A supplementary hash calculator was also used to independently generate the file's hash for cross-verification.

> ⚠️ **Safety Note (from the instructor):** This demonstration was performed in an **isolated practical lab environment** specifically built for this purpose, containing intentionally stored sample malicious files. Viewers/students were explicitly warned **not** to replicate this on personal or production machines, as handling live malware samples can cause real harm to a system.

### 4.3 Step-by-Step Demonstration

**Step 1 — Select a sample malicious file**

- A pre-stored sample file (from a folder containing various malware/Trojan samples kept for lab/testing purposes) was copied to a working directory.

**Step 2 — Upload to VirusTotal**

- The file was uploaded to VirusTotal.
- VirusTotal computes the file's hash and checks it against its aggregated database of antivirus engine results.
- **Result:** The file was flagged as malicious (a Trojan) by a large number (≈72) of antivirus/security vendor engines integrated into VirusTotal.

**Step 3 — Cross-verify via hash value**

- The file's hash (MD5) was also independently calculated using a separate hash calculator tool.
- This hash was then manually searched/submitted on VirusTotal (via hash lookup rather than file upload).
- **Result:** The same malicious classification was returned, confirming that file identification/classification can be done **purely from the hash value**, without needing to re-upload or directly handle the file each time.

### 4.4 Key Lessons

A hash value serves two major forensic purposes:

1. **Integrity verification** — confirming a file hasn't been altered.
2. **Identity/classification lookup** — determining what a file _is_ (including whether it's malicious), by checking its hash against threat-intelligence databases, even for compressed/archived files.

---

## 5. Practical 4: Forensic Disk Imaging

### 5.1 Concept

**Forensic imaging** is the process of creating an exact, bit-for-bit copy of a storage device or partition, called a **forensic image**. This image is what investigators actually work with and analyze — **the original evidence drive is never directly examined**, to avoid any risk of accidental modification or data loss to the source evidence.

### 5.2 Tool Used

- **R-Drive Image** (offers a 30-day free trial)
- Other industry tools mentioned: **FTK Imager**

### 5.3 Imaging Options

When creating an image, you can choose to image:

1. **An entire physical drive** (combining all partitions), or
2. **A single specific partition** of a drive.

In the demo, the USB pen drive had multiple partitions plus areas of "unallocated space" (deleted/free space). The investigator chose to image the **smallest partition first** to save time during the demonstration, but explained that in real casework, you select based on where the relevant evidence resides.

### 5.4 Step-by-Step Imaging Procedure

**Step 1 — Launch R-Drive Image and select "Create Image"**

**Step 2 — Select source**

- All connected drives/partitions are listed (system partitions, the OS drive, and the USB drive's partitions).
- The relevant partition was selected (deselecting irrelevant ones to reduce imaging time).

**Step 3 — Select destination**

> ⚠️ **Same rule as data recovery applies here:** The image must **never** be saved to the same drive being imaged. A separate destination drive was attached and selected (e.g., drive H).

**Step 4 — Configure imaging options**

- **Verification:** Option to verify the image immediately after creation (recommended for evidentiary integrity).
- **Compression:** Choice between faster speed (less compression) or smaller file size (more compression).
- **Backup mode:** "Actual data only" vs. **sector-by-sector** imaging (a full sector-by-sector copy captures everything, including deleted/unallocated data — generally preferred for forensic completeness).
- **Image size:** Automatic or fixed.
- **Password protection:** Optional, for securing the image file.
- **Backup/snapshot option:** Default Windows Volume Snapshot service was used (captures a consistent point-in-time copy).

**Step 5 — Review and start**

- The tool displays a summary (source, destination, compression settings, estimated size/duration) before starting.
- The imaging process was initiated and completed successfully.

**Step 6 — Resulting image file**

- The completed forensic image file (e.g., `Image.RDR` format in this case, named for later reference) was saved to the destination drive.
- This image file can now be used for analysis **as if it were the original drive**, without any risk to the actual physical evidence.

### 5.5 Key Lessons

- Forensic imaging preserves the original evidence completely untouched.
- All subsequent investigation/analysis is performed only on the image copy.
- Sector-by-sector imaging is preferred in forensic contexts because it captures deleted and unallocated data too, not just active/visible files.
- Always record/calculate a hash of the image immediately after creation, to later prove the image itself has not been altered since acquisition (chain of custody).

---

## 6. Practical 5: Investigating a Forensic Image

### 6.1 Tool Used

- A case-management/forensic analysis tool (referred to as a "TopC"-style tool in the session — functions similarly to industry-standard forensic suites such as Autopsy or EnCase) used to formally open, catalog, and analyze a forensic image as part of a structured **case file**.

### 6.2 Step-by-Step Procedure

**Step 1 — Create a new case**

- Case Name: e.g., "NVH – New Version Hacker" (example/demo name)
- **Base Directory:** Where all case data/output will be stored (selected as a separate drive, e.g., G:).
- **Case Number:** Assigned (can be any identifying reference per organizational policy).
- **Investigator/Examiner details:** Name and contact information of personnel involved are logged — standard practice for maintaining accountability and chain of custody.

**Step 2 — Add data source**

- Source type selected: **Disk Image** (as opposed to local file or unallocated space directly).
- The previously created forensic image file was selected as the source.
- Note: One image attempt was found to be corrupted, illustrating that investigators should always **verify image integrity** before relying on it for casework (a real-world reminder of why hash verification immediately post-imaging is critical) — a backup/alternate image file (in **E01 format**, a common forensic image format) was used instead.

**Step 3 — Configure analysis options**

- Options were presented to include/exclude specific data categories to recover/index (all were left checked for a full scan in this demo).

**Step 4 — Run analysis**

- The tool processes the image and builds an explorable case file structure showing:
  - File system contents
  - Deleted files
  - Downloaded files
  - Email messages (if present)
  - Other recoverable artifacts

- This indexing process can take significant time depending on image size and proceeds in the background.

### 6.3 Key Lessons

- Forensic case-management tools organize evidence into a structured, documented case file — essential for legal defensibility.
- Proper documentation (case number, investigator name, examiner name, base directory) supports **chain of custody**.
- Image integrity must be verified before analysis; corrupted images cannot be reliably used as evidence.
- This step is the bridge into deeper forensic analysis (to be covered in Part 2 of this practical series).

---

## 7. Key Takeaways

1. **Never investigate or recover data onto the original/source evidence drive** — always use a separate destination to preserve evidence integrity.
2. **Hash values are the cornerstone of evidence authenticity** — they prove a file has not been altered, and they uniquely "fingerprint" file content (not filename or metadata).
3. **Even a single character or space change alters a hash value entirely** — this is what makes hashing reliable for tamper detection.
4. **Hash values can also identify file type/maliciousness** by cross-referencing against threat-intelligence platforms like VirusTotal.
5. **Forensic imaging preserves original evidence** — all analysis work happens on a copy (the "image"), never on the original media.
6. **Sector-by-sector imaging** is preferred in forensic work as it captures deleted/unallocated data, not just visible files.
7. **Different tools have different capabilities** — a multi-tool approach is often necessary in real investigations, since no single tool recovers/analyzes everything equally well.
8. **Documentation and chain of custody** (case numbers, investigator details, timestamps, hash logs) are essential for evidence to be legally admissible in court.

---

## 8. Tools Used in This Session

| Tool                                       | Purpose                                                     |
| ------------------------------------------ | ----------------------------------------------------------- |
| **SS Data Recovery**                       | Recovering deleted files from storage media                 |
| **Hash Calculator**                        | Computing MD5/SHA-256/SHA-512 hashes and HMAC values        |
| **VirusTotal**                             | Hash-based malware/file identification                      |
| **R-Drive Image**                          | Creating forensic disk/partition images                     |
| **Forensic Case-Management Tool ("TopC")** | Creating and analyzing forensic case files from disk images |

---

_End of Part 1 — Practical continues in Part 2 with deeper image analysis._
