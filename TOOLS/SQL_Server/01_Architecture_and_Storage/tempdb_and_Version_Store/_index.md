---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/tempdb_and_Version_Store"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# tempdb and the Version Store

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** tempdb is the instance-wide scratch database — temp tables, table variables, sort/hash spills, worktables and the row version store all live there — so its configuration and monitoring affect every database on the server.

### 1. The big picture
Every SQL Server instance has exactly one tempdb, shared by all databases and sessions. It is recreated from `model` at each restart, so nothing in it survives a restart. You use it explicitly (temp tables `#t`, global temp tables `##t`, table variables) and the engine uses it implicitly: sorts and hash joins that don't fit their memory grant **spill** to tempdb, spools and cursors use **worktables**, online index rebuilds and `SORT_IN_TEMPDB` stage data there, and — crucially — **row versions** for snapshot isolation, read committed snapshot (RCSI), triggers, MARS and online index operations are kept in the **version store** inside tempdb.

Because it is shared, one bad workload (a huge spill, a forgotten open snapshot transaction) can fill tempdb or create contention that slows unrelated applications.

### 2. Core concepts and vocabulary
- **User objects** — local temp tables (`#name`, visible only to the creating session), global temp tables (`##name`, visible to all sessions), table variables (`@t`), temp stored procedures.
- **Internal objects** — worktables (spools, cursors, `UNION`, LOB variables), workfiles (hash spills), sort runs.
- **Spill** — when a sort or hash operator's memory grant is too small, it writes intermediate data to tempdb; shown as a warning triangle in the actual plan.
- **Version store** — copies of the previous versions of modified rows, linked by a 14-byte pointer added to the row; used by readers under snapshot/RCSI to see consistent data without shared locks.
- **Version store cleanup** — a background task removes versions no longer needed by any active snapshot transaction.
- **Allocation contention** — many sessions creating/dropping temp objects concurrently compete for latches on PFS, GAM and SGAM pages (`PAGELATCH_UP/EX` on `2:1:1`, `2:1:2`, `2:1:3`).
- **Temp table caching** — SQL Server can cache temp table metadata created inside stored procedures to reduce create/drop overhead.
- **Memory-optimized tempdb metadata** (2019+) — moves tempdb system tables into memory to remove metadata contention.
- **Persistent Version Store (PVS)** — with Accelerated Database Recovery, versions are stored in the *user* database instead of tempdb.

### 3. How it works, step by step
**Row versioning (snapshot / RCSI)**
1. Transaction T1 updates a row under a versioning-enabled database.
2. Before modifying, the engine copies the old row image into the **version store** in tempdb, stamped with the transaction sequence number (XSN) of the transaction that created that version.
3. The current row gets a 14-byte versioning tag pointing to the old version (forming a version chain).
4. A reader T2 using RCSI (statement-level) or SNAPSHOT (transaction-level) needs the data as of its start point. If the current row is too new, it follows the chain to the version that was committed before its start — no shared lock, no blocking by T1.
5. When no active transaction needs a version any more, the cleanup task removes it. A single long-running snapshot transaction prevents cleanup of everything created after it started, and the version store keeps growing.

**Temp table lifecycle**
1. `CREATE TABLE #myCustomers (...)` creates an object in tempdb with a session-unique name suffix. It can have primary keys, defaults, indexes and statistics like a normal table.
2. It's visible to the creating session (and procedures it calls).
3. It's dropped automatically when the session ends (or the creating procedure finishes), but it's good practice to `DROP` it explicitly.

**Table variables** (`DECLARE @t TABLE (...)`) also live in tempdb (not "only in memory"), are scoped to the batch, can have constraints/inline indexes, but have **no column statistics**; before SQL Server 2019 the optimizer assumed 1 row, from 2019 *deferred compilation* uses the actual row count at first execution.

**Spills**
1. Before execution the query gets a memory grant based on estimated rows and row size.
2. If actual rows exceed the estimate, the sort or hash operator writes runs/partitions to tempdb and reads them back — much slower than memory.
3. The actual plan shows a warning; Query Store and Extended Events record spill details.

