# Chapter 7 — Disk Storage, File Structures, and Hashing

!!! abstract "Chapter at a glance"
    **Coverage:** disk storage devices · files of records · unordered, sorted, and hashed files · dynamic and extendible hashing · RAID.

---

## Learning outcomes

1. Describe the physical structure of a magnetic disk and the cost of block access.
2. Compare file organisations: heap, sorted, hashed.
3. Explain static, dynamic, and extendible hashing.
4. Describe RAID levels 0–6 and their trade-offs.

---

## 7.1 Disk storage devices

A magnetic **hard disk** consists of **platters**, each divided into **tracks**, each track into **sectors**. A **block** is the unit of disk I/O (typically 4 KB).

**Access cost:**

$$
T_{\text{access}} = T_{\text{seek}} + T_{\text{rotational}} + T_{\text{transfer}}
$$

| Component | Typical value |
|---|---|
| Seek time | 3–10 ms |
| Rotational latency | ~4 ms (7200 RPM) |
| Transfer time | ~0.1 ms per 4 KB |

**Seek time dominates** — disk scheduling algorithms (FCFS, SSTF, SCAN) are designed to reduce total head movement.

**SSDs** have no moving parts: constant-time random access, but finite write cycles and asymmetric read/write performance.

---

## 7.2 Files of records

A **file** is a sequence of **records** stored on disk. Each record consists of **fields** (attributes).

**Fixed-length records** — all records the same size; direct access by `(record_number × record_size)`.

**Variable-length records** — records differ in size; use a **slotted-page** layout with a slot directory and record offsets.

---

## 7.3 File organisations

### 7.3.1 Unordered (heap) file

Records inserted at the end. New records go wherever there is free space.

- **Insert:** $O(1)$ (if space is available).
- **Search by key:** $O(b)$ where $b$ is the number of blocks — full scan.

### 7.3.2 Sorted (sequential) file

Records kept in key order.

- **Search by key:** $O(\log_2 b)$ — binary search over blocks.
- **Insert:** $O(b)$ — shifting.

### 7.3.3 Hashed file

A **hash function** $h(k)$ maps the key to a **bucket** (block or set of blocks).

- **Search by key:** $O(1)$ on average.
- **Range queries:** $O(b)$ — hashing destroys order.

---

## 7.4 Static hashing

A fixed number of buckets $M$ is allocated. Each key $k$ hashes to bucket $h(k) \bmod M$.

**Overflow problem.** If a bucket fills, overflow blocks are chained. As the file grows, overflow chains lengthen, and search degrades from $O(1)$ to $O(n)$.

**The fix: dynamic hashing.**

---

## 7.5 Dynamic hashing

Two main variants:

### 7.5.1 Extendible hashing

Uses a **directory** (an array of pointers to buckets) indexed by the first $d$ bits of the hash value.

- **Global depth** $d$ — number of bits used by the directory.
- **Local depth** $d_i$ — number of bits used by bucket $i$.
- When a bucket overflows, split it and increment its local depth; double the directory if needed.

**Properties.**

- $O(1)$ search in the **directory**, plus one block access.
- Directory doubles at most $\log_2 n$ times for $n$ records.
- No overflow chains (except transiently during splits).

### 7.5.2 Linear hashing

Grows the file **one bucket at a time** — no directory. Uses a **split pointer** and a **level** counter. Splits happen when the load factor exceeds a threshold.

**Properties.**

- Simpler than extendible hashing (no directory).
- Splits are incremental, so the file grows smoothly.
- Slightly slower search (need to check the split state).

---

## 7.6 RAID

**RAID** (Redundant Array of Independent Disks) combines multiple disks for performance, redundancy, or both.

| Level | Technique | Redundancy | Performance | Cost |
|---|---|---|---|---|
| **RAID 0** | Striping (no redundancy) | None | Fastest | Low |
| **RAID 1** | Mirroring | Full | Fast read, slower write | 2× storage |
| **RAID 5** | Block-level striping with **distributed parity** | 1-disk failure | Good read, slower write | 1 disk overhead |
| **RAID 6** | Striping with **double parity** | 2-disk failure | Good read, slower write | 2 disks overhead |

**RAID 0** is fast but has no fault tolerance — one disk failure loses everything.

**RAID 1** duplicates every write; storage efficiency 50%, but read can be served from either copy.

**RAID 5** distributes parity across all disks; reading is fast, writing has a "write penalty" (read-modify-write of parity). Rebuild after a failure is slow.

**RAID 6** is RAID 5 with two parity blocks per stripe — survives two simultaneous failures. Often used for large arrays (e.g., > 12 TB).

---

## Worked example 7.1 — choosing a RAID level

**Scenario A — a database server with high read/write ratio and tolerance for 1-disk failure.**

**Choice:** RAID 5 or RAID 10. RAID 5 offers good capacity efficiency; RAID 10 (mirror + stripe) gives better write performance but 50% storage overhead.

**Scenario B — a backup server, sequential writes dominate, capacity matters.**

**Choice:** RAID 6. Two-disk redundancy, good capacity efficiency, write performance is acceptable.

**Scenario C — a scratch/scratch-adjacent server with maximum speed, no durability requirement.**

**Choice:** RAID 0. Fastest, but any failure loses data — acceptable only if the data is regenerable.

**Scenario D — a small server with only two disks.**

**Choice:** RAID 1. The only level that gives redundancy with two disks.

---

## 7.7 Quiz

1. List the three components of disk access time and rank them by typical magnitude.
2. Compare heap, sorted, and hashed files on search and insert cost.
3. What problem does dynamic hashing solve that static hashing cannot?
4. Explain extendible hashing — what is the role of the directory and global/local depth?
5. Choose a RAID level for a video-editing workstation and justify.

---

## Answer key

??? success "Click to reveal"

    1. Seek (~3–10 ms) > rotational (~4 ms) > transfer (~0.1 ms).
    2. Heap: O(b) search, O(1) insert. Sorted: O(log b) search, O(b) insert. Hashed: O(1) search, O(1) insert.
    3. Static hashing uses a fixed number of buckets; as the file grows, overflow chains lengthen and search degrades. Dynamic hashing grows the bucket count.
    4. Directory is an array of pointers indexed by the top $d$ bits of the hash; local depth controls how many bits distinguish a given bucket's keys; when a bucket overflows, its local depth increments and the directory may double.
    5. RAID 0 for maximum sequential throughput (scratch space); RAID 5 if resilience is required and budget is limited.

---

## Further reading

- Elmasri & Navathe, §16.1–§16.8, §17.1–§17.4.
- Silberschatz et al., §11.1–§11.6, §12.1–§12.4.