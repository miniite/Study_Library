---
id: "TOOLS/SQL_Server/04_Query_Processing/Plan_Cache_and_Reuse"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Plan Cache and Plan Reuse

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Compiling a plan costs CPU, so SQL Server keeps compiled plans in the plan cache and reuses them when the same query text (with the same settings) comes back. Parameterized code reuses plans; ad hoc code with literals fills the cache with single-use plans.

### 1. The big picture
The optimizer (previous lessons) is expensive: a complex query can take tens or hundreds of milliseconds of CPU to compile, sometimes longer than it takes to run. Most workloads repeat the same queries over and over with different values. So SQL Server stores plans in a memory area called the **plan cache** (part of the buffer pool's memory, also called the procedure cache) and looks there before optimizing.

Good reuse saves CPU and makes performance predictable. Poor reuse shows up as:
- **High compile CPU**: the same query shape compiled thousands of times with different literals.
- **Plan cache bloat**: memory taken by single-use plans, which pushes out useful plans and data pages.

Reuse also has a downside. A plan built for one parameter value is reused for every value. That is the *parameter sensitivity* problem (separate lesson). Recompilation (also a separate lesson) is the process that throws plans away.

### 2. Core concepts and vocabulary
- **Plan cache**: memory holding compiled plans (and some other cache entries), split into cache stores: *Object Plans* (procedures, functions, triggers), *SQL Plans* (ad hoc and prepared statements) and *Bound Trees* (views, defaults).
- **Compiled plan**: the reusable, read-only plan. **Execution context**: per-execution data (parameter values, local variables) created from the compiled plan for each concurrent user.
- **plan_handle / sql_handle**: identifiers for a cached plan and for the batch text.
- **Ad hoc query**: a statement sent with literal values (`WHERE CustomerID = 42`). It is reused only if the next query matches **exactly**, including case, spaces and comments.
- **Simple parameterization (auto-parameterization)**: SQL Server replaces literals with parameters for very simple, "safe" queries (trivial plans) so they can share one plan.
- **Forced parameterization**: a database setting (`PARAMETERIZATION FORCED`) that parameterizes almost all literals.
- **Prepared workload**: parameterized statements: stored procedures, `sp_executesql`, or the client prepare/execute model (`sp_prepare`/`sp_execute`).
- **Optimize for ad hoc workloads**: a server setting that stores only a small **plan stub** the first time a query runs and the full plan only on the second execution.
- **query_hash / query_plan_hash**: hashes that identify queries with the same *shape* (ignoring literals) and plans with the same shape. Useful for finding near-duplicate ad hoc queries.
- **Plan cache key / cache attributes**: besides text, things like SET options, user's default schema (for unqualified names), and database ID must match for reuse.

### 3. How it works, step by step
1. A batch arrives. SQL Server hashes the text and looks it up in the cache, together with the cache key attributes (database, SET options, and so on).
2. **Hit:** the compiled plan is reused. An execution context is created or reused and the query runs. `usecounts` goes up.
3. **Miss:** the batch is compiled. For a simple query, simple parameterization may produce a parameterized form (you see `@1`, `@2` in the plan text, and the cached object type is *Prepared*), and the original ad hoc text gets a small *shell* entry pointing to it.
4. The plan is inserted into the cache with a **cost** based on how expensive it was to compile.
5. **Aging:** when memory is needed, a background process lowers each plan's current cost. A plan whose cost reaches zero, and which is not in use, is removed. Each reuse resets its cost. So cheap, rarely used plans leave first and expensive, frequently used plans stay.
6. Plans also leave on: memory pressure, `DBCC FREEPROCCACHE`, some server configuration changes, database state changes (restore, detach, offline, changes to some database options), and schema or statistics changes that make the plan invalid (that last one is a *recompile*).

**Which kinds of code reuse plans:**
| Code style | Reuse | Note |
|---|---|---|
| Ad hoc with literals | Only exact text match | Each new value = new plan |
| Ad hoc, simple query | Via simple parameterization | Only trivial, safe queries qualify |
| Stored procedure | Yes, by object | Best for reuse and for security |
| `sp_executesql` with parameters | Yes, by parameterized text | Best for dynamic SQL |
| `EXEC('…' + @value)` string concatenation | No | Also an SQL injection risk |
| Prepare/execute (ADO.NET/JDBC with parameters) | Yes | Parameter **types and lengths** must be consistent |

### 4. Worked example
```sql
-- Clean test environment only!
DBCC FREEPROCCACHE;

-- Ad hoc with literals: three different texts → three plans
SELECT soh.SalesOrderNumber FROM Sales.SalesOrderHeader AS soh
JOIN Sales.SalesOrderDetail AS sod ON sod.SalesOrderID = soh.SalesOrderID
WHERE soh.CustomerID = 29825;
-- (repeat with 29672 and 29734)

-- Parameterized: one plan reused
EXEC sp_executesql
  N'SELECT soh.SalesOrderNumber FROM Sales.SalesOrderHeader AS soh
    JOIN Sales.SalesOrderDetail AS sod ON sod.SalesOrderID = soh.SalesOrderID
    WHERE soh.CustomerID = @CustomerID',
  N'@CustomerID int', @CustomerID = 29825;
-- (repeat with 29672 and 29734)

-- Look at the cache
SELECT cp.usecounts, cp.objtype, cp.cacheobjtype, cp.size_in_bytes, st.text
FROM sys.dm_exec_cached_plans AS cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) AS st
WHERE st.text LIKE N'%SalesOrderNumber%' AND st.text NOT LIKE N'%dm_exec%';
```
What you see:
- Three `Adhoc` rows with `usecounts = 1`, one per literal (a join query is not trivial, so simple parameterization did not apply).
- One `Prepared` row with `usecounts = 3` for the `sp_executesql` version.

**Find duplicate ad hoc shapes in production:**
```sql
SELECT TOP (20) qs.query_hash,
       COUNT(*)                     AS plans_in_cache,
       SUM(qs.execution_count)      AS executions,
       MIN(st.text)                 AS sample_text
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
GROUP BY qs.query_hash
HAVING COUNT(*) > 1
ORDER BY plans_in_cache DESC;
```
Many plans with the same `query_hash` = one query shape compiled again and again. Parameterize it in the application, wrap it in a procedure, or (as a database-level fix) use forced parameterization or a plan guide.

**How much of the cache is single-use?**
```sql
SELECT objtype, COUNT(*) AS plans,
       SUM(CAST(size_in_bytes AS bigint)) / 1048576 AS size_mb,
       SUM(CASE WHEN usecounts = 1 THEN 1 ELSE 0 END) AS single_use
FROM sys.dm_exec_cached_plans
GROUP BY objtype ORDER BY size_mb DESC;
```

### 5. Under the hood
- The cache is a set of **hash tables** (cache stores) keyed by text/object ID plus cache key attributes. `sys.dm_exec_plan_attributes(plan_handle)` lists the attributes. Those with `is_cache_key = 1` (for example `set_options`, `user_id`, `dbid`, `objectid`) must all match for reuse. Two connections with different `ANSI_NULLS` or `ARITHABORT` settings get separate plans. This is the classic "fast in SSMS, slow from the app" cause: SSMS sets `ARITHABORT ON` by default and most drivers don't, so the two get different cached plans (and possibly different sniffed parameters).
- **Unqualified object names** (`SELECT … FROM Orders`) make the plan depend on the user's default schema, so users with different default schemas can't share it. Always write `dbo.Orders`.
- **Plan stub** (optimize for ad hoc): first execution caches only about 300 bytes (`cacheobjtype = Compiled Plan Stub`). A second execution compiles again and caches the full plan. The cost is one extra compile for queries that repeat. The gain is far less memory wasted on one-off queries.
- **Client parameter types**: if an ORM sends `nvarchar(5)` for "Smith" and `nvarchar(7)` for "Johnson", these are different parameter declarations, so different plans. Fix by setting a fixed parameter length in the client.
- **Cache size limits**: the plan cache can use a large share of buffer pool memory (the limit scales with *target memory*, around 75% of the first 4 GB plus smaller percentages above). Under pressure, plans compete with data pages.

### 6. Performance and design implications
- Parameterize high-frequency queries. It cuts CPU and gives stable performance. Stored procedures and `sp_executesql` are the standard tools.
- Turn on **optimize for ad hoc workloads** on servers with a lot of ad hoc traffic. It is almost always safe.
- Keep connection SET options consistent across applications, and schema-qualify object names.
- Watch the ratio of `SQL Compilations/sec` to `Batch Requests/sec` (perfmon *SQLServer:SQL Statistics*). Above about 10–20% of batches compiling suggests poor reuse.
- Reuse is not always good. If one plan cannot fit all parameter values, see *Parameter Sensitivity*.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_exec_cached_plans` | One row per cached plan: usecounts, objtype, size | Cache composition, single-use plans |
| `sys.dm_exec_query_stats` | Per-statement runtime totals, `query_hash`, `query_plan_hash` | Top consumers, duplicate shapes |
| `sys.dm_exec_plan_attributes(plan_handle)` | Cache key attributes (set_options…) | "Why do I have two plans?" |
| `sp_configure 'optimize for ad hoc workloads', 1` | Stub on first execution | Ad hoc-heavy servers |
| `ALTER DATABASE … SET PARAMETERIZATION FORCED` | Parameterizes most literals | Apps you can't change, with many ad hoc shapes |
| `EXEC sp_executesql @sql, @params, …` | Parameterized dynamic SQL | All dynamic SQL |
| `DBCC FREEPROCCACHE (plan_handle)` | Removes **one** plan | Evicting a single bad plan |
| `ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE` | Clears one database's plans | Rarely; test environments |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Stored procedures / `sp_executesql`: repeated queries | Strong data skew → one plan may not fit all values |
| Ad hoc literals: rare reporting queries where each value deserves its own plan | High-frequency OLTP |
| Forced parameterization: legacy app floods cache | It also exposes many queries to parameter sniffing |
| Optimize for ad hoc: lots of single-use text | Almost no downside; slight extra compile for repeats |
| `DBCC FREEPROCCACHE` (whole cache) | Production: every query recompiles and CPU spikes |

### 9. Common mistakes
- Building dynamic SQL by concatenating values → a plan per value plus SQL injection → `sp_executesql` with parameters.
- ORM sending variable-length string parameters → many plans for one query → configure fixed parameter sizes.
- Testing in SSMS with different SET options than the app → different plan, can't reproduce "slow in app" → match `ARITHABORT` and the other options, or get the app's plan from the cache.
- Flushing the whole cache to fix one slow query → CPU spike, the real cause (sniffing, stats) remains → evict one plan, then fix the cause.
- Unqualified object names → plans per user schema → always use `schema.object`.

### 10. 🎯 Interview kit
**30-second answer:** "Plans are expensive to compile, so SQL Server caches them and reuses them when an identical query arrives with matching cache keys such as SET options and database. Stored procedures, sp_executesql and prepared statements reuse plans across values; ad hoc literals only match exact text unless simple or forced parameterization kicks in. Plans age out by a cost-based clock under memory pressure. I monitor single-use plans and compile rates, parameterize hot queries and enable optimize for ad hoc workloads."
**Structure a longer answer:** why cache → lookup and cache keys → kinds of parameterization → aging and eviction → measuring (DMVs, compilations/sec) → the reuse vs parameter sensitivity trade-off.
**Likely follow-ups:** *Why is a query fast in SSMS but slow in the app?* → different SET options (ARITHABORT) → different plan and sniffed values. *What does optimize for ad hoc do?* → stub first, full plan on second call. *How do you find plan cache bloat?* → `sys.dm_exec_cached_plans` grouped by objtype with usecounts = 1.
**Signals interviewers look for:** knows reuse depends on exact text plus SET options; can name parameterization methods; knows reuse has a trade-off (sniffing); never flushes the whole cache casually.

### 11. Check yourself
- [ ] Show three ad hoc plans vs one prepared plan for the same query with three values.
- [ ] Use `sys.dm_exec_plan_attributes` to explain why one procedure has two cached plans.
- [ ] Write the `query_hash` query that finds repeated ad hoc shapes.
- [ ] Explain how plan aging decides which plan to remove first.
- [ ] Explain the trade-off of `PARAMETERIZATION FORCED`.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Best Practices for Plan Cache Management
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — load customer names › Chapter 23 A Query Tuning Methodology
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 6 Using The Query Store For Query Performance And Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Best Practices for Plan Reuse
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Best Practices Recap for Plan Cache
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Plan Cache and Reuse
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Execution Plan Reuse
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Execution Plan Caching and Reuse

