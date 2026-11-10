---
revealjs:
  mode: diapo
  highlight:
    light: vs
    dark:  vs2015
  autoAnimate:  true
  autoAnimateDuration: 1.0
  transition: slide
  transitionSpeed: default
  js: assets/lecture-07.js

---

# ECEG-4191

## Chapter 7 — Disk Storage, File Structures, and Hashing

---

## Learning outcomes

<div class="fragment">

1. Describe the **physical structure** of magnetic and solid-state storage.

</div>

<div class="fragment">

2. Compute **disk access time** (seek + rotation + transfer) for a given workload.

</div>

<div class="fragment">

3. Design **file organizations**: heap, sorted, hashed.

</div>

<div class="fragment">

4. Explain **static, dynamic, and extendible hashing** and their trade-offs.

</div>

<div class="fragment">

5. Choose an appropriate **RAID level** for a given workload.

</div>

---

## 7.1 — Disk Storage: Physical Structure

???+ note "Magnetic hard disk (HDD)"
    ```
    Platter (rotates at 5400–15000 RPM)
      │
      ├── Tracks (concentric circles)
      │     │
      │     └── Sectors (fixed-size blocks, ~512 B–4 KB)
      │
      └── Read/write head per surface
    ```

<div class="fragment">

???+ note "Access time components"
    | Component | Typical value | Cause |
    |---|---|---|
    | **Seek** | 3–10 ms | Move the head to the right track |
    | **Rotation** | 2–6 ms (half a revolution) | Wait for the sector to rotate under |
    | **Transfer** | 0.03–0.2 ms (per block) | Read the data |

</div>

<div class="fragment">

!!! warning "Seek dominates"
    For random access, seek + rotation is 100–500× the transfer time. This is why **disk scheduling** (Chapter 8 of OS) and **B-tree indexing** exist — both reduce the number of seeks.

</div>

<div class="fragment">

???+ note "Solid-state drive (SSD)"
    - NAND flash cells, no moving parts.
    - **No seek, no rotation.**
    - Random access ≈ 50–100 µs (vs. ~10 ms for HDD).
    - Finite write cycles (~10⁴–10⁵ per cell); wear-leveling firmware.

</div>

---

## 7.2 — File Organization

???+ note "What is a file?"
    A **file** is a sequence of **records** stored on disk. Each record consists of **fields** (attributes).

<div class="fragment">

???+ note "Three fundamental organizations"
    | Organization | Insert | Search by key | Range query |
    |---|---|---|---|
    | **Heap** | $O(1)$ | $O(b)$ full scan | $O(b)$ |
    | **Sorted** | $O(b)$ | $O(\log_2 b)$ binary search | $O(\log b + \text{matches})$ |
    | **Hashed** | $O(1)$ | $O(1)$ average | $O(b)$ (destroys order) |

</div>

<div class="fragment">

!!! tip "The choice depends on the workload"
    - **Heap** — if most queries scan the whole file.
    - **Sorted** — if range queries and ordered scans dominate.
    - **Hashed** — if point lookups dominate and range queries are rare.

</div>

---

## Worked example 7.1 — Disk access time

!!! tip "Problem:"
    HDD with 9 ms seek, 7200 RPM, 150 MB/s transfer. Compute the time to read a 4 KB block.

<div class="fragment">

???+ note "**Step 1 — rotational latency.**"
    One revolution: $60 / 7200 = 8.333$ ms.
    Average latency: half a revolution = **4.17 ms**.

</div>

<div class="fragment">

??? info "**Step 2 — transfer time.**"
    $$
    T_{\text{transfer}} = \frac{4096}{150 \times 10^6} \approx 0.027\ \text{ms}.
    $$

</div>

<div class="fragment">

??? success "**Step 3 — total.**"
    $$
    T = 9 + 4.17 + 0.027 \approx \mathbf{13.2\ \text{ms per block}}.
    $$
    
    **IOPS** $\approx 1 / 0.0132 \approx \mathbf{76}$.

</div>

<div class="fragment">

??? tip "**Comparison with SSD.**"
    SSD random read: ~50–100 µs = **~0.075 ms**. That's **175× faster** than the HDD above. Random-access workloads (databases, VM) hit this difference hardest.

