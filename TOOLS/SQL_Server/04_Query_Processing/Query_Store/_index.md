---
id: "TOOLS/SQL_Server/04_Query_Processing/Query_Store"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Query Store

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Query Store is a per-database flight recorder. It saves every query, every plan it has used and aggregated runtime and wait statistics to disk, so you can see when performance changed, which plan caused it, and force a better plan or hint without touching the code.

### 1. The big picture
The plan cache only shows the **current** plan, only while it is in memory, and loses everything on restart or eviction. That makes questions like "this was fast last Tuesday, what changed?" very hard. Query Store (SQL Server 2016+, Azure SQL) answers them by persisting history **inside the user database**:
- the query text and its properties,
- every plan the query has used,
- runtime statistics (duration, CPU, reads, writes, memory, rows, DOP) per plan, per time interval,
- wait statistics per plan (2017+).

Because history is kept per plan, a **plan regression** (new plan, worse performance) is easy to see and to undo with **plan forcing**. Since SQL Server 2022 you can also attach **query hints** to a query from Query Store (`sp_query_store_set_hints`) without changing application code. Query Store is also the foundation of Automatic Tuning, PSP optimization variants and several feedback features (next lessons). It is **on by default** for new databases from SQL Server 2022.

### 2. Core concepts and vocabulary
- **query_text / query**: the statement text and the query entry (one per statement and context settings). A statement inside a procedure records its `object_id`.
- **plan**: one row per distinct plan used for a query (`plan_id`), with the plan XML.
- **runtime stats interval**: a time bucket (default 60 minutes) in which execution statistics are aggregated (avg, min, max, stdev, last, count).
- **Regressed query**: a query whose recent metrics are worse than its history, usually because the plan changed.
- **Plan forcing**: telling the optimizer to use a specific `plan_id` for a query whenever it compiles.
- **Morally equivalent plan**: on rare occasions, the forced plan cannot be reproduced exactly and the optimizer produces a nearly identical one. This still counts as forcing working.
- **Query Store hints**: hints (`OPTION (…)` text) attached to a `query_id` and applied at compile time.
- **Capture mode**: which queries are recorded: `ALL`, `AUTO` (skip trivial or infrequent ones, the default), `NONE`, or `CUSTOM` (your thresholds).
- **Operation mode**: `READ_WRITE` (capturing) or `READ_ONLY` (no new data; happens automatically when the store is full).

