---
id: "TOOLS/SQL_Server/04_Query_Processing/Execution_Plans/Memory_Grants_and_Spills"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Memory Grants and Spills

**L5 Expert** · [↑ Reading Execution Plans](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Before a query runs, SQL Server reserves a fixed amount of workspace memory (the memory grant) for its sorts and hashes, sized from *estimated* rows. Too small and the operators spill to tempdb; too large and queries queue on `RESOURCE_SEMAPHORE` while memory sits unused.

### 1. The big picture
Most operators stream rows and need almost no memory. Some need a workspace: **Sort**, **Hash Match** (join and aggregate), **Exchange** buffers in parallel plans, and some batch-mode operators. They need memory proportional to the data they hold. SQL Server can't grow that memory freely while the query runs. Instead, the plan carries an estimate of how much it needs, and execution must be **granted** that memory *before* it starts.

This makes memory grants the place where cardinality-estimation errors turn into real resource problems:
- **Underestimate** → the grant is too small → the operator writes overflow rows to **tempdb** (a *spill*) and reads them back. Runtime can jump from milliseconds to seconds.
- **Overestimate** → the grant is too big → memory is reserved but unused. Concurrent queries wait for grants (`RESOURCE_SEMAPHORE` waits), and the buffer pool shrinks.

This lesson follows on from *Join Operators* (hash joins) and leads into *Intelligent Query Processing* (memory grant feedback).

### 2. Core concepts and vocabulary
- **Workspace memory**: memory used by memory-consuming operators (sort, hash, exchange). It is taken from the buffer pool's budget.
- **Memory grant**: the amount reserved for one query execution. Plan properties: *Desired Memory*, *Requested Memory*, *Granted Memory*, *Max Used Memory*.
- **Required memory**: the minimum a query needs to start at all (enough for the operator structures).
- **Resource semaphore**: the gatekeeper that hands out grants. Queries that don't fit wait in a queue, and the wait type is **`RESOURCE_SEMAPHORE`**.
- **Spill**: rows written to tempdb because the grant was too small. Shown as **Sort Warning**, **Hash Warning**, or a yellow warning triangle with *Spill Level* and *Pages spilled*.
- **Spill level**: how many recursive passes were needed. Level 1 = one write-and-read pass. Higher = worse.
- **Excessive grant warning**: plan warning when used memory is far below the granted amount.
- **Memory grant feedback (MGF)**: since 2017 (batch) and 2019 (row mode), later executions adjust the grant up or down based on what the previous run used. From 2022 it can persist in Query Store and use percentiles.
- **Resource Governor**: can cap the grant per query (`REQUEST_MAX_MEMORY_GRANT_PERCENT`, default 25% of the pool).

### 3. How it works, step by step
1. **Compile.** The optimizer estimates rows and row width into each sort/hash operator and computes the **ideal** (desired) memory to do all work in memory.
2. **Request.** At execution start, the query asks the resource semaphore for its grant. The request is capped by Resource Governor (by default about 25% of the workspace pool per query) and the server's max.
3. **Wait or run.** If enough memory is free, the grant is given and the query starts. If not, the query waits on `RESOURCE_SEMAPHORE` (small queries have their own, faster queue). It may eventually time out with error 8645.
4. **Execute.** The operators use only the memory granted. *A grant does not grow during execution.*
5. **Overflow.** If a Sort gets more rows than fit, it sorts runs in memory, writes them to tempdb and merge-reads them later. If a Hash build input doesn't fit, it partitions buckets to tempdb and processes them in extra passes (*recursion*, ultimately *bailout*).
6. **Release.** When the operator (or the query) finishes, the memory is returned.
7. **Feedback** (if enabled): the actual *Max Used Memory* is compared with the grant, and the next execution's grant is adjusted.

```
estimated rows ──► desired grant ──► semaphore ──► granted ──► operators
                                       │ (not enough free)          │ (more rows than fit)
                                       ▼                            ▼
                               RESOURCE_SEMAPHORE wait        spill to tempdb
```

### 4. Worked example
```sql
-- Provoke a spill: a sort whose estimate is wrong.
-- A table variable is estimated at 1 row (or 100 with deferred compilation), so tiny grant.
DECLARE @t TABLE (id int, payload char(200));
INSERT @t SELECT TOP (200000) object_id, 'x'
FROM sys.all_objects a CROSS JOIN sys.all_objects b;

SELECT id, payload FROM @t ORDER BY payload, id   -- Sort with a tiny grant
OPTION (RECOMPILE);   -- run once WITHOUT RECOMPILE first: the estimate stays low, the Sort spills
```
What to look for in the actual plan:
- A yellow ⚠ on **Sort**: "Operator used tempdb to spill data during execution with spill level 1 and 1 spilled thread(s)…".
- Root **SELECT → MemoryGrantInfo**: *GrantedMemory* small, *MaxUsedMemory* = granted (fully used), *DesiredMemory* small.
- With `OPTION (RECOMPILE)` the optimizer sees the actual table-variable row count, the grant is right, and the warning disappears.

Live view while queries run:
```sql
SELECT session_id, requested_memory_kb, granted_memory_kb, used_memory_kb,
       ideal_memory_kb, wait_time_ms, queue_id, dop
FROM sys.dm_exec_query_memory_grants;      -- granted_memory_kb NULL = still waiting

SELECT * FROM sys.dm_exec_query_resource_semaphores;  -- pool size, waiter counts

-- Historical: which queries spill or over-grant
SELECT TOP (20) qs.total_spills, qs.max_grant_kb, qs.max_used_grant_kb,
       qs.execution_count, st.text
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
ORDER BY qs.total_spills DESC;
```

### 5. Under the hood
- **Grant size math.** Sort memory ≈ estimated rows × average row size (plus overhead). Hash memory ≈ build-side rows × row size. **Wide columns** (`varchar(max)`, `nvarchar(4000)`) are estimated at half their declared size. That's why oversized column definitions inflate grants even when the data is short.
- **Parallel plans.** The grant covers all threads plus exchange buffers. If one thread gets most rows (skew), it can spill while the others have spare memory, because the grant is divided between threads.
- **Sort spill mechanics.** Runs that fit in memory are sorted and written to tempdb as sorted runs, then merged (multi-pass if many runs). **Hash spill mechanics.** Partitions are written to tempdb and processed one by one. If a partition still doesn't fit, it is re-partitioned (deeper spill level). If that repeats too many times, it falls back to a nested-loops-like *bailout*.
- **Semaphore queues.** There are two resource semaphores (regular and small queries < 5 MB). Inside each, queues are ordered by cost, so a large query can wait longer than a cheap one that arrived later. A waiting query has used no memory yet, but the user sees it as "hung."
- **Feedback details.** Row-mode MGF (2019, compat 150) adjusts when used memory is < 50% of the grant (decrease) or when a spill occurred (increase). If it keeps oscillating it disables itself for that plan (*IsMemoryGrantFeedbackAdjusted = NoFeedbackDisabled*). Percentile and persistence modes (2022, compat 140+/160) smooth this and survive restarts via Query Store.
- **Index builds.** `CREATE/ALTER INDEX` sorts also need grants. `SORT_IN_TEMPDB = ON` puts intermediate sort runs in tempdb instead of the user database. That's faster when tempdb is on faster storage, but tempdb then needs room for the whole sort while the user DB holds the new index.

![Intermediate index-build sorts placed on faster tempdb storage](<../../../../../_assets/100_SQL_Server_Mistakes_and_How_to_Avoid_-_Peter_A_Carter/CH11_F10_Carter.png>)

### 6. Performance and design implications
- **Spills are a symptom of bad estimates.** Fix the estimate first (stale statistics, table variables, multi-statement TVFs, parameter sniffing, non-SARGable expressions), and the grant fixes itself.
- **Avoid unnecessary sorts.** An index that supplies the `ORDER BY`/`GROUP BY` order removes the Sort and its grant entirely. `TOP n` with `ORDER BY` uses a much smaller *Top N Sort*.
- **Right-size columns.** Declaring `nvarchar(4000)` "to be safe" can multiply grant sizes.
- **Reduce rows early.** Filter and aggregate before sorting or hashing.
- **Watch the server level.** `RESOURCE_SEMAPHORE` waits in `sys.dm_os_wait_stats`, the *Memory Grants Pending* perfmon counter (should be ~0), and *Memory Grants Outstanding*.
- **Excessive grants hurt everyone.** A report query granted 4 GB that uses 50 MB can block twenty other queries.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_exec_query_memory_grants` | current grants and waiters | live "why is it stuck?" |
| `sys.dm_exec_query_resource_semaphores` | semaphore capacity and waiters | server-wide grant pressure |
| `sys.dm_exec_query_stats` (`*_grant_kb`, `*_spills`) | historical grant/spill per cached plan | find spilling queries |
| Query Store `sys.query_store_runtime_stats` (`avg_query_max_used_memory`) | memory used over time | trend and regressions |
| XE `sort_warning`, `hash_warning`, `hash_spill_details`, `memory_grant_updated_by_feedback` | capture spill/feedback events | ongoing monitoring |
| `OPTION (MIN_GRANT_PERCENT = n, MAX_GRANT_PERCENT = n)` | bounds the grant for one query | a known problem query |
| Resource Governor `REQUEST_MAX_MEMORY_GRANT_PERCENT` | caps grants per workload group | protect OLTP from reports |
| `ALTER DATABASE SCOPED CONFIGURATION SET ROW_MODE_MEMORY_GRANT_FEEDBACK = ON/OFF` | toggles row-mode MGF | troubleshooting feedback |
| `ALTER INDEX ... REBUILD WITH (SORT_IN_TEMPDB = ON)` | sorts index build in tempdb | faster rebuild with fast tempdb |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fix statistics / estimates: grants are wrong because rows are mis-estimated | (always try this first) |
| `MAX_GRANT_PERCENT`: one query over-grants and starves others | you cap it below what it really needs → spills |
| `MIN_GRANT_PERCENT`: a query spills despite correct-looking stats | it increases memory pressure for everyone |
| Resource Governor caps: mixed OLTP + reporting workloads | caps set too low make all reports spill |
| Rely on memory grant feedback: repetitive workloads | highly variable parameters (feedback oscillates and disables itself) |
| Add an index to remove a Sort | write-heavy tables where another index is costly |

### 9. Common mistakes
- Treating `RESOURCE_SEMAPHORE` waits as "add more RAM" → the over-granting queries keep growing → find and fix the over-granting queries first.
- Ignoring the yellow spill warning because "the query finished" → intermittent slowness and tempdb contention → investigate the estimate.
- Declaring huge variable-length columns → inflated grants → size columns to the data.
- Using table variables or multi-statement TVFs for large row sets → 1-row (or 100-row) estimates → tiny grants and spills → use temp tables, or rely on 2019 deferred compilation.
- Setting `MIN_GRANT_PERCENT` high everywhere → memory exhaustion → use it only per query, with evidence.
- Using `SORT_IN_TEMPDB` without sizing tempdb → the rebuild fails for lack of space → check free space for the full sort.

### 10. 🎯 Interview kit
**30-second answer:** Sorts and hash operators need workspace memory. SQL Server reserves it up front as a memory grant, sized from estimated rows and row width, and the grant can't grow during execution. If the estimate is too low, the operator spills to tempdb and the query slows a lot. If it's too high, memory is wasted and other queries queue on `RESOURCE_SEMAPHORE`. I diagnose spills from plan warnings and `sys.dm_exec_query_stats`, and waiters from `sys.dm_exec_query_memory_grants`. Then I fix the cardinality estimate, remove unnecessary sorts with indexes, or use memory grant feedback and `MIN/MAX_GRANT_PERCENT`.
**Structure a longer answer:** what needs memory → how the grant is computed and handed out → two failure modes (spill vs over-grant) → how to see each → root causes (estimates, wide columns, skew) → fixes in order of preference → mention feedback in 2019/2022.
**Likely follow-ups:**
- *Can a grant grow mid-query?* → no; that's why spills happen.
- *What does spill level 2 mean?* → recursion: a spilled partition didn't fit on the second pass either.
- *Why would `nvarchar(max)` increase grants?* → variable columns are estimated at half their declared size (with special handling for max types), regardless of the actual data.
- *What is the small-query semaphore?* → a separate queue for grants under about 5 MB, so tiny queries aren't stuck behind huge ones.
**Signals interviewers look for:** connecting grants to cardinality estimation, knowing both failure modes, naming the DMVs and wait type, preferring root-cause fixes over hints.

### 11. Check yourself
- [ ] Explain why a grant is sized at compile time and what happens if the estimate is wrong.
- [ ] Reproduce a Sort spill with a table variable and remove it with `OPTION (RECOMPILE)`.
- [ ] Query `sys.dm_exec_query_memory_grants` during a large report and interpret `granted_memory_kb` = NULL.
- [ ] List three design changes that reduce grant size.
- [ ] Describe how row-mode memory grant feedback decides to increase or decrease a grant.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Spill to TempDB
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Investigating Sort Operators
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.8.2 Considerations for SORTINTEMPDB
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9 Instance and database management

