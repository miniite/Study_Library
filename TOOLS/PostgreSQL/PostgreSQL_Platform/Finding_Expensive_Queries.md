---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Finding_Expensive_Queries"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: ["PostgreSQL"]
---

# Finding Expensive Queries

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Find what's expensive with `pg_stat_statements` (cumulative cost per normalized query), then explain *why* with `EXPLAIN (ANALYZE, BUFFERS)`, and cap the damage with timeouts, sane pooling and autovacuum tuning.

### 1. The big picture
Slow-database incidents have two questions: *which* statements consume the most time/IO overall, and *why* is a given one slow. Individual slow-query logs miss "cheap query run 50,000 times", so you start from aggregated statistics. Then you read the plan to see whether the time is in scanning, joining, sorting/spilling, waiting on locks, or fetching from disk. Finally you prevent recurrence: timeouts, connection limits, and maintenance health (bloat, stale stats).

Build on: query plans, indexes, [vacuum and bloat], and lock diagnostics from the deadlocks lesson.

### 2. Core concepts and vocabulary
- **`pg_stat_statements`** — extension that aggregates executions per *normalized* statement (constants replaced by `$1, $2…`) keyed by `queryid`, user and database.
- **Normalization** — literals are stripped so `WHERE id = 5` and `= 6` count as one entry; `IN (…)` lists of different lengths can still fragment.
- **`total_exec_time`, `mean_exec_time`, `calls`, `rows`, `shared_blks_hit/read`** — the core columns; `total = mean × calls`.
- **`EXPLAIN` vs `EXPLAIN ANALYZE`** — estimate only vs actually run and measure; `ANALYZE` executes the statement (wrap writes in a transaction you roll back).
- **`BUFFERS`** — shows pages hit in `shared_buffers` vs read from OS/disk.
- **Hash spill** — hash/sort exceeding `work_mem` writes temp files (`Batches: > 1`, `Sort Method: external merge`).
- **Parallel query** — gather workers scan/aggregate chunks; bounded by `max_parallel_workers_per_gather`.
- **`idle in transaction`** — session holding a transaction open doing nothing; blocks vacuum and locks.