</div>

---

## 7.3 — Fixed vs Variable-Length Records

???+ note "Fixed-length records"
    - Every record has the **same size**.
    - Direct access: `address = file_start + record_size × record_number`.
    - Simple but wastes space (padding).

<div class="fragment">

???+ note "Variable-length records"
    - Records differ in length (e.g., a `VARCHAR` field).
    - **Slotted page** structure: a header, a slot directory, and variable-length record data.

</div>

<div class="fragment">

???+ info "Slotted page layout"
    ```
    ┌─────────────────────────────────┐
    │  Header (record count, free ptr) │
    ├─────────────────────────────────┤
    │  Slot 1 → offset                 │
    │  Slot 2 → offset                 │
    │  ...                             │
    ├─────────────────────────────────┤
    │  Free space                      │
    ├─────────────────────────────────┤
    │  Record N                        │
    │  Record ...                      │
    │  Record 1                        │
    └─────────────────────────────────┘
    ```
    
    The slot directory points to record offsets. Records can move within the page without affecting external pointers.

</div>

---

## 7.4 — Static Hashing

???+ note "Definition"
    A **hash file** uses a hash function $h(k)$ to map a key to a **bucket** (typically one disk block). All records with the same hash go to the same bucket.

<div class="fragment">

???+ note "Static hashing"
    - The number of buckets $M$ is fixed at creation.
    - $h(k) = k \bmod M$.
    - **Problem:** if $M$ is too small, overflow chains grow.
    - If $M$ is too large, most buckets are empty.

</div>

<div class="fragment">

!!! warning "The overflow problem"
    When a bucket is full, new records go to an **overflow block** chained from the primary. As the file grows, chains lengthen, and search degrades from $O(1)$ to $O(n)$.

</div>

<div class="fragment">

???+ info "Example"
    $M = 5$, keys $\{10, 15, 20, 25, 30, 35\}$, $h(k) = k \bmod 5$.
    
    | Bucket | Keys |
    |---|---|
    | 0 | 10, 20, 30 |
    | 1 | — |
    | 2 | — |
    | 3 | 15, 25, 35 |
    | 4 | — |
    
    Buckets 0 and 3 hold multiple records — overflow if capacity is less than 3.

</div>

---

## 7.5 — Dynamic Hashing

???+ note "Why dynamic hashing?"
    Static hashing cannot grow or shrink gracefully. **Dynamic hashing** adjusts the number of buckets as the file grows.

<div class="fragment">

???+ note "Two main variants"
    | Variant | Structure | Growth |
    |---|---|---|
    | **Extendible hashing** | Directory + buckets | Doubles directory when needed |
    | **Linear hashing** | No directory | Splits one bucket at a time |

</div>

<div class="fragment">

!!! tip "The trade-off"
    - Extendible hashing has **$O(1)$ search** (via directory) but pays for directory maintenance.
    - Linear hashing has **no directory** but may probe more than one bucket during splits.

</div>

---

## 7.6 — Extendible Hashing

???+ note "The structure"
    - **Directory** — an array of $2^d$ pointers, where $d$ is the **global depth**.
    - **Buckets** — hold the actual records; each has a **local depth** $d_i$.
    - To find a record: use the first $d$ bits of $h(k)$ to index the directory.

<div class="fragment">

???+ note "Splitting a bucket"
    When a bucket overflows:
    1. If local depth $d_i < d$: increment $d_i$, split the bucket's records into two by the next bit, update directory pointers.
    2. If local depth $d_i = d$: double the directory (increment $d$), then split.

</div>

<div class="fragment">

???+ info "Search cost"
    - **Directory lookup:** $O(1)$.
    - **Bucket access:** $O(1)$.
    - Total: **$O(1)$** — no overflow chains.

</div>

---

## Worked example 7.2 — Extendible hashing insert

!!! tip "Problem:"
    Start with $d = 2$ (4 directory entries). Buckets contain keys $\{0, 4\}$, $\{1, 5\}$, $\{2, 6\}$, $\{3, 7\}$. Bucket capacity = 2. Insert key 12.

<div class="fragment">

