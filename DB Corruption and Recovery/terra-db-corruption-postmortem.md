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

## Additional Engineering Highlights

### AI-Powered Non-Human Identity (NHI) Threat Detection
* [cite_start]Architected an end-to-end AI-driven behavioral analytics platform for detecting anomalous machine identity activities[cite: 79].
* [cite_start]Designed a distributed event-driven architecture utilizing FastAPI, Kafka, and PostgreSQL for large-scale telemetry ingestion[cite: 80].
* [cite_start]Implemented real-time event streaming using cloud-hosted Kafka infrastructure with secure SSL/TLS authentication[cite: 81].

### Distributed Radiology Triage Platform (AegisRad-AI)
* [cite_start]Built scalable backend services using Python, FastAPI, Redis, Kafka, Docker, AsyncIO, and WebSockets to enable real-time study processing pipelines[cite: 82].
* [cite_start]Engineered a study lifecycle management system featuring state machines, timeline tracking, and priority routing[cite: 83].

### Legacy Modernization & Performance Tuning
* [cite_start]Revitalized a 20-year-old legacy project on Linux by creating debuggable modules, resolving year-old issues within days[cite: 84].
* [cite_start]Optimized product memory management for legacy environments operating under strict 4–6 GB system memory constraints[cite: 85].
* [cite_start]Developed critical server features to capture 20 minutes of event data and managed shared memory across MFC applications for seamless IPC[cite: 86].