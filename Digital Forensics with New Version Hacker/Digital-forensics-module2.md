# Digital Forensics Course — Module 2: Cyber Crime Classification, Investigation Procedure & Storage Internals

**Course:** Digital Forensics (Complete Course)
**Type:** Theory + Light Practical (Course Introduction, Cyber Crime Classification, 13-Step Investigation Process, Hard Disk Deep-Dive, SSD Architecture, First Practical Data Recovery Demo)

---

## Table of Contents

1. [Course Introduction](#1-course-introduction)
2. [What Is Forensics? What Is Digital Forensics?](#2-what-is-forensics-what-is-digital-forensics)
3. [Why Digital Forensics Is Needed](#3-why-digital-forensics-is-needed)
4. [Roadmap to Learning Digital Forensics](#4-roadmap-to-learning-digital-forensics)
5. [Certifications](#5-certifications)
6. [Career, Roles & Salary](#6-career-roles--salary)
7. [Classification of Cyber Crimes by Motive/Target](#7-classification-of-cyber-crimes-by-motivetarget)
8. [Internal vs. External Attacks](#8-internal-vs-external-attacks)
9. [Digital Evidence: Sources and Admissibility Rules](#9-digital-evidence-sources-and-admissibility-rules)
10. [Types of Digital Forensics (Recap)](#10-types-of-digital-forensics-recap)
11. [The 13-Step Investigation Procedure](#11-the-13-step-investigation-procedure)
12. [Bit-Stream Copy vs. Normal Copy/Clone](#12-bit-stream-copy-vs-normal-copyclone)
13. [First Responder Tools](#13-first-responder-tools)
14. [Four Phases of Investigation (Detailed)](#14-four-phases-of-investigation-detailed)
15. [Investigation Challenges](#15-investigation-challenges)
16. [Hard Disk Drive (HDD) — Deep Dive](#16-hard-disk-drive-hdd--deep-dive)
17. [How Hard Disks Store Data (Magnetic Encoding)](#17-how-hard-disks-store-data-magnetic-encoding)
18. [Disk Formatting and Size Calculation](#18-disk-formatting-and-size-calculation)
19. [SSD (Solid State Drive) — Deep Dive](#19-ssd-solid-state-drive--deep-dive)
20. [Storage Interfaces (ATA, SATA, SCSI, PCIe/NVMe)](#20-storage-interfaces-ata-sata-scsi-pcienvme)
21. [Practical: Data Recovery from a Pen Drive](#21-practical-data-recovery-from-a-pen-drive)
22. [Lost Clusters, Bad Sectors & Slack Space — Concepts and Practical (CHKDSK)](#22-lost-clusters-bad-sectors--slack-space--concepts-and-practical-chkdsk)
23. [Key Takeaways](#23-key-takeaways)

---

## 1. Course Introduction

This module opens a complete digital forensics course consisting of **22+ modules**, delivered as live classes (with recordings provided for revision). The course is designed to take a learner from foundational theory through hands-on practical investigation techniques.

---

## 2. What Is Forensics? What Is Digital Forensics?

**Forensics (general definition):** The **detailed, in-depth study of anything** — knowing a subject thoroughly, from A to Z. (Analogy: someone who deeply studies science becomes a "scientist"; similarly, deep study of any domain area constitutes "forensics" of that domain.)

**Digital Forensics:** Applying that same principle of deep, detailed investigation to **digital devices** — i.e., it is the **process used to investigate cyber crimes** using/through digital devices and data.

**Cyber Crimes (definition):** Crimes in which digital devices are involved or used as the medium — e.g., lottery scam calls (using a phone), unauthorized bank withdrawals, and other crimes enabled by phones, laptops, smartwatches, and networked devices.

### Illustrative Example: When Is a Digital Forensic Analyst Needed?

At a murder crime scene, a phone is found near the victim. Two separate investigative tracks exist:

- The **murder case itself** → handled by police.
- The **phone found at the scene** → handled by a **cyber/digital forensic analyst**, who examines who the victim last contacted, what data exists on the device, and how it might help solve the case.

This illustrates the complementary relationship between traditional law enforcement and digital forensic investigation.

---

## 3. Why Digital Forensics Is Needed

As cybercrime increases alongside technological advancement, the need for a dedicated investigative discipline to **investigate, resolve, and prosecute** these crimes has grown correspondingly. Digital forensics exists specifically to fill this need.

---

## 4. Roadmap to Learning Digital Forensics

| Step | Topic                            | Notes                                                                                                                 |
| ---- | -------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| 1    | **Networking**                   | A universally required foundational topic — relevant across nearly all cyber security domains                         |
| 2    | **Ethical Hacking**              | Considered the starting/entry-level stage of cyber security                                                           |
| 3    | **Basic Programming**            | Not mandatory, but useful for scripting; **Python** is recommended for its simplicity and minimal coding requirements |
| 4    | **Operating System Proficiency** | General comfort running and navigating operating systems                                                              |
| 5    | **Digital Forensics**            | The specialization itself, built atop the above foundation                                                            |

> Digital forensics is also referred to by multiple names depending on context/certification body: **Computer Forensics**, **Cyber Forensics**, **CHFI** (as a certification name), etc. — these largely overlap, covering the broad scope of investigating digital devices.

---

## 5. Certifications

| Certification | Full Form / Provider                                                                                                               |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **CHFI**      | Computer Hacking Forensic Investigator — provided by **EC-Council** (most well-known)                                              |
| **CDFE**      | Certified Digital Forensic Examiner                                                                                                |
| **CFCE**      | Certified Forensic Computer Examiner — provided by the **International Association of Computer Investigative Specialists (IACIS)** |
| **EnCE**      | Certification tied to the **EnCase** forensic tool/vendor                                                                          |

> The instructor notes that most of these certifications cover largely similar content, with only modest differences (described as "a difference of about 19-20" i.e., minor variation) — learners can choose based on budget and preference without major functional difference between them.

---

## 6. Career, Roles & Salary

### Job Roles in Digital Forensics

| Role                                      | Description                                                                                                                                                                                        |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Digital Forensic Analyst / Examiner**   | Examines cases across digital devices, prepares reports for law enforcement                                                                                                                        |
| **Incident Responder**                    | Responds to and investigates the aftermath of a crime/incident (note: digital forensics work happens **after** a crime occurs, unlike penetration testers who work proactively/before an incident) |
| **Cyber Security Consultant**             | Advises companies/organizations on securing their systems                                                                                                                                          |
| **Law Enforcement Officer**               | Works closely with agencies to investigate cyber crimes                                                                                                                                            |
| **Legal Consultant / Expert Witness**     | Appears in court to testify and explain investigative findings to the judge                                                                                                                        |
| **Digital Forensic Researcher/Developer** | Researches forensic tools and techniques, contributes to tool development                                                                                                                          |
| **Cyber Threat Intelligence Analyst**     | Tracks and traces individuals issuing threats                                                                                                                                                      |
| **Forensic Accountant**                   | Specializes in investigating financial crimes                                                                                                                                                      |

> **Pen testers vs. digital forensic investigators (recap of the distinction):** Pen testers work proactively, identifying and reporting security gaps _before_ a crime occurs. Digital forensic investigators step in _after_ a crime has occurred — even when robust security measures (applied by pen testers/security teams) were in place but a breach still occurred.

### Salary

Approximate **7–8 lakhs** as a general benchmark (varies by city, state, and individual knowledge/experience level) — based on general market research and the instructor's personal industry experience.

---

## 7. Classification of Cyber Crimes by Motive/Target

Cyber crimes are classified based on their **underlying motive** and **who/what is being targeted**.

### 7.1 Cyber Crime Against Individual

**Target:** A single specific person.

| Crime                                 | Description                                                                                                                                                                                        |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Identity Theft**                    | Stealing someone's identity — e.g., impersonating a bank employee to extract Aadhar/personal details, or impersonating a specific person (e.g., "Hello, I am Dev, I need money") to deceive others |
| **Online Harassment & Cyberbullying** | Threatening, harassing, or mentally tormenting someone online (e.g., via social media platforms)                                                                                                   |
| **Online Stalking (Cyberstalking)**   | Continuously monitoring/following a specific individual online — described as a very common crime that often serves as a gateway to other crimes                                                   |

### 7.2 Financial Crimes

**Target/Motive:** Money — crimes solely aimed at stealing/extorting funds.

| Crime                      | Description                                                                                                                                                                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Payment Card Fraud**     | Cloning cards (carding), unauthorized withdrawal using stolen card numbers/OTPs                                                                                                                                                                    |
| **Phishing / Email Scams** | Sending a fraudulent copy of a legitimate website (e.g., a fake "amazonbusiness.in") via a link; victims unknowingly enter credentials which are then captured and misused by the attacker                                                         |
| **Ransomware Attack**      | Malware (often delivered via a malicious website/application) encrypts the victim's files, rendering them inaccessible; a ransom message demands payment for decryption — with **no guarantee** that files will actually be decrypted upon payment |

### 7.3 Cyber Crime Against Organizations

**Target:** An entire company/organization.

| Crime                           | Description                                                                                                                                                             |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data Breach**                 | Unauthorized access and leaking of an organization's data                                                                                                               |
| **Intellectual Property Theft** | Theft of digital/online properties — copyrights, software, source code, applications, websites (e.g., cloning someone's application constitutes stealing its copyright) |
| **Corporate Espionage**         | One organization (or a criminal) secretly spies on another to learn internal secrets, ongoing projects, or employee activities, and may leak that information           |

### 7.4 Cyber Terrorism & State-Sponsored Attacks

**Target:** An entire state/nation.

| Crime                                 | Description                                                                                                                                    |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cyber Warfare**                     | Waging cyber conflict against a state using digital means                                                                                      |
| **Cyber Propaganda & Disinformation** | Spreading fabricated stories/false information against a government or state, often without the spreader even being fully aware of its falsity |
| **Cyber Espionage (state-level)**     | Spying on a state government or its high-value targets — conceptually similar to corporate espionage but operating at the state/national level |

### 7.5 Cyber Crime Against Government

**Target:** The government/entire country.

| Crime                            | Description                                                                                                                                                        |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Attack on Government Systems** | Direct attacks on government digital infrastructure/properties                                                                                                     |
| **Cyber Terrorism**              | Targeting an entire nation, or leaking/exfiltrating national information to foreign entities — functionally similar to traditional terrorism, but conducted online |

---

## 8. Internal vs. External Attacks

All attack types fundamentally fall into one of two categories: the threat originates **inside** the organization or **outside** it.

### 8.1 Internal Attacks

Committed by someone with inside access (e.g., an employee).

| Attack                             | Description                                                                                                                                                                    |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Espionage (Spying)**             | Internally accessing and leaking confidential information                                                                                                                      |
| **Theft of Intellectual Property** | Physically extracting digital assets (e.g., copying applications/files to a pen drive and removing them)                                                                       |
| **Manipulation of Records**        | Altering company records to cause harm/loss, requiring time and effort to restore                                                                                              |
| **Trojan/Horse Attacks**           | Installing a malicious application that runs in the background, granting remote access to the system even from outside — once planted internally, it can be exploited remotely |

> Internal attacks require **physical or system-level insider access** — e.g., espionage cannot occur "from a distance"; the perpetrator must have been inside the organization at some point to observe/access the targeted information.

### 8.2 External Attacks

Committed by someone outside the organization, without insider access.

| Attack                             | Description                                                                                                                                                                                           |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SQL Injection**                  | Exploiting a vulnerability/missing validation in a website by injecting malicious code, often to access or manipulate the underlying database                                                         |
| **Brute Force Attack**             | Systematically trying many password combinations until access is gained                                                                                                                               |
| **Identity Theft**                 | (As described above) stealing someone's information/identity                                                                                                                                          |
| **Phishing / Spoofing (Email)**    | Sending deceptive links that appear legitimate to steal information                                                                                                                                   |
| **Spoofing**                       | The attacker conceals their own identity/origin by using another person's (or system's) IP address, email, or identifying information to commit a crime                                               |
| **DoS (Denial of Service) Attack** | Overwhelming a system beyond its handling capacity (e.g., a website built to handle 100 simultaneous requests is flooded with 1,000 requests), causing it to crash despite security measures in place |
| **Cyber Defamation**               | Damaging someone's reputation online without valid cause, by spreading false information                                                                                                              |

### 8.3 Computer Vandalism (Separate Category)

**Physical damage** to a digital device — i.e., the device itself is physically broken/damaged, as opposed to a purely digital/data-based attack.

---

## 9. Digital Evidence: Sources and Admissibility Rules

### 9.1 Sources of Digital Evidence

| Source                | Notes                                                                                                                                                                                  |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Internet**          | All online activity records — a primary source since most cybercrimes occur via internet-connected systems                                                                             |
| **Mobile Phone**      | A major source today, given how frequently crimes (e.g., social engineering scams) are committed via phones; phones found at crime scenes can reveal prior activity relevant to a case |
| **Laptop / Computer** | Includes RAM data, system logs, and hard disk contents                                                                                                                                 |
| **Pen Drives**        | If found at a crime scene with case-relevant data, can be presented as evidence                                                                                                        |
| **Digital Camera**    | Recorded footage directly relevant to a crime can be used as evidence                                                                                                                  |
| **Digital Watches**   | Modern smartwatches can yield call logs, conversation details, viewed content, location/motion tracking, and more                                                                      |

> **Key principle:** Any evidence presented in court must be **directly related to the case** at hand.

### 9.2 Five Rules for Admissible Evidence

1. **Understandable** — the judge must be able to comprehend the evidence directly.
2. **Admissible (Relevant)** — the evidence must be directly related to the fact/crime in question, not extraneous.
3. **Authentic** — the evidence must show no signs of tampering; verified via **hash value generation** (if tampering occurred, the hash value would change, proving inauthenticity).
4. **Reliable** — the investigator must demonstrate that the entire investigative process was conducted properly, under appropriate supervision/authority, and explain this process clearly to the judge.
5. **Complete** — the evidence must be sufficiently thorough to prove guilt or innocence on its own; incomplete evidence risks the case being deferred or the investigator being asked to re-investigate.

### 9.3 Volatile vs. Non-Volatile Data (Recap)

| Type             | Description                                               | Examples                                                                                            |
| ---------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Volatile**     | Data that changes or is lost once a system is powered off | RAM contents (captured via a **RAM dump** while the system is still running), network activity logs |
| **Non-Volatile** | Data that persists in storage regardless of power state   | Hard disk contents, downloaded files                                                                |

> **RAM Dump Practice:** When a powered-on computer is found at a crime scene, investigators capture a **RAM dump** to preserve a snapshot of what was actively running/open at that moment — since this data is irretrievably lost once the system shuts down.

### 9.4 Types of Presentable Digital Evidence

Computer files (with strong/concrete relevance to the case), internet and social media activity/chat history, mobile device data (calls, photos, videos), network traffic and logs, **forensic images and copies** (duplicates made during investigation are themselves admissible as evidence — see Section 12), **artifacts** (small supplementary items found at a scene that aid the investigation, e.g., a pen drive lying near a laptop), encryption keys and passwords, digital signatures and timestamps (online footprints — websites visited, when, etc.), and electronic transaction/financial records.

---

## 10. Types of Digital Forensics (Recap)

| Type                           | Focus                                                                                                                               |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Computer Forensics**         | Complete forensic investigation of a computer system                                                                                |
| **Mobile Forensics**           | Investigation of mobile devices                                                                                                     |
| **Network Forensics**          | Investigating network performance/activity for evidence of compromise                                                               |
| **Memory Forensics**           | Recovering deleted items and storage-device evidence                                                                                |
| **Database Forensics**         | Forensic investigation of large structured datasets                                                                                 |
| **Cloud Forensics**            | Investigating cloud storage platforms (e.g., OneDrive)                                                                              |
| **IoT Forensics**              | Investigating small internet-connected devices (smartwatches, CCTV cameras, etc.)                                                   |
| **Operating System Forensics** | Identifying whether manipulation/tampering has occurred at the OS level                                                             |
| **Audio Forensics**            | _(implied)_ Analyzing audio recordings relevant to a case                                                                           |
| **Email Forensics**            | Investigating whether emails are connected to a crime                                                                               |
| **Dark Web Forensics**         | Investigating illegal activity/businesses operating on the dark web (a portion of the internet outside conventional/indexed access) |
| **Malware Forensics**          | Investigating malware — what damage it caused, what data it extracted, etc.                                                         |

---

## 11. The 13-Step Investigation Procedure

A condensed overview of the standard investigation workflow (matching the structure introduced in Module 1, restated here with additional framing):

| Step   | Action                                                              | Key Notes                                                                                                        |
| ------ | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **1**  | **Identification**                                                  | First confirm whether the incident is genuinely a cyber crime before proceeding                                  |
| **2**  | **Gather initial (primary) evidence**                               | Collect evidence confirming the cyber crime nature of the incident (e.g., a PC, phone, or any involved device)   |
| **3**  | **Secure authorization from court/authorized entity for seizure**   | Required for private/personal cases; police cases typically already carry pre-obtained authorization             |
| **4**  | **Utilize first responder tools**                                   | Tool selection depends on the nature of the crime scene (see Section 13)                                         |
| **5**  | **Confiscate/seize evidence at the scene**                          | Evidence is sealed (e.g., in antistatic bags) and brought under investigator control                             |
| **6**  | **Produce a two-bit-stream (bit-by-bit) duplicate of the evidence** | A full bit-level copy is made — distinct from a normal file copy (see Section 12)                                |
| **7**  | **Generate a hash value on the duplicate**                          | Establishes the evidence's unique "identity"/fingerprint for later integrity verification                        |
| **8**  | **Ensure continuity of evidence (maintain chain of custody)**       | Every action/finding/process during investigation is documented step-by-step                                     |
| **9**  | **Transfer evidence to the forensic facility/lab**                  | Both the original and duplicate copies are transported to a secure forensic lab                                  |
| **10** | **Securely store the original evidence**                            | Stored in a controlled, access-restricted environment (only authorized lab personnel)                            |
| **11** | **Analyze the duplicate for evidence**                              | All investigative work is performed on the duplicate — never the original                                        |
| **12** | **Compile a forensic report (65B report)**                          | This report is delivered to police or the requesting client                                                      |
| **13** | **Appear in court and provide expert testimony, if required**       | The investigator may need to demonstrate/explain the duplicate evidence and original alongside it before a judge |

---

## 12. Bit-Stream Copy vs. Normal Copy/Clone

A core distinction reiterated with a clear illustrative example:

### Normal Copy/Clone

- Only the **actual data** present on a drive gets copied.
- **Example:** A 10GB pen drive containing only 2GB of actual data — a normal copy/clone transfers only that 2GB to the destination.

### Bit-to-Bit Stream Copy (Forensic Copy/Image)

- Captures the **entire drive**, sector by sector — including both the data **and** the blank/unallocated space.
- **Example:** The same 10GB pen drive (with 2GB of actual data) — a bit-to-bit stream copy duplicates the full 10GB structure, not just the 2GB of visible data.
- This is the standard required for forensic investigation, since unallocated/blank space may still contain remnants of previously deleted files.

> This distinction is functionally identical to the "forensic imaging vs. normal copying" concept covered in Module 1, reinforced here with a numeric example for clarity.

---

## 13. First Responder Tools

| Tool                                               | Purpose                                                                                                                                                                                   |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Forensic Imager / Forensic Replicator software** | Creates the bit-to-bit stream (forensic) copy of evidence                                                                                                                                 |
| **Hash Calculator**                                | Generates the hash value to establish the evidence's unique identity for integrity verification                                                                                           |
| **Write Blocker (Bridge/Bright Blocker Tool)**     | Available in both hardware and software form; prevents any accidental writes to the original evidence drive — only reading is permitted                                                   |
| **SIM Card Reader**                                | Used to extract and analyze data from SIM cards found at the scene                                                                                                                        |
| **SIM Card Analysis Software**                     | Analyzes the data extracted from a SIM card                                                                                                                                               |
| **Faraday Bag (Signal Blocking Bag)**              | Blocks all incoming/outgoing signals to a device (e.g., a found phone) to prevent remote access, hacking, or accidental network reconnection                                              |
| **Mobile Data Recovery Software**                  | E.g., Dr.Fone, Wondershare — used to recover deleted mobile data                                                                                                                          |
| **Evidence Collecting Bag (Antistatic Bag)**       | Protects metallic/magnetic components (HDDs and other metal-based devices) from static charge damage — the human body carries static charge that can damage sensitive hardware on contact |
| **Memory Card Reader**                             | Needed to read any memory cards found at the scene                                                                                                                                        |

---

## 14. Four Phases of Investigation (Detailed)

### 14.1 Primary Investigation Phase

Covers all preparatory work performed **before** the actual investigation begins:

- **Planning and Budget** — determining team size, resource allocation, and overall investigation cost.
- **Physical and Structural Design Consideration** — lab size, environment, HVAC/ventilation suitability to avoid harming evidence.
- **Work Area Consideration** — lighting, networking, workstation placement, and critically, **emergency power backup** to ensure no investigation disruption during power outages.
- **Physical Security Consideration** — security guards, receptionists, and especially **CCTV cameras** monitoring access to the evidence room; the lab's surrounding area/location must also be secure.
- **Human Resources Consideration** — determining staffing needs and role assignments.
- **Forensic Lab Licensing** — operating without a license can render reports inadmissible in court; licensing is mandatory for legitimate lab operation.
- **Building the Investigation Team** — assembling the various functional roles (see below); a single person can perform multiple roles, but all functions must be covered.

#### Investigation Team Roles

| Role                               | Responsibility                                                                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Photographer**                   | Documents the crime scene visually for evidentiary and self-protective purposes (guards against false allegations against the investigator) |
| **Incident Responder**             | Responsible for securing evidence found at the crime scene                                                                                  |
| **Incident Analyst**               | Analyzes the type and extent of damage/crime that occurred                                                                                  |
| **Evidence Examiner/Investigator** | Examines evidence and investigates leads/clues                                                                                              |
| **Evidence Documenter**            | Documents all evidence found (chain of custody documentation)                                                                               |
| **Evidence Manager**               | Manages overall evidence security and storage                                                                                               |
| **Expert Witness**                 | Available to testify in court if needed, explaining the scene and findings                                                                  |
| **Attorney/Legal Advisor**         | Provides legal guidance on what is/isn't permissible under the law                                                                          |

#### Hardware Requirements for a Forensic Lab

- **Two or more forensic workstations** (powerful processing capability) — redundancy in case one is damaged.
- **Specialized cables** for data transfer/connectivity across various device types.
- **Write blockers** to prevent alteration of original evidence.
- **Drive duplicators**.
- **Archive and restore devices/storage** with strong capacity for evidence retention.
- **Media sterilization systems** to ensure storage media integrity before/after use.
- A **complete computer forensic toolkit** including chargers, power backup equipment, and miscellaneous supporting hardware.
- **First Responder bundles** and **deep-scan disk imagers**.
- An additional **dedicated, high-powered forensic workstation**.

#### Software Requirements for a Forensic Lab

- **OS-specific forensic investigation tools** (different operating systems require tailored forensic software).
- **Data recovery tools.**
- **Password cracking tools.**
- **Data acquisition and data analysis tools.**
- **File viewers** (supporting various formats — PDF, PPT, etc.).
- **File type conversion tools.**
- **Security/utility software**, including properly installed antivirus protection on lab systems.
- **Computer forensic tools** (e.g., Wireshark for network analysis, AccessData/FTK-family tools).

### 14.2 First Response Phase

Activities performed immediately upon reaching the crime scene:

- **Document the electronic crime scene** thoroughly.
- **Collect incident information** — gather context on what happened and from whom (witnesses, etc.).
- **Plan and search** for and **seize evidence**, bringing it under investigator control.
- **Identify evidence** for transport to the lab (following all procedures established in the Primary Investigation phase).

### 14.3 Investigation Phase

The core analytical work — performed in the lab, on the duplicate copies:

- Analyzing all collected drives, computers, and files to identify case-relevant evidence.
- Determining what evidence is admissible and how it can be effectively presented in court.

### 14.4 Post-Investigation Phase

- **Compile the forensic report** — documenting the full investigation process, methodology, evidence names/identifiers, hash values, and findings.
- **Attend court proceedings**, if required, to explain the investigation and findings to the judge.

---

## 15. Investigation Challenges

| Challenge                                   | Description                                                                                                                                                                                                                                                      |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Speed**                                   | While technology advances investigative tools, bureaucratic processes (e.g., obtaining court permission/warrants) can significantly slow investigations, since legal authorization is a government process that takes time                                       |
| **Anonymity**                               | Criminals use VPNs and identity-masking techniques (altering system identity, serial numbers, IP addresses, MAC addresses) to evade detection — though the instructor notes no crime is ever truly "100% perfect," meaning some trace/clue almost always remains |
| **Volatile Evidence**                       | Time-sensitive evidence (e.g., RAM contents, internet history) can be lost or altered before it's captured, making recovery difficult once lost                                                                                                                  |
| **Evidence Size**                           | Large organizations may have massive hard drives/servers, making investigation a lengthy, resource-intensive process                                                                                                                                             |
| **Anti-Forensic Techniques**                | Criminals deliberately use anti-forensic tools/processes to delete or obscure evidence, complicating investigation — though investigators do have countermeasures (covered in later modules)                                                                     |
| **Global/Jurisdictional Laws**              | Different countries have different legal frameworks, making cross-border investigation and law compliance challenging                                                                                                                                            |
| **Lack of Legal Knowledge (Criminal Side)** | Ironically, criminals sometimes commit further crimes due to a lack of legal awareness, requiring investigators to navigate the investigation with that context in mind                                                                                          |

---

## 16. Hard Disk Drive (HDD) — Deep Dive

### 16.1 Why Hard Disks Matter Most

Hard disks remain central to forensic investigation since the majority of investigated crimes involve laptops/computers, and HDDs continue to be widely used despite the rise of SSDs.

### 16.2 Core Properties

- **Full form:** Hard Disk Drive.
- **Storage device** — similar in basic function to a pen drive, though architecturally different.
- **Non-volatile** — data persists unless deliberately deleted.
- **Has movable parts** — increasing risk of data loss/damage from physical shock (e.g., dropping the drive), representing a key structural weakness compared to SSDs.
- **Stores data magnetically** — via internal platters.

### 16.3 Physical Components and External Structure

- **Metal case/casing** — the outer protective shell, secured with screws, with a label detailing drive specifications.
- **Form factor differs by device type** — desktop PC hard drives are physically larger than laptop hard drives (which are more compact), despite similar storage capacities; using a larger drive in a laptop would also increase battery power consumption.
- **Ports:** A power port and a data port (commonly visible as distinct connector sockets on the drive's edge).

### 16.4 Internal Components (Upon Opening the Casing)

| Component           | Function                                                                                                          |
| ------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Platter (Disk)**  | The metal disc(s) where data is magnetically stored; arranged in stacked layers (multiple platters per drive)     |
| **Spindle**         | The central rotating shaft, driven by a motor, that spins the platter(s)                                          |
| **Read/Write Head** | The small component making (near-)contact with the platter surface, responsible for both reading and writing data |
| **Actuator Arm**    | The arm structure connecting to the read/write head — analogous to a human arm                                    |
| **Actuator Axis**   | Enables the arm's left-right movement across the platter                                                          |

> **RPM (Rotations Per Minute):** The platter's rotational speed is **directly proportional** to the drive's read/write speed — faster RPM means faster data access and transfer capability.

> **Component interchangeability:** If a drive's PCB or arm/actuator becomes damaged, the platter can sometimes be transferred into another compatible hard disk housing to attempt data recovery.

### 16.5 Platter Internal Structure: Tracks, Sectors, Clusters

| Term        | Definition                                                                                                                                                                   |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Track**   | A complete circular ring on the platter surface, used to store data; analogous to a complete road/street in a colony                                                         |
| **Sector**  | A track divided into smaller segments — the smallest physical storage unit of the disk; analogous to a small lane branching off the main road, leading to a specific address |
| **Cluster** | A group of sectors combined together to store data exceeding a single sector's capacity, or to group sectors collectively allocated to one file                              |

#### Track Specifics

- Older/typical platters can contain up to **10,000 tracks** on a single side; the latest hard drives can have **500,000+ tracks**.
- All platters within a single hard drive maintain **identical track density** — if one platter has N tracks, every platter has the same N tracks.
- Tracks are **numbered starting from 0** at the **outermost edge**, increasing inward toward the center.

#### Sector Specifics

- The term "sector" is borrowed from the mathematical concept (area of a circular sector).
- **Sector size:**
  - Older hard drives: **512 bytes** per sector.
  - CDs/DVDs: approximately **2048 bytes (~2KB)** per sector.
  - **Latest hard drives: 4096 bytes (4KB)** per sector — sometimes called **4K sectors**, combining what would have been **8 old-style 512-byte sectors** into a single modern sector.
- **Address/Locating mechanism:** Just as a physical address uses plot number, area name, and sector number to locate a house, the hard disk uses **track number** and **sector number** to precisely locate where specific data resides — this addressing system is what enables forensic data recovery, even from physically damaged (burnt/water-damaged) drives, by identifying which surviving sectors can still be read.

#### Internal Structure Within Each Sector

| Field                                     | Function                                                                                                                                                              |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Synchronization Zone / ID Information** | Provides the read/write head with rotational/speed information about the disk                                                                                         |
| **Address**                               | Identifies the sector's specific track number and sector number — its precise location                                                                                |
| **Data**                                  | The actual stored content; matches the sector's capacity (512 bytes or 4KB depending on drive generation)                                                             |
| **ECC (Error Correcting Code)**           | Verifies data integrity — confirms whether stored data has been tampered with or corrupted, and supports proper read/write verification                               |
| **Gap**                                   | A small buffer between consecutive sectors, giving the read/write head time to process the just-read data before moving to the next sector, preventing read confusion |

### 16.6 Cylinders

A **cylinder** is formed by combining the **same-numbered track across all platters** in a multi-platter drive (e.g., track 0 on platter 1 + track 0 on platter 2 + track 0 on platter 3, etc., together form one cylinder). Since one read/write arm assembly can access the corresponding track position across all platters simultaneously, this collection is grouped conceptually as a single cylinder.

### 16.7 Disk Size Calculation

To estimate a hard disk's total size (e.g., when working with a damaged/burnt drive), multiply:

**Number of platters × Number of surfaces (typically 2 per platter) × Number of tracks per platter × Number of sectors per track × Bytes per sector**

The result is obtained in **bytes**, which must then be converted to GB (or other units) for practical interpretation. (Note: when a "heads" count is given instead of separately stating platters × 2 surfaces, the heads figure already accounts for both surfaces — so multiplying by "2" separately would double-count; use whichever figure correctly represents total readable surfaces.)

### 16.8 Byte/Bit Conversion Reference Table

| Unit   | Equivalent             |
| ------ | ---------------------- |
| 1 Bit  | 0 or 1                 |
| 1 Byte | 8 bits                 |
| 1 KB   | 1024 bytes = 2¹⁰ bytes |
| 1 MB   | 1024 KB = 2²⁰ bytes    |
| 1 GB   | 1024 MB = 2³⁰ bytes    |
| 1 TB   | 1024 GB = 2⁴⁰ bytes    |

> Since most forensic size calculations are ultimately done in bytes, converting between units using powers of 2 (2¹⁰, 2²⁰, 2³⁰, 2⁴⁰) is a standard practice.

---

## 17. How Hard Disks Store Data (Magnetic Encoding)

Hard disks store binary data using **magnetic polarity**, with two described conceptual methods:

### Method 1: Polarity-Based Encoding

- **North pole facing up** = binary **1**.
- **South pole facing up** (i.e., North facing down) = binary **0**.
- The read/write head's internal coil generates the appropriate magnetic charge (North or South) at each storage point based on the bit being written.

### Method 2: Field-Change-Based Encoding

- When the magnetic field **changes** between two adjacent points (e.g., South → North), this is interpreted as binary **1**.
- When the magnetic field **remains the same** between adjacent points (e.g., South → South), this is interpreted as binary **0**.

> Both encoding theories were presented as conceptual models — what matters is that the read/write head contains internal coils capable of generating and reading these magnetic field states to encode and decode binary data, with the platter material (often containing metals/titanium-based compounds) suited to magnetic charging.

### Read/Write Head Internal Mechanism

- Contains **separate read and write coil components**.
- Functions: **generating the magnetic field** (for writing) and **detecting/changing magnetic direction** (for both writing and reading).

---

## 18. Disk Formatting and Size Calculation

**What happens during formatting:** When a disk is formatted, the **clusters are broken apart**, releasing the grouped sectors back into their base track/sector structure, organized according to the relevant **file system** in use (e.g., NTFS, FAT32, ext4 — these are explored in subsequent modules).

---

## 19. SSD (Solid State Drive) — Deep Dive

### 19.1 Two Types of SSD-Related Storage Discussed

| Type                                 | Description                                                                                                                                                                       |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **NAND-based SSD**                   | The primary storage type — non-volatile (data persists without power); includes pen drives, laptop SSDs, memory cards                                                             |
| **DRAM-based volatile memory (RAM)** | While technically a different category (volatile, short-term storage), it's discussed alongside SSD architecture since modern systems integrate DRAM caching into SSD performance |

### 19.2 Core SSD Properties

- **Non-volatile** (NAND-based) — data does not change/disappear when power is removed.
- **No moving/movable parts** — fixed internal components, making SSDs significantly more reliable and shock-resistant than HDDs (lower data-loss risk if dropped).
- Uses **NAND flash memory chips** to store data.
- **5–10x faster** than traditional hard disks for read/write operations.

### 19.3 Why RAM Matters Forensically (Recap with New Framing)

- RAM is a **short-term, volatile storage device** — used to temporarily hold copies of files/applications actively in use, which the system loads from the SSD/HDD into RAM for faster processor access.
- Applications "open" in RAM, not directly from the SSD/HDD — this is why restarting a system doesn't restore previously open application windows exactly as they were (RAM contents are wiped on shutdown).
- This is precisely **why investigators capture RAM dumps** from powered-on systems at a crime scene — to preserve a record of exactly what was running/open at that moment, since this volatile evidence is permanently lost once the system powers down.
- RAM is also called **DRAM (Dynamic RAM)** — "dynamic" referring to its constantly changing data state (similar conceptually to a "dynamic IP" that changes).

### 19.4 NAND Flash Memory — Conceptual Basis

- **NAND** refers to a **logic gate** (a digital electronics/physics concept) used to control how data signals are passed, stored, and retrieved.
- **Digital vs. Analog signals:** Digital signals are binary (high/low, 1/0) with no in-between state; analog signals continuously vary (e.g., used in radio frequency communication). Computers — and therefore SSDs — operate using digital (binary) signals exclusively.
- NAND gates (and related logic gates such as AND, NOT, NOR) are used to **store and transmit these digital signals** within the flash memory architecture, ensuring that once written, data remains stable and unchanged unless deliberately modified or deleted — even after the SSD is disconnected from the host system for extended periods.

### 19.5 Capacitor/Transistor-Based Storage Concept

- Conceptually similar to how a capacitor holds an electrical charge: when **charged**, a storage cell represents binary **1**; when **discharged**, it represents binary **0**.
- In SSDs, this storage function is performed by **millions of microscopic transistors** (rather than large capacitors), each capable of being charged/discharged to represent the bits of stored data, using a three-terminal transistor structure.
- This is how SSD data persists reliably over the long term, even without continuous power — the charge state is retained until deliberately changed.

### 19.6 SSD Internal Architecture

| Component                           | Function                                                                                                                                                                      |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Flash Memory Chips (NAND-based)** | Contain the actual storage cells where data is held via charge/discharge states                                                                                               |
| **Controller**                      | Manages incoming/outgoing data, maintains data integrity, and functions similarly to a CPU for the SSD                                                                        |
| **PCB (Printed Circuit Board)**     | The green board connecting/wiring all internal components together; provides the physical and electrical interconnection layer                                                |
| **DRAM (onboard)**                  | Caches recently/frequently accessed data references for faster access; speeds up overall SSD performance, though this cached information is lost when disconnected from power |
| **Power Port**                      | Supplies power to the SSD's internal components                                                                                                                               |
| **Host Interface / Host Port**      | The connectivity point (e.g., SATA, PCIe) enabling data transfer between the SSD and the host computer                                                                        |

> **Form factor note:** SATA-type SSDs are physically larger (similar dimensions to a 2.5" HDD), while **M.2 SSDs** are slim, strip-like form factors with smaller flash chips, used in more compact/modern systems.

---

## 20. Storage Interfaces (ATA, SATA, SCSI, PCIe/NVMe)

Storage interfaces are the **connectors/standards** that allow hard disks and SSDs to physically connect to and communicate with a computer.

| Interface      | Full Form                                                                       | Notes                                                                                                                              |
| -------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **ATA**        | Advanced Technology Attachment                                                  | General umbrella term for attachment technologies                                                                                  |
| **PATA**       | Parallel ATA (Parallel Attachment Technology of Advanced Technology Attachment) | Used a wide ribbon-style 40-pin cable; could connect **two hard drives** on a single cable (older/legacy technology)               |
| **IDE**        | Integrated Drive Electronics                                                    | Older interface type, largely obsolete                                                                                             |
| **EIDE**       | Enhanced Integrated Drive Electronics                                           | An 80-pin redesign/successor to IDE; also legacy technology, rarely seen except in very old laptops                                |
| **SATA**       | Serial ATA (Serial Advanced Technology of Attachment)                           | A widely used standard for both HDDs and SSDs; serial (single-line) data transfer, directly integrated into the motherboard        |
| **SCSI**       | Small Computer System Interface                                                 | Has multiple versions varying by speed and supported cable length                                                                  |
| **SAS**        | Serial Attached SCSI                                                            | A more advanced evolution of SCSI; commonly used in Mac systems and similar professional/enterprise environments                   |
| **PCIe**       | Peripheral Component Interconnect Express                                       | Modern, very high-speed interface used especially for newer SSDs                                                                   |
| **NVMe (M.2)** | Non-Volatile Memory Express, via the M.2 form factor                            | The fastest current SSD storage protocol/technology, built on PCIe; represents the cutting edge of consumer storage transfer speed |

### Practical Forensic Relevance

> The instructor notes that **most real-world forensic cases still involve older technology** (parallel/serial ATA-era drives) rather than the newest NVMe/M.2 drives, since a significant proportion of crimes investigated involve older computers. Understanding legacy interfaces therefore remains practically important for working investigators, even as newer technology continues to emerge.

---

## 21. Practical: Data Recovery from a Pen Drive

### 21.1 Tool Used

**"SS Data Recovery"-type tool** (referred to generically as such; available via a free demo/trial version bundled with multiple tool offerings on the vendor's site) — a paid tool with a usable free/demo mode for practice purposes.

### 21.2 Step-by-Step Demonstration

1. **Setup** — A pen drive (labeled as the "F drive" in the demo) was connected to the system.
2. **Create test deletions** — Two files were deliberately deleted from the pen drive:
   - A `.exe` file (VLC media player executable).
   - A `.jpg` thumbnail image file (a video thumbnail).
3. **Launch the recovery tool** — The tool displays all connected drives/storage devices (internal hard disks, SSDs, external/USB drives, camera storage, etc.) for selection.
4. **Select the target drive** — The USB/pen drive (F drive) was selected as the recovery source.
5. **Initiate scan** — Selecting "Search Lost Data" begins a **sector-by-sector scan** of the entire drive, which can take time depending on drive size; the scan can be paused or stopped mid-process.
6. **Review results** — The tool categorizes findings, including previously existing folders and a separate listing of **recovered/lost files**. A live percentage-complete indicator tracks scan progress.
7. **Locate and verify recovered files** — By filtering by file type and browsing results (including a **preview** feature for supported file types), the two previously deleted files (the VLC `.exe` and the `.jpg` thumbnail) were located and confirmed as recoverable, each appearing as approximately 40KB in this demo.
8. **Select destination for recovery** — Critically, recovered files **must be saved to a different drive** than the one being scanned (e.g., recovering to a "D" or "E" drive rather than back onto the original "F" pen drive) — restoring files back onto the same source drive being scanned is not possible/advisable.
9. **Complete recovery** — Files were successfully restored to the selected destination drive and visually confirmed to match the originally deleted files.

### 21.3 Key Takeaways from the Demo

- Free/demo versions of forensic recovery tools can still be effective for genuine recovery tasks, useful for building practical confidence before investing in paid licenses.
- The underlying recovery mechanism scans the drive **sector by sector**, consistent with the HDD/SSD sector architecture covered earlier in this module.
- Recovered evidence destination must always differ from the source drive — directly reinforcing the imaging/chain-of-custody principles covered in Sections 11–12.
- A follow-up practical (covered in a subsequent module/part) extends this into **hash value calculation** on recovered evidence, to formally establish and verify its integrity for evidentiary purposes.

---

## 22. Lost Clusters, Bad Sectors & Slack Space — Concepts and Practical (CHKDSK)

### 22.1 Lost Cluster

**Definition:** A cluster that the operating system has marked as "in use," but which is **not actually associated with any file** — essentially an orphaned allocation record.

**Two primary causes:**

1. **File Allocation Table (FAT) inconsistency** — when storage allocation records become misaligned/incorrect for a given file, the associated cluster can become "lost" — not actually used by any real file, yet still marked as allocated.
2. **Improper application/file closure** — when a user does not properly close a file/application, or shuts down the system directly without closing running applications, this can result in lost cluster errors.

**Important distinction:** A lost cluster is a **logical error** (a software/file-system-level inconsistency), **not physical damage** to the disk. The underlying hard disk hardware is not actually broken — the operating system has simply mismarked or mismanaged the allocation record. This logical nature means lost clusters **can typically be repaired**.

> Also referred to as an **"orphan cluster."**

### 22.2 Bad Sector

**Definition:** A sector that has suffered actual **physical damage** (e.g., due to scratches or other hardware-level defects) and can no longer function correctly.

**Key distinction from lost clusters:**
| Aspect | Lost Cluster | Bad Sector |
|---|---|---|
| **Nature of error** | Logical (software/file-system) | Physical (hardware damage) |
| **Repairable?** | Generally yes | Rarely repairable — physical damage is usually permanent |
| **Cause** | FAT inconsistency, improper shutdown | Physical defects (scratches, wear, manufacturing faults), sometimes exacerbated by bad drivers or resource conflicts |

### 22.3 Practical Demonstration: CHKDSK Command

**Tool:** Windows Command Prompt, run as **Administrator** (required for the command to execute properly).

**Basic syntax:**

```
chkdsk
```

Run from within the target drive's directory context (e.g., navigate to the relevant drive first using `cd \` to return to root, then switch drives by typing the drive letter followed by a colon, e.g., `D:`).

**What it checks:** Scans the selected drive across multiple stages, reporting:

- Total disk space (in KB).
- Number of files present.
- Index information.
- **Bad sectors** present (if any).
- Whether any **lost/orphan clusters** exist.

**Demonstrated results:**

- A scan of the C: drive returned a clean result — no bad sectors found.
- A scan of the D: drive similarly returned zero bad sectors and no lost clusters.
- A scan of an external hard drive (mounted as K: in the demo) also returned no bad sectors, with comparable scan speed to the internal SSD.

**Repair mode:**

```
chkdsk /f
```

Adding the `/f` (or similarly noted "scan" repair flag in the demo's spoken instructions) initiates **repair mode**, attempting to fix any detected logical issues (such as lost clusters) automatically. The repair process runs through stages including security examination/verification and file examination, ultimately reporting whether any issues were found and resolved (e.g., "Windows has scanned the file system and found no problems" when the drive is clean).

> **File system identification:** The demo's scanned drive reported its file system as **NTFS** (New Technology File System) — one of several file systems (others include FAT, FAT32) used differently across operating systems, to be explored in greater depth in later modules.

### 22.4 Slack Space (Slack Area)

**Definition:** The **wasted/unused leftover space** within the last sector or cluster allocated to a file, when the file's actual size is smaller than the total space reserved for it.

**Worked example (as demonstrated):**

- A file is 700 bytes in size.
- One sector holds 512 bytes — therefore the 700-byte file requires **two sectors** to be stored (since it cannot fully fit in just one).
- Total space allocated across the two sectors: 512 + 512 = 1024 bytes.
- Actual file size: 700 bytes.
- **Remaining unused space: 1024 − 700 = 324 bytes** — this leftover, unused portion within the allocated cluster is the **slack space**.

**Where slack space can occur:** Not limited to hard disks — can also occur in RAM and on any external storage device/drive installed in a system.

**Forensic significance:** Slack space can retain **remnants of previously deleted data** from earlier file allocations at that same physical location, making it a valuable area for forensic recovery of deleted/historical information — directly reinforcing the same principle covered regarding forensic imaging in Module 1 (Section 19, "Forensic Imaging vs. Normal Copying").

---

## 23. Key Takeaways

1. **Forensics, at its core, means deep and thorough investigation** — digital forensics simply applies this principle specifically to digital devices and data, in service of investigating cyber crimes.
2. **Cyber crimes are classified by motive/target** into five main categories: against individuals, financial crimes, against organizations, cyber terrorism/state-sponsored attacks, and against government — each requiring different investigative approaches.
3. **All attacks ultimately originate either internally (insider access) or externally** — internal attacks require physical/system-level inside access, while external attacks (SQL injection, brute force, phishing, spoofing, DoS, defamation) are launched remotely.
4. **The 13-step investigation procedure and the bit-stream copy principle remain the procedural backbone** of all forensic work — never investigate the original evidence; always work from a verified, hashed duplicate.
5. **Five rules govern evidence admissibility:** understandable, admissible/relevant, authentic, reliable, and complete — all five must be satisfied for evidence to hold up in court.
6. **Hard disk and SSD internals (platters, tracks, sectors, clusters, NAND flash, transistors) directly underpin practical forensic techniques** — understanding _how_ data is physically/magnetically/electronically stored is what enables accurate data recovery, including from damaged media.
7. **Lost clusters (logical/software errors) and bad sectors (physical/hardware damage) are fundamentally different problems** — the former is generally repairable via tools like CHKDSK, while the latter typically is not.
8. **Slack space — the leftover unused portion of an allocated cluster — is forensically significant**, as it can retain recoverable fragments of previously deleted data, reinforcing why full bit-by-bit forensic imaging (rather than simple file copying) is the investigative standard.
9. **Practical recovery tools (e.g., SS Data Recovery) operate via sector-by-sector scanning**, consistent with the underlying disk architecture — and recovered files must always be restored to a destination separate from the original evidence source.

---

_This concludes Module 2. It builds directly on Module 1's legal/procedural foundation while going deeper into storage media internals and introducing the first hands-on data recovery practical. Subsequent modules (see Part 1 and Part 2 practical notes) continue with hash verification, malware identification, forensic imaging tools, The Sleuth Kit command-line workflow, and MFT analysis._
