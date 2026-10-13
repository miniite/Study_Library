---
id: "01_Databases_and_SQL/Database_Internals/Storage_Engines/B-tree_vs_LSM_Storage_Engines"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Internals/Storage_Engines"
tools: []
---

# B-tree vs LSM Storage Engines

**L5 Expert** · [↑ Storage Engines](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** B-trees update pages in place (cheap reads, random writes), LSM trees only append and merge sorted immutable files (cheap writes, more read and background work) — each trades read, write and space amplification differently.

### 1. The big picture
Every storage engine must answer: *where on disk does a record live, and how do I change it?* Two families dominate.

- **Mutable, in-place structures — B-trees/B+-trees.** The record has a home page; an update modifies that page (via the buffer pool and a write-ahead log). SQL Server, PostgreSQL, InnoDB, Oracle use them.
- **Immutable, log-structured structures — LSM trees.** Writes go to an in-memory sorted buffer and a sequential log; the buffer is flushed as an immutable sorted file; files are merged in the background (**compaction**). Cassandra, RocksDB, LevelDB, HBase, Bigtable, WiredTiger (optionally) use them.

Neither is "faster" universally. The decision is driven by workload shape (read/write ratio, key distribution, range scans), storage medium (SSD wear, HDD seeks) and operational constraints (predictable latency, disk headroom). The key analytic tool is the three **amplification** factors: read, write, space — the "RUM conjecture": you can optimise two of Read, Update, Memory overhead only at the expense of the third.

### 2. Core concepts and vocabulary
- **B+-tree** — balanced tree; internal nodes hold separator keys, leaves hold records and are linked for range scans.
- **Fanout** — child pointers per node (hundreds), determined by page size ÷ key+pointer size; tree height is typically 3–4 even for billions of rows.
- **Fence/separator key** — key in an internal node that routes a lookup to the correct child.
- **Page split** — full page divided in two, parent updated.
- **Write-ahead log (WAL)** — sequential log of changes written before the data page, for crash recovery.
- **Memtable** — in-memory sorted structure (skip list/tree) absorbing LSM writes.
- **SSTable (sorted string table)** — immutable sorted file with an index and Bloom filter.
- **Compaction** — merging SSTables to drop overwritten and deleted records and reduce file count.
- **Tombstone** — marker written for a delete; physically removed only at compaction.
- **Read amplification** — physical reads (pages/files) per logical read.
- **Write amplification** — bytes written to storage per byte of user data.
- **Space amplification** — bytes stored per byte of live data.
- **Bloom filter** — probabilistic structure: "definitely not present" or "possibly present"; no false negatives.

### 3. How it works, step by step
**B-tree point write (update key K):**
1. Descend root → leaf (≈3–4 page reads, upper levels cached).
2. Log the change to the WAL (sequential append, fsync at commit).
3. Modify the leaf page in the buffer pool; mark it dirty.
4. If the leaf is full, split it and update the parent.
5. A checkpoint/background writer later flushes the dirty page to its home location (random write).

**LSM point write:**
1. Append the record to the WAL (sequential, for durability).
2. Insert into the memtable (sorted, in memory).
3. When the memtable is full, freeze it and flush it as a new immutable SSTable (sequential write).
4. Background compaction merges SSTables, keeping the newest version per key and dropping tombstones that have aged out.

**LSM point read:**
1. Check the memtable (and immutable memtables).
2. For each SSTable from newest to oldest: check key range, then Bloom filter, then the sparse index and the data block.
3. Return the first (newest) version; a tombstone means "not found".

**Range scans:** B+-tree walks linked leaves — one contiguous sorted structure. LSM merges iterators across the memtable and every overlapping SSTable.

| | B-tree | LSM tree |
|---|---|---|
| Write pattern | Random page writes + sequential WAL | Sequential only |
| Point read | ~H page reads (cached upper levels) | Memtable + several files (Bloom filters trim) |
| Range scan | Excellent (linked leaves) | Good once compacted; merges many iterators |
| Write amplification | From page rewrites, splits, full-page writes | From repeated compaction rewrites (10–30× leveled) |
| Space amplification | Fragmentation (50–90% fill) | Obsolete versions awaiting compaction; temp space during compaction |
| Latency profile | Predictable; checkpoint spikes | Write stalls when compaction falls behind |
| Concurrency | Latches on pages | Immutable files → lock-free reads; compaction races |

### 4. Worked example
Back-of-the-envelope: 1 billion rows × 100 B = 100 GB, 8 KB pages, ~80 rows/page.

- **B-tree height:** leaves ≈ 12.5 M pages; with fanout ~400, levels ≈ log₄₀₀(12.5 M) ≈ 2.7 → height 3–4. Upper levels (a few hundred KB) sit in memory, so a cold point read is ~1–2 disk reads.
- **Random updates:** each 100-byte update dirties an 8 KB page — a physical write of 8 KB (and sometimes a full-page image in the log) → write amplification of ~80× before batching.
- **LSM leveled, size ratio 10, 4 levels:** each byte is rewritten about once per level × ratio/2 ≈ 10–30× over its life, but all of it sequential I/O.

Inspect them in the wild:
```sql
-- SQL Server: B-tree depth, fill, fragmentation
SELECT index_depth, avg_page_space_used_in_percent, avg_fragmentation_in_percent, page_count
FROM   sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.Orders'), 1, NULL, 'DETAILED')
WHERE  index_level = 0;
```
```text
# Cassandra: LSM state for one table
nodetool tablestats ks.users
  SSTable count: 14
  Bloom filter false ratio: 0.01
  Local read latency: 0.4 ms
  Pending compactions: 3
```
Reading these: a B-tree with `avg_page_space_used` of 60% and high fragmentation shows split-driven space and read amplification; an LSM with growing SSTable count and pending compactions signals read amplification and impending write stalls.

### 5. Under the hood
- **Why B-trees need WAL and full-page writes.** Pages are updated in place; a torn write (crash mid-page) corrupts a page. Engines use double-write buffers (InnoDB), full-page images after checkpoints (Postgres), or checksums plus torn-page detection (SQL Server).
- **Where write amplification comes from.** B-tree: writing back whole pages for small changes and repeated writes to the same hot page between checkpoints. LSM: moving data from level to level during compaction. Comparing the two naively is misleading because the sources differ.
- **Compaction strategies.** *Size-tiered*: merge similar-sized tables when enough accumulate — low write amplification, high space and read amplification (up to ~2× space during merges). *Leveled*: each level 10× larger, non-overlapping files within a level — a key exists in at most one file per level, so reads are bounded, at the cost of ~10–30× write amplification. *Time-window / FIFO*: for TTL time-series, drop whole files.
- **Bloom filters** with ~10 bits/key give ~1% false positives, avoiding most useless file reads for absent keys; they don't help range scans.
- **Deletes.** B-tree: remove the cell (or mark a ghost record). LSM: write a tombstone; the space is only reclaimed after compaction reaches the data and past the retention grace period (`gc_grace_seconds` in Cassandra). Mass deletes can make reads scan huge numbers of tombstones.
- **Log stacking.** SSDs run their own log-structured flash translation layer and filesystems may too; stacking LSM on a log-structured FS on an SSD can multiply write amplification and wear.
- **Hybrids.** Fractal/Bε-trees buffer updates in internal nodes; WiredTiger offers both B-tree and LSM tables; FD-trees keep small writes in a head tree and merge into sorted runs sized for flash.

### 6. Performance and design implications
- **Write-heavy ingestion** (time-series, logs, events, IoT), large datasets on SSD, high compressibility → LSM wins on throughput and often on storage via better compression (immutable, sorted, block-compressed).
- **Read-heavy, latency-sensitive OLTP** with point lookups and range scans, in-place updates, transactions with secondary indexes → B-tree is usually simpler and more predictable.
- **Hot small key range** with strong read-latency SLO: B-tree (upper levels and hot pages cached, one path); LSM will also cache blocks but pays the multi-file lookup tail.
- **Space headroom:** leveled LSM needs ~10% extra; size-tiered needs up to 50% free for large compactions.
- **Predictability:** compaction debt is the LSM's failure mode (write stalls); checkpoints/vacuum are the B-tree's.
- **Measure:** read/write/space amplification from engine counters (RocksDB `stats`, `iostat` bytes vs logical bytes), p99 latency, compaction backlog, cache hit ratio, page fill and fragmentation.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `compaction = {class: LeveledCompactionStrategy}` (Cassandra) | Leveled compaction | Read-heavy LSM, bounded read amp |
| `SizeTieredCompactionStrategy` / `TimeWindowCompactionStrategy` | Tiered / time-bucketed | Write-heavy / TTL time-series |
| `bloom_filter_fp_chance` | Memory vs false positives | Tune read path |
| RocksDB `write_buffer_size`, `max_bytes_for_level_base` | Memtable & level sizing | Throughput tuning |
| `FILLFACTOR` / `fillfactor` | Page free space | Reduce B-tree splits |
| `sys.dm_db_index_physical_stats` | B-tree depth, fill, fragmentation | SQL Server index health |
| `nodetool compactionstats / tablestats` | LSM backlog & SSTable counts | Diagnosing Cassandra |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **B-tree** for mixed OLTP with range scans and strong consistency | Sustained high-rate random writes on slow disks |
| **LSM** for write-dominated, append-style or time-series workloads | Latency SLOs hostile to compaction stalls |
| Leveled compaction for read-heavy LSM | Heavy overwrite with tight write budget on SSD wear |
| Size-tiered for write-heavy | Need low space overhead |
| Hybrid/two engines per workload | Operational complexity outweighs gains |

### 9. Common mistakes
- Declaring one structure "faster" → wrong choice → define the workload and amplification budget first.
- Ignoring compaction backlog → read latency climbs, then write stalls → monitor pending compactions and provision I/O headroom.
- Mass-deleting in an LSM and expecting instant space → tombstones, slow range reads → TTL/time windows or drop whole tables.
- Random UUID keys in a B-tree → splits, half-empty pages → sequential keys or lower fill factor.
- Sizing disk at 1× data for size-tiered → compaction can't finish → keep 50% free.
- Forgetting write amplification on SSDs → premature wear → measure bytes written to device.

### 10. 🎯 Interview kit
**30-second answer:** B-trees keep sorted pages and update them in place, so reads and range scans are efficient but updates cause random page writes and splits. LSM trees buffer writes in a memtable, flush immutable sorted files and merge them by compaction, so writes are sequential and fast but reads must consult multiple files and compaction adds write and space amplification.

**Structure a longer answer:** the two write paths → read paths → the three amplifications → compaction strategies → workload-based choice and a concrete scenario (metadata store vs event ingestion).

**Likely follow-ups:**
- *Why does B-tree fanout depend on page and key size?* → Fanout ≈ page ÷ (key + pointer); long keys reduce fanout, raise height, add I/O.
- *Why B+-tree rather than B-tree for databases?* → Data only in leaves, linked leaves make range scans sequential and internal nodes denser.
- *How do deletes work in an LSM?* → Tombstones, removed at compaction after grace period.
- *Which has lower write amplification?* → Depends on the source of the amplification; neither wins universally.

**Signals interviewers look for:** you reason with amplification, mention WAL and Bloom filters unprompted, and let the workload decide.

### 11. Check yourself
- [ ] Draw the write and read paths of an LSM tree.
- [ ] Compute B-tree height for a billion rows with given page/key sizes.
- [ ] Explain read, write and space amplification and give an example of each in both structures.
- [ ] Compare leveled and size-tiered compaction.
- [ ] Argue for B-tree or LSM for a metadata store with 80% reads and a hot key range.
<!-- explainer:end -->

## Practice questions

- [Basic] [Explain the concept of a fence / separator key in B-Tree internal nodes and how a B+Tree keeps all values in leaves with linked leaf pages. Why does this matter specifically for range scans versus a plain B-Tree?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20533_Explain_the_concept_of_a_fence_separator_key_in_B-Tree_inter.md>)
- [Deep] [Explain how you'd estimate the total I/O cost of ingesting 1TB into a leveled-compaction LSM with amplification factor 20. What does that imply about the disk endurance (SSD write budget) and cost planning?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20535_Explain_how_you_d_estimate_the_total_I_O_cost_of_ingesting_1.md>)
- [Deep] [You're standardizing observability so any engineer can debug either a B-Tree or LSM engine with the same mental model. Design the unified metrics/tracing abstraction over fundamentally different internals (page splits vs compaction) without hiding the details that matter during incidents.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20601_You_re_standardizing_observability_so_any_engineer_can_debug.md>)
- [Deep] [A team wants to run analytics directly against the LSM primary using long-range iterators, and it keeps evicting the block cache and hurting OLTP latency. As principal, design the org-wide policy that prevents this pattern rather than fixing it per incident, and the enforcement mechanism.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20636_A_team_wants_to_run_analytics_directly_against_the_LSM_prima.md>)
- [Expert] [Explain how you'd design a durable append-only event log storage engine from scratch. Would you use an LSM, a B-Tree, or a simpler segment-file design like Kafka? Justify with write pattern, read pattern, and retention.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20530_Explain_how_you_d_design_a_durable_append-only_event_log_sto.md>)
- [Expert] [Define capacity-planning models for a fleet of LSM nodes where write amplification, compaction debt, and SSD endurance all interact. How would you forecast disk exhaustion and drive-wear replacement 12 months out, and what leading indicators feed your capacity SLOs?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20581_Define_capacity-planning_models_for_a_fleet_of_LSM_nodes_whe.md>)
- [Expert] [You're designing a global multi-tenant SaaS where tenant sizes span 6 orders of magnitude. Decide how storage-engine choice and per-tenant isolation (one big LSM vs DB-per-tenant vs column families) should vary by tenant tier, and how you'd avoid noisy-neighbor compaction across tiers.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20605_You_re_designing_a_global_multi-tenant_SaaS_where_tenant_siz.md>)
- [Expert] [You must decide whether the org standardizes on one WAL/replication paradigm (physical page-level vs logical row-level) across all engines to simplify DR and CDC. Reason through the trade-offs when B-Tree and LSM engines coexist, and what CDC pipeline you'd build on top.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20631_You_must_decide_whether_the_org_standardizes_on_one_WAL_repl.md>)

## Related concepts

- [Choosing a Database for the Workload](<../../../Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [B-tree and BRIN Indexes](<../../../Query_Performance/Indexing/B-tree_and_BRIN_Indexes.md>) *(Indexing)*
- [Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>) *(Data Structures and Algorithms)*
- [Database Cost Optimization](<../../../Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Advantages of LSM-trees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Downsides of LSM-trees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Shared-Nothing Architectures
- *Designing Data-Intensive Applications* — Martin Kleppmann — J › L
- *Database Internals* — Alex Petrov — B-Tree Variants › Bw-Trees
- *Database Internals* — Alex Petrov — Log-Structured Storage › LSM Trees
- *Database Internals* — Alex Petrov — Log-Structured Storage › Multicomponent LSM Trees
- *Database Internals* — Alex Petrov — Log-Structured Storage › Maintenance in LSM Trees
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Engineering's Role › Points to remember
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Implementations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — � D › � O
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — handling. Instead of failing immediately on a bad input line (or worse, failing silently and

