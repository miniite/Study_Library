---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Index Tuning in Practice

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Index tuning is a loop — find the expensive query, read its plan, change one thing (predicate shape, index, or maintenance), and measure reads and time again — keeping the write cost of every index in view.

### 1. The big picture
Most slow queries come from a small set of causes: a missing index, a predicate that can't use an existing index, a plan that does too many lookups, stale statistics, or an index set that is wrong for the write pattern. Practical tuning is mostly *recognising which one* and applying the smallest fix.

The order that works: (1) understand and measure the problem — which query, how often, how long, what resources; (2) read the execution plan for scans, lookups, sorts, spills and estimate errors; (3) fix the **query shape** (sargable predicates, narrow select list) before adding structures; (4) add or change an index only when the plan shows it will remove work; (5) maintain indexes and statistics; (6) re-measure, and keep the baseline.

### 2. Core concepts and vocabulary
- **Missing index** — a predicate/join/sort column with no supporting index → scan.
- **Non-sargable predicate** — a condition on a column wrapped in a function/expression/conversion, e.g. `YEAR(OrderDate) = 2025`.
- **Key lookup** — extra per-row hop to the base table for columns not in a nonclustered index.
- **Logical reads** — pages read from cache; the most stable, repeatable measure of work.
- **Fragmentation** — index pages out of logical order or sparsely filled after inserts/updates/deletes/shrinks.
- **Fill factor** — percentage of each leaf page filled at build time, leaving room for growth.
- **Right-edge (last-page) contention** — many concurrent inserts into the highest key of an ever-increasing index.
- **Write amplification** — each index adds work to every insert/update/delete.
- **Index maintenance** — rebuild/reorganize, statistics updates, bloat removal.

### 3. How it works, step by step
A tuning session:
1. **Reproduce and baseline.** Run the query with realistic parameters; record duration, CPU, logical reads, rows. Use `SET STATISTICS IO, TIME ON` / `EXPLAIN (ANALYZE, BUFFERS)`. Don't capture the actual plan and IO stats while measuring timing — they distort it.
2. **Read the plan.** Scan on a big table? Key Lookup repeated 50 k times? Sort or hash spill? Estimated vs actual rows far apart? Implicit conversion warnings?
3. **Fix the predicate.**
   ```sql
   -- non-sargable
   WHERE YEAR(OrderDate) = 2025
   -- sargable
   WHERE OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'
   ```
4. **Trim the select list.** `SELECT *` forces lookups and prevents covering.
5. **Index design.** Equality columns first, then sort, then range; `INCLUDE` the output columns if the query is hot and result rows many.
6. **Test the change** on production-like data; compare reads and neighbours' plans.
7. **Deploy safely**, then re-check usage after a business cycle.

