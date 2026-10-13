---
id: "01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/Storage_Engines"
tools: []
---

# Tiered Storage in LSM Engines

**L4 Advanced** · [↑ Storage Engines](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** "Tiered" in LSM engines means two things — tiered *compaction* (merge similar-sized files) and tiered *storage* (place hot, warm and cold data on different media) — and the Bloom filter and level layout decide how much each tier costs you on reads.

### 1. The big picture
An LSM tree naturally produces data of different ages: the memtable and L0 hold the newest data, deeper levels hold older data. Two design choices exploit that.

1. **Tiered (size-tiered) compaction** — a *strategy* that merges SSTables of similar size into one larger table, instead of rewriting data level by level (leveled). It minimises write amplification and is popular for write-heavy loads.
2. **Tiered storage** — a *placement* policy that puts recent/hot data on fast, expensive media (NVMe, memory) and older/cold data on cheaper media (HDD, object storage). LSM immutability makes this easy: a file never changes, so it can be moved wholesale.

Both interact with read amplification: more tiers and more overlapping files means more places to look, which is why per-file **Bloom filters** and key-range metadata exist. This lesson separates the two meanings, explains how Bloom filters cut lookups, and shows when tiering is the right lever.

### 2. Core concepts and vocabulary
- **Size-tiered compaction (STCS)** — when N SSTables of similar size exist (e.g. 4), merge them into one larger table.
- **Leveled compaction (LCS)** — levels with exponentially increasing capacity and non-overlapping files per level.
- **Time-window compaction (TWCS)** — group SSTables by time bucket; merge within a window; drop whole windows on TTL expiry.
- **Hot / warm / cold data** — access-frequency tiers: hot = frequently accessed, fastest media; warm = infrequent; cold = archival.
- **Tiered storage** — placing levels/files on different devices or tiers based on age or level.
- **Bloom filter** — bit array + k hash functions; "no" is definite, "yes" is probable.
- **False-positive rate (FPR)** — probability of "possibly present" for an absent key; ≈ 0.6185^(bits per key).
- **Key-range pruning** — skipping files whose min/max key excludes the lookup key.
- **Read amplification** — number of files (and blocks) consulted per read.
- **Space amplification** — stored bytes ÷ live bytes (stale versions, tombstones, compaction temp space).

### 3. How it works, step by step
**Size-tiered compaction**
1. Memtable flushes produce small SSTables of similar size.
2. When a bucket contains N tables of comparable size, merge them into one table roughly N× larger (merge-sort, drop shadowed versions).
3. Larger tables accumulate in bigger buckets and eventually merge again.
4. Each byte is rewritten about log_N(total/flush size) times — few rewrites.
5. But a key may live in many tables of different sizes, and tables overlap in key ranges.

**Leveled (for contrast):** a file moves down one level at a time, merged with overlapping files; ≤1 file per level can contain a key, but each byte is rewritten up to ~ratio times per level.

| Property | Size-tiered | Leveled | Time-window |
|---|---|---|---|
| Write amplification | Low (≈ log N) | High (10–30×) | Low |
| Read amplification | High (many overlapping files) | Low (≈ 1 per level) | Low within window |
| Space amplification | High (up to ~2× + temp space) | Low (~1.1×) | Low |
| Best for | Ingest-heavy, rare updates | Read-heavy, many updates | TTL time-series |

**Bloom filter lookup**
1. At flush/compaction time, hash every key in the SSTable into a bit array with k hash functions.
2. At read time, hash the lookup key with the same functions.
3. If any corresponding bit is 0 → the key is **definitely absent**; skip this file.
4. If all bits are 1 → **maybe present**; do the index and data block read (with probability FPR it's a wasted read).

**Tiered storage placement**
1. Newest files (memtable, L0, upper levels) stay on NVMe/SSD.
2. Older data — the bottom levels, or time windows past a threshold — is written (by compaction output) directly to a cheaper tier or object storage.
3. Metadata (index, filter) is often kept locally so negative lookups never touch the cold tier.

### 4. Worked example
Sizing a Bloom filter and the benefit of it.

- Table: 100 million keys across 20 SSTables, a lookup for a key that does not exist (a common case for existence checks and upserts).
- Without filters: 20 index lookups, ~20 block reads on a cold cache.
- With 10 bits/key (FPR ≈ 1%): expected wasted block reads = 20 × 0.01 = 0.2 — a ~100× reduction.
- Memory: 100 M × 10 bits = 125 MB total.

Cassandra configuration and observation:
```sql
-- CQL: choose compaction strategy and Bloom filter accuracy per table
CREATE TABLE sensor_readings (
  sensor_id uuid, bucket date, ts timestamp, value double,
  PRIMARY KEY ((sensor_id, bucket), ts)
) WITH compaction = {
        'class': 'TimeWindowCompactionStrategy',
        'compaction_window_unit': 'DAYS',
        'compaction_window_size': 1 }
  AND default_time_to_live = 2592000          -- 30 days
  AND bloom_filter_fp_chance = 0.01;
```
```text
nodetool tablestats ks.sensor_readings
  SSTable count: 31
  Space used (live): 48.2 GiB
  Bloom filter false ratio: 0.009
  Bloom filter space used: 61 MiB
  Average tombstones per slice: 0
```
Read it as: 31 SSTables = ~30 daily windows + current; the false ratio matches the configured target; TTL lets entire expired windows be dropped as files — no tombstone scans, no compaction rewrite of cold data.

### 5. Under the hood
- **Bloom filter math.** For n keys and m bits, optimal k = (m/n)·ln 2 hash functions gives FPR ≈ (0.6185)^(m/n). 10 bits/key → ~1%; 15 bits/key → ~0.1%. The filter can be built once because SSTables are immutable; it never supports deletion (counting Bloom filters do).
- **Cost of the filter.** Memory scales with key count, not value size; for tiny values the filter can be a large share of the table's memory. RocksDB lets you skip filters on the last level (`optimize_filters_for_hits`) because most hits land there.
- **Range queries bypass Bloom filters.** They need key-range metadata and prefix Bloom filters; scans still merge iterators over every overlapping file.
- **Size-tiered space blowup.** A major merge of N tables needs temporary space close to the sum of the inputs; with tables near half the disk, the merge can't run. Overwritten or deleted data persists until its tables are merged together.
- **Time-window compaction invariants.** Out-of-order writes into old windows create extra small files and defeat dropping whole windows; mixed TTLs in one window block expiry until the longest TTL passes.
- **Tiered storage and immutability.** Immutability means moving a file is a copy + manifest swap; no in-place invalidation. Cold-tier reads are high-latency, so engines cache blocks locally, prefetch index/filter blocks and disallow compaction from reading cold files unless necessary (compaction rewrites cost egress).
- **Hot/warm/cold in practice** — cloud object storage tiers (infrequent access, nearline, archive) give lower storage cost and higher retrieval cost/latency; the policy is a cost model: storage $/GB vs access count × retrieval $ vs latency SLO.
- **Block cache vs OS page cache.** Hot data can be held in memory; with LSM, compactions can evict hot blocks (cache pollution) unless compaction reads bypass or are read-ahead limited.

### 6. Performance and design implications
- Choose compaction by workload: **TWCS** for TTL time-series; **STCS** for ingest-heavy write-once; **LCS** for read-latency-sensitive overwritten keys.
- Read amplification under STCS: set `bloom_filter_fp_chance` lower, keep key ranges tight via good partitioning, and keep SSTable count per read low (monitor `SSTables per read` histograms).
- Provision at least 50% free disk for STCS, ~10–20% for LCS.
- Put cold windows on cheaper storage only when access is truly rare, otherwise p99 latency tails will exceed SLO.
- Keep Bloom filter and index blocks pinned in memory/local SSD even when data is remote.
- Measure: SSTables per read, bloom false ratio, compaction pending bytes, percent of reads served by each tier, storage cost per GB-month, cold-tier retrieval counts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `compaction = {'class': 'SizeTieredCompactionStrategy', 'min_threshold': 4}` | STCS with merge threshold | Write-heavy tables |
| `LeveledCompactionStrategy` / `sstable_size_in_mb` | LCS | Read-heavy, overwritten data |
| `TimeWindowCompactionStrategy` + window options | Time buckets | TTL time-series |
| `bloom_filter_fp_chance` | Filter accuracy vs memory | Reduce wasted disk reads |
| `default_time_to_live` | Row expiry | Time-window dropping |
| `nodetool tablehistograms` | SSTables per read, latencies | Verify read amplification |
| RocksDB `optimize_filters_for_hits`, `bloom_bits_per_key` | Filter placement/size | Tune memory vs reads |
| Object-store lifecycle rules | Move/expire cold files | Cost tiering |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Size-tiered for high write throughput with few updates | Disk is near full or reads are latency-critical |
| Leveled for hot overwritten keys and point reads | Disk write budget (SSD endurance) is tight |
| Time-window for TTL data with in-order arrival | Late/out-of-order writes or mixed TTLs |
| Cold tier on object storage for compliance/archive | Frequent random reads on cold data |
| Lower Bloom FPR (more bits) | Memory-constrained nodes with enormous key counts |

### 9. Common mistakes
- Using size-tiered on a read-heavy overwrite workload → many files per read, stale versions linger → leveled.
- Using TWCS with out-of-order or updated data → many small overlapping files, no dropping → keep writes in time order or use another strategy.
- Assuming Bloom filters speed up range scans → they don't → rely on partition design and prefix filters.
- Letting disks fill with size-tiered → compaction stuck, writes fail → keep headroom, alert at 50%.
- Placing hot data's index/filters on the cold tier → every negative lookup pays remote latency → pin metadata locally.
- Equating "tiered storage" with "tiered compaction" → wrong configuration → clarify which one is being discussed.

### 10. 🎯 Interview kit
**30-second answer:** In an LSM engine, tiered compaction merges similarly sized SSTables, giving low write amplification but higher read and space amplification; leveled compaction does the opposite. Separately, tiered storage places older immutable files on cheaper media. Bloom filters keep read amplification down by letting a lookup skip files that definitely don't contain the key.

**Structure a longer answer:** define both "tiered" meanings → compare compaction strategies on the three amplifications → Bloom filter mechanics and sizing → tiered storage placement and costs → scenario (30-day sensor data).

**Likely follow-ups:**
- *Why can a Bloom filter have false positives but not false negatives?* → Bits are only ever set, never cleared; collisions can make absent keys look present.
- *How do you size it?* → ~10 bits/key for 1%; FPR ≈ 0.6185^(bits per key).
- *Why is TWCS good for time-series?* → Whole expired windows are dropped as files; no tombstones, no rewrites of cold data.
- *What does immutability buy for tiering?* → Files can be copied/moved without coordination.

**Signals interviewers look for:** you distinguish the two uses of "tiered", quantify with FPR and amplification, and choose by workload.

### 11. Check yourself
- [ ] Contrast size-tiered, leveled and time-window compaction across the three amplifications.
- [ ] Compute Bloom filter memory and expected wasted reads for a given key count and FPR.
- [ ] Explain why range scans don't benefit from standard Bloom filters.
- [ ] Describe a hot/warm/cold policy for 30-day vs 1-year retention data.
- [ ] Explain why STCS needs large free-space headroom.
<!-- explainer:end -->

## Go deeper

- [B-tree Concurrency and Latches](<B-tree_Concurrency_and_Latches.md>)
- [LSM Write Path WAL and Compaction](<LSM_Write_Path_WAL_and_Compaction.md>)
- [B-tree vs LSM Storage Engines](<B-tree_vs_LSM_Storage_Engines.md>)

## Practice questions

- [Intermediate] [Explain how a covering index eliminates the B-Tree double lookup (secondary index then clustered index). When would you deliberately not create a covering index despite the read win?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20501_Explain_how_a_covering_index_eliminates_the_B-Tree_double_lo.md>)
- [Intermediate] [You see high CPU from checksum verification on every block read in your storage engine. Explain when per-block checksums (CRC32C / xxHash) are worth the CPU, and how you'd decide to relax them for a read-hot path.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20563_You_see_high_CPU_from_checksum_verification_on_every_block_r.md>)
- [Deep] [Explain how a bloom filter reduces read amplification for negative lookups in an LSM. If your false-positive rate is 1% and you have 6 SSTables to check, what's the expected number of wasted disk reads for a missing key?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20483_Explain_how_a_bloom_filter_reduces_read_amplification_for_ne.md>)
- [Deep] [Explain how prefix and dictionary block compression work inside an SSTable data block. What's the CPU vs I/O trade-off, and how would you decide whether to enable it for a latency-sensitive read path?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20499_Explain_how_prefix_and_dictionary_block_compression_work_ins.md>)
- [Deep] [Explain how you'd size a bloom filter for an LSM: bits per key vs false-positive rate vs memory budget. If RAM is scarce and the workload is mostly existing-key reads, is investing in bloom filters worth it? Reason it out.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20512_Explain_how_you_d_size_a_bloom_filter_for_an_LSM_bits_per_ke.md>)
- [Deep] [Explain the read path for a key in an LSM: memtable, immutable memtables, then L0..Ln with bloom + index. For a key present only in L3, count exactly how many components are consulted and where caches short-circuit.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20525_Explain_the_read_path_for_a_key_in_an_LSM_memtable_immutable.md>)
- [Deep] [Explain the concept of block cache vs OS page cache in a storage engine. If you have 64GB RAM, how would you split between RocksDB block cache and letting the OS cache compressed blocks, and why not just give all to one?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20554_Explain_the_concept_of_block_cache_vs_OS_page_cache_in_a_sto.md>)
- [Expert] [Explain how partition-level (partitioned) bloom filters and per-block filters differ, and why RocksDB moved toward partitioned filters for large SSTables. What problem with monolithic filters did this solve?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20560_Explain_how_partition-level_partitioned_bloom_filters_and_pe.md>)

## Related concepts

- [Large Values and TOAST](<../../../SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*

## References

- *Database Internals* — Alex Petrov — Introduction and Overview › Buffering, Immutability, and Ordering
- *Database Internals* — Alex Petrov — B-Tree Basics › Hard Disk Drives
- *Database Internals* — Alex Petrov — B-Tree Basics › On-Disk Structures
- *Database Internals* — Alex Petrov — File Formats › Motivation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Hash Indexes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › SSTables and LSM-Trees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Constructing and maintaining SSTables
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Performance optimizations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Heap Tables
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 8 Deleting Form Heaps Results in Unrecoverable Space › Myth 8: Deleting Form Heaps Results in Unrecoverable Space
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 9 Every Table Should Have a HeapClustered Index › Myth 9: Every Table Should Have a Heap/Clustered Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Other Heap Scenarios
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — To integrate MX4J into a Cassandra installation, download the
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Memory Structures
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Sorted-String Tables and Log-Structured Merge Trees
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Bloom filters
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Bloom Filter Index