### 3. How it works, step by step
1. A query compiles as usual. The plan goes into the plan cache. (If a plan is forced, the optimizer uses the forced plan during compilation, see below.)
2. An **asynchronous** process copies the query text and plan into a Query Store memory area.
3. The query runs. When it finishes, its runtime metrics are added (asynchronously) into in-memory aggregates for the current interval and plan.
4. Every `DATA_FLUSH_INTERVAL_SECONDS` (default 900 s = 15 min), the in-memory data is written to internal tables in the user database (in the PRIMARY filegroup). Because they are database tables, **Query Store data is backed up and restored with the database** and travels with it on failover (on readable secondaries it's supported from 2022).
5. A crash can lose up to the last flush interval of aggregates. That is acceptable for statistical data.
6. When you query the `sys.query_store_*` views, SQL Server combines in-memory and on-disk data automatically.
7. Cleanup: data older than `STALE_QUERY_THRESHOLD_DAYS` is removed, and size-based cleanup removes the least valuable queries as the store approaches `MAX_STORAGE_SIZE_MB`. If it fills anyway, it switches to `READ_ONLY` and stops capturing (check `actual_state_desc`).

**Main catalog views:**
| View | Contents |
|---|---|
| `sys.query_store_query_text` | Statement text |
| `sys.query_store_query` | Query entries, object_id, compile stats |
| `sys.query_store_plan` | Plans, `is_forced_plan`, `force_failure_count`, plan XML |
| `sys.query_store_runtime_stats` | Metrics per plan per interval |
| `sys.query_store_runtime_stats_interval` | Interval start/end times |
| `sys.query_store_wait_stats` | Waits per plan per interval (category level) |
| `sys.query_store_query_hints` | Hints applied via Query Store |
| `sys.database_query_store_options` | Configuration and current state |

**Built-in SSMS reports** (database → Query Store): Regressed Queries, Overall Resource Consumption, Top Resource Consuming Queries, Queries With Forced Plans, Queries With High Variation, Query Wait Statistics, Tracked Queries.

### 4. Worked example
```sql
-- 1. Enable and configure
ALTER DATABASE AdventureWorks SET QUERY_STORE = ON;
ALTER DATABASE AdventureWorks SET QUERY_STORE
 (OPERATION_MODE = READ_WRITE,
  QUERY_CAPTURE_MODE = AUTO,
  MAX_STORAGE_SIZE_MB = 2048,
  INTERVAL_LENGTH_MINUTES = 30,
  DATA_FLUSH_INTERVAL_SECONDS = 900,
  CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30),
  SIZE_BASED_CLEANUP_MODE = AUTO,
  WAIT_STATS_CAPTURE_MODE = ON);

-- 2. Find queries for a procedure, with each plan's average duration
SELECT q.query_id, p.plan_id, p.is_forced_plan,
       SUM(rs.count_executions)                          AS executions,
       SUM(rs.avg_duration * rs.count_executions)
         / NULLIF(SUM(rs.count_executions), 0) / 1000.0  AS avg_ms,
       MAX(rs.last_execution_time)                       AS last_run
FROM sys.query_store_query AS q
JOIN sys.query_store_plan AS p           ON p.query_id = q.query_id
JOIN sys.query_store_runtime_stats AS rs ON rs.plan_id = p.plan_id
WHERE q.object_id = OBJECT_ID(N'dbo.ProductTransactionHistoryByReference')
GROUP BY q.query_id, p.plan_id, p.is_forced_plan
ORDER BY q.query_id, avg_ms;
```
Reading the output: if `query_id 121` has `plan_id 11` averaging 2 ms and `plan_id 14` averaging 900 ms, and plan 14 appeared recently, you have a regression. Note the weighted average: `avg_duration` is per interval, so weight it by executions when combining intervals.

```sql
-- 3. Force the good plan
EXEC sys.sp_query_store_force_plan @query_id = 121, @plan_id = 11;

-- Check forcing is succeeding
SELECT plan_id, is_forced_plan, force_failure_count, last_force_failure_reason_desc
FROM sys.query_store_plan WHERE query_id = 121;

-- Undo
EXEC sys.sp_query_store_unforce_plan @query_id = 121, @plan_id = 11;

-- 4. Or apply a hint without code changes (SQL Server 2022+, Azure SQL)
EXEC sys.sp_query_store_set_hints @query_id = 550,
     @query_hints = N'OPTION (OPTIMIZE FOR UNKNOWN)';
SELECT query_hint_id, query_id, query_hint_text, source_desc
FROM sys.query_store_query_hints;
EXEC sys.sp_query_store_clear_hints @query_id = 550;
```

### 5. Under the hood
- **Forcing happens at compile time.** A recompile still occurs normally (statistics changed, schema changed…), but the optimizer is constrained to produce the forced plan. If it cannot (for example the forced plan uses an index that was dropped), forcing **fails**: the newly optimized plan is used, `force_failure_count` increases and `last_force_failure_reason_desc` says why (such as `NO_INDEX`). The forced flag remains, and forcing resumes if the index comes back.
- **Optimized plan forcing (2022):** Query Store can store a *compilation replay script* for forced plans so the optimizer skips most of the search work, reducing compile time for complex forced queries.
- **Query identity:** Query Store identifies a query by its text hash **plus** context settings (SET options, language…) and parent object. The same text with different SET options becomes a different `query_id`. Ad hoc queries with different literals are different queries too, which is why `AUTO` capture mode and `query_hash` grouping matter.
- **Overhead:** the asynchronous design keeps overhead low (typically single-digit percent at most). The main costs are memory for in-flight aggregates and storage. Very ad hoc workloads with `ALL` capture can bloat the store.
- **Readable secondaries (2022+):** Query Store can capture workload from Always On readable secondaries, sending it to the primary.
- **Hints vs plan guides:** Query Store hints replace most uses of plan guides. They are easier to apply, visible in DMVs and keyed by `query_id` instead of exact text.

### 6. Performance and design implications
- Turn it on everywhere that matters. Size `MAX_STORAGE_SIZE_MB` for your retention (often 1–10 GB) and monitor `actual_state_desc`. `READ_ONLY` means you're blind.
- Shorter intervals (15–30 min) show short regressions better, but use more space.
- Treat forced plans as **temporary** stabilizers. Record why each one was forced, review regularly, and fix the underlying cause (statistics, index, code). Forced plans can become wrong as data changes and they block improvements from new indexes.
- Use Query Store before and after upgrades or compatibility-level changes: capture a baseline at the old level, switch, compare regressed queries, force old plans where needed. This is the recommended upgrade method.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE … SET QUERY_STORE = ON (…)` | Enable and configure | Every important database |
| `QUERY_CAPTURE_MODE = AUTO / CUSTOM` | Skip trivial/rare queries | Ad hoc-heavy workloads |
| `sys.sp_query_store_force_plan` / `unforce_plan` | Force or release a plan | Fixing regressions |
| `sys.sp_query_store_set_hints` / `clear_hints` | Attach or remove hints | Fix without code changes (2022+) |
| `sys.sp_query_store_remove_plan` / `remove_query` | Delete history | Cleaning bad data |
| `ALTER DATABASE … SET QUERY_STORE CLEAR` | Purge all data | Rarely, starting over |
| `sys.database_query_store_options` | `actual_state_desc`, sizes, reasons for READ_ONLY | Health check |
| SSMS Regressed Queries / Top Resource Consuming reports | Visual analysis and forcing | Daily triage |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plan forcing: a clearly better older plan exists | As a permanent fix; data drift makes it wrong |
| Query Store hints: need a hint, can't change code | Hints that over-constrain (forcing join types) |
| `CAPTURE_MODE = ALL`: short diagnostic window | Long-term on ad hoc-heavy systems (bloat) |
| Short intervals: catching brief regressions | Storage is tight |
| Plan cache DMVs: right-now analysis | You need history across restarts |

### 9. Common mistakes
- Leaving default small size limits → store fills and goes `READ_ONLY` → raise `MAX_STORAGE_SIZE_MB`, use AUTO capture and cleanup.
- Forcing plans and forgetting them → months later the plan is bad and nobody knows why → document, review `Queries With Forced Plans` regularly.
- Averaging `avg_duration` across intervals without weighting → misleading numbers → weight by `count_executions`.
- Not checking `force_failure_count` → believing a plan is forced when forcing fails → check after schema changes.
- Typos in hint text (`OPTIMIZE FOR UNKOWN`) → hint rejected → verify in `sys.query_store_query_hints` and the plan.

### 10. 🎯 Interview kit
**30-second answer:** "Query Store records queries, every plan they use and aggregated runtime and wait stats per interval inside the database, asynchronously. It survives restarts and is backed up with the database. I use the Regressed Queries and Top Resource Consumers reports to find problems, compare plans for one query, and force the good plan or, on 2022+, apply a Query Store hint without code changes. I treat forcing as a temporary fix, monitor failures, and keep the store sized so it never goes read-only."
**Structure a longer answer:** problem with plan cache only → what QS stores and how (async, intervals, flush) → finding regressions → forcing and hints → operations (size, capture mode, cleanup) → upgrade scenario.
**Likely follow-ups:** *What happens if a forced plan's index is dropped?* → forcing fails, normal plan used, failure recorded. *Why did Query Store stop collecting?* → reached max size → READ_ONLY. *Query Store vs plan cache?* → history and persistence vs current in-memory only.
**Signals interviewers look for:** uses QS as standard practice; understands forcing mechanics and risks; knows configuration pitfalls; uses it for upgrades.

### 11. Check yourself
- [ ] Enable Query Store with sensible settings and check `actual_state_desc`.
- [ ] Create a regression (via parameter sniffing), find it in the Regressed Queries report, and force the old plan.
- [ ] Drop an index used by a forced plan and read `last_force_failure_reason_desc`.
- [ ] Apply and then clear a Query Store hint.
- [ ] Describe an upgrade process that uses Query Store to avoid regressions.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Forcing Plans in Query Store
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Common Pitfalls
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Forcing Plans and Analyzing Regressions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Key Takeaways
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Advanced Plan Forcing Techniques
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Using Query Store for Plan Analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Using Query Store for Hinting
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 6 Using The Query Store For Query Performance And Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Query Store
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Overview