???+ note "**Step 1 — hash 12.**"
    Assume $h(12) = 12$ (mod some power of 2). Binary: `1100`. First 2 bits: `11` → directory index 3.

</div>

<div class="fragment">

??? info "**Step 2 — bucket 3 is full.**"
    Bucket 3 currently holds $\{3, 7\}$ (local depth 2 = global depth).

</div>

<div class="fragment">

??? info "**Step 3 — split requires doubling the directory.**"
    Global depth becomes 3. Directory grows to 8 entries. Bucket 3's local depth becomes 3. Split $\{3, 7, 12\}$ by the third bit:
    - `011` → bucket for 3
    - `111` → bucket for 7, 12 (if capacity > 2, they stay; else split again).

</div>

<div class="fragment">

??? success "**Result.**"
    The directory doubled; buckets redistributed by one extra bit. **No overflow chains.** Search remains $O(1)$.

</div>

---

## 7.7 — Linear Hashing

???+ note "The idea"
    No directory. A **split pointer** moves through the buckets, splitting one bucket at a time when the load factor exceeds a threshold.

<div class="fragment">

???+ note "The algorithm"
    - Buckets numbered $0, \ldots, M-1$, with a **level** $L$.
    - Insert uses $h_L(k) = k \bmod (M \cdot 2^L)$.
    - When load factor > threshold, split bucket at the split pointer (using $h_{L+1}$), then advance the pointer.
    - When the pointer reaches the end, increment $L$ and reset the pointer.

</div>

<div class="fragment">

???+ info "Trade-offs"
    | Aspect | Extendible | Linear |
    |---|---|---|
    | Directory | Yes | No |
    | Search | Always $O(1)$ | Usually $O(1)$, may need 2 lookups during split |
    | Space overhead | Directory | Minimal |
    | Simplicity | Moderate | Complex logic |

</div>

<div class="fragment">

!!! tip "Which is used in practice?"
    Most production systems use **B-trees** rather than hashing for their primary index — because B-trees support range queries. Hashing wins for **point lookups only** (e.g., in-memory caches, key-value stores like Redis).

</div>

---

## 7.8 — RAID: The Idea

???+ note "Definition"
    **RAID** (Redundant Array of Independent Disks) combines multiple physical disks into a single logical unit for **performance**, **reliability**, or both.

<div class="fragment">

???+ note "Two fundamental techniques"
    - **Striping** — split data across disks (performance).
    - **Mirroring / parity** — duplicate data (reliability).

</div>

<div class="fragment">

!!! warning "RAID is not backup"
    RAID protects against **disk failure**, not against accidental deletion, corruption, ransomware, or site loss. Backup is a separate concern.

</div>

---

## 7.9 — RAID Levels 0–6

???+ note "The main levels"
    | Level | Technique | Redundancy | Performance | Storage efficiency |
    |---|---|---|---|---|
    | **RAID 0** | Striping | None | Fastest | 100% |
    | **RAID 1** | Mirroring | Full | Fast read, slow write | 50% |
    | **RAID 2** | Bit striping + Hamming ECC | Yes | Medium | Low |
    | **RAID 3** | Byte striping + dedicated parity | 1 disk | Medium | $\frac{n-1}{n}$ |
    | **RAID 4** | Block striping + dedicated parity | 1 disk | Medium | $\frac{n-1}{n}$ |
    | **RAID 5** | Block striping + distributed parity | 1 disk | Good read, slower write | $\frac{n-1}{n}$ |
    | **RAID 6** | Block striping + double parity | 2 disks | Good read, slower write | $\frac{n-2}{n}$ |

<div class="fragment">

!!! tip "The practical winners"
    - **RAID 0** — scratch space, no durability needed.
    - **RAID 1** — small servers, two disks.
    - **RAID 5** — general-purpose storage, read-heavy.
    - **RAID 6** — large arrays, write-tolerant.
    - **RAID 10** (not in the standard list) — mirror + stripe, best performance + reliability, 50% storage.

</div>

<div class="fragment">

!!! warning "RAID 5 write penalty"
    Every write requires **read-modify-write** of the parity block — the bottleneck for write-heavy workloads. RAID 10 avoids this; that's why databases often prefer it.

</div>

