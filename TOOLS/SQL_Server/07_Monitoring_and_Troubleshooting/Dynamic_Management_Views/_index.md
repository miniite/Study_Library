---
id: "TOOLS/SQL_Server/07_Monitoring_and_Troubleshooting/Dynamic_Management_Views"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Dynamic Management Views

**L4 Advanced** · [↑ Monitoring and Troubleshooting](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Dynamic management views and functions (DMVs/DMFs, `sys.dm_*`) expose SQL Server's internal state as queryable tables: sessions, requests, cached plans and their runtime statistics, index usage, missing indexes, waits, I/O, memory. They are the first tool for live diagnosis, but most are volatile and must be read with care.

### 1. The big picture
Since SQL Server 2005, the engine publishes its internal counters and structures as views. Instead of a black box, you can ask: what is running right now, what is it waiting for, which cached query has used the most CPU since it was compiled, which indexes are never read, which files are slow. Together they're called **dynamic management objects (DMOs)**: views (DMVs) and table-valued functions (DMFs).

They sit between performance counters (broad, numeric) and Extended Events (precise, event-level). Query Store complements them with persistent history. Most query-tuning work starts with a DMV query.

### 2. Core concepts and vocabulary
- **DMV / DMF**: dynamic management view / function in the `sys` schema, prefixed `dm_`. DMFs take parameters (e.g. a handle).
- **Scope**: *server-scoped* DMOs need `VIEW SERVER STATE` (2022+: `VIEW SERVER PERFORMANCE STATE`); *database-scoped* ones need `VIEW DATABASE STATE`.
- **Families by prefix**:
  - `dm_exec_*`: sessions, connections, requests, cached plans and query statistics.
  - `dm_os_*`: operating system layer: waits, schedulers, memory, performance counters.
  - `dm_db_*`: database level: index usage, physical stats, missing indexes, file space, statistics properties.
  - `dm_io_*`: I/O: file stats, pending I/O.
  - `dm_tran_*`: transactions, locks (`dm_tran_locks`), version store.
  - `dm_xe_*`, `dm_hadr_*`, `dm_resource_governor_*` and others.
- **sql_handle**: token identifying a batch's text; turned into text with `sys.dm_exec_sql_text`.
- **plan_handle**: token identifying a cached plan; turned into XML with `sys.dm_exec_query_plan` (or `dm_exec_text_query_plan` per statement).
- **statement offsets**: `statement_start_offset` / `statement_end_offset` locate one statement in a batch, in bytes (Unicode, so divide by 2).
- **query_hash / query_plan_hash**: fingerprints that group queries differing only in literal values, or plans with the same shape.
- **Cumulative vs. current**: some DMVs show current state (`dm_exec_requests`); others accumulate since startup or since the plan was cached (`dm_exec_query_stats`, `dm_os_wait_stats`, `dm_db_index_usage_stats`).

### 3. How it works, step by step
**How DMV data is produced:** the views are not tables on disk. When you query them, SQL Server reads in-memory structures (the plan cache, scheduler lists, lock manager hash tables) and presents them as rows. So:
1. Data is lost on **restart** (and per-database data when a database goes offline or is detached).
2. `dm_exec_query_stats` rows disappear when a plan leaves the cache (memory pressure, recompile, `DBCC FREEPROCCACHE`, `OPTION (RECOMPILE)` queries are never cached).
3. Values are cumulative since their start, so you compare snapshots or divide by `execution_count`.
4. Reading some DMVs (e.g. `dm_tran_locks` on a busy server, `dm_db_index_physical_stats` in DETAILED mode) has a real cost.

**The core diagnostic chain:**
1. **Who's connected?** `sys.dm_exec_sessions` (login, host, program, open transaction count) + `sys.dm_exec_connections` (protocol, client address).
2. **What's running?** `sys.dm_exec_requests`: status, command, wait type, blocking session, CPU, reads, percent complete for backups/DBCC.
3. **What's the text and plan?** `CROSS APPLY sys.dm_exec_sql_text(sql_handle)` and `sys.dm_exec_query_plan(plan_handle)`. For in-flight actual statistics: `sys.dm_exec_query_statistics_xml(session_id)` (lightweight profiling on by default since SQL 2019) and `sys.dm_exec_query_profiles` (live query statistics).
4. **What's expensive over time?** `sys.dm_exec_query_stats` (per statement), `dm_exec_procedure_stats`, `dm_exec_trigger_stats`, `dm_exec_function_stats`.
5. **Where's the bottleneck?** `sys.dm_os_wait_stats`, `sys.dm_exec_session_wait_stats`, `sys.dm_io_virtual_file_stats`.
6. **Index and statistics health?** `sys.dm_db_index_usage_stats`, `sys.dm_db_index_operational_stats`, `sys.dm_db_missing_index_*`, `sys.dm_db_stats_properties`, `sys.dm_db_index_physical_stats`.

### 4. Worked example
**Top 10 statements by CPU since they were cached:**
```sql
SELECT TOP (10)
       qs.execution_count,
       qs.total_worker_time / 1000                        AS total_cpu_ms,
       qs.total_worker_time / qs.execution_count / 1000   AS avg_cpu_ms,
       qs.total_logical_reads / qs.execution_count        AS avg_reads,
       qs.total_elapsed_time / qs.execution_count / 1000  AS avg_duration_ms,
       qs.creation_time, qs.last_execution_time,
       SUBSTRING(st.text, qs.statement_start_offset / 2 + 1,
                 (CASE qs.statement_end_offset WHEN -1 THEN DATALENGTH(st.text)
                       ELSE qs.statement_end_offset END - qs.statement_start_offset) / 2 + 1) AS stmt,
       qp.query_plan
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) AS qp
ORDER BY qs.total_worker_time DESC;
```
**How to read it:** `total_worker_time` is in microseconds. A statement with a moderate average but a huge `execution_count` (e.g. 2 ms × 5 million) can matter more than one 20-second report. Check `creation_time`: a plan cached 5 minutes ago has only 5 minutes of totals, so comparing totals across plans of different ages misleads. Click `query_plan` to open the cached (estimated) plan.

**Grouping ad hoc queries by fingerprint** (finds non-parameterized code):
```sql
SELECT TOP (10) qs.query_hash,
       COUNT(*)                     AS cached_plans,
       SUM(qs.execution_count)      AS execs,
       SUM(qs.total_worker_time)/1000 AS cpu_ms,
       MIN(st.text)                 AS sample_text
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
GROUP BY qs.query_hash
ORDER BY cached_plans DESC;
```
Hundreds of plans for one `query_hash` means the same query with different literals is compiled every time.

**Unused indexes (cost without benefit):**
```sql
SELECT OBJECT_NAME(i.object_id) AS table_name, i.name AS index_name,
       ISNULL(us.user_seeks + us.user_scans + us.user_lookups, 0) AS reads,
       ISNULL(us.user_updates, 0) AS writes
FROM sys.indexes AS i
LEFT JOIN sys.dm_db_index_usage_stats AS us
       ON us.object_id = i.object_id AND us.index_id = i.index_id AND us.database_id = DB_ID()
WHERE OBJECTPROPERTY(i.object_id, 'IsUserTable') = 1 AND i.index_id > 1
ORDER BY reads, writes DESC;
```
Only meaningful after a full business cycle since restart (month-end reports!). Check `sqlserver_start_time` in `sys.dm_os_sys_info`.

**Missing index suggestions:**
```sql
SELECT TOP (10)
       mid.statement AS table_name, mid.equality_columns, mid.inequality_columns, mid.included_columns,
       migs.user_seeks, migs.avg_user_impact,
       migs.user_seeks * migs.avg_total_user_cost * migs.avg_user_impact / 100.0 AS score
FROM sys.dm_db_missing_index_details AS mid
JOIN sys.dm_db_missing_index_groups AS mig ON mig.index_handle = mid.index_handle
JOIN sys.dm_db_missing_index_group_stats AS migs ON migs.group_handle = mig.index_group_handle
ORDER BY score DESC;
```
Treat these as hints: they ignore column order subtleties, overlap with existing indexes, and suggest many similar indexes. Consolidate before creating anything.

**Statistics freshness:**
```sql
SELECT s.name, sp.last_updated, sp.rows, sp.rows_sampled, sp.modification_counter
FROM sys.stats AS s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) AS sp
WHERE s.object_id = OBJECT_ID('Sales.Orders');
```

**Per-file I/O latency:**
```sql
SELECT DB_NAME(vfs.database_id) AS db, mf.physical_name,
       vfs.io_stall_read_ms  / NULLIF(vfs.num_of_reads, 0)  AS avg_read_ms,
       vfs.io_stall_write_ms / NULLIF(vfs.num_of_writes, 0) AS avg_write_ms
FROM sys.dm_io_virtual_file_stats(NULL, NULL) AS vfs
JOIN sys.master_files AS mf ON mf.database_id = vfs.database_id AND mf.file_id = vfs.file_id
ORDER BY avg_read_ms DESC;
```

### 5. Under the hood
- **Plan cache and dynamic SQL.** Statements run via `sp_executesql` are cached as *prepared* plans and appear in `dm_exec_query_stats` like procedure statements. Concatenated `EXEC(@sql)` with literals creates one *ad hoc* plan per distinct string; `sys.dm_exec_cached_plans` shows `objtype = 'Adhoc'` with `usecounts = 1` for these. `optimize for ad hoc workloads` stores only a small stub on first execution.
- **Missing-index DMVs** are filled by the optimizer during compilation when it would have liked an index. They hold at most 600 groups, reset at restart, and are cleared for a table when any index on it changes.
- **Index usage stats** are counted per *plan execution*, not per row: one seek operator executed once = 1 user_seek, even if it reads a million rows. `dm_db_index_operational_stats` gives lower-level counters (leaf inserts, page latch waits, lock escalations) and survives as long as the metadata is cached.
- **`dm_db_index_physical_stats`** reads the index pages themselves: LIMITED mode reads only upper levels (cheap), SAMPLED reads 1%, DETAILED reads everything (expensive on large tables).
- **Live actual plans.** Lightweight query profiling (on by default since SQL 2019) collects per-operator row counts for all queries at low cost, enabling `dm_exec_query_statistics_xml` and Live Query Statistics in SSMS. SQL 2019 also added `sys.dm_exec_query_plan_stats` (last actual plan, needs `LAST_QUERY_PLAN_STATS` database scoped configuration).
- **Azure SQL** exposes database-scoped variants (`sys.dm_db_resource_stats` every 15 s for the last hour, `sys.resource_stats` in master for longer history).

### 6. Performance and design implications
- Snapshot important DMVs into tables on a schedule (waits, file stats, query stats) to survive restarts and to compute deltas.
- Sort by total *and* average; high-frequency small queries are often the real cost.
- Validate index decisions over a full business cycle; never drop an index just because usage stats reset yesterday.
- Combine DMVs with Query Store: Query Store for history and regressions, DMVs for right-now and server-wide views.
- Keep diagnostic queries light on busy servers (avoid DETAILED physical stats and unfiltered lock dumps during peak load).

### 7. Commands and settings you should know
| DMO | What it shows | When to use |
|---|---|---|
| `sys.dm_exec_requests` / `sessions` / `connections` | Current activity | Live slowness, blocking |
| `sys.dm_exec_sql_text(handle)` / `dm_exec_query_plan(handle)` | Text / cached plan | With any handle |
| `sys.dm_exec_query_statistics_xml(spid)` | In-flight actual plan | Long-running query now |
| `sys.dm_exec_query_stats` / `procedure_stats` | Cumulative cost per cached statement/proc | Top consumers |
| `sys.dm_exec_cached_plans` | Plan cache contents | Ad hoc bloat |
| `sys.dm_os_wait_stats` / `dm_exec_session_wait_stats` | Waits server / per session | Bottlenecks |
| `sys.dm_io_virtual_file_stats` | Per-file latency | Storage issues |
| `sys.dm_db_index_usage_stats` / `operational_stats` | Index reads/writes, contention | Index cleanup |
| `sys.dm_db_missing_index_*` | Optimizer index wishes | Index hints |
| `sys.dm_db_stats_properties` | Stats age and modifications | Statistics maintenance |
| `sys.dm_db_index_physical_stats` | Fragmentation, page density | Maintenance decisions |
| `sys.dm_tran_locks` / `dm_tran_active_transactions` | Locks / open transactions | Blocking, log growth |
| `sys.dm_os_sys_info` | Start time, CPUs, memory | Context for cumulative data |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| DMVs for fast, free, live diagnosis | Need history across restarts → Query Store / snapshots |
| `dm_exec_query_stats` for top cached consumers | RECOMPILE queries and evicted plans are missing |
| Missing-index DMVs as a starting list | Creating every suggestion → duplicate, overlapping indexes |
| Index usage stats for cleanup | Short uptime; seasonal workloads |
| `dm_db_index_physical_stats` LIMITED | DETAILED on huge tables during business hours |

### 9. Common mistakes
- Trusting cumulative numbers without checking uptime or `creation_time` → misleading comparisons → normalize by time/executions.
- Creating all missing-index suggestions → write overhead, duplicates → consolidate, test.
- Dropping "unused" indexes after a restart → month-end report breaks → wait a full cycle.
- Forgetting statement offsets → you see the whole procedure text, not the statement → use the SUBSTRING pattern.
- Expecting `dm_exec_query_stats` to show `OPTION(RECOMPILE)` queries → they're not cached → Query Store.
- Granting `sysadmin` for monitoring → excessive rights → `VIEW SERVER STATE` (or the 2022 granular permissions).

### 10. 🎯 Interview kit
**30-second answer:** DMVs and DMFs are system views over SQL Server's in-memory state. I use dm_exec_requests and sessions for what's running and blocking, dm_exec_query_stats with sql_text and query_plan for top consumers, dm_os_wait_stats and dm_io_virtual_file_stats for bottlenecks, and index usage, missing-index and stats-properties DMVs for index and statistics health. Most data resets at restart or when plans leave the cache, so I snapshot it and use Query Store for history.

**Structure a longer answer:** families by prefix → the diagnostic chain (sessions → requests → text/plan → stats → waits) → caveats (volatility, cumulative values, permissions) → a real finding (ad hoc plan bloat via query_hash, unused index cleanup).

**Likely follow-ups:**
- *How do you find the most expensive queries?* → dm_exec_query_stats by total/avg CPU and reads, or Query Store.
- *Are missing-index suggestions reliable?* → hints only: no column-order analysis, overlap, 600-group limit.
- *Why might a query be absent from query_stats?* → RECOMPILE, eviction, restart.
- *How to see a running query's actual plan?* → dm_exec_query_statistics_xml / Live Query Statistics.

**Signals interviewers look for:** fluent DMV joins with handles and offsets; awareness of volatility and cumulative semantics; skepticism about missing-index advice.

### 11. Check yourself
- [ ] Write the top-10-by-CPU query from memory, including the statement-offset SUBSTRING.
- [ ] Find ad hoc plan bloat with `query_hash` grouping.
- [ ] List unused indexes and explain why uptime matters.
- [ ] Find the slowest data file by average read latency.
- [ ] Get the in-flight actual plan of a long-running session.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Monitoring Plan Cache
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Dynamic Management Objects
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Monitoring and Performance Tuning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Performance Monitoring of Dynamic SQL
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Using Live Query Statistics
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbstatsproperties
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Introduction to Dynamic Data Masking (DDM)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Where to Go Next › Launching SQL Server Management Studio
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Views › Creating Views

