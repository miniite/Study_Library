---
id: "TOOLS/SQL_Server/05_Concurrency/Latches"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Latches and Latch Contention

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Latches are short, lightweight locks that protect in-memory structures (mainly buffer pages) while a thread touches them. `PAGEIOLATCH_*` waits mean "waiting for the page to come from disk," `PAGELATCH_*` waits mean "too many threads fighting over the same page in memory."

### 1. The big picture
Locks protect *logical* data for the duration of a transaction and follow isolation rules. **Latches** protect *physical* structures for microseconds: a page in the buffer pool must not be read while another thread is halfway through changing it, or while it is still being read in from disk. Latches aren't controlled by isolation levels or hints, and they don't show up in `sys.dm_tran_locks`. `NOLOCK` doesn't avoid them.

Latch waits matter in two situations:
- **I/O-bound workloads** → lots of `PAGEIOLATCH_SH/EX`: pages aren't in memory and storage is slow, or queries read too much.
- **High-concurrency inserts or tempdb churn** → `PAGELATCH_EX/SH` on a few hot pages: the last page of an ever-increasing index, or tempdb allocation pages (PFS, GAM, SGAM).

This lesson sits beside the lock lessons in the concurrency block. It's the "physical" counterpart to logical locking.

### 2. Core concepts and vocabulary
- **Latch**: a short-term synchronization object protecting an in-memory structure. Held only while the structure is being accessed.
- **Buffer latch**: protects a page (buffer) in the buffer pool. The two families are:
  - **`PAGEIOLATCH_<mode>`**: waiting for a page that is being read from (or written to) disk. It's an I/O wait.
  - **`PAGELATCH_<mode>`**: waiting for a page that is *already in memory* but latched by another thread. It's contention.
- **Non-buffer latch** (`LATCH_<mode>`): protects other internal structures. Look in `sys.dm_os_latch_stats` for the class (e.g. `ACCESS_METHODS_DATASET_PARENT` for parallel scans, `FGCB_ADD_REMOVE` for file growth).
- **Latch modes**: `KP` (keep), `SH` (shared, reading), `UP` (update), `EX` (exclusive, modifying), `DT` (destroy).
- **Last-page insert contention**: many sessions inserting into a clustered index on an identity/sequence/`GETDATE()` key all need EX on the same last leaf page.
- **Allocation pages**: PFS (page free space, one per 8,088 pages), GAM and SGAM (extent allocation maps, one per ~4 GB). These are hot in tempdb when many sessions create and drop temp objects.
- **IOCOMPLETION / WRITELOG**: related I/O waits for non-data-page I/O and for log flushes.

### 3. How it works, step by step
**Reading a page that isn't in memory**
1. A thread needs page 1:5000. It looks it up in the buffer pool hash table and doesn't find it.
2. It allocates a buffer, puts an **EX latch** on it, and issues an asynchronous read.
3. Any thread (including itself) that wants to *read* the page requests an **SH latch** and waits: **`PAGEIOLATCH_SH`**.
4. When the I/O completes, the EX latch is released, waiters get SH, and they read the page.
→ High `PAGEIOLATCH_SH` average wait (e.g. > 10–20 ms) means slow storage or excessive physical reads.

**Modifying a page that is in memory**
1. A thread wants to insert a row on page 1:9000 (already cached). It takes an **EX latch** on the buffer.
2. Other threads wanting SH or EX on that page wait: **`PAGELATCH_SH/EX`**.
3. The thread changes the row, writes the log record, marks the page dirty, and releases the latch (microseconds).
→ If hundreds of threads target the *same* page, they queue on that latch, and throughput flattens no matter how many CPUs you add.

**Latch vs lock**

| | Lock | Latch |
|---|---|---|
| Protects | logical data (rows, keys, tables) | physical in-memory structures (pages, internal objects) |
| Duration | until statement/transaction end (by isolation) | while the structure is touched (µs–ms) |
| Controlled by | isolation level, hints | engine only |
| Deadlock detection | yes | no (designed to be deadlock-free by ordering) |
| Visible in | `sys.dm_tran_locks`, `LCK_M_*` | `sys.dm_os_latch_stats`, `PAGELATCH_*`, `PAGEIOLATCH_*` |

