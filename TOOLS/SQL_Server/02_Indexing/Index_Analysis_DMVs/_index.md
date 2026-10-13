---
id: "TOOLS/SQL_Server/02_Indexing/Index_Analysis_DMVs"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Index Analysis with DMVs

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Three families of dynamic management objects tell you how indexes are really used — **usage stats** (how often each index is sought, scanned, looked up, updated), **operational stats** (low-level I/O, locking, latching, splits) and **missing-index DMOs** (what the optimizer wished it had) — and reading them correctly, with their reset and accuracy limits, is the evidence base for every indexing decision.

### 1. The big picture
The design-methodology lesson described a Monitor → Analyze → Implement cycle. This lesson is its toolbox. SQL Server tracks index activity automatically, at almost no cost, in memory-resident DMVs (dynamic management views) and DMFs (functions) — together, DMOs. They answer questions like:

- Which indexes are never read but constantly updated (pure overhead)?
- Which indexes cause many key lookups (candidates for INCLUDE)?
- Where is lock or latch contention concentrated?
- Which indexes would the optimizer have liked to use, and how much would they help?

All of them are **cumulative since the last restart** (or since the database came online / the metadata was evicted), not historical. That single fact drives most of the mistakes people make with them. The structural DMF `sys.dm_db_index_physical_stats` (fragmentation, page density) belongs to the maintenance lesson.

### 2. Core concepts and vocabulary
- **DMO / DMV / DMF** — dynamic management object; view (no parameters) or function (with parameters).
- **`sys.dm_db_index_usage_stats`** — per index: counts and last times of seeks, scans, lookups and updates, split into *user* and *system* activity. Counts *plan operator executions*, not rows.
- **User vs system columns** — user = queries; system = internal activity like statistics updates and index maintenance.
- **`sys.dm_db_index_operational_stats(db, object, index, partition)`** — per index *partition*: leaf/non-leaf inserts, updates, deletes, ghosts, page allocations (splits), merges, range scans, singleton lookups, row/page lock counts and waits, latch and I/O latch waits, lock escalations, compression attempts.
- **Missing-index DMOs** — `sys.dm_db_missing_index_details` (the suggested index), `…_columns` (DMF listing its columns), `…_groups` (links index to group), `…_group_stats` (benefit statistics), and since 2019 `…_group_stats_query` (per query).
- **Equality / inequality / included columns** — how the missing-index feature classifies columns: `=` predicates, other comparisons (`>`, `<`, `<>`, BETWEEN…), and output columns for covering.
- **avg_total_user_cost** — average optimizer cost of the queries that would benefit.
- **avg_user_impact** — estimated % reduction of that cost if the index existed.
- **Persisted snapshot** — your own table into which you copy DMO output periodically so history survives restarts.

### 3. How it works, step by step
**A. Usage stats — `sys.dm_db_index_usage_stats`**
1. A row appears for an index the first time it is used after startup. *No row* = not used since the counters began.
2. Every time a plan operator touches the index, the matching counter increments once — a seek that returns 10,000 rows counts as one seek; a scan of the whole index counts as one scan.
3. `user_lookups` increments on the *clustered index* (or heap) when a key/RID lookup happens — so lookups show up on the clustered index, not on the NC index that triggered them.
4. `user_updates` counts insert/update/delete operations that modified the index (statement-level, not rows).
5. Interpretation patterns:

| Pattern | Meaning | Action |
|---|---|---|
| Seeks high, updates moderate | Healthy, useful index | Keep |
| Scans high, seeks ~0 on a large NC index | Used as a narrow copy of the table, or poor key order | Review queries/key order |
| Reads 0, updates high | Pure write overhead | Candidate to disable/drop (after a full cycle) |
| `user_lookups` high on clustered index | Many key lookups | Find the NC indexes/queries; add INCLUDEs |
| No row at all | Not touched since restart | Check uptime before concluding |

