# Technical Profile & Engineering Postmortem

**RAHUL JAGDISH POL**
Senior Systems Engineer | [cite_start]Distributed Systems Developer [cite: 51]

## Profile Summary
* [cite_start]Senior engineer with ~15 years of experience architecting and delivering real-time, high-performance enterprise applications[cite: 52].
* [cite_start]Specialized in C++11/14/17, Multithreading, TCP/IP socket programming, and Linux[cite: 53].
* [cite_start]Expertise in modernizing complex legacy systems, transforming long-standing codebases into debuggable architectures, and migrating build frameworks from Make to CMake[cite: 54].
* [cite_start]Proven ability to drop down to the binary level to reverse silent data corruption and prevent catastrophic data loss in critical healthcare systems[cite: 55].
* [cite_start]Designed distributed event-driven architectures utilizing FastAPI, Kafka, PostgreSQL, and Redis for high-throughput telemetry ingestion[cite: 56].

## Core Technical Stack

| Category | Technologies |
| :--- | :--- |
| **Systems Programming** | [cite_start]C++11/14/17, C#.Net, Java, Python, Win32 API, MFC [cite: 58] |
| **Networking & I/O** | [cite_start]TCP/IP Socket Programming, Shared Memory IPC, Multithreading [cite: 59] |
| **Distributed Systems** | [cite_start]Kafka, Redis, FastAPI, PostgreSQL, RabbitMQ [cite: 60] |
| **Tooling & OS** | [cite_start]Linux, WSL 2.0, CMake, GTest, Git, Docker [cite: 61] |

---

## Terra DB Debugging & Recovery Architecture Overview
Refer [Terra DB Debugging & Recovery Architecture Overview.txt](Terra%20DB%20Debugging%20&%20Recovery%20Architecture%20Overview.txt)

## [cite_start]Postmortem: Reversing Silent Binary Corruption and Data Loss in a Legacy Storage Engine [cite: 62]

### The Problem & The Stakes
[cite_start]Within a mission-critical healthcare imaging system (PACS), our proprietary storage engine—Terra DB, which relies on custom binary files for high-speed I/O data retrieval—was experiencing silent data corruption under heavy concurrent usage[cite: 62].

[cite_start]The consequences of these I/O failures were catastrophic in a clinical environment, leading to invisible patient data where the retrieval service would silently skip corrupted records[cite: 63]. [cite_start]This forced radiologists to order rescans, dangerously exposing patients to unnecessary radiation[cite: 64]. [cite_start]Furthermore, silent corruption wiped critical state markers regarding exported diagnostic reports and high-res images, leaving clinicians blind as to what data had reached distributed systems[cite: 65]. 

[cite_start]Previous developers relied purely on guessing, adding random logs near suspected areas without addressing the root cause[cite: 66].

### Debugging by Understanding
[cite_start]Standard tools could not parse the DB once it entered a corrupted state[cite: 67]. [cite_start]To understand exactly how it was failing at the structural level, I engineered a custom offline TerraDB viewer designed to be fault-tolerant[cite: 68]. [cite_start]Instead of crashing on bad bytes, it safely read corrupted binary files to expose the raw state[cite: 69]. [cite_start]This specialized tooling allowed me to map the exact failure points within the proprietary Patient-Study-Series-Image (P-S-S-I) binary tree hierarchy[cite: 70].

### The Architecture of the Fix
[cite_start]I programmed the viewer to cross-reference the corrupted DB against the actual DICOM files on the disk, verifying if a missing record possessed a private DICOM tag explicitly marking it as deleted by a user[cite: 71]. [cite_start]If it was not legitimately deleted, it verified the integrity of the P-S-S-I tree against the physical drive[cite: 72]. [cite_start]I programmatically flipped the corrupted display flags back to an active state, instantly restoring the "lost" patient histories[cite: 73].

[cite_start]For scenarios where binary linkages were completely severed, I wrote automated batch pipelines to forcefully push orphaned images back into TerraDB, rebuilding the DB state without requiring patient rescans[cite: 74]. [cite_start]Finally, I traced the loss of export markers to a critical logic flaw in the legacy DB recovery routine[cite: 75]. [cite_start]I rewrote the transactional logic to strictly ensure that marker backups were kept completely intact irrespective of recovery attempts, isolating them until a successful DB state was verified[cite: 76].