---

## Worked example 7.3 — Choose a RAID level

!!! tip "Problem:"
    Match each scenario to a RAID level.

<div class="fragment">

???+ note "**Scenario A — 10 TB backup server, sequential writes, cost matters.**"
    **RAID 6.** Two-disk redundancy, ~67% storage efficiency, tolerates large-disk rebuilds.

</div>

<div class="fragment">

??? info "**Scenario B — OLTP database, read-heavy, frequent small writes.**"
    **RAID 10.** Fast reads (mirror), avoids RAID 5's parity write penalty, 50% storage.

</div>

<div class="fragment">

??? info "**Scenario C — video editing scratch space, no persistence needed.**"
    **RAID 0.** Maximum throughput, no durability required (data regenerated).

</div>

<div class="fragment">

??? info "**Scenario D — home NAS, four 4 TB disks, mixed read/write.**"
    **RAID 5.** 75% storage (12 TB usable), tolerates one disk failure, good read performance.

</div>

<div class="fragment">

??? success "**The rule of thumb.**"
    - Performance → RAID 0 or 10.
    - Capacity → RAID 5.
    - Safety at scale → RAID 6.
    - Small/cheap → RAID 1.

</div>

---

## 7.10 — Buffering and Caching

???+ note "The disk is slow"
    Disk latency is ~100,000× CPU speed. The DBMS hides this with **memory**:

<div class="fragment">

???+ note "Three layers"
    | Layer | Where | What it caches |
    |---|---|---|
    | **Buffer pool** | DBMS memory | Recently used disk pages |
    | **OS page cache** | OS kernel | File-system pages |
    | **On-disk cache** | Disk controller | Pre-fetched sectors |

</div>

<div class="fragment">

???+ info "Replacement policies"
    - **LRU** — evict least-recently used page.
    - **Clock** — approximate LRU using a reference bit.
    - **LRU-K** — evict page with oldest $K$-th most recent access.
    - **ARC** — adaptive; balances recency and frequency.

</div>

<div class="fragment">

!!! tip "The buffer pool is the DBMS's most valuable resource"
    All query performance depends on it. Tuning its size and replacement policy is one of the DBA's most important jobs.

</div>

---

## Chapter 7 — key takeaways

<div class="fragment">

**1.** Disk access time = **seek + rotation + transfer**. Seek dominates — this is why disk scheduling and indexing exist.

</div>

<div class="fragment">

**2.** **File organizations**: heap (fast insert, slow search), sorted (fast search, slow insert), hashed (fast point lookup, no order).

</div>

<div class="fragment">

**3.** **Static hashing** has overflow chains that grow. **Dynamic hashing** (extendible, linear) grows gracefully.

</div>

<div class="fragment">

**4.** **Extendible hashing** uses a directory; **linear hashing** uses a split pointer and no directory.

</div>

<div class="fragment">

**5.** **RAID 0** = performance only, **RAID 1** = mirror, **RAID 5** = distributed parity (1-failure), **RAID 6** = double parity (2-failures).

</div>

<div class="fragment">

**6.** **Buffer pools** and **caches** are the DBMS's defence against the 100,000× speed gap between CPU and disk.

</div>

---

## Course wrap-up

<div class="fragment">

**The arc of ECEG-4191:**

1. **Chapter 1** — what a DBMS is, and why file-based systems fail.
2. **Chapter 2** — the relational model: relations, keys, constraints.
3. **Chapter 3** — conceptual design: ER/EER modelling.
4. **Chapter 4** — logical design: ER-to-relational mapping and normalization.
5. **Chapter 5** — relational algebra and calculus: the theoretical query language.
6. **Chapter 6** — SQL: the practical query language.
7. **Chapter 7** — storage and file structures: where the data lives.

</div>

<div class="fragment">

!!! tip "The unifying question"
    Every chapter asked: **how do we represent, design, query, and store data so that it is consistent, efficient, and evolvable?**
    
    - **Represent** — the relational model.
    - **Design** — ER modelling and normalization.
    - **Query** — relational algebra and SQL.
    - **Store** — file organizations, hashing, RAID.

</div>

<div class="fragment">

**Good luck with the exam.**

</div>