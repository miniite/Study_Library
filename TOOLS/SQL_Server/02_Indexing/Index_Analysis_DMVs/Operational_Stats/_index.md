---
id: "TOOLS/SQL_Server/02_Indexing/Index_Analysis_DMVs/Operational_Stats"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Index Operational Stats

**L5 Expert** · [↑ Index Analysis with DMVs](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `sys.dm_db_index_operational_stats` counts what *physically* happened inside each index partition: inserts, updates, deletes and ghosts at leaf and non-leaf level, page splits, range scans vs singleton lookups, forwarded fetches, LOB reads, and lock/latch waits. It is the DMV that explains *why* an index is expensive.

### 1. The big picture
The parent lesson introduced the three index-analysis DMV families. `sys.dm_db_index_usage_stats` says *how often* queries used an index (seeks, scans, lookups, updates), counted once per plan operator execution. `sys.dm_db_index_physical_stats` says what the index *looks like* on disk (fragmentation, page density, depth). The **missing-index DMVs** say what the optimizer *wished* existed.

`sys.dm_db_index_operational_stats` fills the gap between them: **what work the storage engine actually did**, row by row and page by page, in each index partition. Usage stats might say an index had 10 scans. Operational stats tell you those scans were range scans over 50 million rows, that 3,000 page splits happened at the leaf, that forwarded records cost 2 million extra reads, or that row-lock waits added up to 40 minutes. It is the DMV for **diagnosing contention and write overhead** at index level. Pair it with Performance Monitor counters (Access Methods, Buffer Manager) to see the same activity server-wide.

### 2. Core concepts and vocabulary
- **Function signature**: `sys.dm_db_index_operational_stats(database_id, object_id, index_id, partition_number)`. Pass NULLs for "all". It returns one row per index *partition*.
- **Leaf vs non-leaf counters**: `leaf_*` counts activity at the leaf level (data rows, or index leaf rows). `nonleaf_*` counts activity in upper levels, usually caused by page splits propagating upward.
- **Ghost record**: a deleted row left in place and flagged, to be cleaned up later by the ghost cleanup task (`leaf_ghost_count`).
- **leaf_allocation_count**: page allocations at the leaf. For a B-tree index, this is essentially **page splits**, including "good" splits at the end of an ever-increasing key.
- **range_scan_count**: number of range scans or full scans started on the index (including heap scans).
- **singleton_lookup_count**: single-row fetches, mostly key or RID lookups and seeks for one row.
- **forwarded_fetch_count**: reads that followed a forwarding pointer in a heap. Each one is an extra page read.
- **lob_fetch_in_pages / row_overflow_fetch_in_pages**: pages read for LOB and row-overflow data.
- **row_lock_count / row_lock_wait_count / row_lock_wait_in_ms**: row locks requested, how many had to wait, and the total wait time. The same exists for `page_lock_*`.
- **index_lock_promotion_attempt_count / index_lock_promotion_count**: lock escalation attempts and successes.
- **page_latch_wait_count / page_latch_wait_in_ms**: in-memory page latch contention (e.g. last-page insert hotspots).
- **page_io_latch_wait_count / page_io_latch_wait_in_ms**: waits for pages to be read from disk into memory.
- **Volatility**: the counters live in the metadata cache. They reset when the object's metadata is evicted from cache (e.g. memory pressure or schema changes), when the database goes offline, and at instance restart. They are cumulative and must be *sampled*.

### 3. How it works, step by step
1. **Snapshot.** Insert the DMV output, with a timestamp, into a table.
2. **Wait.** Let a representative workload run, e.g. one business hour or a batch window.
3. **Snapshot again**, and compute deltas per `(database_id, object_id, index_id, partition_number)`. If a row's counters went *down*, the counters were reset, so discard that pair.
4. **Interpret by category** (the book groups the counters this way):

| Category | Counters | Question it answers |
|---|---|---|
| DML activity | `leaf_insert/update/delete/ghost_count`, `nonleaf_*` | How much write work does this index absorb? |
| SELECT activity | `range_scan_count`, `singleton_lookup_count` | Is it scanned or used for lookups? Many lookups suggest a covering problem |
| Page splits | `leaf_allocation_count`, `nonleaf_allocation_count` | Is the key or fill factor causing splits? |
| Heap/LOB | `forwarded_fetch_count`, `lob_fetch_in_pages`, `row_overflow_fetch_in_pages` | Hidden I/O from forwarding or LOB reads |
| Locking | `row_lock_*`, `page_lock_*`, `index_lock_promotion_*` | Which index is the blocking hotspot? |
| Latching / I/O | `page_latch_*`, `page_io_latch_*`, `tree_page_*` | Insert hotspots, buffer-pool misses |

5. **Rank.** Sort by wait time (`row_lock_wait_in_ms + page_lock_wait_in_ms`, `page_latch_wait_in_ms`, `page_io_latch_wait_in_ms`) to find the indexes where time is lost. Sort by `leaf_allocation_count` to find split-heavy indexes.
6. **Act.** Change the key or fill factor (splits), add INCLUDE columns (lookups), rebuild the heap or add a clustered index (forwarding), change the isolation level or index for locking, or use `OPTIMIZE_FOR_SEQUENTIAL_KEY` (last-page latch contention, 2019+).

### 4. Worked example
```sql
-- Snapshot helper
IF OBJECT_ID('dbo.IOS_Snap') IS NULL
SELECT SYSDATETIME() AS snap_time, ios.*
INTO dbo.IOS_Snap
FROM sys.dm_db_index_operational_stats(DB_ID(), NULL, NULL, NULL) AS ios WHERE 1 = 0;

INSERT dbo.IOS_Snap
SELECT SYSDATETIME(), ios.* FROM sys.dm_db_index_operational_stats(DB_ID(), NULL, NULL, NULL) AS ios;
-- ... run workload, then take a second snapshot the same way ...

-- Delta report between the last two snapshots
WITH s AS (
  SELECT *, DENSE_RANK() OVER (ORDER BY snap_time DESC) AS rn FROM dbo.IOS_Snap)
SELECT OBJECT_NAME(n.object_id) AS tbl, i.name AS idx, n.partition_number,
       n.leaf_insert_count      - o.leaf_insert_count      AS inserts,
       n.leaf_allocation_count  - o.leaf_allocation_count  AS leaf_splits,
       n.range_scan_count       - o.range_scan_count       AS range_scans,
       n.singleton_lookup_count - o.singleton_lookup_count AS singleton_lookups,
       n.forwarded_fetch_count  - o.forwarded_fetch_count  AS forwarded_fetches,
       n.row_lock_wait_in_ms    - o.row_lock_wait_in_ms    AS row_lock_wait_ms,
       n.page_latch_wait_in_ms  - o.page_latch_wait_in_ms  AS page_latch_wait_ms,
       n.page_io_latch_wait_in_ms - o.page_io_latch_wait_in_ms AS io_wait_ms
FROM s AS n
JOIN s AS o ON o.rn = 2 AND n.rn = 1
           AND o.database_id = n.database_id AND o.object_id = n.object_id
           AND o.index_id = n.index_id AND o.partition_number = n.partition_number
JOIN sys.indexes AS i ON i.object_id = n.object_id AND i.index_id = n.index_id
WHERE n.leaf_insert_count >= o.leaf_insert_count         -- skip reset rows
ORDER BY row_lock_wait_ms + page_latch_wait_ms DESC;
```
How to read typical findings:
- **Clustered index with a large `singleton_lookups` delta** and an NC index with many range scans on the same table → queries seek the NC index and then do key lookups → cover them (see *Covering and Included Columns*).
- **`leaf_splits` high on an NC index keyed on a random GUID or a last-name column** → mid-page splits → lower the fill factor, or reconsider the key.
- **`page_latch_wait_ms` high on the clustered index of an identity-keyed table** with heavy concurrent inserts → last-page insert contention → `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON`, or a different key/partitioning strategy.
- **`forwarded_fetches` high on a heap (index_id 0)** → rows grew and moved → `ALTER TABLE … REBUILD`, or add a clustered index.
- **`row_lock_wait_ms` concentrated on one index** → the blocking hotspot. Read the queries touching it, and consider narrower transactions or RCSI.

### 5. Under the hood
- **Granularity.** Counters are incremented by the storage engine's access methods as operations happen, per HoBT (partition). They count *physical* events. A single `SELECT` that seeks once and returns 1,000 rows counts as one range scan, not 1,000.
- **Comparison with usage stats.** `user_seeks` in usage stats counts plan operator executions. `range_scan_count` and `singleton_lookup_count` count storage-engine operations. They differ in definition and in when they reset, so treat them as complementary rather than as precise cross-checks. Usage stats also reset on restart, and in many versions on index rebuild.
- **Ghost cleanup.** `leaf_ghost_count` grows with deletes. If ghosts accumulate faster than cleanup runs, scans read ghost rows. This is visible as a high ghost count in `sys.dm_db_index_physical_stats` (`ghost_record_count`).
- **Page splits vs allocations.** `leaf_allocation_count` includes end-of-index allocations from ascending keys (harmless). To isolate *bad* mid-page splits, use the Extended Events `sqlserver.transaction_log` event filtered on `LOP_DELETE_SPLIT`, or compare with fragmentation.
- **Performance counters** (server-wide, cheap, and continuous):
  - *SQLServer:Access Methods — Full Scans/sec*: unrestricted scans, often from missing indexes or small tables. Judge it against Index Searches/sec.
  - *Index Searches/sec*: seeks and range-scan starts.
  - *Page Splits/sec*: compare with Batch Requests/sec. A high ratio means split-heavy designs.
  - *Forwarded Records/sec*: heap forwarding cost.
  - *SQLServer:Buffer Manager — Page lookups/sec*: logical page reads per second. A sudden jump suggests a new scan-heavy plan.
  - *Page life expectancy / Page reads/sec*: buffer pool pressure behind `page_io_latch` waits.
  - Read them in `sys.dm_os_performance_counters` (per-second counters are cumulative there, so take two samples and divide by elapsed seconds).

### 6. Performance and design implications
- Use operational stats when an index is *known* to be important and you need to know *what is wrong with it*: contention, splits, lookups, forwarding.
- Always use deltas from a known interval. Lifetime totals mix last week's ETL with today's OLTP.
- Combine three views per suspect index: usage (is it used?), operational (what does it cost?), physical (what shape is it in?). Then look at the query plans that touch it.
- Collection is cheap. A scheduled snapshot every 15–60 minutes builds a valuable history.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_operational_stats(db, obj, idx, part)` | Physical activity per index partition | Contention, splits, lookups |
| `sys.dm_db_index_usage_stats` | Seeks/scans/lookups/updates per index | Is the index used? |
| `sys.dm_db_index_physical_stats(…, 'SAMPLED')` | Fragmentation, density, ghosts | Index shape |
| `sys.dm_os_performance_counters` | Perf counters inside SQL | Server-wide trends |
| `ALTER INDEX … SET (OPTIMIZE_FOR_SEQUENTIAL_KEY = ON)` | Mitigates last-page insert latch waits (2019+) | High `page_latch_wait` on ascending keys |
| `ALTER TABLE … REBUILD` | Removes heap forwarding | High `forwarded_fetch_count` |
| `ALTER INDEX … WITH (FILLFACTOR = n)` (on rebuild) | Leaves free space | Mid-page splits |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Diagnosing *why* an index is costly | Treating lifetime totals as current behaviour |
| Ranking indexes by lock/latch wait time | Data reset recently (restart, metadata eviction): sample longer |
| Combining with perf counters for trend context | Over-reading `leaf_allocation_count` as "bad splits" for ascending keys |

### 9. Common mistakes
- Reading cumulative values once → misleading totals → snapshot and compute deltas.
- Ignoring counter resets → negative deltas → filter or detect resets.
- Treating all leaf allocations as harmful splits → tuning a healthy identity index → check whether the key is ascending, and use fragmentation or XE for mid-page splits.
- Fixing lock waits by adding NOLOCK everywhere → wrong results → fix the index or query, shorten transactions, or use RCSI.
- Looking only at usage stats → an index that is "used" but drives massive lookups or contention → add operational stats to the review.

### 10. 🎯 Interview kit
**30-second answer:** "sys.dm_db_index_operational_stats shows the physical work per index partition: leaf and non-leaf inserts, updates and deletes, page allocations (splits), range scans vs singleton lookups, forwarded fetches, LOB reads, and row/page lock and latch waits. It's cumulative and volatile, so I snapshot it and compare deltas, then rank indexes by wait time or splits to find hotspots."
**Structure a longer answer:** where it fits among the index DMVs → counter categories → sampling method → three example diagnoses → corresponding perf counters.
**Likely follow-ups:** *Usage stats vs operational stats?* → logical plan usage vs physical storage-engine activity. *How do you find last-page contention?* → `page_latch_wait_in_ms` on the clustered index of an ascending key; fix with OPTIMIZE_FOR_SEQUENTIAL_KEY. *When do counters reset?* → restart, DB offline, metadata cache eviction.
**Signals interviewers look for:** delta-based thinking, mapping counters to concrete fixes, and knowing the reset behaviour.

### 11. Check yourself
- [ ] Build the snapshot table and delta query for one database.
- [ ] Map five counters to the problem each one reveals and its fix.
- [ ] Explain why `leaf_allocation_count` on an identity clustered index is not alarming.
- [ ] Use perf counters to compare Page Splits/sec with Batch Requests/sec.
- [ ] Diagnose a heap with high forwarded fetches and fix it.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — � D › � O
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DML Activity › SELECT Activity
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — LOB Access
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Singleton Lookup
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DML Activity
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Lookups per Second
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Locking Contention
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full Scans per Second
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance

