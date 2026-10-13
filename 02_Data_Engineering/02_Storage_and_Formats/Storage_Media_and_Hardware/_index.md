---
id: "02_Data_Engineering/02_Storage_and_Formats/Storage_Media_and_Hardware"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Storage Media and Hardware

**L4 Advanced** · [↑ Storage Systems and File Formats](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** All storage performance traces back to a handful of physical ingredients — spinning disks, SSDs, RAM, CPU and network — each with a very different latency, throughput and price, and good data engineering means matching data to the cheapest medium that still meets its access needs.

### 1. The big picture

Before choosing a file format or a cloud service, you should know what is physically underneath. Magnetic disks are cheap and huge but slow to seek; SSDs are fast but roughly ten times the price per GB; RAM is fastest and most expensive but volatile. Networking and CPU turn individual devices into distributed systems with durability and parallelism.

These facts explain nearly every storage design decision: why data lakes use object storage on magnetic disks, why OLTP databases run on SSDs, why compression pays for itself, why hot/warm/cold tiers exist, and why benchmark claims need scepticism.

This lesson is the hardware foundation for *File, Block and Object Storage* (the abstractions) and *Parquet Format* (how data is laid out to exploit these devices).

### 2. Core concepts and vocabulary

- **HDD (magnetic disk drive)** — spinning platters coated with a ferromagnetic film; a read/write head magnetises the film to write and senses it to read.
- **SSD (solid-state drive)** — stores data as charge in flash cells; no moving parts, read electronically.
- **Seek time** — time to move the head to the correct track.
- **Rotational latency** — wait for the data to spin under the head (7,200 RPM drives).
- **Areal density vs linear density** — capacity scales with bits per square inch; transfer speed scales only with bits per inch, so speed grows more slowly than capacity.
- **IOPS** — random I/O operations per second.
- **RAM** — volatile memory attached to the CPU address space; dynamic RAM stores charge in capacitors that must be refreshed.
- **CPU cache** — small, much faster memory on the CPU die or package.
- **RAID** — parallelising disks within one server.
- **Availability zone** — a cloud construct with independent power, water and other resources; multi-zone storage improves durability and availability.
- **Compression ratio** — original size ÷ compressed size.
- **Hot / warm / cold data** — data by access frequency; cold is rarely read and cheap to keep but expensive to retrieve.
- **Spillover** — data moving from a faster, scarcer tier (RAM) to a slower tier (disk/object storage) when capacity runs out.

### 3. How it works, step by step

**Reading from a magnetic disk**

1. The controller moves the head to the right track (seek).
2. It waits for the sector to rotate under the head (rotational latency).
3. It streams bytes at the disk's transfer speed — 200–300 MB/s for data-center drives.
4. At 7,200 RPM, seek plus rotation costs over ~4 ms per random access; sequential reads avoid most of that.

**Reading from an SSD**

1. The controller addresses flash cells electronically; there is no seek.
2. Random lookups complete in under ~0.1 ms (100 µs).
3. Parallel controllers over partitioned flash give multi-GB/s and tens of thousands of IOPS.

**Reading from RAM**

1. The memory controller returns data over the CPU's memory channels in roughly 100 ns (DDR5), about 1,000× faster than SSD, with ~100 GB/s bandwidth and millions of IOPS.
2. Power loss erases it in under a second, so durable data must also live elsewhere.

**Comparing the devices (book figures, approximate)**

| Medium | Typical random latency | Throughput | Cost per GB | Volatile? |
|---|---|---|---|---|
| Magnetic disk | ~4+ ms | 200–300 MB/s | ~\$0.03 | No |
| SSD | < 0.1 ms | several GB/s | ~\$0.20–0.30 | No |
| RAM (DDR5) | ~100 ns | ~100 GB/s per CPU | ~\$10 | Yes |
| CPU cache | far below RAM | highest | n/a (tiny) | Yes |

### 4. Worked example

Scan-time arithmetic for a 30 TB dataset shows why compression and parallelism matter:

```text
Single 300 MB/s HDD:   30 TB / 300 MB/s  = 100,000 s  ≈ 27.8 hours   (book: "more than 20 hours")
With 10:1 compression: 3 TB effective   → ~2.8 hours   (effective ~2 GB/s per disk at 200 MB/s raw)
Same 3 TB across 100 disks in parallel  → ~100 seconds (ignoring network limits)

Network check: EC2 ↔ S3 at 10 Gbps = 1.25 GB/s
With 10:1 compression, effective bandwidth ≈ 12.5 GB/s (~100 Gbps)
```

How to read this: a single big disk is a throughput bottleneck regardless of capacity; compression multiplies effective scan speed *and* effective network bandwidth; spreading data across many disks (object stores, HDFS, RAID) multiplies throughput again. The cost of compression is CPU time to compress and decompress.

### 5. Under the hood

**Why HDD speed lags capacity.** Capacity tracks areal density (bits/in²), while transfer rate tracks linear density (bits/in). If capacity grows 4×, transfer speed grows only ~2×. Newer techniques — heat-assisted magnetic recording (HAMR), shingled magnetic recording (SMR), helium-filled enclosures — raise density but cannot fix seek and rotation physics. That is why a huge drive is slow to read end to end.

**Why SSDs rule OLTP but not bulk analytics.** Sub-millisecond random access lets PostgreSQL, MySQL or SQL Server do thousands of transactions per second, so SSDs are the standard for OLTP. For petabyte-scale scans, cost wins: object storage on magnetic disks remains the default for lakes and cloud warehouses, with SSDs used as caches for frequently accessed data in low-latency OLAP engines.

**RAM details.** Dynamic RAM stores charge in capacitors that leak, so a hardware controller refreshes it constantly; static RAM is used for CPU caches. CPUs use von Neumann architecture (code and data in one memory space) but can mark pages non-executable for security. RAM serves caching, in-process data and indexes; several databases treat it as primary storage, but then durability must come from a log, replica or snapshot.

**Network and CPU as storage ingredients.** Modern storage is distributed: disks are spread across a network and multiple data centers/zones. CPUs service requests, aggregate reads and distribute writes, so a storage service behaves like a web application with an API, backend components and load balancing. Network topology and device performance determine the real speed, and you trade durability/availability (spread out) against performance/cost (close to consumers).

**Compression interaction.** Compression reduces on-disk size, raises effective scan speed per disk and effective network bandwidth, but costs CPU. Columnar formats compress especially well because similar values sit together (see *Parquet Format*).

### 6. Performance and design implications

- **Match tier to access pattern.** Hot data on SSD or memory, warm on HDD-backed object storage, cold on archive. Putting everything hot is "a tremendous price"; putting everything cold makes retrieval slow and costly.
- **Plan for spillover.** Memory is finite; some engines move cold data to disk or object storage automatically. Know what your engine does, and size RAM for the working set.
- **Automate lifecycle policies.** Move data to Infrequent Access after ~a month without reads and to archive after ~180 days without use; weigh retrieval time and money first.
- **Benchmark realistically.** Vendor tests often use datasets small enough to fit in SSD or RAM and query the same data repeatedly; to be honest, simulate real data size and query mix. Also beware comparing ephemeral per-second-priced systems with always-on clusters on cost per second.
- **Measure what matters:** sustained throughput (MB/s), random IOPS, latency percentiles, and cost per TB-month *and* per TB retrieved.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `fio` (or cloud volume benchmarks) | Measures IOPS, throughput, latency of a disk | Sizing and verifying volumes |
| `iostat -x`, `vmstat` | Observe device utilisation, queue depth, memory pressure | Diagnosing I/O bottlenecks |
| Volume class / IOPS provisioning (e.g., EBS tiers) | Chooses SSD vs HDD-backed volume and performance | Matching DB or batch workload |
| Compression codec choice (Snappy, Gzip, Zstd) | Speed vs ratio trade-off | Balancing CPU against I/O and network |
| Object lifecycle rules | Auto-tier by age/access | Controlling cold-data cost |
| Availability-zone / region placement | Chooses failure domain and latency | Durability vs cost and performance |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **SSD** for OLTP, caches, low-latency OLAP hot sets | Storing petabytes of rarely read data (cost ≈ 10× HDD) |
| **HDD / object storage** for bulk lake data scanned sequentially | Latency-sensitive random access |
| **RAM** for caches, indexes and hot working sets | It is your only copy — volatile |
| **Heavy compression** when I/O or network bound | CPU is already saturated or latency per record matters |
| **Archive/cold tiers** for retention | You will need the data frequently or on short notice |
| **Multi-zone storage** for availability/durability | Cross-zone latency or cost is unacceptable |

### 9. Common mistakes

- Trusting a "big data" benchmark that fits in cache → real workloads are far slower → rerun with realistic volume and query diversity.
- Comparing cost per second for ephemeral and always-on systems → misleading TCO → compare total cost for the real usage pattern.
- Putting RAM-only data without a durable copy → loss on power or process failure → write to disk/object storage or replicate.
- Archiving data that is queried regularly → huge retrieval bills and delays → base tiering on observed access.
- Ignoring network bandwidth in distributed designs → "fast disks" starved by a 10 Gbps link → budget end to end.

### 10. 🎯 Interview kit

**30-second answer:** HDDs are cheap with high capacity but seek-bound (a few ms per random access, 200–300 MB/s); SSDs give sub-0.1 ms random access at about ten times the cost; RAM is ~1,000× faster than SSD but volatile and ~\$10/GB. Data engineers keep hot data on fast media and cold data on cheap media, use compression to multiply effective throughput, and spread data across many disks to scale reads.

**Structure a longer answer:** state the numbers → explain why (physics: seek, rotation, density) → tie to systems (OLTP on SSD, lake on object storage, caches in RAM) → discuss tiering and compression → mention network/CPU and failure domains.

**Likely follow-ups:**
- *Why not use SSDs for the whole lake?* → ~10× cost per GB; scans are sequential so HDD-backed object stores are good enough, with SSD caches for hot data.
- *Why does compression help beyond saving space?* → Higher effective scan speed per disk and effective network bandwidth, at the cost of CPU.
- *What is spillover?* → Data moving from scarce fast storage (RAM) to slower tiers automatically when capacity is exceeded.
- *Why can't a 30 TB HDD be read quickly?* → Transfer speed (~300 MB/s) does not scale with capacity; it takes more than 20 hours.

**Signals interviewers look for:** quantitative sense of latency/throughput/price, awareness of volatility and failure domains, and critical reading of benchmarks.

### 11. Check yourself

- [ ] Recite the approximate latency, throughput and \$/GB for HDD, SSD and RAM.
- [ ] Explain why disk capacity grows faster than transfer speed.
- [ ] Compute the time to scan 10 TB at 250 MB/s, then with 5:1 compression and 20 disks in parallel.
- [ ] Describe a hot/warm/cold design for a clickstream table including spillover and lifecycle rules.
- [ ] List three tricks used to make a benchmark look better than reality.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Random Access Memory
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Compression
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Magnetic Disk Drive
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Networking and CPU
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Solid-State Drive
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Big Data\...for the 1990s
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Cold data