### 4. Worked example
```sql
-- 1. What is using tempdb right now?
SELECT SUM(user_object_reserved_page_count)     * 8 / 1024.0 AS user_mb,
       SUM(internal_object_reserved_page_count) * 8 / 1024.0 AS internal_mb,
       SUM(version_store_reserved_page_count)   * 8 / 1024.0 AS version_store_mb,
       SUM(unallocated_extent_page_count)       * 8 / 1024.0 AS free_mb
FROM tempdb.sys.dm_db_file_space_usage;

-- 2. Which sessions are allocating it?
SELECT session_id,
       (user_objects_alloc_page_count + internal_objects_alloc_page_count) * 8 / 1024.0 AS alloc_mb
FROM sys.dm_db_session_space_usage
ORDER BY alloc_mb DESC;

-- 3. Who is holding the version store?
SELECT session_id, transaction_id, elapsed_time_seconds
FROM sys.dm_tran_active_snapshot_database_transactions
ORDER BY elapsed_time_seconds DESC;

-- 4. Allocation contention?
SELECT session_id, wait_type, resource_description
FROM sys.dm_os_waiting_tasks
WHERE wait_type LIKE 'PAGELATCH%' AND resource_description LIKE '2:%';

-- 5. Temp table vs table variable estimates
CREATE TABLE #T (Id INT PRIMARY KEY);  INSERT #T SELECT object_id FROM sys.all_objects;
DECLARE @T TABLE (Id INT PRIMARY KEY); INSERT @T SELECT object_id FROM sys.all_objects;
SELECT * FROM #T WHERE Id > 0;   -- estimate uses statistics
SELECT * FROM @T WHERE Id > 0;   -- no column statistics: cruder estimate
```
How to read it:
- Large `version_store_mb` plus an old transaction in query 3 → a long snapshot transaction (often an idle session with an open transaction or a long report) is pinning versions. End or fix that transaction.
- Large `internal_mb` → spills, worktables, or `SORT_IN_TEMPDB` rebuilds; find the sessions in query 2 and look at their plans for spill warnings.
- `PAGELATCH_UP` waits on `2:1:1` (PFS) or `2:1:3` (SGAM) → allocation contention; add equally sized tempdb data files.

### 5. Under the hood
- **Multiple data files.** Allocation round-robins across files using proportional fill; with several equally sized files, concurrent allocations hit different PFS/GAM/SGAM pages. Common guidance: as many files as logical cores up to 8, then add in groups of 4 only if contention persists. Keep sizes and growth identical (since 2016, autogrow of all tempdb files is uniform by default).
- **Uniform extents by default** in tempdb since 2016 (formerly trace flag 1118) — reduces SGAM contention.
- **Logging in tempdb** is lighter: only undo information is needed (tempdb is never recovered), so no redo logging — but the log still exists and can fill under huge transactions.
- **Versions in tempdb** are not logged for redo and are discarded at restart; with ADR the persistent version store moves to each user database and survives restarts.
- **Temp object caching.** Temp tables created in procedures can be cached and reused if they aren't altered after creation (no DDL like `CREATE INDEX` after creation — define indexes inline instead).
- **Storage placement.** The old rule "put data, log and tempdb on separate drives" only helps if the volumes are on different physical paths. On a SAN where all volumes share the same NIC, fabric and RAID array, separation gives no speed benefit (and some SAN tools need data and log on one volume to take application-consistent snapshots). It *is* worth putting a very busy tempdb on fast local storage such as an NVMe/M.2 drive.