### 3. How it works, step by step
**Triage loop**
1. Enable: `shared_preload_libraries = 'pg_stat_statements'` (restart), then `CREATE EXTENSION pg_stat_statements;`.
2. Rank by `total_exec_time` (what costs the most), then by `mean_exec_time` (what's slow per call), then by `shared_blks_read` (IO hogs), then by `calls` with tiny means (parse/plan overhead, N+1).
3. Reproduce the statement with real parameters; run `EXPLAIN (ANALYZE, BUFFERS, SETTINGS)`.
4. Compare *estimated rows* vs *actual rows* per node; a 100× miss means stale/insufficient statistics or correlated columns.
5. Fix the cheapest cause: missing index, unsargable predicate, stale stats (`ANALYZE`), spill (`work_mem`), bad join order, or application pattern (N+1, chatty).
6. Reset or diff the stats (`pg_stat_statements_reset()` or snapshot deltas) to confirm.

### 4. Worked example
```sql
-- top consumers
SELECT queryid, calls,
       round(total_exec_time::numeric, 0)           AS total_ms,
       round(mean_exec_time::numeric, 2)            AS mean_ms,
       rows,
       round(100.0 * shared_blks_hit /
             NULLIF(shared_blks_hit + shared_blks_read, 0), 1) AS hit_pct,
       left(query, 100) AS query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- why is this one slow?
EXPLAIN (ANALYZE, BUFFERS, SETTINGS)
SELECT c.region, sum(o.amount)
FROM orders o JOIN customers c USING (customer_id)
WHERE o.order_date >= '2026-01-01'
GROUP BY c.region;
```
What to read in the plan: `Seq Scan … Rows Removed by Filter: 9,800,000` (scanning for few rows → index candidate); `Hash Join … Batches: 8  Disk: 412MB` (spill → raise `work_mem` for this role/session or reduce rows earlier); `Buffers: shared read=…` large and repeated (cold cache/IO-bound); `Workers Planned: 2, Launched: 0` (pool of parallel workers exhausted); `actual time` multiplied by `loops`.

Other triage queries:
```sql
-- sessions in trouble
SELECT pid, state, wait_event_type, wait_event, now()-xact_start AS xact_age, query
FROM pg_stat_activity WHERE state <> 'idle' ORDER BY xact_start;

-- bloat/autovacuum health
SELECT relname, n_live_tup, n_dead_tup, last_autovacuum, last_autoanalyze
FROM pg_stat_user_tables ORDER BY n_dead_tup DESC LIMIT 10;
```

### 5. Under the hood   *(optional at this level)*
- Planner cost = sequential/random page cost × pages + CPU costs × tuples; estimates come from `pg_statistic` (histograms, MCVs, n_distinct). Bad stats → bad plan.
- `HashAggregate` is chosen when the estimated group count fits `work_mem × hash_mem_multiplier`; otherwise the planner sorts and uses `GroupAggregate`, or the hash spills in batches (PG13+ supports hash-agg spill).
- Parallel seq scan splits the table into block ranges handed to workers; planner only considers it above `min_parallel_table_scan_size` and when the estimated gain beats `parallel_setup_cost`. On a busy server it can be slower: workers compete for CPU and the limit `max_parallel_workers` is shared.
- Server-side cursors (`DECLARE … CURSOR`, `FETCH n`) stream results without materializing all rows in the client, but hold a transaction (and snapshot) open for their lifetime.

### 6. Performance and design implications
- A cache hit ratio below ~99% on an OLTP system suggests undersized `shared_buffers`/RAM or scans of cold data; `pg_buffercache` shows what's resident.
- Thousands of tiny queries per request: confirm with high `calls` and low `mean_exec_time`; fix by batching (`= ANY($1)`), joins, or caching — not indexes.
- Connection storms: use a pooler (PgBouncer in transaction mode) and size pool ≈ cores × 2–4; note transaction pooling breaks session state (prepared statements in old versions, session advisory locks, `SET`).
- Guardrails: `statement_timeout` per role, `idle_in_transaction_session_timeout`, `log_min_duration_statement`, `auto_explain` for slow plans.
- Autovacuum falling behind: lower `autovacuum_vacuum_scale_factor` / raise `autovacuum_vacuum_cost_limit` per table with `ALTER TABLE … SET (…)`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pg_stat_statements` | per-query aggregates | first look at load |
| `EXPLAIN (ANALYZE, BUFFERS)` | actual plan + I/O | explain one query |
| `auto_explain.log_min_duration` | log plans of slow queries | production capture |
| `log_min_duration_statement` | log slow statements | spotting outliers |
| `pg_stat_activity`, `pg_locks` | live sessions/locks | incidents |
| `work_mem`, `hash_mem_multiplier` | per-node memory | sort/hash spills |
| `max_parallel_workers_per_gather` | parallel degree | analytic queries |
| `ANALYZE t` / `ALTER TABLE … SET STATISTICS n` | refresh / deepen stats | row-estimate misses |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Raise `work_mem` for one role/session | raising it globally × many connections (OOM) |
| Add an index | write-heavy table with many indexes already |
| Rollup/summary table | data must be real-time and cheap to compute |
| More parallel workers | busy OLTP server, many concurrent analytic queries |
| Pooler in transaction mode | app relies on session state |

### 9. Common mistakes
- Tuning from a single slow execution → it was a cold cache or a lock wait → check `BUFFERS` and wait events.
- Running `EXPLAIN ANALYZE` on `DELETE/UPDATE` in prod → it actually modifies → wrap in `BEGIN … ROLLBACK`.
- Ignoring `rows` mismatches → keeps adding indexes → refresh stats/extended statistics.
- Looking only at `mean_exec_time` → misses the cheap-but-ubiquitous query → sort by `total_exec_time`.
- Relying only on app timeouts → DB query keeps running → set `statement_timeout` server-side.

### 10. 🎯 Interview kit
**30-second answer:** "I start with `pg_stat_statements` sorted by total time to find the heavy hitters, reproduce with `EXPLAIN (ANALYZE, BUFFERS)`, compare estimated and actual rows, and fix the cause — index, stats, memory or application pattern — then verify with fresh stats."
**Structure a longer answer:** aggregate → single plan → root cause → fix → guardrails.
**Likely follow-ups:** *What does BUFFERS add?* → hit vs read. *Why did my plan spill?* → `work_mem`. *Parallelism slower?* → worker contention/overhead. *Many literal variants?* → normalization limits, `IN` lists, `query_id` grouping.
**Signals interviewers look for:** measurement before guessing, awareness of normalization and cumulative stats, safe use of `ANALYZE` on writes.

### 11. Check yourself
- [ ] Write the `pg_stat_statements` query that surfaces the top 5 by IO.
- [ ] Read a plan and say where the time went and what the estimate error is.
- [ ] Explain when raising `work_mem` is the wrong fix.
- [ ] List four guardrails that stop one bad query taking the database down.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../_index.md>)

## Practice questions

- [Intermediate] [You want to know how much of a table's data is actually in shared_buffers to explain cache-miss latency; how would you use the pg_buffercache extension and what does the hit ratio tell you?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36247_You_want_to_know_how_much_of_a_table_s_data_is_actually_in_s.md>)
- [Intermediate] [How would you detect and prevent a runaway query from consuming all DB resources; walk me through statement_timeout, per-role settings, and why an application-level timeout alone is insufficient.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36076_How_would_you_detect_and_prevent_a_runaway_query_from_consum.md>)
- [Deep] [A long-running analytics transaction on the primary is preventing vacuum from removing dead tuples table-wide; explain the xmin horizon concept and how you'd identify and cap the offending session.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36287_A_long-running_analytics_transaction_on_the_primary_is_preve.md>)
- [Deep] [What is the pg_stat_activity view and how is it useful for debugging?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35820_What_is_the_pg_stat_activity_view_and_how_is_it_useful_for_d.md>)
- [Deep] [How would you find and terminate a long-running query blocking others?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36398_How_would_you_find_and_terminate_a_long-running_query_blocki.md>)
- [Deep] [A managed Aurora/RDS Postgres shows high IOPS and cost; how do you attribute it to specific queries and reduce it without just scaling the instance up?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36028_A_managed_Aurora_RDS_Postgres_shows_high_IOPS_and_cost_how_d.md>)
- [Expert] [Under high write load your PostgreSQL is slowing down. How do you diagnose the root cause?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35821_Under_high_write_load_your_PostgreSQL_is_slowing_down_How_do.md>)
- [Expert] [Autovacuum can't keep up on a hot table and bloat is growing; how do you diagnose it with pg_stat_user_tables and tune per-table autovacuum without a maintenance window?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35948_Autovacuum_can_t_keep_up_on_a_hot_table_and_bloat_is_growing.md>)

## Related concepts

- [Index-only Scans](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>) *(Indexing)*
- [Statistics and Sargability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*
- [Query Rewrites and Plan Stability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Excessive tempDB Usage
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Identifying High-Frequency Ad Hoc Queries
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — enable tracing on individual queries in order to obtain detailed information on the interactions
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — A Graph For Testing › Performance Tuning Results
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Portfolio data engineering project › Step 6: Connect a Power BI service to your PostgreSQL database and do some exploratory data analysis.
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — J