### 4. Worked example
```sql
-- Problem: MIN over a big table does a full scan
SELECT MIN(UnitPrice) FROM Sales.SalesOrderDetail;
-- Clustered Index Scan -> Stream Aggregate; 1,248 logical reads, ~15 ms for ONE value

CREATE INDEX IX_Detail_UnitPrice ON Sales.SalesOrderDetail (UnitPrice);
-- Same query: Index Scan (TOP 1 / first leaf) -> a few reads
-- MAX works the same way (read the last leaf)

-- Problem: OR over two columns
SELECT * FROM Orders WHERE CustomerID = 101 OR SalesPersonID = 55;
-- Fix: two seeks, concatenated
SELECT * FROM Orders WHERE CustomerID = 101
UNION ALL
SELECT * FROM Orders WHERE SalesPersonID = 55 AND CustomerID <> 101;

-- Problem: ETL loads 50 M rows into an indexed table
ALTER INDEX ALL ON dbo.FactSales DISABLE;        -- (nonclustered only)
-- bulk load ...
ALTER INDEX ALL ON dbo.FactSales REBUILD;
```
Walkthrough: In the first case the "work" isn't rows returned, it's pages touched; an ordered index reduces `MIN/MAX` to reading one end of the leaf chain. The second shows that a rewrite can let two indexes each seek (the book's example went 2.4 ms → 1.56 ms, with reads equal and CPU the difference). The third illustrates that load-time index maintenance dominates bulk loads and that staging tables read once by a transform often need **no** indexes.

### 5. Under the hood
- **Fragmentation.** Page splits occur when a row must be inserted into a full page: half the rows move to a new page that may be physically far away, leaving both half-empty. Effects: more pages per scan, more read-ahead misses, lower buffer-cache efficiency. Causes: random inserts (GUID keys), updates that grow rows, deletes, `SHRINKDATABASE`. Remedy by threshold: `REORGANIZE` for moderate fragmentation (online, incremental), `REBUILD` for heavy (new structure, updates statistics with a full scan). On SSD with cached data, many shops de-emphasise fragmentation and focus on statistics and page density.
- **Heap forwarding.** In a heap, an update that no longer fits creates a forwarding pointer; scans and lookups hop twice. Fix: create a clustered index or rebuild the heap.
- **Monotonic key hotspot.** An ascending key (`timestamp`, identity) sends all concurrent inserts to the rightmost leaf: latch contention (`PAGELATCH_EX` on one page), not lock contention. Mitigations: partition by a hash/bucket prefix (`(bucket, ts)`), use OPTIMIZE_FOR_SEQUENTIAL_KEY (SQL Server 2019+), spread keys (UUIDv7 is time-ordered and has the same hotspot trade-off; random UUIDv4 spreads but fragments), batch inserts, or use an in-memory/append store.
- **IN-list with thousands of values.** Large literal lists hit parser/plan limits and produce poor plans. Load values into a temp table / table-valued parameter / `unnest(array)` and join on an indexed key.
- **COUNT(*) on huge tables.** Needs to visit every row or at least the narrowest index. Row stores answer with a scan of the smallest index; columnstore/warehouses use metadata or compressed segments; maintained counter tables or approximate stats (`pg_class.reltuples`, `sys.dm_db_partition_stats`) give instant approximate counts.
- **Leaderboards: top-N is easy, "rank of user X" is not.** A B-tree gives rank only by counting entries ahead — O(n) without order-statistic augmentation. Approaches: count query with an index on score (`COUNT(*) WHERE score > ?`, fine for moderate size), a sorted-set structure (Redis ZSET with O(log n) rank), or bucketed counts + local count.
- **N+1 queries.** A loop of single-row indexed lookups pays parse, round-trip and per-call overhead N times; a single join/`IN` via an index does one plan and one round trip.
- **Latency budgets.** In a 200 ms page with a 120 ms DB query, index tuning attacks query time; pooling attacks connection acquisition; serialization/ORM mapping attacks the transfer cost. Measure each before tuning.

### 6. Performance and design implications
- **Index selection checklist:** WHERE equality → sort → range; JOIN keys (child FK); ORDER BY/GROUP BY; covering for hot, narrow queries.
- **Write budget:** ten indexes on a constantly written table means ten B-tree updates per insert; keep only those with measurable read benefit.
- **Low selectivity:** a column where one value is 99 % of rows is a poor index candidate for that value, great for the rare values — use a filtered/partial index.
- **Long free-text columns** aren't B-tree indexable in practice (key size limits, huge pages); use full-text/trigram or hash a normalized form.
- **Critical-but-rare queries** (once an hour on checkout path) can justify an index: weigh latency risk to revenue against measured write overhead; mitigate cost via filtered indexes or off-peak builds.
- **Staging tables** loaded nightly and read once: usually no index; add one only on the join/incremental-extract key (`load_timestamp`, `batch_id`) if filtered repeatedly.
- **Wide analytic tables (100+ columns):** don't index every column; use columnstore or partition/sort strategies.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO, TIME ON` / `EXPLAIN (ANALYZE, BUFFERS)` | Reads and time | Every tuning step |
| `ALTER INDEX … REORGANIZE` / `REBUILD` (`WITH (FILLFACTOR = 90, ONLINE = ON)`) | Defragment / rebuild | Measured fragmentation, page density issues |
| `UPDATE STATISTICS … WITH FULLSCAN`, `ANALYZE` | Refresh stats | After big loads, estimate drift |
| `sys.dm_db_index_physical_stats` | Fragmentation, page density | Maintenance decisions |
| `DROP INDEX` | Removes structure; data untouched | Pruning |
| `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` | Reduce last-page insert contention | Identity/timestamp clustered keys |
| `DISABLE`/`REBUILD` around bulk loads | Avoid per-row index maintenance | ETL into indexed targets |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rewriting the predicate fixes the scan | The query is already sargable; adding index won't help |
| Add a covering index for a hot report | Table receives heavy writes and the report is nightly |
| Drop/disable indexes around bulk loads | Table must stay queryable during the load |
| Partition/bucket keys for insert hotspot | Range queries need contiguous time order (adds union of buckets) |
| REBUILD for heavy fragmentation | Tiny indexes (<1000 pages) — not worth it |

### 9. Common mistakes
- **Adding an index first** → bigger write cost, maybe no gain → check predicate shape and select list first.
- **Indexing a boolean/low-cardinality column alone** → optimizer ignores it → filtered index or composite.
- **Timing with plans/IO stats on** → distorted duration → measure separately.
- **Rebuilding all indexes nightly** → log growth, blocking → threshold-based maintenance and statistics focus.
- **Dropping an index and expecting data loss** → none; the table is intact, but plans change → check dependents (FKs, hints, unique constraints).
- **Using `NOT IN`/functions on join keys** → scans → rewrite as `NOT EXISTS`, bare columns.
- **Per-row queries in app loops** → N round trips → set-based joins.

### 10. 🎯 Interview kit
**30-second answer:** I measure first (reads, time, plan), fix the query shape — sargable predicates and a narrow select list — then add the smallest index that removes the scan or lookup, and verify with reads. I weigh every index against write cost and keep statistics and fragmentation maintained.

**Structure a longer answer:** baseline → plan reading (scan, lookup, sort, estimates) → rewrite → index → validate → operations (maintenance, monitoring, retirement) → example.

**Likely follow-ups:**
- *Why not index every column?* → Write amplification, storage, optimizer plan-space, maintenance.
- *Drop indexes before a bulk load?* → Avoids per-row maintenance and splits; rebuild afterwards builds sorted, compact pages and fresh stats.
- *Right-edge contention?* → Concurrent inserts into last leaf; spread keys/partition/sequential-key optimization.
- *IN with thousands of values?* → Temp table/TVP join on an index, or batching.
- *Rank lookup in a leaderboard?* → Needs order-statistic structure or a count; B-tree alone gives top-N.

**Signals interviewers look for:** methodical measurement, preference for query rewrites, awareness of write costs and maintenance, precise vocabulary.

### 11. Check yourself
- [ ] Convert three non-sargable predicates to sargable forms.
- [ ] Explain page splits and name two mitigations.
- [ ] Decide whether to index a nightly-loaded staging table and justify.
- [ ] Describe how to tune a query that spends 120 ms in the DB on a 200 ms page budget.
- [ ] Explain why `COUNT(*)` stays slow on big row-store tables and what alternatives exist.
<!-- explainer:end -->

## Go deeper

- [Indexing Read State at Scale](<Indexing_Read_State_at_Scale.md>)
- [Index Strategy and Governance at Scale](<Index_Strategy_and_Governance_at_Scale.md>)

## Practice questions

- [Basic] [As a data engineer, when a staging table is loaded fresh every night and then read once by a transform, is it worth indexing it? Explain your reasoning.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20269_As_a_data_engineer_when_a_staging_table_is_loaded_fresh_ever.md>)
- [Intermediate] [How would you use a database's slow query log to build a shortlist of tables in your pipeline that most need an index review?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20366_How_would_you_use_a_database_s_slow_query_log_to_build_a_sho.md>)
- [Intermediate] [A report that filters on a date range and groups by category is slow. What single composite index would you try first and how would you validate it against EXPLAIN?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20081_A_report_that_filters_on_a_date_range_and_groups_by_category.md>)
- [Deep] [You must add an index to a large production source table used for extraction. How would you do it without blocking the running pipeline?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20361_You_must_add_an_index_to_a_large_production_source_table_use.md>)
- [Deep] [Explain how Adaptive Query Execution in Spark 3 can switch a sort-merge join to a broadcast join at runtime and coalesce shuffle partitions.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20382_Explain_how_Adaptive_Query_Execution_in_Spark_3_can_switch_a.md>)
- [Deep] [A batch job rebuilds a large reporting table nightly and query latency spikes for the first hour after. How do you tie that to stale statistics or a cold cache and fix it durably?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19670_A_batch_job_rebuilds_a_large_reporting_table_nightly_and_que.md>)
- [Deep] [Your write-heavy table needs an index for a new reporting feature but the write path is already at IOPS limits. How do you quantify the added write amplification before committing?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19828_Your_write-heavy_table_needs_an_index_for_a_new_reporting_fe.md>)
- [Expert] [A query is fast for 99% of users but a 'power user' with 2M rows in a child table times out. How do you design indexes and query shape to be robust to this data-distribution outlier?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19722_A_query_is_fast_for_99_of_users_but_a_power_user_with_2M_row.md>)

## Related concepts

- [Query Rewrites and Plan Stability](<../../Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Leftmost Prefix Rule](<../../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Covering Indexes and Index Strategy for Sharding](<../How_Indexes_Are_Used/Covering_Indexes_and_Index_Strategy_for_Sharding.md>) *(Indexing)*
- [Index Strategy and Governance at Scale](<Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>) *(System Design Core)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 10 Index Behaviors
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 11 Key Lookups And Solutions
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DBCC IND
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties › Hash Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Balance Index Count
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index to Your Environment › Chapter 9
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Performance Considerations with PIVOT/UNPIVOT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Identifying Bottlenecks
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Columnstore Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Negative Impacts of Indexes on Performance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE › Physical Data on Pages
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Row Order (Vertipaq) Optimization
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Other Compression Algorithms
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › Columnstore Index Memory Usage
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Introducing SQL Server › 1.2.1 Overview of the Database Engine
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3.1 #6 Always storing whole numbers in INT
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.2 #44 Failing to rebuild indexes after data file shrink
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.1 #67 Assuming internal fragmentation is always bad
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Cautions and limits