### 6. Performance and design implications
- **Size tempdb up front** for peak workload (spills, version store, index rebuilds) to avoid autogrowth during busy times.
- **Fix spills at the source**: accurate statistics, better estimates, appropriate indexes; memory grant feedback (2017+/2019+) helps repeated queries.
- **Use temp tables for intermediate sets that need statistics/indexes**; table variables for small sets or where recompiles must be avoided.
- **Watch long transactions** when RCSI/snapshot is on — they bloat the version store and lengthen version chains, slowing readers.
- **RCSI adds 14 bytes per modified row** in the user database (versioning tag) — can cause page splits after enabling.
- **Monitor** `Version Store Size (KB)`, `Version Generation/Cleanup rate`, `Free Space in tempdb` counters, and `sys.dm_db_file_space_usage`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `tempdb.sys.dm_db_file_space_usage` | User/internal/version/free pages | First check when tempdb grows |
| `sys.dm_db_session_space_usage` / `dm_db_task_space_usage` | Allocation by session/task | Find the culprit |
| `sys.dm_tran_active_snapshot_database_transactions` | Active snapshot transactions | Version store not shrinking |
| `sys.dm_tran_version_store_space_usage` | Version store by database | Which DB generates versions |
| `ALTER DATABASE tempdb ADD FILE (…)` | Add data files | Allocation contention |
| `ALTER SERVER CONFIGURATION SET MEMORY_OPTIMIZED TEMPDB_METADATA = ON` | In-memory tempdb system tables | Metadata contention (2019+) |
| `CREATE INDEX … WITH (SORT_IN_TEMPDB = ON)` | Sort runs in tempdb | Faster rebuilds if tempdb is fast |
| `ALTER DATABASE db SET READ_COMMITTED_SNAPSHOT ON` | RCSI (uses version store) | Reduce reader/writer blocking |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Temp table — larger sets, need stats/indexes, reused across statements | Tiny sets in very hot procedures (recompiles) |
| Table variable — small sets, no recompiles wanted | Large sets on pre-2019 versions (1-row estimate) |
| RCSI/snapshot — fewer blocking issues | tempdb sized too small; long-running transactions |
| Separate fast local disk for tempdb | Separation on a shared SAN path — no benefit |
| More tempdb files | Unequal sizes (proportional fill skews) |

### 9. Common mistakes
- **One tempdb data file on a busy OLTP server** → `PAGELATCH_UP` contention → multiple equally sized files.
- **Leaving an open transaction in an idle session with RCSI on** → version store grows until tempdb fills → monitor and kill/fix.
- **Believing table variables are memory-only** → surprise tempdb usage, bad estimates → treat them as tempdb objects without stats.
- **Creating indexes on temp tables after creation in procedures** → prevents caching, extra recompiles → declare indexes inline.
- **Ignoring spill warnings** → slow sorts and hashes, tempdb I/O → fix estimates and memory grants.
- **Splitting volumes on the same SAN array "for performance"** → no gain, may break app-consistent snapshots → separate only when hardware paths differ.

### 10. 🎯 Interview kit
**30-second answer:** tempdb is a shared, recreated-at-restart system database holding temp tables, table variables, worktables, sort/hash spills, and the version store used by RCSI, snapshot isolation, triggers and online index operations. Common problems are allocation contention (fix with multiple equal data files), spills (fix estimates/memory grants), and version store growth from long-running snapshot transactions. Monitor with `dm_db_file_space_usage` and session space DMVs.

**Structure a longer answer:** what lives in tempdb → how versioning works (copy old row, 14-byte pointer, readers follow chain) → failure modes → configuration (files, sizing, placement) → scenario: tempdb filled overnight because a report left a snapshot transaction open.

**Likely follow-ups:**
- *Temp table vs table variable?* → stats, scope, recompiles, estimates (deferred compilation in 2019).
- *Why multiple tempdb files?* → spread allocation-page latch contention.
- *What makes the version store grow?* → long snapshot transactions, heavy updates.
- *What's ADR's effect?* → versions move to persistent version store in the user DB.

**Signals interviewers look for:** you know both explicit and implicit tempdb consumers, can triage which one is filling it, and avoid myths (memory-only table variables, separate volumes always faster).

### 11. Check yourself
- [ ] List five things that consume tempdb space, and which DMV column each appears in.
- [ ] Explain step by step how an RCSI reader finds an older row version.
- [ ] Find the session responsible for tempdb growth on a test server.
- [ ] Explain when separating tempdb onto its own volume helps and when it doesn't.
- [ ] Compare estimates for a temp table and a table variable with 10,000 rows.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › TempDB and Intermediate Storage
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Monitoring Row Versioning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Internal Working of Row Versioning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Excessive tempDB Usage
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Best Practices
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Performance Counters
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — � P › � T
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Temporary Tables and Table Variables › Creating Local Temp Tables
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.7 #49 Always placing TempDB and log files on dedicated drives
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance › Solutions to Exercise 12-4: Temporary Tables and Table Variables

