---
id: "01_Databases_and_SQL/Database_Internals/Storage_Engines/LSM_Write_Path_WAL_and_Compaction"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Internals/Storage_Engines"
tools: ["MySQL"]
---

# LSM Write Path WAL and Compaction

**L5 Expert** · [↑ Storage Engines](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An LSM engine makes every write cheap — append to a WAL, insert into a memtable, flush to an immutable sorted file — and pays later by merging files in the background (compaction), which is where read, write and space amplification are traded off.

### 1. The big picture
Disks (and SSDs) are far better at sequential than random writes. The log-structured merge tree turns *all* writes into sequential ones: nothing is modified in place. The cost is that the current value of a key may be spread over several files, so reads must reconcile versions and a background process must continually reorganise data.

Three components make this durable and fast:
- the **write-ahead log (WAL / commit log)** guarantees that an acknowledged write survives a crash;
- the **memtable** keeps recent writes sorted in memory so they can be read and flushed in order;
- **SSTables** (immutable sorted files) hold flushed data, merged over time by **compaction**.

Understanding this path explains LSM behaviours that surprise newcomers: why deletes don't free space, why write throughput collapses when compaction falls behind, why Bloom filters matter, and why recovery after a crash replays a log.

### 2. Core concepts and vocabulary
- **WAL / commit log** — append-only file recording each mutation before acknowledging; replayed on restart.
- **Memtable** — in-memory sorted map (skip list/red-black tree) of recent writes.
- **Immutable memtable** — a full memtable awaiting flush while a new one accepts writes.
- **SSTable** — Sorted String Table: immutable file of key-sorted records with a block index and often a Bloom filter and compression.
- **Flush** — writing a memtable to disk as a new SSTable (level 0 in leveled designs).
- **Compaction** — merge-sorting SSTables into new ones, dropping shadowed versions and expired tombstones.
- **Sequence number / timestamp** — orders versions of a key; newest wins.
- **Tombstone** — delete marker, kept until all older data it shadows has been compacted away and a grace period has elapsed.
- **Level / run** — group of SSTables; in leveled compaction levels are non-overlapping and grow ×10.
- **Read / write / space amplification** — extra reads, bytes written and bytes stored relative to the logical data.
- **Bloom filter** — per-SSTable probabilistic set-membership test; no false negatives.
- **Write stall** — engine throttles or stops writes because flush/compaction can't keep up.

### 3. How it works, step by step
**Write path**
1. The client sends `put(k, v)`.
2. The engine assigns a sequence number and appends the record to the WAL; depending on the sync policy it `fsync`s (per write, per batch/group commit, or periodic — e.g. Cassandra `commitlog_sync = periodic` 10 s, RocksDB `sync` flag).
3. The record is inserted into the memtable (O(log n) in memory). The client is acknowledged. **No read, no seek.**
4. When the memtable reaches its size limit, it becomes immutable and a new memtable is created; a background thread writes the immutable one as a sorted SSTable (sequential), then the corresponding WAL segment can be discarded.

**Read path**
1. Check the memtable(s).
2. Check SSTables newest-first (L0 may overlap, so all L0 files; one file per deeper level for leveled compaction).
3. Per candidate file: key range check → Bloom filter → index lookup → read the data block (possibly from the block cache).
4. Combine results: the highest sequence number wins; a tombstone returns "not found". For partial-row/column stores, merge fragments from several files.

**Compaction (leveled example)**
1. L0 accumulates flushed files, which may overlap in key ranges.
2. When L0 exceeds its threshold, pick files and merge them with the overlapping files in L1.
3. Merge-sort the inputs; for equal keys keep the newest version; drop deleted keys when no older data can exist below; write new non-overlapping L1 files; atomically swap them in via a manifest and delete the inputs.
4. Cascade: when L1 exceeds its size target (e.g. 10× L0), push a file down to L2, and so on.

**Recovery**
1. On restart, read the manifest to learn current SSTables.
2. Replay WAL segments not yet covered by a flush into a fresh memtable.
3. Resume — acknowledged writes are not lost.

### 4. Worked example
Trace the key `user:42` through its life:

| Time | Operation | Where it lives |
|---|---|---|
| t1 | `put(user:42, "Ann")` seq 100 | WAL, memtable |
| t2 | memtable flushed | SSTable S1 (L0) contains seq 100 |
| t3 | `put(user:42, "Anna")` seq 250 | WAL, new memtable |
| t4 | flush | SSTable S2 (L0) contains seq 250 |
| t5 | `delete(user:42)` seq 400 | tombstone in memtable → S3 |
| t6 | read | checks memtable, S3 (tombstone, seq 400) → returns "not found" without consulting S1/S2 |
| t7 | compaction merges S1+S2+S3 | newest per key = tombstone; if no older data exists below and grace period passed, the key is dropped entirely |

Observing it on Cassandra:
```text
nodetool flush ks users          # force memtable → SSTable
nodetool tablestats ks.users     # SSTable count, bloom filter false ratio, pending flushes
nodetool compactionstats         # running/pending compactions
nodetool compact ks users        # major compaction (rarely recommended)
```
```text
# RocksDB LOG excerpt style
[flush_job] memtable flushed: 64MB -> L0 file 000123.sst
[compaction] L0 -> L1: 4 files + 3 files, 310MB in, 280MB out (30MB reclaimed)
```
Interpretation: the difference between "in" and "out" is space reclaimed from overwritten/deleted versions; the sum of bytes written by compaction over bytes ingested is the write amplification.

### 5. Under the hood
- **WAL durability trade-offs.** `fsync` per write is safest (loss window zero) but costs ~device latency; group commit amortises one `fsync` over many concurrent writers; periodic sync risks losing the last interval on power loss. With replication, a periodic sync is often acceptable because other replicas hold the data.
- **Memtable internals.** A skip list allows concurrent lock-free-ish inserts and ordered iteration for flush. Memory is bounded (e.g. 64 MB per memtable, multiple per table); too many tables or column families can exhaust memory.
- **SSTable layout.** Data blocks (4–64 KB, compressed individually), an index block mapping the first key of each block to its offset, a Bloom filter block, and a footer. Because the file is immutable, it can be memory-mapped, cached and replicated safely.
- **Compaction strategies and amplification.** *Size-tiered*: group similar-sized SSTables (e.g. 4 at a time); write amplification ~O(log N) but low; space amp up to 2× and overlapping files hurt reads. *Leveled*: write amp ≈ ratio × levels (often 10–30×); read amp bounded (≈1 file per level); space amp ≈ 1.1×. *Universal/FIFO/time-window*: tailored to time-series and TTL.
- **Sequence numbers and snapshots.** Compaction may only discard an old version if no live snapshot/iterator can still see it; long-running scans pin versions and delay reclamation.
- **Tombstones.** A tombstone must persist until it reaches the bottom level covering all older data, or deleted data "resurrects". Cassandra's `gc_grace_seconds` (default 10 days) also lets replicas repair before tombstones are purged; skipping repair within that window resurrects data.
- **Write stalls.** RocksDB throttles writers when L0 file count or pending compaction bytes cross thresholds (`level0_slowdown_writes_trigger`, `level0_stop_writes_trigger`).
- **Log stacking.** Running log-structured storage on top of a log-structured filesystem and an SSD's FTL compounds write amplification and garbage-collection pauses; preallocate, align, and give the FTL spare capacity.
- **Why sequential?** Appends avoid seeks on HDD and avoid read-modify-erase cycles on SSD, extending flash life even though total bytes written may rise.

### 6. Performance and design implications
- **Write throughput** is bounded by WAL device speed and compaction I/O. Put the WAL on a separate device if fsync-bound.
- **Read latency tails** come from (a) many L0 files, (b) cold cache, (c) Bloom filter misses, (d) tombstone-heavy ranges. Reduce with leveled compaction, more bloom bits per key, larger block cache.
- **Compaction debt.** If ingest rate × write amp > disk bandwidth, backlog grows without bound. Capacity-plan I/O for ingest × amplification, not ingest alone.
- **Space:** need headroom for compaction output (size-tiered can temporarily need as much free space as the data being merged).
- **Deletes and TTL:** prefer TTL + time-window compaction; avoid wide-row mass deletes.
- **Measure:** SSTable count per read (read amp), bytes written to disk ÷ bytes ingested (write amp), on-disk size ÷ live size (space amp), pending compaction bytes, stall time, p99 read latency, bloom false-positive ratio.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `commitlog_sync`, `commitlog_sync_period` (Cassandra) | WAL sync policy | Durability vs latency |
| `memtable_flush_writers`, `memtable_heap_space` | Flush concurrency/size | Heavy ingestion |
| `compaction_throughput_mb_per_sec` | Throttle compaction | Protect foreground latency |
| `gc_grace_seconds` | Tombstone retention | Tune with repair cadence |
| `nodetool compactionstats / tablehistograms` | Backlog and SSTables-per-read | Diagnose read amp |
| RocksDB `write_buffer_size`, `max_write_buffer_number`, `level0_*_trigger` | Memtable and stall tuning | Prevent write stalls |
| `bloom_filter_fp_chance` | FP rate vs memory | Reduce wasted reads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Per-write fsync for no-loss single-node durability | You can rely on replication and want throughput |
| Leveled compaction for read-heavy/overwrite-heavy | Heavy ingestion on limited SSD endurance |
| Size-tiered for write-heavy append workloads | You can't spare 50% free space |
| Time-window compaction for TTL data | Data is updated across windows |
| More memtable memory | Heap pressure and long GC/recovery times |

### 9. Common mistakes
- Disabling the WAL "for speed" → data loss on crash → keep it, or replicate and accept the window knowingly.
- Mass deletes then reading ranges → tombstone scans, timeouts → TTL or partition drops.
- Running repair less often than `gc_grace_seconds` → zombie data → schedule repair within the window.
- Provisioning disks for ingest only → compaction stalls → budget for amplification and headroom.
- Triggering major compaction by habit → huge SSTable that never compacts again (size-tiered) → avoid unless necessary.
- Ignoring long-running scans/snapshots → space not reclaimed → bound query duration.

### 10. 🎯 Interview kit
**30-second answer:** A write goes to the WAL for durability and into the sorted in-memory memtable; when full it is flushed as an immutable SSTable. Reads check memtable and SSTables newest-first using Bloom filters; compaction merges files, keeping the newest version and purging tombstones. It makes writes sequential at the price of read and write amplification.

**Structure a longer answer:** write path → read path → compaction and tombstones → amplification trade-offs → failure modes (stalls, resurrection) → tuning.

**Likely follow-ups:**
- *What if the node crashes before a flush?* → Replay the WAL into a new memtable.
- *Walk a key written, deleted and rewritten through compaction.* → Newest sequence number wins; tombstone dropped only after covering older data and grace period.
- *Leveled vs size-tiered?* → Read/space amp vs write amp.
- *Why can a replica show stale data right after a write?* → Asynchronous replication/apply lag; local durability ≠ replica visibility.

**Signals interviewers look for:** you mention WAL, Bloom filters and tombstones, quantify amplification, and describe operational failure modes.

### 11. Check yourself
- [ ] Draw the LSM write and read paths including the WAL and Bloom filters.
- [ ] Explain why tombstones can't be dropped immediately and what `gc_grace_seconds` protects.
- [ ] Calculate approximate write amplification for leveled compaction with ratio 10 and 5 levels.
- [ ] Explain what a write stall is and how to prevent it.
- [ ] Describe crash recovery for an LSM engine.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Intermediate] [A user reports stale reads right after a write on your primary-replica MySQL setup. Explain the chain from redo/binlog flush to replica apply lag, and how read-your-writes could be violated even with sync=1 on the primary.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20500_A_user_reports_stale_reads_right_after_a_write_on_your_prima.md>)
- [Deep] [Explain how compaction merges overlapping SSTables and resolves duplicate keys by sequence number. Walk through what happens to a key that was written, deleted, then written again as it flows through compaction levels.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20495_Explain_how_compaction_merges_overlapping_SSTables_and_resol.md>)
- [Deep] [You need predictable read latency but your LSM's compaction causes periodic p99 jumps. Would you switch to a B-Tree engine, tune compaction, or add a caching layer? Reason through the trade-offs and how you'd measure success.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20532_You_need_predictable_read_latency_but_your_LSM_s_compaction_.md>)
- [Deep] [Explain how an LSM handles the read-your-own-writes guarantee within a single session. Why is it trivially satisfied by checking the memtable first, and how does this change with a follower/replica read?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20562_Explain_how_an_LSM_handles_the_read-your-own-writes_guarante.md>)
- [Expert] [Your RocksDB compaction is consuming all disk bandwidth and starving foreground reads/writes. How do you rate-limit compaction, and what's the trade-off between throttling compaction and letting read amplification grow?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20490_Your_RocksDB_compaction_is_consuming_all_disk_bandwidth_and_.md>)
- [Expert] [Explain the ARIES recovery algorithm's three phases (analysis, redo, undo) and why WAL must obey write-ahead and force-log-at-commit rules. Where does an LSM diverge from ARIES since it never updates pages in place?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20518_Explain_the_ARIES_recovery_algorithm_s_three_phases_analysis.md>)
- [Expert] [Explain how you'd debug an LSM store where disk usage suddenly doubled overnight with no traffic change. Consider stuck compaction, snapshot pinning old SSTables, and WAL retention. Give the order you'd check them.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20544_Explain_how_you_d_debug_an_LSM_store_where_disk_usage_sudden.md>)
- [Expert] [An interviewer probes: "you picked LSM for write throughput — what breaks at 100x scale that doesn't break at 10x?" Reason through where compaction, bloom-filter memory, and open-file/SSTable counts hit non-linear cliffs, and how you'd architect around each.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20600_An_interviewer_probes_you_picked_LSM_for_write_throughput_wh.md>)

## Related concepts

- [ACID Durability and Consistency](<../../../Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>) *(ACID Transactions)*
- [Batching Writes](<../../DBMS_Concepts/Batching_Writes.md>) *(DBMS Concepts)*
- [Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*
- [Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*
- [Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — literals. DevCenter provides command completion as you type out CQL commands, and interprets the
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — cassandra@cqlsh GRANT MODIFY ON KEYSPACE reservation TO reservationservice; This allows your
- *Database Internals* — Alex Petrov — Implementing B-Trees › Page Defragmentation
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Log Semantics
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Operation Versus Data Log
- *Database Internals* — Alex Petrov — Log-Structured Storage › Chapter 7. Log-Structured Storage
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Vacuum
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Making B-trees reliable
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Write-ahead log (WAL) shipping
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Using logs for message storage
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioned Logs › Disk space usage
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Commits, Redo, and Journaling
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Memory allocation and fragmentation
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Logs and Databases
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Write-ahead logs
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ordering › Pattern: Bin Pack Orderer