**B. Operational stats — `sys.dm_db_index_operational_stats`**
1. Pass `DB_ID()` and NULLs for all objects, or specific IDs. One row per index partition.
2. Counters are kept in the metadata cache; if the object's metadata is evicted (memory pressure), counters can reset — another reason to snapshot.
3. Useful groups of columns:
   - *DML volume:* `leaf_insert_count`, `leaf_update_count`, `leaf_delete_count`, `leaf_ghost_count`.
   - *Access patterns:* `range_scan_count`, `singleton_lookup_count`.
   - *Allocation cycle:* `leaf_allocation_count` (new leaf pages — inserts at the end *and* splits), `nonleaf_allocation_count`, `leaf_page_merge_count`.
   - *Locking:* `row_lock_count`, `row_lock_wait_count`, `row_lock_wait_in_ms`, `page_lock_*`, `index_lock_promotion_attempt_count`, `index_lock_promotion_count` (escalations).
   - *Latching / I/O:* `page_latch_wait_count/_in_ms` (in-memory contention, e.g. hot last page), `page_io_latch_wait_count/_in_ms` (waiting for disk reads).
   - *Forwarded records* (heaps): `forwarded_fetch_count`.
4. Divide waits by counts to get average wait per lock/latch; rank indexes by total wait ms to locate contention.

**C. Missing-index DMOs**
1. During compilation, when the optimizer sees that an index would have reduced a plan's cost, it records a suggestion: table, equality columns, inequality columns, included columns.
2. `…_details` stores one row per suggestion (`index_handle`, `database_id`, `object_id`, `statement` = fully qualified table name, the three column lists).
3. `…_groups` maps each index to a group (always one-to-one in practice).
4. `…_group_stats` accumulates how often it would have been used (`user_seeks`, `user_scans`, `unique_compiles`) and how much it would help (`avg_total_user_cost`, `avg_user_impact`).
5. Rank by an *improvement measure*: `avg_total_user_cost × (avg_user_impact / 100) × (user_seeks + user_scans)`.
6. 2019+: `sys.dm_db_missing_index_group_stats_query` gives the same per `query_hash`, so you can see which query wanted it.

**Limitations of missing-index data** (be explicit about them):
- *Queue size:* a fixed cap on stored suggestions (500 in older documentation, 600 in newer); once full, new suggestions are not recorded.
- *Depth:* considers only the current plan; after you create the index a new plan may suggest another one.
- *Accuracy:* columns are listed in table-column order within each category, not the best key order; overlapping suggestions are not merged; included-column lists can be huge.
- *Types:* only nonclustered rowstore indexes — never clustered, columnstore, filtered or indexed views.
- *No write cost:* it never weighs maintenance overhead.
- *No query link (before 2019):* a suggestion could come from a query run once or a thousand times a second.
- *Reset:* cleared on restart, database offline, and for a table when its metadata changes (e.g. an index is created on it).

