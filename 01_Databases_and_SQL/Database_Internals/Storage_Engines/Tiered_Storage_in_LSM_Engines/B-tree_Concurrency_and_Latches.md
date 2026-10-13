---
id: "01_Databases_and_SQL/Database_Internals/Storage_Engines/B-tree_Concurrency_and_Latches"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Internals/Storage_Engines"
tools: ["MySQL", "PostgreSQL"]
---

# B-tree Concurrency and Latches

**L5 Expert** · [↑ Storage Engines](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A B-tree is shared by many threads, so each page is protected by short-lived latches (physical, in-memory) using protocols like latch coupling and B-link siblings, while transactions are protected separately by logical locks and MVCC.

### 1. The big picture
A B-tree is a balanced tree of fixed-size pages (4–16 KB). Lookups descend from the root through internal nodes to a leaf; inserts and deletes modify leaf pages and sometimes **split** or **merge** pages, propagating changes upward. All of this happens with hundreds of threads in the same buffer pool at once.

Two different kinds of concurrency control are layered:

- **Locks** (and MVCC) protect *logical* data — rows and keys — for the duration of a *transaction*, to give isolation.
- **Latches** protect *physical* structures — a page's bytes, the tree's shape — for the duration of a *single operation* (microseconds), to prevent a reader seeing a half-modified page.

Mixing these up is the most common conceptual error. This lesson explains how latching keeps the tree structurally consistent, why naive approaches collapse at the root, and the protocols (latch coupling, optimistic descent, B-link trees) that real engines use. It builds on B-tree basics (fanout, fences, slotted pages) and leads to the B-tree vs LSM comparison.

### 2. Core concepts and vocabulary
- **Latch** — lightweight, short-term synchronisation primitive on an in-memory structure (a buffer-pool page, hash bucket). Shared (S) or exclusive (X); no deadlock detection, no queueing fairness guarantees beyond the primitive; held microseconds.
- **Lock** — logical, transaction-duration protection of rows/keys/ranges with deadlock detection.
- **Latch coupling (crabbing)** — acquire the child's latch before releasing the parent's during descent.
- **Safe node** — a node that will not split (has room) on insert or not merge (above minimum) on delete; once the child is safe, ancestor latches can be released.
- **Optimistic descent** — latch with S while descending, take X only on the leaf; restart pessimistically if the leaf turns out to be unsafe.
- **Right-link / B-link tree** — each node has a pointer to its right sibling and a *high key*, allowing readers to recover when a concurrent split moved the key they wanted.
- **Page split / merge** — splitting a full page into two and posting a separator key to the parent; merging underfull siblings.
- **Fence (separator) key** — key in an internal node dividing child ranges.
- **Buffer pool / page pinning** — pages are cached; a pinned page cannot be evicted while in use.
- **MVCC** — keeps multiple row versions so readers don't take read locks.

### 3. How it works, step by step
**Naive approach.** One latch on the whole tree: correct but serial. A single latch per page but holding all latches along the path (root to leaf) is correct but the root becomes a permanent bottleneck.

**Latch coupling for reads (S-crabbing):**
1. S-latch the root.
2. Find the child pointer; S-latch the child.
3. Release the parent's latch.
4. Repeat until the leaf; read; release.
At every instant a reader holds at most two latches, and the structure it is traversing cannot change underneath it.

**Pessimistic writes (X-crabbing for insert):**
1. X-latch the root.
2. Descend: X-latch the child. If the child is **safe** (not full), release all ancestor latches. Otherwise keep them, because a split may need to modify them.
3. At the leaf, insert. If the leaf is full: split it, push the median key to the parent (still X-latched), and so on up to the highest unsafe ancestor.
4. Release everything.

**Optimistic variant (the common default):**
1. Descend with S-latches (coupling) to the leaf's parent.
2. X-latch the leaf only.
3. If the leaf is safe, insert and done (the overwhelmingly common case).
4. Otherwise release and restart with the pessimistic X-crabbing path.
Since most inserts touch a leaf with free space, contention on upper levels is almost nil.

**B-link (Lehman–Yao) approach:**
1. Every node stores a pointer to its right sibling and a high key (upper bound of keys in the node).
2. A split is done in two atomic page-level steps: (a) create the new right page and link it, (b) later post the separator to the parent.
3. A reader that arrives at a node after a split checks `key > high_key`; if so, it moves right via the sibling pointer instead of failing.
4. Writers therefore hold only one latch at a time (plus brief parent updates), and readers never block structure modifications.

### 4. Worked example
Trace of two concurrent inserts into a 3-level tree under optimistic coupling.

State: root R → internal node I → leaves L1 [10,20,30] (full, capacity 3) and L2 [50,60].
- T1 inserts 25; T2 inserts 55.
- T1: S(R) → S(I) → release R → X(L1). L1 is full → *unsafe*. T1 releases and restarts pessimistically: X(R), X(I) — I has room, so it is safe for the split; release R; X(L1). Split L1 into [10,20] and [25,30]; post separator 25 to I; release all.
- T2 (concurrently): S(R) → S(I) → X(L2). L2 has room → insert 55 → release. T2 was only briefly blocked while T1 held X(I).

Observing contention in practice (SQL Server):
```sql
-- latch waits by class; high PAGELATCH_EX on one page = hot spot (e.g. ever-increasing key)
SELECT TOP 10 wait_type, waiting_tasks_count, wait_time_ms
FROM   sys.dm_os_wait_stats
WHERE  wait_type LIKE 'PAGELATCH%' OR wait_type LIKE 'LATCH%'
ORDER  BY wait_time_ms DESC;

-- which pages are hot right now
SELECT session_id, wait_type, wait_resource
FROM   sys.dm_exec_requests
WHERE  wait_type LIKE 'PAGELATCH%';
```
`PAGELATCH_*` means waiting on an in-memory page latch (a hot page), `PAGEIOLATCH_*` means waiting for the page to be read from disk — different problems. The last-page insert contention of an identity/clustered key is the textbook case for `PAGELATCH_EX`.

### 5. Under the hood
- **Latch implementation.** Typically a reader–writer spinlock or a futex-backed lock embedded in the buffer-pool frame header; shared mode is a counter. Reader-writer latches themselves can bottleneck on cache-line bouncing at the root even when only taking S — hence optimistic lock coupling (version counter per page: read the version, read the page, re-check the version; no writes to shared memory on the read path).
- **Why the root is special.** Every operation passes through the root; an X latch there serialises the whole tree. Safe-node release and optimistic descent exist to avoid ever taking X on the root except on a root split.
- **Split mechanics.** A leaf split allocates a new page, moves half the cells, writes both pages and updates the parent. Crash safety requires WAL: the split is logged as a *system transaction* (nested top action) that is redone/undone atomically, independent of the user transaction that triggered it.
- **Merge/underflow.** Deletes may leave pages underfull; many engines delay or skip merging (SQL Server leaves ghost records and merges lazily; Postgres vacuum removes dead tuples and index cleanup happens in bulk).
- **MVCC and B-trees.** InnoDB stores old versions in undo logs and keeps the B-tree storing only the latest version plus a roll pointer; Postgres keeps old tuple versions in the heap and index entries point to all of them, so `COUNT(*)` must check visibility for every candidate tuple — one reason exact counts on large tables are expensive.
- **Range scans.** Leaf pages are linked (B+-tree); a scan latch-couples along the leaf chain. Backward scans and concurrent splits complicate sibling latching and can require deadlock-avoidance ordering (always latch left to right).
- **Logical vs latch deadlock.** Locks have deadlock detection; latches rely on strict acquisition order (top-down, left-to-right) for deadlock freedom.

### 6. Performance and design implications
- **Hot right edge.** Monotonic keys (identity, timestamps) concentrate inserts on the last leaf: `PAGELATCH_EX` contention at thousands of inserts per second. Mitigations: hash/prefix the key, `OPTIMIZE_FOR_SEQUENTIAL_KEY` (SQL Server 2019+), partitioning, or accept it as good locality for sequential I/O.
- **Page splits are write amplification.** Random keys (GUIDs) cause frequent mid-tree splits, fragmented pages and ~50–70% fill; mitigations: sequential GUIDs, fill factor, periodic rebuild.
- **Fanout matters.** Long keys reduce fanout, increase height and the number of latches per lookup; keep keys compact (surrogate integer keys, included columns in leaf only).
- **Read-heavy workloads** scale with latch-free/optimistic reads; **write-heavy** hot spots need key design.
- **Measure:** latch wait stats, page split counters (`Page Splits/sec`), `sys.dm_db_index_operational_stats` (`page_latch_wait_count`, `leaf_allocation_count`), fragmentation DMVs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_os_wait_stats` / `sys.dm_os_latch_stats` | Aggregate latch waits | Triage latch contention |
| `sys.dm_db_index_operational_stats` | Per-index latch/lock waits, splits | Locate the hot index |
| `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` | Reduces last-page contention | Identity-keyed high insert rate |
| `FILLFACTOR` | Leaves free space in leaf pages | Random-key inserts, fewer splits |
| `ALTER INDEX … REBUILD/REORGANIZE` | Defragment, restore fill | After heavy splitting |
| `pg_stat_user_indexes`, `pgstattuple`, `pg_stat_activity.wait_event` | Postgres index health/waits | Postgres tuning |
| `SHOW ENGINE INNODB STATUS` | Mutex/RW-lock waits (SEMAPHORES) | InnoDB latch contention |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Sequential keys for write locality and low fragmentation | Very high concurrent insert rate to one table |
| Random keys (UUIDs) to spread insert contention | Large indexes; page splits and cache misses |
| Lower fill factor for churning indexes | Read-mostly indexes (wastes space and I/O) |
| Optimistic latching | Contention on the same leaf is frequent (restarts) |
| Optimistic concurrency (version column) for app-level conflicts | Highly contended rows: retries explode; use pessimistic `FOR UPDATE` |

### 9. Common mistakes
- Treating latch waits as lock blocking → chasing the wrong cause → separate `LCK_M_*` from `PAGELATCH_*`.
- Using random GUIDs as clustered keys → splits and fragmentation → sequential GUIDs or surrogate ints.
- Rebuilding indexes on a schedule without measuring → wasted I/O → rebuild by measured fragmentation and page density.
- Believing an index makes `COUNT(*)` cheap → still scans leaves and checks visibility → maintain a counter or use estimates.
- Assuming latches obey isolation levels → they don't; isolation is logical, latches are physical.

### 10. 🎯 Interview kit
**30-second answer:** Latches protect the physical structure of pages for microseconds; locks protect logical data for the transaction. In a B-tree, threads descend with latch coupling — hold the parent until the child is latched, release ancestors once the child is safe from splitting — or use optimistic descent that latches only the leaf exclusively. B-link trees add right-sibling pointers and high keys so splits don't block readers.

**Structure a longer answer:** distinguish lock vs latch → why the root is a bottleneck → crabbing, safe nodes, optimistic variant → split handling (B-link, WAL system transaction) → hot-spot examples and mitigations.

**Likely follow-ups:**
- *Why not hold the root latch for the whole operation?* → Serialises everything.
- *What is a safe node?* → Won't split/merge, so ancestors can be released.
- *How does MVCC interplay?* → Readers use versions instead of S locks; latches still guard pages.
- *Optimistic vs pessimistic concurrency at row level?* → Version check at commit vs `SELECT FOR UPDATE`; optimistic for low contention.

**Signals interviewers look for:** you keep latch/lock separate, can draw crabbing, and tie it to real symptoms (PAGELATCH hot spots, split storms).

### 11. Check yourself
- [ ] State three differences between a latch and a lock.
- [ ] Walk a concurrent insert that causes a leaf split with latch coupling.
- [ ] Explain what a B-link right pointer and high key allow.
- [ ] Diagnose `PAGELATCH_EX` waits on an identity-keyed insert workload and propose two fixes.
- [ ] Explain why `COUNT(*)` on a large MVCC B-tree table is expensive.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../../TOOLS/MySQL/_index.md>)
- [PostgreSQL](<../../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Intermediate] [Your team is choosing between InnoDB and RocksDB (MyRocks) for a new metadata store. The workload is 80% reads with a hot small key range and strong latency SLOs. Argue which engine you'd pick and what would flip your decision.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20469_Your_team_is_choosing_between_InnoDB_and_RocksDB_MyRocks_for.md>)
- [Intermediate] [A junior engineer proposes using SELECT COUNT(*) on a huge InnoDB table for a dashboard. Explain why this is a full B-Tree scan, how MVCC makes an exact count expensive, and what mid-level alternatives you'd mentor them toward.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20570_A_junior_engineer_proposes_using_SELECT_COUNT_on_a_huge_Inno.md>)
- [Deep] [Your service reads then writes back a value under contention causing lost updates. Explain how optimistic concurrency with a version column versus SELECT FOR UPDATE (pessimistic) behaves differently at the B-Tree/lock level and which fits high contention.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20553_Your_service_reads_then_writes_back_a_value_under_contention.md>)
- [Deep] [A team benchmarked RocksDB at 3x their InnoDB write throughput and wants to migrate immediately. As the reviewing architect, list the ways a synthetic benchmark misleads at production scale (compaction debt, cache warmup, real key distribution) and what you'd require before approving.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20608_A_team_benchmarked_RocksDB_at_3x_their_InnoDB_write_throughp.md>)
- [Expert] [You see InnoDB history list length growing into millions during a nightly batch. Explain the root cause, the production symptoms it will produce, and how you'd remediate without killing the batch outright.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20478_You_see_InnoDB_history_list_length_growing_into_millions_dur.md>)
- [Expert] [Explain how MVCC garbage / vacuum works in Postgres and why heap bloat happens. A table's disk usage keeps growing despite steady row count — walk through diagnosing whether autovacuum is falling behind and how to fix it.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20503_Explain_how_MVCC_garbage_vacuum_works_in_Postgres_and_why_he.md>)
- [Expert] [Your team wants to switch a MySQL table from InnoDB to MyRocks to save disk. Walk through the migration risks: online DDL, secondary index behavior, transaction isolation differences, and how you'd validate parity before cutover.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20519_Your_team_wants_to_switch_a_MySQL_table_from_InnoDB_to_MyRoc.md>)
- [Expert] [You're seeing lock contention and deadlocks on a hot B-Tree page under high concurrency. Explain the difference between row locks and page latches, how gap locks in InnoDB REPEATABLE READ contribute, and how you'd reduce it.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Internals/Storage_Engines/Q-20537_You_re_seeing_lock_contention_and_deadlocks_on_a_hot_B-Tree_.md>)

## Related concepts

- [MySQL Performance Diagnosis](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis.md>) *(MySQL Platform)*
- [Batching Writes](<../../DBMS_Concepts/Batching_Writes.md>) *(DBMS Concepts)*
- [Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*

## References

- *Database Internals* — Alex Petrov — B-Tree Basics › Chapter 2. B-Tree Basics
- *Database Internals* — Alex Petrov — Implementing B-Trees › Vacuum and Maintenance
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Locking Pages in Cache
- *Database Internals* — Alex Petrov — B-Tree Variants › Copy-on-Write
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — G › I
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — M
- *Designing Data-Intensive Applications* — Martin Kleppmann — M
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.5 #47 Allowing logs to become fragmented
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page IO Latch › Page I/O Latch
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — T › V