### The Outcome
[cite_start]This solution eradicated the need for patient rescans due to DB failures, directly mitigating unnecessary radiation exposure[cite: 77]. [cite_start]It ensured that critical clinical state markers were permanently protected from destructive legacy recovery routines, achieving zero data loss[cite: 78].

---

## Bibliography

### Architecture & Diagnostic Framework (Citations 24, 32–46)
*References from "Terra DB Debugging & Recovery Architecture Overview"*

[24] Scenario classification: Invisible patient data — corrupted records causing silent retrieval failures.

[32] Diagnostic capability: Bypass standard tooling to parse raw binary state of corrupted databases.

[33] Diagnostic capability: Parse corrupted binary files without standard tool limitations.

[34] Custom fault-tolerant viewer: Engineered offline diagnostic tool designed for corrupted database inspection.

[35] Diagnostic capability: Safe parsing of binary state despite data corruption.

[36] Data recovery methodology: Reconstruct proprietary Patient-Study-Series-Image (P-S-S-I) binary tree hierarchy from corrupted state.

[38] Recovery mechanism: Verify private DICOM tags to determine explicit user-initiated deletion vs. corruption.

[39] Integrity verification: Cross-validate P-S-S-I tree structure against physical drive data.

[40] Recovery action: Programmatically flip corrupted display flags to restore active record visibility.

[41] Scenario classification: Severed binary linkages — orphaned image files with broken database references.

[42] Recovery mechanism: Automated batch pipeline for re-ingesting orphaned images into TerraDB.

[43] Scenario classification: Fatal recovery logic flaw — destructive loss of critical state markers during recovery operations.

[45] Root cause identification: Legacy recovery routine destructively deletes backup markers on recovery failure.

[46] Transactional redesign: Rewritten recovery logic ensuring marker backups remain intact until successful DB state verification.

### Professional Profile & Expertise (Citations 51–61)

[51] Professional profile and distributed systems development expertise.

[52] Career experience in real-time, high-performance enterprise application architecture.

[53] Technical specialization in C++11/14/17, multithreading, TCP/IP socket programming, and Linux systems.

[54] Expertise in legacy system modernization and build framework migration (Make to CMake).

[55] Binary-level debugging and data corruption investigation in critical healthcare systems.

[56] Event-driven architecture design with FastAPI, Kafka, PostgreSQL, and Redis.

[58] Systems programming languages and APIs: C++11/14/17, C#.Net, Java, Python, Win32 API, MFC.

[59] Network and I/O technologies: TCP/IP socket programming, shared memory IPC, multithreading.

[60] Distributed systems technologies: Kafka, Redis, FastAPI, PostgreSQL, RabbitMQ.

[61] Development tooling and OS platforms: Linux, WSL 2.0, CMake, GTest, Git, Docker.

### Case Study: Terra DB Corruption & Recovery (Citations 62–78)

[62] Postmortem case study: Silent binary corruption and data loss in legacy Terra DB storage engine.

[63] Impact analysis: Silent data corruption in PACS system causing invisible patient data loss.

[64] Clinical consequences: Unnecessary patient radiation exposure due to required diagnostic rescans.

[65] Systemic failure: Loss of critical state markers in export tracking for distributed systems.

[66] Root cause assessment: Previous debugging approach relied on guesswork without addressing underlying issues.

[67] Technical challenge: Standard diagnostic tools unable to parse corrupted database state.

[68] Solution architecture: Custom fault-tolerant TerraDB offline viewer development.

[69] Implementation detail: Safe binary file reading without crashes on corrupted data.

[70] Analysis methodology: Mapping failure points in Patient-Study-Series-Image (P-S-S-I) binary tree hierarchy.

[71] Verification approach: Cross-referencing corrupted DB against DICOM files using private deletion tags.

[72] Integrity validation: P-S-S-I tree verification against physical disk data.

[73] Recovery technique: Programmatic correction of corrupted display flags to restore patient histories.

[74] Batch processing: Automated pipelines for recovering orphaned images without patient rescans.

[75] Root cause analysis: Critical logic flaw identified in legacy DB recovery routine affecting export markers.

[76] Transactional safeguard: Rewritten recovery logic ensuring marker backup integrity and isolation.

[77] Outcome validation: Elimination of patient rescans due to DB failures, reducing unnecessary radiation exposure.

[78] Final achievement: Zero data loss through protection of critical clinical state markers from legacy recovery routines.