### 4. Worked example
```sql
-- 1) Usage: indexes ranked by write overhead vs reads, current database
SELECT OBJECT_NAME(i.object_id) AS table_name, i.name AS index_name, i.type_desc,
       ISNULL(us.user_seeks,0) AS seeks, ISNULL(us.user_scans,0) AS scans,
       ISNULL(us.user_lookups,0) AS lookups, ISNULL(us.user_updates,0) AS updates,
       us.last_user_seek, us.last_user_scan,
       ps.row_count, ps.reserved_page_count * 8 / 1024 AS size_mb
FROM sys.indexes i
JOIN sys.objects o ON o.object_id = i.object_id AND o.is_ms_shipped = 0
LEFT JOIN sys.dm_db_index_usage_stats us
       ON us.database_id = DB_ID() AND us.object_id = i.object_id AND us.index_id = i.index_id
JOIN sys.dm_db_partition_stats ps ON ps.object_id = i.object_id AND ps.index_id = i.index_id
WHERE i.index_id > 1 AND i.is_primary_key = 0 AND i.is_unique_constraint = 0
ORDER BY (ISNULL(us.user_seeks,0)+ISNULL(us.user_scans,0)+ISNULL(us.user_lookups,0)) ASC,
         updates DESC;

-- How long have counters been accumulating?
SELECT sqlserver_start_time FROM sys.dm_os_sys_info;

-- 2) Operational: where is lock/latch waiting?
SELECT TOP (10) OBJECT_NAME(os.object_id) AS table_name, i.name AS index_name,
       os.row_lock_wait_in_ms, os.page_lock_wait_in_ms, os.page_latch_wait_in_ms,
       os.page_io_latch_wait_in_ms, os.index_lock_promotion_count,
       os.leaf_allocation_count, os.forwarded_fetch_count
FROM sys.dm_db_index_operational_stats(DB_ID(), NULL, NULL, NULL) os
JOIN sys.indexes i ON i.object_id = os.object_id AND i.index_id = os.index_id
ORDER BY os.row_lock_wait_in_ms + os.page_lock_wait_in_ms + os.page_latch_wait_in_ms DESC;

-- 3) Missing indexes ranked by estimated improvement
SELECT TOP (20)
       CAST(gs.avg_total_user_cost * gs.avg_user_impact / 100.0
            * (gs.user_seeks + gs.user_scans) AS decimal(18,1)) AS improvement,
       d.statement AS table_name, d.equality_columns, d.inequality_columns,
       d.included_columns, gs.user_seeks, gs.unique_compiles, gs.avg_user_impact
FROM sys.dm_db_missing_index_group_stats gs
JOIN sys.dm_db_missing_index_groups g ON g.index_group_handle = gs.group_handle
JOIN sys.dm_db_missing_index_details d ON d.index_handle = g.index_handle
WHERE d.database_id = DB_ID()
ORDER BY improvement DESC;
```
**How to read it.**
- Query 1: an index with `seeks + scans + lookups = 0`, `updates = 250,000` and 3 GB size after 60 days of uptime is a strong drop candidate — confirm it isn't needed by month-end jobs and doesn't enforce uniqueness, then *disable* it first. If uptime is 2 days, conclude nothing.
- Query 2: a clustered index with high `page_latch_wait_in_ms` and an identity key suggests last-page insert contention; high `row_lock_wait_in_ms` on an NC index points to blocking on a hot range; non-zero `index_lock_promotion_count` shows lock escalation.
- Query 3: suppose the top row is `[dbo].[Orders]`, equality `[Status], [CustomerID]`, inequality `[OrderDate]`, included 12 columns. Don't paste it. Decide key order yourself (which equality column do other queries share?), trim includes to what important queries need, check whether an existing index on `(CustomerID)` can be extended instead, and validate with the query's plan.

### 5. Under the hood
- **Where counters live.** Usage and operational counters sit in in-memory structures attached to the object's metadata; nothing is written to disk, which is why they are cheap and volatile. Rebuilding an index resets its usage row in some versions (2012–2014 era behavior; fixed later) — another reason to snapshot with timestamps and compute deltas.
- **Counting granularity.** Usage stats count *operator executions*: a nested-loop plan with 5,000 lookups registers 5,000 `user_lookups` on the clustered index, but a hash join scanning an index once registers one scan. So "scans" can understate the work (one scan = millions of rows) and "lookups" can look alarming but be cheap per execution.
- **Missing-index generation.** It happens during optimization, in the "index matching" phase, comparing the predicates against available indexes; the cost/impact numbers are optimizer *estimates* in its internal cost units, not measured time — they inherit any cardinality-estimation errors.
- **Measuring index cost directly.** Adding an index raises the cost of writes: in a book example, an `UPDATE` on a heap needed 29 reads, and after adding a clustered index on the updated column it needed 42 reads plus a worktable for sorting — the index had to be reordered. Use Extended Events or `STATISTICS IO` to measure such costs, not just the DMVs.