### 4. Worked example
```sql
-- 1) Are latches a significant share of waits?
SELECT wait_type, waiting_tasks_count, wait_time_ms,
       wait_time_ms * 1.0 / NULLIF(waiting_tasks_count,0) AS avg_ms
FROM sys.dm_os_wait_stats
WHERE wait_type LIKE 'PAGEIOLATCH%' OR wait_type LIKE 'PAGELATCH%' OR wait_type LIKE 'LATCH%'
ORDER BY wait_time_ms DESC;

-- 2) Which page are threads fighting over right now?
SELECT wt.session_id, wt.wait_type, wt.wait_duration_ms, wt.resource_description  -- e.g. 2:1:1 (tempdb PFS), 7:1:123456
FROM sys.dm_os_waiting_tasks AS wt
WHERE wt.wait_type LIKE 'PAGELATCH%';

-- 3) Identify the object behind a page (SQL Server 2019+)
SELECT OBJECT_NAME(pi.object_id, pi.database_id) AS obj, pi.index_id, pi.page_type_desc
FROM sys.dm_db_page_info(7, 1, 123456, 'DETAILED') AS pi;
-- obj = OrderEvents, index_id = 1, DATA_PAGE  → last-page insert contention on the clustered PK

-- 4) Fix options for last-page insert contention
ALTER INDEX PK_OrderEvents ON dbo.OrderEvents
SET (OPTIMIZE_FOR_SEQUENTIAL_KEY = ON);   -- SQL 2019+: controls the rate of threads entering
```
Interpretation guide:
- `resource_description` `2:1:1`, `2:1:2`, `2:1:3` (tempdb PFS/GAM/SGAM) → **tempdb allocation contention** → add tempdb data files (equal size), use 2019+ in-memory tempdb metadata.
- A user-DB page that is the rightmost leaf of a clustered index → **last-page insert contention**.
- High `PAGEIOLATCH_SH` with high average ms → **I/O latency or too many physical reads** → check `sys.dm_io_virtual_file_stats`, reduce reads with indexes and query fixes, add memory.

### 5. Under the hood
- **Latch implementation.** Each buffer header has a latch. Threads spin briefly, then wait. Superlatches (sub-latches per CPU) are promoted automatically for very hot read-only pages, so many SH requests don't contend.
- **`OPTIMIZE_FOR_SEQUENTIAL_KEY`.** This doesn't remove the hot page. It manages the *convoy*: it limits how fast threads can request the latch and favors threads likely to finish quickly. That improves throughput by roughly 2× in heavy cases. The book's point: it suits ever-increasing keys under very high insert concurrency, and adds slight overhead otherwise.
- **Other designs for last-page contention:** a non-sequential leading key (e.g. hash-partition column + identity), a random GUID key (spreads inserts but fragments and enlarges the index), hash partitioning on a computed modulo column, or memory-optimized tables (latch-free).
- **Tempdb.** Multiple data files (one per core up to 8, then add in groups of 4 if contention remains) spread PFS/GAM/SGAM pages. SQL Server 2016+ allocates uniform extents by default (no SGAM mixed-extent hotspot). 2019+ offers **memory-optimized tempdb metadata** to remove system-table latch contention. 2022 adds concurrent GAM/SGAM updates.
- **Parallel scan latch** `ACCESS_METHODS_DATASET_PARENT` shows up with heavily parallel scans. Its root cause is usually a large scan plus high DOP.

