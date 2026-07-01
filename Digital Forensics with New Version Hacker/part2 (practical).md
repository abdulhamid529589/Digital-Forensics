# Digital Forensics — Practical Lab Notes (Part 2)

**Course:** Digital Forensics
**Type:** Practical / Hands-on Lab
**Continues from:** Part 1 (Data Recovery, Hashing/Integrity, Malware Identification via Hash, Forensic Imaging Basics)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [The Sleuth Kit (TSK) — Command-Line File System Analysis](#2-the-sleuth-kit-tsk--command-line-file-system-analysis)
3. [Understanding the Master File Table (MFT)](#3-understanding-the-master-file-table-mft)
4. [Listing and Recovering Files via Command Line](#4-listing-and-recovering-files-via-command-line)
5. [Data Recovery using WinHex](#5-data-recovery-using-winhex)
6. [Forensic Image Formats — Theory](#6-forensic-image-formats--theory)
7. [Creating Forensic Images — Practical (FTK Imager)](#7-creating-forensic-images--practical-ftk-imager)
8. [Creating a Raw (DD) Image via Command Line](#8-creating-a-raw-dd-image-via-command-line)
9. [Key Takeaways](#9-key-takeaways)
10. [Tools Used in This Session](#10-tools-used-in-this-session)

---

## 1. Introduction

This part picks up from where Part 1 ended — understanding the **Windows file system** in depth using command-line forensic tools, followed by a return to **data recovery** (this time using WinHex) and a detailed theoretical and practical exploration of **forensic image formats**.

**Topics covered in this part:**

- The Sleuth Kit (TSK) command-line toolset for file system analysis
- Partition table inspection (`mmls`)
- File system status inspection (`fsstat`)
- Image metadata/statistics inspection (`img_stat`)
- Master File Table (MFT) structure and metadata extraction (`istat`)
- Listing files (`fls`) and recovering files (`tsk_recover`) via command line
- Data recovery using WinHex
- Forensic image formats: RAW/DD, SMART, E01, AFF, and MEM
- Practical image creation using FTK Imager and the command-line `dd` tool

---

## 2. The Sleuth Kit (TSK) — Command-Line File System Analysis

### 2.1 About the Tool

**The Sleuth Kit (TSK)** is a well-known, free, command-line-based digital forensics tool used to analyze file systems and disk images. It does not require installation in the traditional sense — it is run directly from its folder via Command Prompt (CMD).

> **Note:** Another tool, Autopsy, is mentioned as a GUI counterpart that works alongside The Sleuth Kit, but this session focuses on the command-line usage.

### 2.2 Setting Up

1. Navigate to the folder where The Sleuth Kit is installed/extracted.
2. In the address bar of File Explorer, type `cmd` and press Enter — this opens Command Prompt **already pointed at that folder location**, saving the effort of manually navigating via `cd` commands.

### 2.3 `mmls` — Media Management List

**Purpose:** Displays the **partition table** of an evidence file (disk image) — i.e., all partitions present on the imaged drive, including allocated and unallocated space.

**Command structure:**

```
mmls "<path-to-evidence-image>"
```

**Key syntax notes:**

- `mmls` = **M**edia **M**anagement **L**i**s**t.
- Quotation marks (`" "`) around the file path are good practice (not strictly mandatory) — they correctly delimit where the file path starts and ends, especially important if the path contains spaces.
- **Common error encountered:** Forgetting to include a **space** between `mmls` and the quoted path causes a _"system cannot find the path specified"_ error. This is a frequent human/syntax error in command-line forensics work.

**What the output shows:**

- A full **partition table**, equivalent to what you'd see graphically in Windows Disk Management.
- Each partition is numbered (e.g., 0, 1, 2, 3...) and shows:
  - Start and end **sector** numbers
  - A description of each entry (e.g., partition type)
  - Whether the space is **allocated** or **unallocated**

> **Allocated vs. Unallocated Space:**
>
> - **Allocated space** — disk space currently assigned to a partition and usable/visible to the OS.
> - **Unallocated space** — space present on the physical disk but not assigned/usable until formatted/allocated. This is significant in forensics because deleted data often persists in unallocated space until overwritten.

**Getting help:** Typing `mmls` alone (without arguments) displays the tool's built-in help/usage information, including available flags.

**Useful flag — `-h` (hide):**

```
mmls -h "<path-to-evidence-image>"
```

Hides metadata/volume details for a cleaner view of just the partition table (the demonstration showed this toggling between showing and hiding metadata volumes).

---

### 2.4 `fsstat` — File System Status

**Purpose:** Displays detailed status information about the **file system** of a partition/image — including which operating system's file system is in use (e.g., NTFS) and detailed structural parameters.

**Command structure:**

```
fsstat "<path-to-evidence-image>"
```

**Tips:**

- Use the **Up Arrow key** in CMD to recall and edit previous commands instead of retyping the full path each time — a practical time-saver during repeated command-line forensic work.
- This command works reliably in **CMD**, not in PowerShell (noted by the instructor as a practical compatibility issue).

**Key output fields explained:**

| Field                               | Meaning                                                                                 |
| ----------------------------------- | --------------------------------------------------------------------------------------- |
| **Volume Serial Number**            | Unique identifier of the volume                                                         |
| **File System Type**                | e.g., NTFS                                                                              |
| **Volume Name**                     | The label given to the volume                                                           |
| **Master File Table (MFT) info**    | Indicates the system uses MFT to manage all file/folder metadata within the partition   |
| **Index/Record Range**              | Typically 0–128 for core system metadata entries; root directory is commonly at entry 5 |
| **Sector Size**                     | Commonly 512 bytes                                                                      |
| **Cluster Range**                   | Cluster size/allocation details                                                         |
| **Standard Information attributes** | Additional structural/eligible value details                                            |

> **What is the MFT?** The **Master File Table** is the core structure NTFS uses to store metadata about every file and folder in a partition — essentially a master index/catalog of the entire file system. This becomes the central focus of the next section.

**Troubleshooting note:** If `fsstat` returns _"Cannot determine file system,"_ it may indicate an incorrect/mismatched evidence file was specified. The instructor demonstrated swapping to a different evidence file to resolve this.

---

### 2.5 `img_stat` — Image Statistics

**Purpose:** Displays statistics/details about the **image file itself** (as opposed to the file system within it).

**Command structure:**

```
img_stat "<path-to-evidence-image>"
```

**Key output fields:**

- **Image Type:** e.g., RAW or DD (common raw, uncompressed image types)
- **Size in bytes**
- **Sector size**

---

## 3. Understanding the Master File Table (MFT)

### 3.1 Conceptual Overview

The **MFT** is the central structure NTFS uses to track every file and folder. It can be conceptually compared to:

- A **book's index**, where each chapter/section number corresponds to a specific category of information, or
- A **folder system**, where the MFT is the outer folder and each numbered "entry" inside it is like a sub-section containing specific details.

Each entry in the MFT is identified by a **number**, and querying a specific number reveals a corresponding category of metadata.

### 3.2 Extracting MFT Entry Details — `istat`

**Command structure:**

```
istat -f ntfs "<path-to-evidence-image>" <entry-number>
```

**Syntax notes:**

- `-i` is used for general image/info statistics; `-f` is used to **specify the file system type** (e.g., NTFS) — necessary because the system couldn't auto-determine it earlier.
- The **entry number must be separated by a space** from the rest of the command — a missing space was again demonstrated as a common source of command errors.
- `0` (zero) as the entry number refers to **entry zero of the MFT** — the root/primary metadata entry for the MFT itself.

### 3.3 Key MFT Entry Numbers and What They Represent

| Entry #   | Section                                       | Description                                                                                                                                                                          |
| --------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **0**     | $MFT (Master File Table itself)               | Complete metadata about the MFT; root entry point for system metadata                                                                                                                |
| **1**     | $MFTMirr (MFT Mirror)                         | A backup/duplicate copy of part of the MFT, used to **cross-verify/compare** in case of corruption — similar in principle to comparing two copies of evidence to confirm consistency |
| **2**     | $LogFile (Log File)                           | Records all actions/operations performed on the file system — i.e., a system activity log                                                                                            |
| **3**     | $Volume                                       | Stores volume-level information: when the volume was created, size allocated, etc.                                                                                                   |
| **4**     | $AttrDef (Attribute Definition Table)         | Defines all possible attribute types and their properties used across the file system                                                                                                |
| **5**     | (Root directory / standard info, second copy) | Standard information details, similar structure to entry 0 but representing root-level metadata                                                                                      |
| **6**     | $Bitmap                                       | Maps which clusters/sectors are allocated to which files — handles size/space mapping across the file system                                                                         |
| **7–8**   | $Boot / Bad Cluster file                      | Entry 7 relates to boot file data; entry 8 (`$BadClus`) lists **bad clusters** — clusters that exist physically but cannot reliably store data                                       |
| **9**     | $Secure                                       | Stores **security descriptor** data — permissions, ownership (e.g., administrator vs. local user access)                                                                             |
| **10–11** | $Upcase and related system files              | Case-mapping and other system definition tables                                                                                                                                      |
| **12–15** | Reserved                                      | Reserved for future system use; **no file name is associated** with these entries since they are placeholders                                                                        |
| **16**    | User directory / file entries begin           | Marks where actual user-created file and directory entries start being indexed                                                                                                       |

### 3.4 Sub-Attributes Within Each MFT Entry

When examining a specific MFT entry (e.g., entry 0), several **sub-sections (attributes)** appear within it:

1. **Standard Information**
   - Contains core timestamps and properties: **created date**, **last modified date**, **last MFT-record-modified date**, **last accessed date**.
   - Also includes an **Attribute ID** (used for verification/security key referencing).

2. **File Name**
   - Stores the actual **name of the file**, its **parent entry** (i.e., which directory it belongs to), file size (in bytes), and creation details specific to the name attribute.
   - Demonstrated distinction: Standard Information holds _when_ things happened; File Name holds _what it's called and where it sits in the hierarchy_.

3. **Data**
   - Contains (or points to) the **actual content/data** of the file — e.g., the real bytes of a document, image, or video stored within that entry.
   - Includes **size** and **initial size** fields (in bytes) describing how much actual data is stored.

4. **Bitmap** (within an entry context)
   - Performs **bit-mapping** — tracking how much space/size is allocated to which file, helping manage storage distribution at a granular level.

5. **Security Descriptor** (when present)
   - Contains security-related details: **permissions** and **ownership** (e.g., whether the administrator or a local user has access rights).

### 3.5 Practical Demonstration Summary

- Running `istat ... 0` displayed entry 0's full metadata (the MFT's own record), including header values like:
  - **Entry number**, **sequence number**, **log file sequence number**
  - **Allocated/unallocated status** of the file
  - **Link count**
- The instructor deliberately omitted required spaces in commands at multiple points to illustrate **common syntax errors** and how to self-correct them — reinforcing that most command-line errors in forensic tools are human/formatting errors rather than tool malfunctions.
- Running `istat` with different entry numbers (1 for MFT Mirror, 2 for Log File, 3 for Volume, etc.) returned the corresponding metadata sections as outlined in the table above.
- Entry 16 (reserved) was shown to return incomplete output (no file name) since it is a placeholder, reinforcing the reserved-entry concept.

---

## 4. Listing and Recovering Files via Command Line

### 4.1 `fls` — File List

**Purpose:** Lists all files (including deleted/orphan files) present within the file system of the evidence image.

**Command structure:**

```
fls -f ntfs "<path-to-evidence-image>"
```

**Notes:**

- `fls` = **F**i**l**e **L**i**s**t.
- The output includes regular files as well as **orphan files** (files with metadata entries but disconnected/orphaned references — effectively "useless" or incomplete file records).
- This step is useful for getting a full inventory of what exists (and what was deleted) within the file system before deciding what to recover.

### 4.2 `tsk_recover` — Recovering Files via Command Line

**Purpose:** Recovers files (both allocated and unallocated/deleted) directly from the evidence image into a specified destination folder — entirely through the command line, as an alternative to GUI-based recovery tools.

**Preparation:**

- Create an **empty destination folder** beforehand (confirmed empty before running the command) to receive recovered files.

**Command structure:**

```
tsk_recover -i raw -e "<path-to-evidence-image>" "<destination-folder-path>"
```

**Flag explanations:**

- `-i` — specifies the **image type** (e.g., `raw`).
- `-e` — instructs the tool to recover **both allocated and unallocated (deleted)** files (a comprehensive recovery, rather than only currently-existing files).
- A **space** must separate the destination path from the rest of the command (again flagged as a common error source).

**Result of the demonstration:**

- The recovery process completed successfully, recovering **1,097 files**.
- Minor errors during recovery are common and can generally be **ignored** unless they stem from an incorrect command syntax.
- Recovered content included:
  - Files originally found in the **Recycle Bin**
  - **Audio files** (songs)
  - **Images**
  - A **password-protected PDF** (confirmed still password-protected post-recovery — demonstrating that recovery preserves file-level protections)
  - **PPT files**, **text files**, and other miscellaneous documents

### 4.3 Key Lessons

- The Sleuth Kit provides a complete command-line workflow: partition inspection (`mmls`) → file system status (`fsstat`) → image statistics (`img_stat`) → MFT entry inspection (`istat`) → file listing (`fls`) → file recovery (`tsk_recover`).
- The MFT is the structural backbone of NTFS, and understanding its entry numbering scheme allows an investigator to manually inspect any category of file system metadata.
- Command-line forensic work is precise — small formatting errors (missing spaces, wrong flags) are common and must be carefully debugged, but they do not indicate tool failure.

---

## 5. Data Recovery using WinHex

### 5.1 About the Tool

**WinHex** is a hex-editing and data-recovery tool with a dedicated website for download. Like other forensic recovery tools, it is used to recover deleted files from a **copy (image)** of the evidence — reinforcing the principle that **original evidence is never directly investigated**.

### 5.2 Step-by-Step Procedure

**Step 1 — Open the evidence image**

- Launch WinHex.
- Use **File → Open** (or "Create New Case" workflow) and navigate to the location of the stored forensic image file.
- WinHex displays options like **Data Interpreter** settings (8-bit, 16-bit, 32-bit) for how data should be interpreted/displayed.

**Step 2 — Access File Recovery by Type**

- Navigate to: **Tools → Disk Tools → File Recovery by Type**.
- This presents categories of recoverable file types, including:
  - Images
  - Documents
  - Emails
  - Internet activity artifacts
  - Archive files
  - Music/Videos
  - Program files

**Step 3 — Select and recover**

- Select the desired file type/category to recover (a smaller file type was chosen in the demo for speed).
- Click **OK**, then specify a destination (e.g., Desktop) and confirm.
- The tool processes and recovers matching files to the chosen destination, showing progress (e.g., in MB processed).

**Step 4 — Review recovered files**

- Recovered files may be flagged with statuses such as:
  - **"File recovered"**
  - **"Mutilated"** (i.e., damaged/incomplete) or **"Corrupt"**
- This distinction matters forensically — a recovered file marked as corrupt/mutilated may still hold partial evidentiary value but should be noted as such in documentation.

### 5.3 Practical Observations

- Recovering a large number of files to one location (e.g., Desktop) can overload system resources/performance — the instructor experienced slowdowns from too many recovered files accumulating in one folder during the demo, and cleaned up afterward.
- WinHex serves as another tool in the forensic toolkit alongside SS Data Recovery (Part 1) and The Sleuth Kit's `tsk_recover` — reinforcing the lesson that **using multiple tools** increases the chances of comprehensive recovery.

---

## 6. Forensic Image Formats — Theory

### 6.1 Why This Matters

Just as regular photos/documents have formats (JPG, PNG, PDF, DOCX), **forensic disk images** also come in multiple standardized formats — each with different properties (compression, metadata support, tool compatibility). Understanding these is essential before creating an image, since the format chosen affects what data/metadata can be preserved and which tools can later open it.

### 6.2 RAW / DD Format

| Property             | Details                                                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **Definition**       | The exact, uncompressed, bit-by-bit (sector-by-sector) copy of the original media                                                            |
| **Extension**        | No fixed/special extension — commonly `.dd` or `.img`                                                                                        |
| **Compression**      | None — file size equals the original exactly                                                                                                 |
| **Metadata storage** | None — no case details, examiner info, or investigation metadata stored within the image itself                                              |
| **Tool support**     | Very wide — supported by Autopsy, The Sleuth Kit, FTK, EnCase ("Guy Manager" in transcript likely refers to EnCase), and most forensic tools |
| **Best for**         | Preserving an exact, unaltered copy of evidence where raw fidelity is the priority                                                           |

### 6.3 SMART Format

| Property             | Details                                                                                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Full form**        | Simple Management of Artifact Recovery Tools                                                                                                                              |
| **Tool support**     | Limited — primarily supported by its native tool ("SMART" forensic software), described as **proprietary** (tied to a specific tool, unlike RAW/DD's broad compatibility) |
| **Compression**      | Supported — can meaningfully reduce image size (e.g., compressing a 10GB image to ~5GB), useful when storage is limited during field investigations                       |
| **Metadata storage** | Supported — stores timestamps and investigation-related details                                                                                                           |
| **Checksum/Hash**    | Generates checksum/hash values automatically at creation time to support integrity verification                                                                           |
| **Advantage**        | Saves storage via compression                                                                                                                                             |
| **Disadvantage**     | Poor tool compatibility (proprietary/limited support)                                                                                                                     |

### 6.4 E01 (EnCase Evidence File) Format

| Property                   | Details                                                                                                                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Origin**                 | Not invented by any single specific forensic case workflow — it is a widely adopted industry-standard format (commonly associated with EnCase)                                                                                        |
| **Tool support**           | Broad — supported by many major forensic tools (e.g., FTK, Autopsy, EnCase-family tools)                                                                                                                                              |
| **Metadata storage**       | Yes — records full case details, examiner info, and investigation metadata, similar to SMART                                                                                                                                          |
| **Segmentation**           | Divides the image into **fixed-size segments/chunks** — commonly described in this session as ~2GB chunks (e.g., a 10GB drive could be split into multiple ~2GB segments) — making large images easier to handle, store, and transfer |
| **Compression**            | Supported — saves storage                                                                                                                                                                                                             |
| **Integrity verification** | Performs hash/checksum calculations automatically                                                                                                                                                                                     |
| **Limitation**             | Considered semi-proprietary — not universally supported by every single tool, though support is wide in practice                                                                                                                      |

### 6.5 AFF (Advanced Forensic Format)

| Property               | Details                                                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Key distinction**    | **Open-source** — unlike SMART and (to a lesser extent) E01, AFF is not tied to or restricted by any specific vendor/tool |
| **Tool support**       | Broad/unrestricted in principle, due to its open nature                                                                   |
| **Features supported** | Compression, metadata collection, **and encryption** (a feature not highlighted for the other formats in this session)    |
| **Design purpose**     | Specifically designed for forensic evidence preservation and analysis, combining the strengths of the other formats       |
| **Considered**         | A strong, flexible, "best of all worlds" option among the non-RAW formats                                                 |

### 6.6 MEM (Memory Image) Format

| Property                               | Details                                                                                                                                                                                                                     |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Definition**                         | Used specifically for capturing **volatile memory (RAM)** — i.e., a memory dump                                                                                                                                             |
| **Key distinction from other formats** | All other formats (RAW/DD, SMART, E01, AFF) are intended for **non-volatile** storage media (hard disks, drives — data that persists without power). MEM is for **volatile** memory, which is lost when a system shuts down |
| **When used**                          | If a system is found **powered on** during an investigation, a RAM/memory dump may be captured (in addition to disk imaging) before shutdown, since volatile data is otherwise lost                                         |
| **Tool support**                       | Broadly usable for any volatile memory dump scenario — effectively the standard/open approach for RAM imaging                                                                                                               |

### 6.7 Comparative Summary Table

| Feature                      | RAW/DD                          | SMART                   | E01                               | AFF                                | MEM                           |
| ---------------------------- | ------------------------------- | ----------------------- | --------------------------------- | ---------------------------------- | ----------------------------- |
| **Compression**              | ❌ No                           | ✅ Yes                  | ✅ Yes                            | ✅ Yes                             | N/A (memory-specific)         |
| **Metadata collection**      | ❌ No                           | ✅ Yes                  | ✅ Yes                            | ✅ Yes                             | N/A                           |
| **Encryption support**       | ❌ No                           | ❌ No                   | ❌ No                             | ✅ Yes                             | N/A                           |
| **File size (vs. original)** | Exact (1:1)                     | Reduced                 | Reduced                           | Reduced                            | N/A                           |
| **Tool compatibility**       | Wide                            | Limited                 | Wide (but not universal)          | Wide (open source)                 | Used for RAM dumps            |
| **Best suited for**          | Exact raw evidence preservation | Basic compression needs | Comprehensive case-managed images | Open, secure, comprehensive images | Volatile memory (RAM) capture |

> **Practical guidance given:** Choose the format based on your specific needs — if metadata and encryption matter, RAW/DD is insufficient (it stores neither); if broad tool compatibility combined with strong feature support is the priority, AFF or E01 are generally preferred over SMART.

---

## 7. Creating Forensic Images — Practical (FTK Imager)

### 7.1 Tool Used

**FTK Imager** — a widely used, GUI-based forensic imaging tool capable of creating images in multiple formats (RAW/DD, SMART, E01, AFF).

### 7.2 Step-by-Step Procedure

**Step 1 — Start image creation**

- Open FTK Imager → **File → Create Disk Image**.

**Step 2 — Select source type**
Options presented:

- **Physical Drive** — the entire physical disk as installed in the system.
- **Logical Drive** — a specific partition created via the operating system's partitioning (a "logical" division, since it's created in software rather than being a physically separate disk).
- **Image File** — to create an image from an already-existing image file.
- Other options: CD/DVD, contents of a folder, etc.

> In the demo, **Physical Drive** was selected to image a complete drive.

**Step 3 — Select destination and format**

- Click **Add** to specify the destination for the new image.
- Format options shown: **RAW/DD, SMART, E01, AFF**.
- The instructor notes **E01** is often considered a strong/preferred choice since it includes broad metadata and reasonable compatibility, though **AFF** was earlier described as best for open/secure needs — the choice depends on the investigation's specific requirements (as summarized in the comparison table above).

**Step 4 — Enter case metadata**
For formats that support metadata (SMART, E01, AFF), the following fields are typically completed:

- **Case Number**
- **Evidence Number**
- **Unique Description**
- **Examiner** name

**Step 5 — Choose destination path and image filename**

- Browse to select the destination drive/folder (e.g., an M: drive in the demo).
- Provide an **image filename**.

**Step 6 — Configure verification and finish**

- Option to **verify the image after creation** (recommended — confirms the image was created correctly and matches the source via hash comparison).
- Start the imaging process.

### 7.3 Reviewing the Completed Image (E01 Example)

After completion, FTK Imager provides an **Image Summary**, which includes:

- **Sector count** and verification calculation results (confirming no errors).
- **MD5/hash algorithm results** — used to confirm the image is a verified, unaltered match of the source. (Any bad sectors/dead blocks would be reported here if present — none were found in this demo.)
- **Metadata** — case number, examiner, creation timestamp, etc.
- **Segmentation details** — the demo image was automatically split into **7 segments**, each approximately **1.46 GB**, with the final segment typically being smaller. This reflects the E01 format's chunk-based segmentation behavior described in the theory section.

**On disk**, the resulting files appear as:

- A **.E01** file (the first/primary segment, which others attach to when opened) and accompanying **.txt** record files documenting full hash values and image details.
- Opening the first segment automatically links and loads all associated segments together as one complete image.

---

## 8. Creating a Raw (DD) Image via Command Line

### 8.1 Purpose of This Section

To demonstrate an **alternative, command-line-based method** for creating a forensic image — specifically in **RAW/DD format** — useful in scenarios where a GUI tool like FTK Imager isn't available or preferred.

### 8.2 Tool Used

- **DD tool** for Windows (downloadable by searching "DD tool" online; in this demo it was placed on the Desktop).
- **PowerShell** (run as Administrator) was used as the command-line environment for this section (distinct from the CMD environment used for The Sleuth Kit commands earlier).

### 8.3 Step-by-Step Procedure

**Step 1 — Identify available drives**

Command used:

```
wmic diskdrive list brief
```

_(Transcribed loosely as "WMIC Disk Drive List Brief" in the session — this is the standard Windows command to list physical disks.)_

- This lists all physical drives connected to the system (e.g., **Physical Drive 0**, **Physical Drive 1**), along with identifying details (internal vs. external, drive names/models).
- In the demo: **Drive 0** was identified as the internal/primary system drive (C:), and another listed drive was identified as an **external drive** based on its name/label.

> **Tip:** Always carefully verify which physical drive number corresponds to your actual target before imaging — imaging the wrong drive is a critical, irreversible mistake in a real investigation.

**Step 2 — Navigate to the DD tool location**

```
cd <path-to-DD-tool-folder>
```

**Step 3 — Run the DD imaging command**

Command structure used:

```
dd.exe if=\\.\physicaldrive0 of=E:\WinVI0_Evidence.dd bs=512k --progress
```

**Flag explanations:**
| Flag | Meaning |
|---|---|
| `if=` | **Input file** — the source being imaged (here, `\\.\physicaldrive0`, referring to Physical Drive 0) |
| `of=` | **Output file** — destination path and filename for the resulting image (e.g., `E:\WinVI0_Evidence.dd`) |
| `bs=` | **Block size** — set to `512k` (512 kilobytes) in this demo, controlling the chunk size used during copying |
| `--progress` | Displays a live progress indicator during the copy/imaging process |

**Naming convention used in the demo:** `WinVI0_Evidence.dd` — reflecting the source (Windows, drive 0) and labeling it clearly as evidence, a good practice for maintaining clear documentation/chain of custody.

### 8.4 Critical Storage Requirement

> ⚠️ **Important:** Since **RAW/DD format applies no compression**, the destination drive must have **free space equal to or greater than the full size of the source drive** being imaged. In the demo, the source (C:) was ~60GB, so the destination needed at least 60GB free before starting — this was verified before proceeding.

### 8.5 Result

- The DD imaging process completed successfully, producing a `.dd` image file matching the exact size of the source drive (~60GB in the demo).
- This RAW/DD image can now be analyzed using The Sleuth Kit commands (`mmls`, `fsstat`, `istat`, `fls`, `tsk_recover`) covered earlier in this session, or other forensic tools, exactly as demonstrated with the other evidence images throughout this lab.

---

## 9. Key Takeaways

1. **The Sleuth Kit provides a full command-line forensic workflow**: from partition table inspection (`mmls`) to file system status (`fsstat`), image statistics (`img_stat`), detailed MFT entry analysis (`istat`), file listing (`fls`), and file recovery (`tsk_recover`).
2. **The Master File Table (MFT) is the structural core of NTFS** — every file/folder's metadata is stored in numbered entries, each containing standardized sub-attributes (Standard Information, File Name, Data, Bitmap, Security Descriptor).
3. **Command-line forensic tools are syntax-sensitive** — missing spaces and incorrect flags are the most common sources of errors, not tool malfunction. Always double-check command structure.
4. **Multiple recovery tools should be used together** (SS Data Recovery, WinHex, `tsk_recover`) since no single tool guarantees complete recovery of all deleted/damaged data.
5. **Forensic image format selection matters significantly**:
   - **RAW/DD** — exact copy, no compression, no metadata, but universal tool support.
   - **SMART** — compression + metadata, but poor tool compatibility.
   - **E01** — compression + metadata + segmentation, wide (though not universal) tool support.
   - **AFF** — open-source, supports compression + metadata + **encryption**, broadest flexibility.
   - **MEM** — specifically for volatile memory (RAM) dumps, used only when a system is captured while still powered on.
6. **Always verify available destination storage before imaging** — RAW/DD images require space equal to the full source drive size since no compression is applied.
7. **Image verification (hashing) immediately after creation** is essential to confirm the image is an accurate, unaltered copy of the source — this underpins the entire chain-of-custody and evidentiary integrity process covered across both parts of this series.

---

## 10. Tools Used in This Session

| Tool                     | Purpose                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------- |
| **The Sleuth Kit (TSK)** | Command-line file system/image analysis: `mmls`, `fsstat`, `img_stat`, `istat`, `fls`, `tsk_recover` |
| **WinHex**               | Hex editing and file recovery by type from forensic images                                           |
| **FTK Imager**           | GUI-based forensic image creation (supports RAW/DD, SMART, E01, AFF formats)                         |
| **DD (Windows port)**    | Command-line creation of RAW/DD forensic images                                                      |
| **PowerShell / CMD**     | Command-line environments used for running forensic tool commands                                    |

---

_End of Part 2 — this concludes the file-system analysis and image-format segment of the practical series. Subsequent parts will build on these recovered images and formats for deeper investigative analysis._