### 6. Performance and design implications
- Snapshot usage and operational stats (e.g. hourly or daily) into tables with a timestamp; compute deltas between snapshots and handle resets (counter goes down → restart).
- Ratio heuristic: indexes where `user_updates` is far larger than all reads, on write-heavy tables, are the first to question.
- Missing-index data is a lead for investigation, best combined with Query Store to see the query and its frequency.
- Querying the DMOs is cheap, except `sys.dm_db_index_operational_stats` across thousands of partitions — scope it to a database.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_usage_stats` | Seeks/scans/lookups/updates per index | Unused or lookup-heavy indexes |
| `sys.dm_db_index_operational_stats(DB_ID(),NULL,NULL,NULL)` | Low-level DML, lock, latch, I/O, split counters | Contention and churn analysis |
| `sys.dm_db_missing_index_details / _groups / _group_stats` | Optimizer suggestions with benefit | Create candidates |
| `sys.dm_db_missing_index_group_stats_query` (2019+) | Suggestions per query hash | Linking a suggestion to queries |
| `sys.dm_db_missing_index_columns(index_handle)` | Columns of one suggestion with usage type | Detailed review |
| `sys.dm_db_partition_stats` | Rows and pages per index partition | Index size in reports |
| `sys.dm_os_sys_info.sqlserver_start_time` | Instance start time | Knowing how long counters have run |
| `ALTER INDEX … DISABLE` | Removes data, keeps definition | Reversible test before dropping |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Usage stats to find drop candidates | Uptime is short, or the index enforces a constraint |
| Operational stats to locate lock/latch hot spots | You need per-query detail — use Extended Events / Query Store |
| Missing-index ranking to prioritize investigation | Creating suggestions verbatim — merge, reorder and trim first |
| Disabling an index as a reversible step | Disabling a clustered index — it makes the table inaccessible |

### 9. Common mistakes
- Treating "no row in usage stats" as "unused forever" → dropping indexes needed weekly/monthly → check uptime and persist history.
- Expecting lookups to appear on the NC index → they're recorded on the clustered index/heap → trace back via plans.
- Creating every missing index → overlapping wide indexes, write slowdown → consolidate.
- Trusting suggested column order → suboptimal seeks → decide order from predicates and reuse.
- Confusing `user_updates` with row counts → it counts operations → use operational stats for row-level DML counts.
- Disabling the clustered index "to test" → table becomes unreadable → only disable nonclustered indexes.

### 10. 🎯 Interview kit
**30-second answer:** "I use `sys.dm_db_index_usage_stats` to see seeks, scans, lookups and updates per index — indexes with no reads and many updates are candidates to remove; high lookups on the clustered index point to missing INCLUDEs. `sys.dm_db_index_operational_stats` shows row and page lock waits, latch waits, escalations and page allocations per index. The missing-index DMOs give optimizer suggestions with estimated impact, but they don't merge, order or weigh write cost. Everything resets on restart, so I snapshot the data over a full business cycle."

**Structure a longer answer:** three families → what each answers → reset/accuracy caveats → workflow (snapshot, delta, rank, validate in plans) → scenario: dropped 6 unused indexes on an order table, insert latency fell 30%; or found last-page latch contention via operational stats.

**Likely follow-ups:**
- *Why might a used index show zero seeks?* → counters reset (restart, offline, metadata eviction) or it's used only for scans/lookups/constraint checks.
- *How do you rank missing indexes?* → cost × impact × (seeks + scans), then validate per query.
- *What limitations do missing-index DMVs have?* → cap on entries, no merge, column order, rowstore NC only, no write cost.
- *How to identify lock escalation per index?* → `index_lock_promotion_count`.

**Signals interviewers look for:** knows counters are volatile and operation-based; can interpret patterns; treats suggestions critically; links findings to concrete actions.

### 11. Check yourself
- [ ] Write a query listing NC indexes with zero reads, their updates and size, plus instance uptime.
- [ ] Explain why key lookups are counted on the clustered index.
- [ ] Rank missing-index suggestions and turn the top one into a reviewed, consolidated index.
- [ ] Use operational stats to find the index with the most row-lock wait time.
- [ ] Design a snapshot table and the delta query that handles a restart.
<!-- explainer:end -->

## In this section (study in order)

1. [Index Operational Stats](<Operational_Stats/_index.md>) (L5 Expert)

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbmissingindexgroupstats
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Detecting Missing Indexes via Operator Analysis
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbmissingindexdetails
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — T-SQL Scripts › Explaining the DMOs
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbmissingindexdetails › sys.dmdbmissingindexgroups
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Unindexed Foreign Keys › Unused Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Identifying Missing Indexes from Execution Plans
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — T-SQL Scripts › Missing Index DMOs
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — User Columns

