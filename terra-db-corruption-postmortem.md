
# Title: Postmortem: Reversing Silent Binary Corruption and Data Loss in a Legacy Healthcare Storage Engine
---
**Author:** Rahul Jagdish Pol  
**Role:** Senior Systems Engineer  
**Technologies:** C++, Custom Binary Storage Engine (Terra DB), DICOM/PACS, Batch Scripting  

---
### 1. The Problem & The Stakes
---
Within a mission-critical healthcare imaging system (PACS), our proprietary storage engine—Terra DB, which relies on custom binary files for high-speed I/O data retrieval—was experiencing silent data corruption under heavy concurrent usage.

The consequences of these I/O failures were catastrophic in a clinical environment:

* **Invisible Patient Data:** The imaging viewer's retrieval service would silently skip corrupted records. Radiologists could not see patient data or histories in their study lists, despite the underlying data consuming massive disk space.
* **Radiation Hazards:** If a radiologist took a batch of X-rays and the DB silently corrupted the ingestion, the radiologist was forced to order a rescan. This wasn't just an economic issue; it dangerously exposed patients to unnecessary radiation.
* **Lost Diagnostic State:** The DB maintained critical state markers regarding whether diagnostic reports, CD films, or high-res images had been successfully exported or sent to distributed systems. Silent corruption wiped these markers, leaving radiologists completely blind as to which clinical data had actually reached its destination.

Previous attempts to solve this issue over a year had failed. Because standard tooling couldn't reproduce the concurrent I/O corruption, previous developers relied purely on guessing—adding random logs near the suspected areas and closing the tickets without addressing the root cause.

### 2. Debugging by Understanding (Bypassing Standard Tooling)
---
To fix a proprietary binary engine, I had to understand exactly how it was failing at the structural level. Standard tools could not parse the DB once it entered a corrupted state.

I engineered a custom offline TerraDB viewer designed to be fault-tolerant. Instead of crashing on bad bytes, it safely read corrupted binary files to expose the raw state. This allowed me to map the exact failure points within the proprietary Patient-Study-Series-Image (P-S-S-I) binary tree hierarchy.

### 3. The Architecture of the Fix
---
Using the custom viewer, I isolated the core issues and implemented a multi-stage recovery and root-cause resolution strategy:

* **Restoring the P-S-S-I Binary Tree:** I programmed the viewer to cross-reference the corrupted DB against the actual DICOM files on the disk. It first verified if a missing record possessed a private DICOM tag explicitly marking it as "deleted" by a user. If it was not legitimately deleted, it verified the integrity of the P-S-S-I tree against the physical drive. Finally, I programmatically flipped the corrupted display flags back to an active state, instantly restoring the "lost" patient histories to the radiologists' study lists.
* **Orchestrating Re-ingestion:** For scenarios where the binary linkages were completely severed but the raw image files survived on disk, I wrote automated batch pipelines. These scripts forcefully pushed the orphaned images back into TerraDB one by one, effectively re-orchestrating the ingestion process and rebuilding the DB state without requiring patient rescans.
* **Fixing the Fatal Recovery Logic Flaw:** I traced the loss of the export markers (print/send status) to a critical logic flaw in the legacy DB recovery routine. During a recovery attempt, the system backed up the marker files. However, if the DB recovery process *failed*, the system destructively deleted the backups anyway. I rewrote the transactional logic to strictly ensure that marker backups were kept completely intact irrespective of recovery attempts, isolating them until a successful DB state was verified.

```
+-----------------------------------------------------------------------------------+
|                           LEGACY PACS ENVIRONMENT                                 |
|                                                                                   |
|  +--------------------+       +------------------------------------------------+  |
|  | Corrupted Terra DB |       |         Physical Drive (Raw DICOM Files)       |  |
|  | (Binary Files)     |       |                                                |  |
|  +---------+----------+       +-----------------------+------------------------+  |
|            |                                          |                           |
+------------|------------------------------------------|---------------------------+
             |                                          |
             v                                          v
+-----------------------------------------------------------------------------------+
|                     CUSTOM FAULT-TOLERANT VIEWER                                  |
|                                                                                   |
|  1. Bypass standard tooling & parse raw binary state.                             |
|  2. Reconstruct proprietary P-S-S-I (Patient-Study-Series-Image) Tree.            |
+----------------------------+------------------------------------------------------+
                             |
                             v
+-----------------------------------------------------------------------------------+
|                        DIAGNOSTIC & RECOVERY ENGINE                               |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  | SCENARIO A: Invisible Patient Data                                          |  |
|  | -> Check private DICOM tag for explicit user "deletion".                    |  |
|  | -> If NOT deleted: Verify P-S-S-I tree integrity against drive.             |  |
|  | -> Action: Programmatically flip corrupted display flags to active.         |  |
|  +-----------------------------------------------------------------------------+  |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  | SCENARIO B: Severed Binary Linkages                                         |  |
|  | -> Raw image files survive on disk, but DB links are broken.                |  |
|  | -> Action: Trigger Automated Batch Pipelines.                               |  |
|  | -> Result: Forcefully push/re-ingest orphaned images into TerraDB.          |  |
|  +-----------------------------------------------------------------------------+  |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  | SCENARIO C: Fatal Recovery Logic Flaw (Lost State Markers)                  |  |
|  | -> Legacy system destructively deleted backups on recovery failure.         |  |
|  | -> Action: Rewrote transactional logic.                                     |  |
|  | -> Result: Marker backups kept strictly intact until DB state is verified   |  |
|  |    successful.                                                              |  |
|  +-----------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```
### 4. The Outcome
---
* **Patient Safety:** Eradicated the need for patient rescans due to DB failures, directly mitigating unnecessary radiation exposure.
* **Zero Data Loss:** Ensured that critical clinical state markers (print/send/export) were permanently protected from destructive legacy recovery routines.
* **Operational Integrity:** Transformed an opaque, failing binary storage engine into a debuggable, recoverable system, restoring absolute confidence in the clinical diagnostic pipeline.
---