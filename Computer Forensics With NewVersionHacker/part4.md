# Computer Forensics — Part 4

## Hard Disk Architecture, Data Storage & Evidence Recovery

### Cybersecurity Department | Comprehensive Lab Notes

> **Course Context:** Digital forensics investigators must understand storage media at a hardware level to recover evidence from damaged, destroyed, or wiped drives. This module covers the physical and logical structure of HDDs, data encoding, and the forensic implications of each layer.

---

## Table of Contents

1. [Why Hard Disk Knowledge Matters in Forensics](#1-why-hard-disk-knowledge-matters-in-forensics)
2. [Hard Disk Physical Components](#2-hard-disk-physical-components)
3. [The Platter — Tracks, Sectors & Clusters](#3-the-platter--tracks-sectors--clusters)
4. [Sector Internals & Structure](#4-sector-internals--structure)
5. [4K Sectors — Modern Hard Drives](#5-4k-sectors--modern-hard-drives)
6. [Cylinders & Head Assemblies](#6-cylinders--head-assemblies)
7. [Magnetic Data Encoding](#7-magnetic-data-encoding)
8. [The Read/Write Head — Mechanism](#8-the-readwrite-head--mechanism)
9. [Data Units — Bits, Bytes & Conversions](#9-data-units--bits-bytes--conversions)
10. [Hard Disk Size Calculation](#10-hard-disk-size-calculation)
11. [Disk Formatting & File Systems Overview](#11-disk-formatting--file-systems-overview)
12. [Forensic Lab Exercises](#12-forensic-lab-exercises)
13. [Evidence Recovery from Damaged Media](#13-evidence-recovery-from-damaged-media)
14. [Tools Reference](#14-tools-reference)
15. [Quick Reference Cheatsheet](#15-quick-reference-cheatsheet)

---

## 1. Why Hard Disk Knowledge Matters in Forensics

A forensic investigator who does not understand storage hardware cannot effectively:

- Recover data from physically damaged drives
- Explain evidence integrity to a court
- Identify signs of tampering or wiping
- Calculate what data could or could not have survived damage
- Perform sector-level imaging correctly

```
Criminal destroys hard disk
         ↓
Throws in water / burns / smashes
         ↓
Investigator recovers device
         ↓
Hardware-level analysis begins:
  ├── Which platters survived?
  ├── Which sectors are readable?
  ├── Can the read/write head assembly be replaced?
  ├── Can data be reconstructed from partial sectors?
  └── Can evidence be presented to court with integrity intact?
```

**Key forensic principle:** There is no such thing as a perfectly destroyed hard drive. As long as **any sectors survive**, data can potentially be recovered.

---

## 2. Hard Disk Physical Components

### 2.1 Component Overview

```
┌─────────────────────────────────────────────────────────┐
│                    HARD DISK DRIVE                       │
│                                                          │
│   ┌──────────────────────────────────────────────────┐  │
│   │            Top Cover (dust-proof seal)            │  │
│   └──────────────────────────────────────────────────┘  │
│                                                          │
│        ┌─────┐   ← Read/Write Arm (Actuator Arm)        │
│   ━━━━━┿━━━━━┿━━━━━━━━━━━━  ← Platter                  │
│   ━━━━━┿━━━━━┿━━━━━━━━━━━━  ← Platter                  │
│   ━━━━━┿━━━━━┿━━━━━━━━━━━━  ← Platter                  │
│        │  │  │                                           │
│        │  ↑  │                                           │
│        │Spindle (Motor)                                  │
│        │  │  │                                           │
│        └──┴──┘                                           │
│                                                          │
│   [SATA Port] [Power Port]  ← Interface Connectors      │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Components in Detail

| Component           | Also Called            | Function                                               |
| ------------------- | ---------------------- | ------------------------------------------------------ |
| **Platter**         | Disk                   | Circular magnetic disk that stores data                |
| **Spindle**         | Motor                  | Rotates all platters at constant speed (5400–7200 RPM) |
| **Read/Write Arm**  | Actuator Arm           | Positions the R/W head over the correct track          |
| **Actuator**        | VCM (Voice Coil Motor) | Moves the arm assembly back and forth                  |
| **Read/Write Head** | R/W Head               | Reads magnetic fields and writes new ones              |
| **Actuator Axis**   | Pivot                  | Point around which the arm swings                      |
| **Top Cover**       | Lid                    | Hermetically sealed — dust proof                       |
| **PCB**             | Logic Board            | Controls all electronics                               |
| **SATA/SAS Port**   | Interface              | Connects drive to motherboard                          |

### 2.3 The Platter

The platter is the most critical component for forensics.

```
     Side View of a Single Platter:

     ┌────────────────────────────────┐
     │     Face-UP Side               │ ← R/W Head reads/writes here
     ├────────────────────────────────┤
     │     Platter Material           │ (aluminum or glass substrate
     │     (typically aluminum/glass) │  coated with magnetic material)
     ├────────────────────────────────┤
     │     Face-DOWN Side             │ ← Separate R/W Head below
     └────────────────────────────────┘
```

**Key facts about platters:**

- Data is stored on **both sides** (face-up and face-down)
- A single drive can contain **1–6+ platters** stacked on the spindle
- Platter material: aluminum or glass coated with magnetic oxide or thin-film alloy
- Modern platters use **perpendicular magnetic recording (PMR)** or **shingled magnetic recording (SMR)**

### 2.4 The Critical Gap — 15 Nanometers

One of the most important hardware facts for understanding both disk operation and damage:

```
     Read/Write Head
          ↓
     ═══════════════   ← Head flying height: 15 nanometers
     ───────────────   ← Platter surface
```

| Object                      | Size               |
| --------------------------- | ------------------ |
| Human hair                  | ~70,000 nanometers |
| Smoke particle              | ~10,000 nanometers |
| Dust particle (smallest)    | ~10,000 nanometers |
| **Hard disk flying height** | **~15 nanometers** |
| DNA molecule width          | ~2.5 nanometers    |

**Why this matters forensically:**

- The head **never touches** the platter during normal operation
- If the head crashes (head crash), it physically gouges the platter → **permanent data loss** in that area
- The gap is smaller than any dust particle → the casing must be hermetically sealed
- A hard disk opened outside a **cleanroom** will be contaminated → never open a hard disk in a normal room

### 2.5 Multiple Platters

```
Spindle →  │
           ├── Platter 1  (Head 0 top, Head 1 bottom)
           │
           ├── Platter 2  (Head 2 top, Head 3 bottom)
           │
           ├── Platter 3  (Head 4 top, Head 5 bottom)
           │
           ├── Platter 4  (Head 6 top, Head 7 bottom)
           │
           └── (up to 6+ platters)
```

**Rule:** Each platter surface gets its own dedicated read/write head. A 4-platter drive = 8 heads (2 per platter).

---

## 3. The Platter — Tracks, Sectors & Clusters

### 3.1 Tracks

A **track** is a complete circular ring on one side of a platter where data is stored.

```
     Top view of a platter:

               Track 0 (outermost)
            ┌──────────────────────────┐
           /   Track 1                  \
          /      ┌──────────────────┐    \
         │       │  Track 2          │    │
         │       │    ┌──────────┐   │    │
         │       │    │ Track 3  │   │    │
         │       │    │  (inner) │   │    │
         │       │    └──────────┘   │    │
         │       └──────────────────┘    │
          \                             /
           └────────────────────────────┘

     Tracks are numbered from OUTSIDE → INSIDE
     Track 0 = outermost track
     Track N = innermost track
```

**Track facts:**

- Old HDDs: ~10,000 tracks per platter side
- Modern HDDs: 500,000+ tracks per platter side
- **Track density is identical across all platters** in a drive — if platter 1 has 10 tracks, every platter has exactly 10
- Track size is **not fixed** — outer tracks are physically longer than inner tracks, but store the same amount of data (ZBR — Zoned Bit Recording varies this)

### 3.2 Sectors

A **sector** is a subdivision of a track — the smallest individually addressable unit of storage on a hard disk.

```
     One Track divided into Sectors:

     ╔════╦════╦════╦════╦════╦════╦════╦════╗
     ║ S0 ║ S1 ║ S2 ║ S3 ║ S4 ║ S5 ║ S6 ║ S7 ║  ← One track
     ╚════╩════╩════╩════╩════╩════╩════╩════╝
      ↑
      One sector (512 bytes or 4096 bytes)
```

**The name "sector" comes from mathematics** — just as a sector is a "pizza slice" of a circle, a disk sector is a slice of a circular track.

| Drive Type                   | Sector Size                     |
| ---------------------------- | ------------------------------- |
| Old HDD                      | 512 bytes                       |
| CD-ROM / DVD                 | 2048 bytes (2 KB)               |
| Modern HDD (Advanced Format) | 4096 bytes (4 KB)               |
| SSD                          | Variable (typically 4096 bytes) |

**Forensic importance:**

- Sectors are the **fundamental unit of forensic evidence**
- Even a partially burnt or water-damaged drive may have readable sectors
- Data recovery tools work **sector by sector**
- **Every sector has a unique address** (track number + sector number)

### 3.3 Analogy — Tracks and Sectors

```
CITY = Hard Disk Platter
├── COLONY (e.g., Krishnanagar) = Track
│   └── The entire colony boundary = one full circular track
│
└── STREETS within the colony = Sectors
    ├── Street 1 (Sector 0)
    ├── Street 2 (Sector 1)
    ├── Street 3 (Sector 2)
    └── ...

To find your friend's house:
"Krishnanagar Colony, Street 5, House 12"
= "Track 3, Sector 5, Offset 12"

This address system is exactly how the OS finds data on disk.
```

### 3.4 Clusters

A **cluster** is a group of consecutive sectors treated as a single allocation unit by the file system.

```
     Sectors on a track:

     [S0][S1][S2][S3][S4][S5][S6][S7][S8][S9]...
      ╰─────────────╯  ╰──────────────────────╯
       Cluster 0         Cluster 1
       (4 sectors)       (6 sectors — variable)

     More common depiction:
     [S0][S1] = Cluster 0
     [S2][S3] = Cluster 1
     [S4][S5] = Cluster 2
```

**Why clusters exist:**

If a file system managed individual sectors, the overhead of tracking millions of sectors would be enormous. Instead, the OS allocates clusters (groups of sectors) to files.

**Example:**

```
File size: 700 bytes
Sector size: 512 bytes

Sector allocation:
├── Sector 100: first 512 bytes of file
└── Sector 101: remaining 188 bytes (+ 324 bytes SLACK SPACE)

These two sectors together = 1 cluster

"Slack space" = the unused portion of the last sector/cluster
              = FORENSICALLY IMPORTANT — can contain remnants
                of previously deleted files
```

**Cluster size examples (NTFS, default):**

| Volume Size   | Cluster Size |
| ------------- | ------------ |
| 512 MB – 1 GB | 512 bytes    |
| 1 GB – 2 GB   | 1 KB         |
| 2 GB – 32 GB  | 4 KB         |
| 32 GB – 2 TB  | 8–64 KB      |

### 3.5 Relationship Summary

```
┌──────────────────────────────────────────────────────────┐
│                        PLATTER                           │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                   TRACK  (full circle)            │   │
│  │                                                   │   │
│  │  [SECTOR 0][SECTOR 1][SECTOR 2][SECTOR 3][...]   │   │
│  │   ╰────────────────╯                              │   │
│  │        CLUSTER 0                                  │   │
│  │                   ╰───────────────────╯           │   │
│  │                        CLUSTER 1                  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Platter > Track > Cluster > Sector                      │
│  (largest)                  (smallest addressable unit)  │
└──────────────────────────────────────────────────────────┘
```

---

## 4. Sector Internals & Structure

Each sector is not just raw storage — it contains a structured set of fields that enable the drive to operate, self-identify, and maintain data integrity.

### 4.1 Sector Layout

```
┌─────────────────────────────────────────────────────────────┐
│                        ONE SECTOR                           │
├──────────────┬───────────────┬───────────────┬─────────────┤
│  SYNC ZONE   │    ADDRESS    │     DATA      │    ECC      │
│  (ID Info)   │  (Location)   │  (512B/4KB)   │  (Error     │
│              │               │               │  Correction)│
├──────────────┴───────────────┴───────────────┴─────────────┤
│        GAP        │        GAP        │        GAP         │
│  (between zones)  │  (between zones)  │  (between sectors) │
└───────────────────┴───────────────────┴────────────────────┘
```

### 4.2 Field-by-Field Explanation

#### Field 1 — Synchronization Zone (ID Information)

```
Purpose: Tells the read/write head key operational parameters

Contains:
✓ Platter rotation speed (RPM)
✓ Disk spin-up confirmation signal
✓ Synchronization pattern (allows head to sync its clock to disk)
✓ Preamble for signal calibration
```

The head must synchronize with the rotating disk before it can read. This zone provides the timing reference.

#### Field 2 — Address Field (Sector Location)

```
Contains:
✓ Track number    (which circular ring)
✓ Head number     (which platter side)
✓ Sector number   (which slice of the track)

Together: CHS Address (Cylinder-Head-Sector)

Example address: Track 0, Head 0, Sector 100
Meaning: "I am the 100th sector on track 0, upper side of platter 1"
```

**Forensic importance:** Even if data is deleted or overwritten, the sector's **own address** remains. This is how forensic tools can map out every sector on a damaged drive.

#### Field 3 — Data Area

```
Old drives:    512 bytes of actual data
Modern drives: 4096 bytes (4 KB) of actual data

Example — storing a 600-byte image in 512-byte sectors:
┌────────────────────────┐  ┌────────────────────────┐
│ Sector 100             │  │ Sector 101             │
│ 512 bytes of image     │  │ 88 bytes of image      │
│ [████████████████████] │  │ [███░░░░░░░░░░░░░░░░░] │
└────────────────────────┘  └────────────────────────┘
                                   ↑
                              424 bytes = SLACK SPACE
                              (potentially contains old data)
```

#### Field 4 — ECC (Error Correcting Code)

```
Functions:
✓ Detects whether stored data has been corrupted
✓ Corrects small errors automatically (up to certain bit count)
✓ Verifies data integrity on every read
✓ Detects tampering (if data was modified, ECC won't match)

Algorithm: Reed-Solomon ECC (most common)

Forensic importance:
- ECC mismatch on a forensic image = sector corruption detected
- Tools like ddrescue and TestDisk report ECC errors per sector
- ECC can sometimes reconstruct partially corrupted sectors
```

#### Field 5 — Gaps (Inter-field and Inter-sector)

```
Why gaps exist:

After the R/W head reads 512 bytes of data, it needs
processing time before it can read the next field.

Without gap:                  With gap:
[DATA][ECC][DATA][ECC]...    [DATA][ECC][GAP][DATA][ECC][GAP]...
  ↑ Head cannot keep up         ↑ Head processes during gap
  → data corruption             → reliable reading
```

**Gap sizes are determined by drive firmware** and vary by manufacturer and drive generation.

### 4.3 The Forensic Importance of Sector Structure

```
Scenario: Criminal burns laptop and throws in river

What survives?  → Individual sectors may still be intact
                  Even if platter is damaged in some areas,
                  other sectors remain readable

What we extract:
├── Address field  → map of where every sector is
├── Data field     → actual evidence (documents, images, etc.)
├── ECC field      → verify data hasn't been corrupted
└── Sync zone      → helps read platter even if damaged

Sector-by-sector recovery:
[  OK  ][  OK  ][DAMAGE][  OK  ][  OK  ][DAMAGE][  OK  ]
   ↑        ↑      ↑       ↑       ↑       ↑       ↑
Recover  Recover  Skip  Recover  Recover  Skip   Recover

Result: Partial but valid evidence
```

---

## 5. 4K Sectors — Modern Hard Drives

### 5.1 History and Evolution

```
Timeline:
1956 – First HDD (IBM 350): 3.75 MB, 50 platters
1980s – 512-byte sector standard established
2010s – 4K sector (Advanced Format) introduced
Now   – 4K sectors standard in all modern drives

Why change?
Old: [SYNC][ADDR][512B DATA][ECC][GAP]
     Each 512-byte sector has overhead fields

New: [SYNC][ADDR][4096B DATA][ECC][GAP]
     8× more data per sector with only slightly more overhead
     → More efficient use of platter space
     → More storage in same physical size
```

### 5.2 512 Bytes vs 4096 Bytes

```
Old drive: one track = 100 sectors × 512 bytes = 51,200 bytes
New drive: one track = 100 sectors × 4096 bytes = 409,600 bytes

For the same physical space on the platter,
the new drive stores 8× more data.

This is why modern 2.5" laptop drives can hold 2TB
while older 3.5" desktop drives held only 40GB.
```

### 5.3 512e vs 4Kn Drives

| Type                    | Physical Sector | Logical Sector       | Compatibility       |
| ----------------------- | --------------- | -------------------- | ------------------- |
| **512n** (old)          | 512 bytes       | 512 bytes            | All systems         |
| **512e** (transitional) | 4096 bytes      | 512 bytes (emulated) | Most systems        |
| **4Kn** (native)        | 4096 bytes      | 4096 bytes           | Modern systems only |

**Forensic implication:** When imaging a 512e drive, your tool must be aware of the 4K physical sector size to avoid alignment errors that corrupt evidence.

---

## 6. Cylinders & Head Assemblies

### 6.1 What Is a Cylinder?

```
A cylinder is the set of all tracks at the same position
across all platters and both faces.

Side view of a 3-platter drive:

Position X (e.g., Track 5 on all platters)

Platter 1 top:    ════════════════  ← Track 5, Head 0
                  [platter]
Platter 1 bottom: ════════════════  ← Track 5, Head 1

Platter 2 top:    ════════════════  ← Track 5, Head 2
                  [platter]
Platter 2 bottom: ════════════════  ← Track 5, Head 3

Platter 3 top:    ════════════════  ← Track 5, Head 4
                  [platter]
Platter 3 bottom: ════════════════  ← Track 5, Head 5

All these tracks at position X = CYLINDER 5
```

**Why cylinders matter:**

The actuator arm moves all heads **simultaneously**. When the arm is at position X, it can read/write Track X on **all platters at once** without moving. This is faster than moving the arm for each platter separately.

### 6.2 CHS Addressing (Cylinder-Head-Sector)

The traditional method for addressing any location on a hard disk uses three values:

```
CHS = (Cylinder, Head, Sector)

Example: (5, 2, 100)
Means:
├── Cylinder 5    → arm at position 5 (5th track from outside)
├── Head 2        → select head 2 (platter 2, top face)
└── Sector 100    → read sector 100 on that track

Maximum values (old drives):
├── Cylinders: 1024
├── Heads: 255
└── Sectors per track: 63

Maximum addressable: 1024 × 255 × 63 × 512 bytes ≈ 8.46 GB
(This is why old BIOSes couldn't see drives larger than 8 GB)
```

### 6.3 LBA Addressing (Logical Block Addressing)

Modern drives use **LBA** — a simpler linear numbering system:

```
LBA replaces CHS with a single sequential number:

LBA 0   = first sector of drive
LBA 1   = second sector
LBA 2   = third sector
...
LBA N   = last sector

Conversion formula:
LBA = (C × HPC + H) × SPT + (S - 1)
Where: C=Cylinder, HPC=Heads Per Cylinder, H=Head, SPT=Sectors Per Track, S=Sector

Modern drives support 48-bit LBA → max 128 PB (petabytes)
```

**Forensic tools use LBA** — when you see output like `sector 2048`, that is an LBA number.

---

## 7. Magnetic Data Encoding

### 7.1 The Binary-to-Magnetic Mapping

All data in a computer is binary (0s and 1s). The hard disk stores these by magnetizing tiny regions of the platter.

```
Binary representation:  1  0  1  1  0  0  1  0
                        ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓
Magnetic orientation:   N  S  N  N  S  S  N  S

Where:
N = North pole facing up  = represents 1
S = South pole facing up  = represents 0
```

### 7.2 Method 1 — Direct Polarity Encoding

```
Binary:   1   0   1   0   1   1   0
          ↓   ↓   ↓   ↓   ↓   ↓   ↓
Poles:    N   S   N   S   N   N   S
         [↑] [↓] [↑] [↓] [↑] [↑] [↓]

Where [↑] = North pole up = 1
      [↓] = South pole up = 0

The R/W head detects field direction → reads 1 or 0
```

### 7.3 Method 2 — Transition Encoding (NRZ / NRZI)

Modern drives use transition encoding — it is a **change in direction** (transition) that represents a 1, while **no change** represents 0:

```
Binary:    1   0   1   1   0   0   1
           ↓   ↓   ↓   ↓   ↓   ↓   ↓

Poles:    S→N  N   N→S  S→N  N   N   N→S
            ↑       ↑    ↑            ↑
        transition  transition   transition
           = 1        = 1           = 1
         ↑      ↑
       no change between transitions = 0

Advantage: More resistant to errors from noise
```

### 7.4 How Data Is Written

```
Step-by-step write process:

1. OS sends write command with LBA address and data
2. Drive firmware translates LBA → physical CHS address
3. Actuator arm moves to correct cylinder
4. Correct head selected
5. Spindle rotates platter to correct sector position
6. Write current flows through write coil in head
7. Magnetic field generated flips domain orientation in platter
8. Domains are permanently magnetized (until overwritten)

Physics: The write coil generates a field > 1000 Oersteds
         to overcome the platter's coercivity
```

### 7.5 How Data Is Read

```
Step-by-step read process:

1. Actuator arm positions head over correct track
2. Head flies at 15nm above platter surface
3. As magnetized domains pass beneath the head,
   the changing field induces a voltage in the read element
4. Signal amplified by pre-amplifier chip (inside drive)
5. Signal sent through flexible cable to PCB
6. ADC converts analog signal to digital
7. ECC checked and corrected
8. Data returned to OS

Modern read head technology: GMR (Giant Magnetoresistance)
or TMR (Tunneling Magnetoresistance)
```

### 7.6 Platter Materials

| Layer           | Material                                | Function                      |
| --------------- | --------------------------------------- | ----------------------------- |
| Substrate       | Aluminum alloy or glass                 | Structural rigidity           |
| NiP layer       | Nickel-phosphorus                       | Hard surface, smoothness      |
| Magnetic layer  | CoCrPt alloy (cobalt-chromium-platinum) | Actually stores data          |
| Carbon overcoat | Diamond-like carbon (DLC)               | Protects against head contact |
| Lubricant       | Perfluoropolyether (PFPE)               | Reduces friction              |

**Forensic implication:** Even if the platter is scratched, the magnetic layer beneath the scratch may still be intact. Specialists can sometimes read data through scratches using specialized heads.

---

## 8. The Read/Write Head — Mechanism

### 8.1 Head Structure

```
                    ┌──────────────────┐
                    │   Actuator Arm   │
                    └────────┬─────────┘
                             │ (flexible cable carries signals)
                    ┌────────┴─────────┐
                    │   Head Gimbal    │ (suspension mechanism)
                    │   Assembly (HGA) │
                    └────────┬─────────┘
                             │
                    ┌────────┴─────────┐
                    │   Slider         │ (aerodynamic body)
                    │                  │
                    │  ┌────┐ ┌─────┐  │
                    │  │Wrt │ │Read │  │
                    │  │Head│ │Head │  │
                    │  └────┘ └─────┘  │
                    └──────────────────┘
                    (flies 15nm above platter)
```

### 8.2 Write Head

```
Write Head mechanism:

        Coil (carries write current)
           ↓
    ┌──────────────┐
    │      ∩       │  ← Magnetic core (iron/permalloy)
    │   ╱   ╲      │
    │  ╱     ╲     │
    └──────────────┘
           ↕
    15 nm gap
    ═══════════════  ← Platter magnetic layer

Write current → magnetic field in core → field at gap
→ magnetizes platter domains in North or South direction
→ binary 1 or 0 stored permanently
```

### 8.3 Read Head

Modern drives use **GMR (Giant Magnetoresistance)** or **TMR (Tunneling Magnetoresistance)** read heads:

```
Read Head mechanism (GMR):

        Sensing element
           ↓
    ┌──────────────┐
    │   [GMR]      │  ← Resistance changes with magnetic field
    └──────────────┘
           ↕
    15 nm gap
    ═══════════════  ← Platter (domains rotating past)

As domains pass:
→ North domain: GMR resistance = HIGH  → voltage = HIGH → "1"
→ South domain: GMR resistance = LOW   → voltage = LOW  → "0"
→ Signal amplified, digitized, ECC checked
```

### 8.4 Head Crash — The Worst Failure

```
Normal operation:
Head flying at 15nm
═══════════════════  ← Platter surface — intact

Head crash (caused by shock, dust, power surge):
Head touches platter
═══════════════╤════  ← Platter gouged
               ↑
         Physical scratch
         Data in this area = PERMANENTLY DESTROYED
         (magnetic material removed)
```

**Head crash forensics:**

- The scratch pattern on the platter can indicate **direction of impact**
- Areas outside the crash may still have recoverable data
- In a lab, damaged heads can be swapped from a **donor drive** (same model/firmware)
- This procedure must be done in a **Class 100 cleanroom**

---

## 9. Data Units — Bits, Bytes & Conversions

### 9.1 Fundamental Units

```
BIT (Binary Digit)
└── Smallest unit of digital data
└── Value: 0 or 1 (South or North pole on disk)
└── Symbol: b (lowercase)

BYTE
└── 8 bits grouped together
└── Can represent values 0–255
└── Symbol: B (uppercase)

Example:
Letter 'A' in ASCII = 01000001 = 8 bits = 1 byte
```

### 9.2 Storage Unit Table

```
Unit         Symbol    Size in Bytes         Power of 2
─────────────────────────────────────────────────────────
Bit           b        1/8 byte              2^0 (bits)
Byte          B        1 byte                2^0
Kilobyte      KB       1,024 bytes           2^10
Megabyte      MB       1,048,576 bytes       2^20
Gigabyte      GB       1,073,741,824 bytes   2^30
Terabyte      TB       ~1.099 × 10^12 bytes  2^40
Petabyte      PB       ~1.126 × 10^15 bytes  2^50
Exabyte       EB       ~1.153 × 10^18 bytes  2^60
```

### 9.3 Memory Tricks

```
2^10 = 1,024  ≈ 1,000  → Kilo
2^20 = 1,048,576        → Mega
2^30 = 1,073,741,824    → Giga
2^40 = 1,099,511,627,776 → Tera

Pattern: every 10 powers of 2 = next unit
```

### 9.4 Conversion Examples

```python
# Python conversions (for lab practice)

def bytes_to_gb(bytes_val):
    return bytes_val / (2**30)

def bytes_to_mb(bytes_val):
    return bytes_val / (2**20)

def bits_to_bytes(bits):
    return bits / 8

# Example: 512 bytes sector
sector = 512
print(f"512 bytes = {sector * 8} bits")
print(f"1024 bytes = {1024/1024} KB")
print(f"1 GB = {2**30} bytes = {2**30 * 8} bits")

# Common conversions:
# 1 KB = 1024 bytes = 8192 bits
# 1 MB = 1,048,576 bytes
# 1 GB = 1,073,741,824 bytes
# 512 byte sectors per GB = 1,073,741,824 / 512 = 2,097,152 sectors
```

---

## 10. Hard Disk Size Calculation

### 10.1 The Formula

```
Disk Size (bytes) =
    Number of Platters
  × Surfaces per Platter (always 2)
  × Tracks per Surface
  × Sectors per Track
  × Bytes per Sector
```

### 10.2 Step-by-Step Calculation Example

**Given (from a recovered burned drive):**

```
Number of platters:      2
Surfaces per platter:    2  (always — top and bottom)
Tracks per surface:      256
Sectors per track:       512
Bytes per sector:        512
```

**Calculation:**

```
Step 1: Total surfaces
= Platters × 2
= 2 × 2 = 4 surfaces

Step 2: Total tracks
= Total surfaces × Tracks per surface
= 4 × 256 = 1,024 tracks

Step 3: Total sectors
= Total tracks × Sectors per track
= 1,024 × 512 = 524,288 sectors

Step 4: Total bytes
= Total sectors × Bytes per sector
= 524,288 × 512 = 268,435,456 bytes

Step 5: Convert to GB
= 268,435,456 / (1024 × 1024 × 1024)
= 268,435,456 / 1,073,741,824
= 0.25 GB = 256 MB

Answer: This is a 256 MB hard drive (old drive)
```

### 10.3 Practice Problem 1 (from lecture)

**Given:**

```
Platters:              2
Surfaces per platter:  2
Tracks per platter:    100,000 (1 lakh)
Sectors per track:     500
Bytes per sector:      512
```

**Your calculation:**

```
Total bytes = 2 × 2 × 100,000 × 500 × 512
            = 4 × 100,000 × 500 × 512
            = 4 × 25,600,000,000
            = 102,400,000,000 bytes
            = 102,400,000,000 / 1,073,741,824 GB
            = ≈ 95.4 GB
```

### 10.4 Practice Problem 2 (from lecture)

**Given:**

```
Platters:              4
Surfaces per platter:  2  (heads = 8, given — do NOT multiply by heads separately)
Tracks per platter:    200,000 (2 lakh)
Sectors per track:     1,000
Bytes per sector:      512
```

**Your calculation:**

```
Total bytes = 4 × 2 × 200,000 × 1,000 × 512
            = 8 × 200,000 × 1,000 × 512
            = 8 × 102,400,000,000
            = 819,200,000,000 bytes
            = 819,200,000,000 / 1,073,741,824
            ≈ 763 GB ≈ 0.74 TB

Note: The 8 heads mentioned = 4 platters × 2 sides
      Already accounted for in "2 surfaces per platter"
      DO NOT multiply by heads separately — that is double counting
```

### 10.5 Python Script for Any Drive

```python
#!/usr/bin/env python3
"""
Hard Disk Size Calculator
Used in forensics to determine drive capacity from physical parameters
"""

def calculate_disk_size(platters, tracks_per_surface, sectors_per_track, bytes_per_sector):
    """
    Calculate total disk capacity in bytes, KB, MB, GB, TB

    Parameters:
        platters          : number of disk platters
        tracks_per_surface: tracks on one side of one platter
        sectors_per_track : sectors on one track
        bytes_per_sector  : size of one sector (512 or 4096)
    """
    surfaces = platters * 2   # each platter has top and bottom

    total_tracks   = surfaces * tracks_per_surface
    total_sectors  = total_tracks * sectors_per_track
    total_bytes    = total_sectors * bytes_per_sector

    total_kb = total_bytes / 1024
    total_mb = total_kb / 1024
    total_gb = total_mb / 1024
    total_tb = total_gb / 1024

    print(f"\n{'='*50}")
    print(f"HARD DISK SIZE CALCULATION")
    print(f"{'='*50}")
    print(f"Input Parameters:")
    print(f"  Platters:             {platters}")
    print(f"  Surfaces (×2):        {surfaces}")
    print(f"  Tracks per surface:   {tracks_per_surface:,}")
    print(f"  Sectors per track:    {sectors_per_track:,}")
    print(f"  Bytes per sector:     {bytes_per_sector}")
    print(f"\nResults:")
    print(f"  Total surfaces:       {surfaces}")
    print(f"  Total tracks:         {total_tracks:,}")
    print(f"  Total sectors:        {total_sectors:,}")
    print(f"  Total bytes:          {total_bytes:,}")
    print(f"  Total KB:             {total_kb:,.2f}")
    print(f"  Total MB:             {total_mb:,.2f}")
    print(f"  Total GB:             {total_gb:,.4f}")
    print(f"  Total TB:             {total_tb:,.6f}")
    print(f"{'='*50}\n")

    return total_bytes

# Example 1 from lecture
calculate_disk_size(
    platters=2,
    tracks_per_surface=256,
    sectors_per_track=512,
    bytes_per_sector=512
)

# Example 2 from lecture
calculate_disk_size(
    platters=2,
    tracks_per_surface=100_000,
    sectors_per_track=500,
    bytes_per_sector=512
)

# Example 3 from lecture
calculate_disk_size(
    platters=4,
    tracks_per_surface=200_000,
    sectors_per_track=1_000,
    bytes_per_sector=512
)

# Your own test
calculate_disk_size(
    platters=1,
    tracks_per_surface=10_000,
    sectors_per_track=63,
    bytes_per_sector=512
)
```

---

## 11. Disk Formatting & File Systems Overview

### 11.1 What Happens During Format

```
Before format:
Platter has tracks and sectors with random (previous) data
Cluster allocations exist pointing to old files

During format (quick):
├── Master File Table (MFT) or FAT cleared/reset
├── Cluster allocation table wiped
├── Data in sectors: UNTOUCHED (this is why "quick format" is recoverable)
└── OS now treats all clusters as "free"

During format (full / low-level):
├── MFT/FAT cleared
├── Every sector overwritten with zeros
├── Bad sectors mapped out
└── Data in sectors: OVERWRITTEN (harder to recover)

After format:
Drive behaves as if empty but sectors still
contain old data until overwritten by new data
```

**Forensic implication:** A "quick formatted" drive can be imaged and analyzed — the old data is still physically present on the platters. Tools like Autopsy, Recuva, or TestDisk can recover it.

### 11.2 File System Overview

| File System  | OS                      | Max File Size | Max Volume | Notes            |
| ------------ | ----------------------- | ------------- | ---------- | ---------------- |
| **FAT16**    | DOS, old Windows        | 2 GB          | 2 GB       | Very old         |
| **FAT32**    | Windows 95+, USB drives | 4 GB          | 8 TB       | Common on USB    |
| **NTFS**     | Windows XP–11           | 16 TB         | 256 TB     | Standard Windows |
| **exFAT**    | Flash drives, SD cards  | 128 PB        | 128 PB     | Cross-platform   |
| **ext2/3/4** | Linux                   | 2 TB / 16 TB  | 32 TB      | Linux standard   |
| **APFS**     | macOS 10.13+            | 8 EB          | 8 EB       | Modern Apple     |
| **HFS+**     | macOS (older)           | 8 EB          | 8 EB       | Older Apple      |
| **XFS**      | Linux servers           | 8 EB          | 8 EB       | High-performance |

**Forensic relevance:** Different file systems store file metadata differently. The forensic analyst must know:

- **NTFS:** stores timestamps (created, modified, accessed, MFT changed) — the $MFT is a goldmine of evidence
- **FAT32:** minimal metadata — easier to understand but less evidence
- **ext4:** stores journal (transaction log) — can reconstruct recent file operations

---

## 12. Forensic Lab Exercises

### Lab 1 — Disk Structure Exploration (Linux)

```bash
# View partition table of your drive
sudo fdisk -l /dev/sda

# Output shows:
# Device     Start       End   Sectors   Size Type
# /dev/sda1   2048   1050623   1048576   512M EFI System
# /dev/sda2 1050624 500118158 499067535 238G Linux filesystem

# Each "Start" and "End" value is an LBA sector number

# View disk geometry
sudo hdparm -g /dev/sda

# Output:
# geometry = 60801/255/63, sectors = 976773168, start = 0

# Display drive information
sudo hdparm -I /dev/sda
# Shows: model, firmware, serial, sector count, LBA support

# Check sector size
sudo fdisk -l /dev/sda | grep "Sector size"
# Sector size (logical/physical): 512 bytes / 4096 bytes
#                                   ↑               ↑
#                               Logical (512e)   Physical (4K native)
```

### Lab 2 — Sector-Level Reading with `dd`

```bash
# WARNING: Use ONLY on your own drives or lab drives
# dd reads/writes raw sector data

# Read the first sector (MBR — Master Boot Record)
sudo dd if=/dev/sda of=mbr_backup.bin bs=512 count=1

# Read the first 100 sectors
sudo dd if=/dev/sda of=first_100_sectors.bin bs=512 count=100

# Hexdump the MBR to see its structure
xxd mbr_backup.bin | head -50

# Look for the MBR signature at bytes 510-511: 55 AA
xxd mbr_backup.bin | grep "55 aa"

# Read a specific sector (LBA 2048 = typical start of partition)
sudo dd if=/dev/sda of=sector_2048.bin bs=512 skip=2048 count=1
xxd sector_2048.bin | head -30
```

### Lab 3 — Create a Disk Image (Forensic Imaging)

```bash
# Install tools
sudo apt install dcfldd ewf-tools

# Method 1: dd (basic)
sudo dd if=/dev/sdb of=/mnt/evidence/drive.img bs=4096 status=progress

# Method 2: dcfldd (with hashing)
sudo dcfldd if=/dev/sdb of=/mnt/evidence/drive.img \
  hash=sha256 \
  hashlog=/mnt/evidence/drive_hash.txt \
  bs=4096 \
  status=on

# Method 3: ddrescue (for damaged drives — preferred)
sudo ddrescue -d -r3 /dev/sdb /mnt/evidence/drive.img /mnt/evidence/drive.log
# -d = direct mode (no kernel cache)
# -r3 = retry bad sectors 3 times

# Verify image integrity
sha256sum /dev/sdb           # Hash of original
sha256sum /mnt/evidence/drive.img  # Hash of image
# These MUST match for court admissibility

# Mount image read-only for analysis
sudo mount -o ro,loop /mnt/evidence/drive.img /mnt/analysis
```

### Lab 4 — Examining Sector Contents

```bash
# Install Sleuth Kit
sudo apt install sleuthkit

# Show partition information from image
mmls drive.img

# Show file system details
fsstat -i raw drive.img

# List all files (including deleted)
fls -r drive.img 2> /dev/null | head -50
# Output:
# r/r 4-128-1: $MFT
# r/r 8-128-2: $BadClus
# d/d 11-144-3: $Extend
# r/- 0: image.jpg   ← r/- = deleted file, still recoverable

# Extract a specific file by inode
icat drive.img 32 > recovered_file.jpg

# Analyze a specific sector
blkcat drive.img 2048 | xxd | head -20
```

### Lab 5 — File Recovery from Formatted Drive

```bash
# Install TestDisk (includes PhotoRec)
sudo apt install testdisk

# Run TestDisk for partition recovery
sudo testdisk drive.img

# Inside TestDisk:
# → Analyse → Quick Search → [shows lost partitions]
# → Write [to restore partition table]

# Run PhotoRec for file carving (recovers files from any sector)
sudo photorec drive.img
# → Select partition type
# → Select file types to recover
# → Choose output directory
# PhotoRec carves files based on file headers, ignoring file system

# Alternative: Foremost (command-line file carving)
sudo apt install foremost
foremost -i drive.img -o ./recovered_files/ -v
# Recovers: jpg, gif, png, bmp, avi, exe, mpg, wav, riff, wmv, mov, pdf, ole, doc, zip, rar, htm, cpp

# Scalpel (faster alternative to foremost)
sudo apt install scalpel
scalpel drive.img -o ./scalpel_output/
```

### Lab 6 — Analyzing NTFS with Autopsy

```bash
# Install Autopsy (GUI forensic tool)
sudo apt install autopsy

# Launch
autopsy &
# Open browser to: http://localhost:9999/autopsy

# Or use command line Sleuth Kit:
# 1. Open case with image
# 2. Analyze $MFT for file metadata
# 3. Look at timeline
# 4. Check deleted files
# 5. Extract artifacts

# Command-line NTFS analysis:
# List all files including deleted
fls -r -p drive.img

# Get file metadata (MAC times — Modified, Accessed, Changed)
istat drive.img 32
# Output shows:
# Modified:  2024-01-15 14:32:11
# Accessed:  2024-01-16 09:12:44
# Changed:   2024-01-15 14:32:11
# Born:      2023-12-01 10:00:00

# Timeline analysis — generate body file
fls -r -m / drive.img > bodyfile.txt
# Create timeline sorted by MAC times
mactime -b bodyfile.txt -d > timeline.csv
```

### Lab 7 — Magnetic Data Visualization (Python)

```python
#!/usr/bin/env python3
"""
Visualize binary data as it would be stored magnetically on a platter
Educational tool for understanding sector storage
"""

def binary_to_magnetic(data_string):
    """Convert a string to its binary representation and show magnetic encoding"""

    # Convert string to binary
    binary = ''.join(format(ord(c), '08b') for c in data_string)

    print(f"Original data: '{data_string}'")
    print(f"Binary:        {binary}")
    print(f"Length:        {len(binary)} bits = {len(binary)//8} bytes")
    print()

    # Show magnetic encoding
    print("Magnetic encoding (N=1, S=0):")
    print("-" * 60)

    magnetic = ""
    for bit in binary:
        if bit == '1':
            magnetic += "N "
        else:
            magnetic += "S "

    # Display in groups of 8 (one byte per group)
    print("Binary:    ", end="")
    for i, bit in enumerate(binary):
        print(bit, end=" ")
        if (i + 1) % 8 == 0:
            print(" ", end="")
    print()

    print("Magnetic:  ", end="")
    for i, bit in enumerate(binary):
        pole = "N" if bit == "1" else "S"
        print(pole, end=" ")
        if (i + 1) % 8 == 0:
            print(" ", end="")
    print()
    print("-" * 60)

def show_sector_structure(data_bytes=512):
    """Show the structure of a sector"""
    print(f"\nSECTOR STRUCTURE ({data_bytes} byte data area)")
    print("=" * 70)

    fields = [
        ("SYNC ZONE",    20,  "Drive speed, synchronization pattern"),
        ("ADDRESS",      40,  "Track#, Head#, Sector# (CHS address)"),
        ("GAP",          10,  "Processing pause for R/W head"),
        ("DATA AREA",   data_bytes, f"Actual data ({data_bytes} bytes)"),
        ("ECC",          50,  "Error Correction Code (Reed-Solomon)"),
        ("GAP",          20,  "Inter-sector gap before next sector"),
    ]

    total = sum(f[1] for f in fields)

    for name, size, description in fields:
        bar_len = max(3, int(size / total * 50))
        bar = "█" * bar_len
        pct = size / total * 100
        print(f"{name:12} [{bar:<52}] {pct:5.1f}%")
        print(f"{'':12}  → {description}")
        print()

def calculate_storage(platters, tracks, sectors, sector_bytes=512):
    """Calculate and display storage breakdown"""
    surfaces = platters * 2
    total_tracks = surfaces * tracks
    total_sectors = total_tracks * sectors
    total_bytes = total_sectors * sector_bytes

    print(f"\nSTORAGE CALCULATION")
    print("=" * 50)
    print(f"Platters:           {platters}")
    print(f"Surfaces (×2):      {surfaces}")
    print(f"Tracks/surface:     {tracks:,}")
    print(f"Sectors/track:      {sectors:,}")
    print(f"Bytes/sector:       {sector_bytes}")
    print(f"{'─'*40}")
    print(f"Total sectors:      {total_sectors:,}")
    print(f"Total bytes:        {total_bytes:,}")
    print(f"Total KB:           {total_bytes/1024:,.1f}")
    print(f"Total MB:           {total_bytes/1024**2:,.2f}")
    print(f"Total GB:           {total_bytes/1024**3:,.4f}")
    print(f"Total TB:           {total_bytes/1024**4:,.6f}")

# Run demonstrations
if __name__ == "__main__":
    # Show binary/magnetic encoding
    binary_to_magnetic("Hi")

    # Show sector structure
    show_sector_structure(512)   # Old drive
    show_sector_structure(4096)  # Modern drive

    # Calculate drive sizes
    calculate_storage(2, 256, 512, 512)
    calculate_storage(4, 200_000, 1_000, 512)
```

---

## 13. Evidence Recovery from Damaged Media

### 13.1 Types of Damage and Recovery Approaches

| Damage Type                             | Recovery Approach                                      | Success Rate |
| --------------------------------------- | ------------------------------------------------------ | ------------ |
| **Logical damage** (deleted, formatted) | Software tools (Autopsy, Recuva, PhotoRec)             | Very High    |
| **Water damage**                        | Dry carefully, replace PCB if shorted                  | High         |
| **Fire damage**                         | Depends on temperature — platters survive up to ~400°C | Medium       |
| **Physical shock**                      | Head replacement in cleanroom                          | High         |
| **Platter scratch**                     | Specialist tools, partial recovery                     | Low–Medium   |
| **Degaussed**                           | No recovery possible                                   | Zero         |
| **Encrypted + wiped**                   | Key required — computational infeasibility             | Zero         |
| **SSD wiped**                           | TRIM makes recovery very difficult                     | Very Low     |

### 13.2 Water Damage Recovery Protocol

```
Step-by-step (do NOT do this on evidence without documentation):

1. DO NOT power on the wet drive
   → Powering on = short circuit = permanent damage

2. Rinse with deionized or distilled water
   → Tap water has minerals that cause corrosion
   → Distilled water is safe to rinse with

3. Allow to air dry (room temperature, 24–48 hours)
   OR use dry nitrogen gas to speed up

4. Inspect PCB for corrosion:
   → If corroded, transplant PCB from identical donor drive
   → Must match: manufacturer, model, firmware version

5. Attempt to mount in forensic duplicator:
   → If drive spins, image immediately
   → Use ddrescue with log file to track bad sectors

6. If drive does not spin:
   → Professional cleanroom intervention required
   → Head swap or platter transfer
```

### 13.3 Fire Damage Recovery Protocol

```
Temperature effects on HDD components:

< 100°C  → No damage (normal operating temp is 30-50°C)
100–200°C → PCB may fail, drive electronics damaged
200–350°C → Lubricant burns off, head crash likely
350–450°C → Platter coating may be affected
> 500°C  → Aluminum platters melt (MP = 660°C)
           Glass platters survive longer
> 1000°C → Complete destruction of all magnetic material

Recovery process:
1. Document the condition photographically
2. Remove platters carefully (cleanroom)
3. Clean platter surface if soot/ash present
4. Transfer platters to identical donor drive assembly
5. Attempt imaging with ddrescue
```

### 13.4 Deleted File Recovery — How It Works

```
When a file is "deleted" in Windows/Linux:

What changes:
├── MFT entry marked as "not in use" (NTFS)
│   OR FAT entry set to 0x00 (FAT32)
└── That's it — no data is touched

What does NOT change:
└── The actual sectors containing file data
    → Data remains until overwritten by new files

Recovery:
1. Image the drive (always image before recovery)
2. Scan for "unallocated" clusters
3. Look for file signatures (headers) in those clusters
4. Reconstruct files from those clusters
5. Verify with ECC data

File carving (when MFT/FAT is gone):
→ PhotoRec, Foremost, Scalpel scan every byte
→ Find file headers: JPEG = FF D8 FF, PDF = 25 50 44 46
→ Carve from header to file footer (or next header)
→ Recover files regardless of file system state
```

### 13.5 Overwritten Data — Can It Be Recovered?

```
Single overwrite (zeros):
→ Magnetic domains weakly retain previous orientation
→ Specialized equipment (MFM - Magnetic Force Microscopy)
   can sometimes detect residual magnetism
→ Very expensive, time-consuming, rare in practice
→ NIST SP 800-88 says single overwrite is sufficient
   (contradicts older DoD 5220.22-M standard)

Multiple overwrites:
→ 3–7 passes make recovery practically impossible
→ Gutmann method (35 passes) — considered overkill now

Modern drives (2007+):
→ High bit density makes residual signal detection extremely difficult
→ Single secure overwrite is considered forensically unrecoverable

SSDs with TRIM:
→ TRIM command instructs SSD to zero erased blocks immediately
→ Deleted data on SSD is almost immediately unrecoverable
→ This is a significant difference from HDD forensics
```

---

## 14. Tools Reference

### 14.1 Imaging Tools

| Tool           | Type          | Best For                          |
| -------------- | ------------- | --------------------------------- |
| **dd**         | CLI           | Basic imaging, healthy drives     |
| **dcfldd**     | CLI           | dd with hashing and logging       |
| **ddrescue**   | CLI           | Damaged drives, bad sectors       |
| **Guymager**   | GUI           | Fast imaging, E01 format          |
| **FTK Imager** | GUI (Windows) | Industry standard, court-accepted |
| **Paladin**    | Live CD       | Bootable forensic toolkit         |

### 14.2 Analysis Tools

| Tool                   | Type | Best For                       |
| ---------------------- | ---- | ------------------------------ |
| **Autopsy**            | GUI  | Full case management, timeline |
| **Sleuth Kit (TSK)**   | CLI  | Command-line disk analysis     |
| **Volatility**         | CLI  | Memory forensics               |
| **Wireshark**          | GUI  | Network traffic analysis       |
| **Bulk Extractor**     | CLI  | Fast artifact extraction       |
| **Plaso/log2timeline** | CLI  | Timeline generation            |
| **RegRipper**          | CLI  | Windows registry analysis      |

### 14.3 Recovery Tools

| Tool         | Type          | Best For                         |
| ------------ | ------------- | -------------------------------- |
| **PhotoRec** | CLI           | File carving (200+ formats)      |
| **TestDisk** | CLI           | Partition recovery               |
| **Recuva**   | GUI (Windows) | Simple deleted file recovery     |
| **Foremost** | CLI           | File carving                     |
| **Scalpel**  | CLI           | File carving (faster)            |
| **R-Studio** | GUI           | Professional commercial recovery |

### 14.4 Command Reference

```bash
# DISK INFO
sudo fdisk -l                              # List all disks
sudo lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
sudo hdparm -I /dev/sda                    # Drive details
sudo smartctl -a /dev/sda                  # SMART health data

# IMAGING
sudo dd if=/dev/sdb of=image.img bs=4096 status=progress
sudo ddrescue -d -r3 /dev/sdb image.img image.log
sudo dcfldd if=/dev/sdb of=image.img hash=sha256 hashlog=hash.txt

# HASHING (Evidence Integrity)
md5sum image.img
sha256sum image.img
sha1sum image.img

# ANALYSIS (Sleuth Kit)
mmls image.img                  # Partition table
fsstat image.img                # File system details
fls -r image.img                # File listing (incl. deleted)
istat image.img [inode]         # File metadata (MAC times)
icat image.img [inode]          # File content
blkcat image.img [LBA]          # Sector content

# HEX EXAMINATION
xxd image.img | head -50        # Hex dump beginning
xxd image.img | grep "ff d8"    # Find JPEG headers
hexdump -C image.img | head -50

# FILE CARVING
photorec image.img              # GUI-guided recovery
foremost -i image.img -o out/   # Automatic carving
scalpel image.img -o out/       # Fast carving

# MOUNTING
sudo mount -o ro,loop image.img /mnt/analysis  # Read-only mount
```

---

## 15. Quick Reference Cheatsheet

### Hard Disk Components

```
Platter      → magnetic disk storing data (both sides)
Spindle      → motor rotating platters (5400–7200 RPM)
R/W Head     → reads and writes magnetic data (flies at 15nm)
Actuator Arm → positions R/W head over correct track
PCB          → control electronics
Flying height→ 15 nanometers (smaller than any dust particle)
```

### Storage Hierarchy (large → small)

```
Drive → Platter → Track → Cluster → Sector → Bit
```

### Track Facts

```
= Complete circular ring on platter surface
= Old drives: 10,000+ tracks per surface
= Modern drives: 500,000+ tracks per surface
= Numbered from 0 (outer) inward
= Same track count on every platter
```

### Sector Facts

```
= Subdivision of a track (smallest addressable unit)
= Old/standard: 512 bytes
= CD/DVD: 2048 bytes
= Modern HDD/SSD: 4096 bytes (4K sector)
= Contains: sync zone + address + data + ECC + gaps
```

### Cluster Facts

```
= Group of consecutive sectors
= Allocated by file system (not hardware)
= Size varies with volume size and file system
= "Slack space" at end of last cluster = forensically valuable
```

### Cylinder Facts

```
= All tracks at same position across all platters
= All heads move together → cylinder read without arm movement
```

### Magnetic Encoding

```
North pole up = 1
South pole up = 0
Modern: flux transitions = 1, no transition = 0
```

### Size Calculation Formula

```
Bytes = Platters × 2 × Tracks_per_surface × Sectors_per_track × Bytes_per_sector
```

### Data Units

```
1 byte    = 8 bits
1 KB      = 1,024 bytes     = 2^10 bytes
1 MB      = 1,048,576 bytes = 2^20 bytes
1 GB      = 2^30 bytes
1 TB      = 2^40 bytes
```

### Forensic Principles

```
Quick format → data still present (only allocation table cleared)
Full format  → sectors overwritten (harder to recover)
Deleted file → sectors freed, data untouched until overwritten
File carving → find files by header/footer, bypasses file system
Evidence integrity → hash before and after imaging (must match)
Chain of custody → document every step, every person who touched drive
```

### Key Forensic File Signatures (for file carving)

```
JPEG:   FF D8 FF          (header)  FF D9 (footer)
PNG:    89 50 4E 47       (header)  49 45 4E 44 AE 42 60 82 (footer)
PDF:    25 50 44 46 2D    (header)  %%EOF (footer)
ZIP:    50 4B 03 04       (header)
RAR:    52 61 72 21       (header)
DOCX:   50 4B 03 04       (same as ZIP — Office Open XML)
EXE:    4D 5A             (MZ header)
GIF:    47 49 46 38       (GIF8)
MP3:    FF FB or ID3
AVI:    52 49 46 46       (RIFF)
```

---

## Homework & Lab Assignments

### Assignment 1 — Size Calculation

Calculate the size of the following drives. Show all working:

```
Drive A:
- Platters: 3
- Tracks per surface: 500,000
- Sectors per track: 1,000
- Bytes per sector: 4,096

Drive B:
- Platters: 6
- Tracks per surface: 100,000
- Sectors per track: 512
- Bytes per sector: 512
- Heads: 12 (hint: don't use this separately)
```

### Assignment 2 — Sector Hex Analysis

```bash
# Create a test disk image
dd if=/dev/zero of=test_disk.img bs=1M count=100
mkfs.ext4 test_disk.img

# Copy some files into it
mkdir /tmp/mnt_test
sudo mount -o loop test_disk.img /tmp/mnt_test
echo "Forensic evidence text" | sudo tee /tmp/mnt_test/evidence.txt
sudo umount /tmp/mnt_test

# Delete the file
sudo mount -o loop test_disk.img /tmp/mnt_test
sudo rm /tmp/mnt_test/evidence.txt
sudo umount /tmp/mnt_test

# Now recover it:
# 1. Use PhotoRec on test_disk.img
# 2. Use foremost on test_disk.img
# 3. Manually grep for the string: grep -a "Forensic" test_disk.img
# Document: what did each method find?
```

### Assignment 3 — MBR Analysis

```bash
# Extract the MBR from your practice disk image
dd if=test_disk.img of=mbr.bin bs=512 count=1

# Analyze with xxd
xxd mbr.bin

# Identify:
# 1. Bootstrap code (bytes 0–445)
# 2. Partition table (bytes 446–509) — 4 entries × 16 bytes
# 3. Boot signature (bytes 510–511) — should be 55 AA

# Write a short report explaining each section
```

### Assignment 4 — Build a File Carver

```python
#!/usr/bin/env python3
"""
Build a simple JPEG file carver
Carves JPEG files from a raw disk image based on headers/footers
"""

import os
import sys

JPEG_HEADER = b'\xFF\xD8\xFF'
JPEG_FOOTER = b'\xFF\xD9'

def carve_jpegs(image_path, output_dir):
    """Carve all JPEG files from a disk image"""

    os.makedirs(output_dir, exist_ok=True)
    found = 0

    with open(image_path, 'rb') as f:
        data = f.read()

    pos = 0
    while pos < len(data):
        # Find next JPEG header
        header_pos = data.find(JPEG_HEADER, pos)
        if header_pos == -1:
            break

        # Find corresponding footer
        footer_pos = data.find(JPEG_FOOTER, header_pos + 3)
        if footer_pos == -1:
            pos = header_pos + 3
            continue

        # Extract JPEG (include 2-byte footer)
        jpeg_data = data[header_pos:footer_pos + 2]

        # Minimum JPEG size check
        if len(jpeg_data) > 1024:  # At least 1KB
            output_file = os.path.join(output_dir, f"carved_{found:04d}.jpg")
            with open(output_file, 'wb') as out:
                out.write(jpeg_data)

            print(f"[+] Carved JPEG at offset {header_pos:,} "
                  f"(size: {len(jpeg_data):,} bytes) → {output_file}")
            found += 1

        pos = footer_pos + 2

    print(f"\nTotal carved: {found} JPEG files")
    return found

# Usage
if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <disk_image> <output_dir>")
        print(f"Example: {sys.argv[0]} disk.img ./carved_files/")
    else:
        carve_jpegs(sys.argv[1], sys.argv[2])
```

---

_Computer Forensics Part 4 — Hard Disk Architecture & Data Storage_
_Cybersecurity Department | Lab Notes Version 1.0_
_Next: Part 5 — File Systems in Depth (NTFS, FAT32, ext4) & Evidence Analysis_
