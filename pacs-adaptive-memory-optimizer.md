# Postmortem: Architecting an Adaptive Memory Optimizer for Resource-Constrained PACS Viewers

**Role:** Senior Systems Engineer

**Technologies:** C++, VC++, Win32, JPEG Lossless, DCMTK C++ Library, log4cxx

### 1. The Problem & Clinical Impact
---
In healthcare systems, Patient Archiving and Communication Systems (PACS) deal with massive, memory-hungry DICOM datasets. Many of our legacy hospital clients were operating on severely resource-constrained hardware, typically limited to 4–8 GB of system RAM.

The existing imaging viewer utilized a greedy memory strategy: it loaded all transmitted compressed images into system memory, uncompressed the entire dataset simultaneously, and held it in RAM for rendering. On 8 GB machines, this quickly exhausted available memory, causing the application to hang, forcefully close, or crash entirely. This resulted in interrupted workflows, radiologist frustration, and potentially delayed diagnoses.

### 2. System Profiling & Root Cause
---
I profiled the application's memory consumption and identified two major flaws:

1. **Inefficient Memory Layouts:** The core C++ structs and classes handling image metadata and pixel data were poorly aligned, resulting in massive memory bloat due to compiler padding.
2. **Unbounded RAM Allocation:** The application assumed infinite memory and made no distinction between actively viewed images and background data. Since a radiologist cannot view 1,000 images simultaneously, holding all uncompressed pixel data in RAM was an architectural oversight.

### 3. The Architecture of the Fix
---
To solve this, I engineered an adaptive, multi-tiered Memory Optimizer that operates by default, intelligently paging data between RAM and disk based on dynamic memory pressure.

**Phase 1: Memory Layout Optimization**
I refactored the C++ memory layout across the application. By strictly packing and aligning struct members and classes, I eliminated unnecessary padding bytes. This foundational C++ optimization instantly lightened the application's baseline memory footprint by 20%–30% before any caching logic was even applied.

**Phase 2: Tiered Cache Eviction Strategy**
I implemented a dual-threshold caching system with configurable limits (defaulting to 85% and 95% of available RAM).

* **Primary Cache Hit (85% Threshold):** When RAM usage hits 85%, the system aggressively cleans memory by releasing the *uncompressed* pixel data of non-visible images. To maintain a seamless user experience, the engine keeps the currently visible images, plus a pre-fetch window of 10 adjacent images (forward and backward). If a user jumps to a released image, the engine seamlessly uncompresses the requested target and its adjacent 10 images, while simultaneously releasing the previously viewed batch.
* **Secondary Cache Hit (95% Threshold - Critical Pressure):** If memory pressure hits 95%, the system drops the pre-fetch window from ±10 down to ±2 adjacent images. It then begins evicting even the *compressed* DICOM data from RAM for non-visible images to prevent an OS-level Out-of-Memory (OOM) termination.

**Phase 3: Disk-Backed Paging & Observability**
Evicting compressed data at the 95% threshold risked leaving the viewer with a blank screen if the user scrolled rapidly. To mitigate this, I designed a disk-backed paging mechanism.

* Evicted compressed data is sequentially cached to a secondary physical drive.
* If the user navigates to an image no longer in RAM, the engine performs a fast read from the secondary drive, loads the compressed payload into memory, uncompresses it for display, and pages out the older images.
* I integrated **log4cxx** to provide deep, troubleshooting-level telemetry on cache hits, eviction cycles, and disk-paging latency.

```
+-------------------------------------------------------------------------+
|                LEGACY PACS CLIENT ENVIRONMENT (4-8 GB RAM)              |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
|               PHASE 1: MEMORY LAYOUT OPTIMIZATION (C++)                 |
|  - Strict C++ Struct Alignment & Padding Elimination                    |
|  - Result: 20-30% Baseline Memory Bloat Reduction                       |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
|               PHASE 2: ADAPTIVE TIERED CACHING ENGINE                   |
|                                                                         |
|  +-----------------------+       +-----------------------------------+  |
|  | Normal State (< 85%)  |       | Primary Cache Hit (85% RAM)       |  |
|  | Standard operations.  | ----> | - Evict uncompressed hidden data  |  |
|  | Full allocation.      |       | - Keep Visible + Pre-fetch (±10)  |  |
|  +-----------------------+       +-----------------------------------+  |
|                                                    |                    |
|                                                    v                    |
|                                  +-----------------------------------+  |
|                                  | Secondary Cache Hit (95% RAM)     |  |
|                                  | - CRITICAL PRESSURE (OOM Risk)    |  |
|                                  | - Shrink Pre-fetch to ±2 images   |  |
|                                  | - Evict compressed data to DISK   |  |
|                                  +-----------------------------------+  |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
|                  PHASE 3: DISK-BACKED PAGING (I/O)                      |
|                                                                         |
|  +-----------------------+             +-----------------------------+  |
|  |    System RAM         | <========== | Secondary Physical Drive    |  |
|  |  (Active Rendering)   |  Fast Read  | (Evicted Compressed DICOM)  |  |
|  +-----------------------+             +-----------------------------+  |
+-------------------------------------------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
|                  OBSERVABILITY & TELEMETRY (log4cxx)                    |
|  - Tracks Cache Hits/Misses | Eviction Cycles | Disk-Paging Latency     |
+-------------------------------------------------------------------------+

```
### 4. The Outcome
---
* **Eliminated OOM Crashes:** The adaptive memory manager completely stabilized the viewer on legacy 4–8 GB hardware, eliminating memory exhaustion crashes.
* **Optimized Performance:** The combination of strict C++ struct alignment and the ±10 sliding window pre-fetch logic ensured that radiologists experienced zero perceived latency or rendering delays, despite the aggressive memory cleanup happening in the background.