# Technical Profile & Engineering Postmortem

**RAHUL JAGDISH POL**
Senior Systems Engineer | Distributed Systems Developer

## Profile Summary
* Senior engineer with ~15 years of experience architecting and delivering real-time, high-performance enterprise applications.
* Specialized in C++11/14/17, Multithreading, TCP/IP socket programming, and Linux.
* Expertise in modernizing complex legacy systems, transforming long-standing codebases into debuggable architectures, and migrating build frameworks from Make to CMake.
* Proven ability to drop down to the binary level to reverse silent data corruption and prevent catastrophic data loss in critical healthcare systems.
* Designed distributed event-driven architectures utilizing FastAPI, Kafka, PostgreSQL, and Redis for high-throughput telemetry ingestion.

## Core Technical Stack

| Category | Technologies |
| :--- | :--- |
| **Systems Programming** | C++11/14/17, C#.Net, Java, Python, Win32 API, MFC |
| **Networking & I/O** | TCP/IP Socket Programming, Shared Memory IPC, Multithreading |
| **Distributed Systems** | Kafka, Redis, FastAPI, PostgreSQL, RabbitMQ |
| **Tooling & OS** | Linux, WSL 2.0, CMake, GTest, Git, Docker |

---

## Terra DB Debugging & Recovery Architecture Overview
Refer [Terra DB Debugging & Recovery Architecture Overview.txt](Terra%20DB%20Debugging%20&%20Recovery%20Architecture%20Overview.txt)

## Postmortem: Reversing Silent Binary Corruption and Data Loss in a Legacy Storage Engine

### The Problem & The Stakes
Within a mission-critical healthcare imaging system (PACS), our proprietary storage engine—Terra DB, which relies on custom binary files for high-speed I/O data retrieval—was experiencing silent data corruption under heavy concurrent usage.

The consequences of these I/O failures were catastrophic in a clinical environment, leading to invisible patient data where the retrieval service would silently skip corrupted records. This forced radiologists to order rescans, dangerously exposing patients to unnecessary radiation. Furthermore, silent corruption wiped critical state markers regarding exported diagnostic reports and high-res images, leaving clinicians blind as to what data had reached distributed systems.

Previous developers relied purely on guessing, adding random logs near suspected areas without addressing the root cause.

### Debugging by Understanding
Standard tools could not parse the DB once it entered a corrupted state. To understand exactly how it was failing at the structural level, I engineered a custom offline TerraDB viewer designed to be fault-tolerant. Instead of crashing on bad bytes, it safely read corrupted binary files to expose the raw state. This specialized tooling allowed me to map the exact failure points within the proprietary Patient-Study-Series-Image (P-S-S-I) binary tree hierarchy.

### The Architecture of the Fix
I programmed the viewer to cross-reference the corrupted DB against the actual DICOM files on the disk, verifying if a missing record possessed a private DICOM tag explicitly marking it as deleted by a user. If it was not legitimately deleted, it verified the integrity of the P-S-S-I tree against the physical drive. I programmatically flipped the corrupted display flags back to an active state, instantly restoring the "lost" patient histories.

For scenarios where binary linkages were completely severed, I wrote automated batch pipelines to forcefully push orphaned images back into TerraDB, rebuilding the DB state without requiring patient rescans. Finally, I traced the loss of export markers to a critical logic flaw in the legacy DB recovery routine. I rewrote the transactional logic to strictly ensure that marker backups were kept completely intact irrespective of recovery attempts, isolating them until a successful DB state was verified.

### The Outcome
This solution eradicated the need for patient rescans due to DB failures, directly mitigating unnecessary radiation exposure. It ensured that critical clinical state markers were permanently protected from destructive legacy recovery routines, achieving zero data loss.