### 6. Performance and design implications
- **PAGEIOLATCH is a symptom of I/O demand.** First reduce reads (indexes, SARGable queries, covering), then check storage latency (> 20 ms for data reads is usually slow), then memory (Page Life Expectancy, buffer pool size).
- **PAGELATCH contention gets worse as concurrency rises.** It shows up only at scale, in load tests or peak traffic.
- Choose clustering keys with insert concurrency in mind. An identity key is fine for most systems, but not for thousands of inserts per second from many threads.
- Size and configure tempdb correctly from day one (multiple equal files, fast storage).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_os_wait_stats` | cumulative waits including latch types | baseline, triage |
| `sys.dm_os_waiting_tasks` (`resource_description`) | current latch waits with db:file:page | live hot-page hunting |
| `sys.dm_os_latch_stats` | non-buffer latch classes | `LATCH_*` investigation |
| `sys.dm_db_page_info()` (2019+) / `DBCC PAGE` | maps page id to object and page type | identify the hot object |
| `sys.dm_io_virtual_file_stats` | I/O latency per file | PAGEIOLATCH diagnosis |
| `ALTER INDEX … SET (OPTIMIZE_FOR_SEQUENTIAL_KEY = ON)` | mitigates last-page insert convoys | high-rate identity inserts |
| `ALTER SERVER CONFIGURATION SET MEMORY_OPTIMIZED TEMPDB_METADATA = ON` | latch-free tempdb system tables (restart needed) | temp-object-heavy workloads |
| Additional tempdb files (`ALTER DATABASE tempdb ADD FILE`) | spreads allocation pages | PFS/GAM/SGAM contention |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `OPTIMIZE_FOR_SEQUENTIAL_KEY`: proven last-page contention, SQL 2019+ | low concurrency (slight overhead, no gain) |
| Random/GUID keys to spread inserts | range scans by key, index size, fragmentation |
| Hash-partitioned key | queries that can't eliminate partitions |
| Memory-optimized tables | large data, feature limitations |
| More tempdb files | uneven file sizes (proportional fill breaks) |
| More memory for PAGEIOLATCH | queries that scan needlessly (fix those first) |

### 9. Common mistakes
- Treating `PAGEIOLATCH` as a locking problem → wrong fixes (isolation changes) → it's I/O: reduce reads, check storage.
- Using `NOLOCK` to avoid latch waits → no effect → latches are always taken.
- One tempdb data file on a busy server → PFS/GAM contention → multiple equally sized files.
- Switching to GUID keys to cure contention that isn't there → bigger, fragmented indexes → measure `PAGELATCH_EX` on the last page first.
- Reading only total wait time → missing per-wait average latency → compute `wait_time_ms / waiting_tasks_count`.

### 10. 🎯 Interview kit
**30-second answer:** Latches are lightweight, short-term locks on in-memory structures, mainly buffer pages, held only while a thread touches the structure, regardless of isolation level. `PAGEIOLATCH` waits mean threads are waiting for a page to be read from disk, so it's an I/O and memory issue that I fix by reducing reads, checking storage latency and memory. `PAGELATCH` waits mean contention on a page already in memory: typically last-page inserts on an identity clustered key, or tempdb allocation pages. I fix those with `OPTIMIZE_FOR_SEQUENTIAL_KEY`, key design, more tempdb files, or memory-optimized tempdb metadata.
**Structure a longer answer:** latch vs lock → the two families → how to find the page (`waiting_tasks`, `page_info`) → patterns and fixes → version-specific features.
**Likely follow-ups:** *Does NOLOCK avoid latches?* → no. · *What's in `resource_description` 2:1:1?* → tempdb file 1, page 1 = PFS. · *Why doesn't adding CPUs help last-page contention?* → every thread serializes on one page latch.
**Signals interviewers look for:** clear lock/latch distinction, IO vs in-memory latch, knowing tempdb and sequential-key remedies.

### 11. Check yourself
- [ ] Explain the difference between `PAGEIOLATCH_SH` and `PAGELATCH_EX` in one sentence each.
- [ ] Find the current hot page with `sys.dm_os_waiting_tasks` and map it to an object.
- [ ] Describe last-page insert contention and two fixes.
- [ ] List the tempdb settings that reduce allocation contention.
- [ ] Compute average wait per latch wait type from `sys.dm_os_wait_stats`.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page IO Latch › Page I/O Latch
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Latch Contention
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Latch
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — CXPACKET › IOCOMPLETION

