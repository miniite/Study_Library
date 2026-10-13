---
id: "TOOLS/SQL_Server/03_Statistics_and_Cardinality/Statistics_Maintenance"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Statistics Maintenance

**L4 Advanced** · [↑ Statistics and Cardinality](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Let SQL Server create and update statistics automatically (with async update for busy OLTP), and add targeted manual updates (FULLSCAN or larger samples) after big loads, for skewed columns and for statistics that index maintenance doesn't touch.

### 1. The big picture
Statistics describe the data *as it was* when they were built. As rows are inserted, updated and deleted, the histogram drifts away from reality. New key values appear beyond the last step, heavy values change, row counts grow. **Statistics maintenance** keeps them current enough that estimates stay good.

SQL Server does most of this itself through three database options. But the automatic process has blind spots: it waits for a percentage of rows to change, it samples, and it only runs when a query compiles. This lesson explains the automatic rules exactly, when they are not enough, and how to supplement them without wasting resources.

### 2. Core concepts and vocabulary
- **AUTO_CREATE_STATISTICS**: builds single-column stats on predicate columns that have none. Default ON.
- **AUTO_UPDATE_STATISTICS**: refreshes a statistic when its change count passes a threshold and a query that uses it compiles. Default ON.
- **AUTO_UPDATE_STATISTICS_ASYNC**: the query does not wait for the update; it compiles with the old statistics and a background task refreshes them. Default OFF. Requires auto-update ON.
- **Modification counter**: changes to the statistic's leading column since the last update.
- **Threshold**: number of changes that marks a statistic stale.
- **Sample / FULLSCAN**: build stats from a subset of rows (default, faster) or all rows (exact, slower).
- **PERSIST_SAMPLE_PERCENT**: makes a chosen sample rate stick for later automatic updates.
- **RESAMPLE**: update using the sample rate last used.
- **NORECOMPUTE**: excludes a statistic from automatic updates.
- **Incremental statistics**: per-partition statistics, so you can update just the partitions that changed.

### 3. How it works, step by step
**3a. Thresholds for automatic update**

| Table type | Rows (n) | Threshold (compat 130+, default) | Old threshold (compat < 130) |
|---|---|---|---|
| Temporary | < 6 | 6 changes | 6 |
| Temporary | 6–500 | 500 changes | 500 |
| Permanent | < 500 | 500 changes | 500 |
| Both | > 500 | **MIN(500 + 0.20 × n, √(1000 × n))** | 500 + 0.20 × n |

Changes = inserts + updates + deletes combined. Example with n = 5,000,000:
- Old rule: 500 + 1,000,000 = **1,000,500** changes needed.
- New rule: √(5,000,000,000) ≈ **70,711** changes. Large tables now refresh far more often. (Before 2016, trace flag 2371 provided this dynamic threshold.)

**3b. What happens when the threshold is crossed**
1. Data changes only increment the modification counter; nothing is updated yet.
2. A query that uses the statistic is compiled (or its cached plan is checked for validity).
3. The optimizer sees the statistic is stale.
4. **Synchronous** (default): the query waits while the statistic is rebuilt with a default sample, then compiles with fresh stats. Cached plans that depend on it are recompiled.
5. **Asynchronous**: the query compiles immediately with the old stats; a background job updates them; later compilations use the new stats.

So a table that crossed its threshold but is never queried keeps stale stats, and the update cost lands on whichever query happens to compile first.

**3c. Sampling**
- Default sample size is chosen automatically; it shrinks as a percentage as tables grow (small tables are effectively full-scanned).
- `WITH FULLSCAN` reads every row: exact histogram and densities, more I/O and time.
- `WITH SAMPLE n PERCENT | ROWS` sets the sample; `PERSIST_SAMPLE_PERCENT = ON` keeps it for future auto-updates (otherwise auto-update reverts to the default sample).
- Index **rebuild** updates that index's statistics as if with FULLSCAN, because it reads all rows. Index **reorganize** does not touch statistics. Column statistics are untouched by both.

**3d. When automatic maintenance is not enough**
- **Ascending keys**: daily loads add new dates/IDs above the last histogram step long before 20 % (or √1000n) of the table changes.
- **Skew changed by few rows**: e.g. 10 % of rows updated from many values to one value. Cardinality changes a lot; the counter may not reach the threshold.
- **Sample too small** for a skewed column.
- **After large ETL loads**, the first morning query pays the sync update cost, or (async) runs with stale stats.
- **NORECOMPUTE** stats or auto-update turned off.

**3e. Manual maintenance options**
1. `UPDATE STATISTICS table [stat] WITH FULLSCAN | SAMPLE | RESAMPLE` – precise, per table or statistic. Preferred.
2. `sp_updatestats` – every user table in the database, only stats with at least one modification, default sample (or `'resample'`). Blunt, but cheap to schedule.
3. Maintenance solutions (e.g. Ola Hallengren's scripts) – update only stats with modifications, with chosen sample, in a SQL Agent job.
4. Maintenance plans – GUI-driven, less flexible.

### 4. Worked example
```sql
-- 1) Database settings
SELECT name, is_auto_create_stats_on, is_auto_update_stats_on, is_auto_update_stats_async_on
FROM sys.databases WHERE name = DB_NAME();
ALTER DATABASE CURRENT SET AUTO_UPDATE_STATISTICS_ASYNC ON;   -- busy OLTP

-- 2) Which statistics are stale? (dynamic threshold computed per stat)
SELECT OBJECT_SCHEMA_NAME(s.object_id) + '.' + OBJECT_NAME(s.object_id) AS table_name,
       s.name AS stat_name, sp.last_updated, sp.rows, sp.rows_sampled,
       CAST(100.0 * sp.rows_sampled / NULLIF(sp.rows, 0) AS DECIMAL(5,1)) AS sample_pct,
       sp.modification_counter,
       CASE WHEN sp.rows > 500
            THEN IIF(500 + 0.2 * sp.rows < SQRT(1000.0 * sp.rows), 500 + 0.2 * sp.rows, SQRT(1000.0 * sp.rows))
            ELSE 500 END AS auto_threshold
FROM sys.stats s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE OBJECTPROPERTY(s.object_id, 'IsUserTable') = 1
ORDER BY sp.modification_counter * 1.0 / NULLIF(sp.rows, 0) DESC;

-- 3) Out-of-date statistics in action
--    iFirstIndex on dbo.Test1(C1); auto-update OFF; bulk change data
--    SELECT C1, C2, C3 FROM dbo.Test1 WHERE C1 = 51;  → estimated 5,001, actual 1
UPDATE STATISTICS dbo.Test1 iFirstIndex WITH FULLSCAN;
--    Rerun: estimate ≈ actual, plan switches to an index seek.

-- 4) Persist a sample rate for a skewed column
UPDATE STATISTICS Sales.Orders st_CustomerID WITH SAMPLE 50 PERCENT, PERSIST_SAMPLE_PERCENT = ON;

-- 5) Partitioned table: incremental stats, update only the changed partition
UPDATE STATISTICS dbo.BigSales WITH RESAMPLE, INCREMENTAL = ON;      -- convert existing stats
UPDATE STATISTICS dbo.BigSales (st_SaleDate) WITH RESAMPLE ON PARTITIONS (37);

-- 6) After an ETL load, refresh only what changed
EXEC sp_updatestats;
```
**How to read it:** Query 2 ranks statistics by the fraction of changed rows and shows the auto threshold, so you can see which ones are close to (or past) refresh and which have low samples. In example 3, the estimated vs actual row gap (5,001 vs 1) disappears after the update; that is the signature of stale statistics. Note the plan shows *no warning* for stale stats: you only see it by comparing estimates with actuals.

### 5. Under the hood
- The modification counter (`modification_counter`) is tracked per statistic on its **leading column** (column-level modification tracking since SQL Server 2008). Changes to non-leading columns don't count.
- Statistics updates take a **schema stability lock** and read data with a sample; they don't block normal DML.
- A statistics update **invalidates cached plans** that depend on that statistic, causing recompiles on next use. That's how new stats reach the plans.
- Async updates run as a background task. Since SQL Server 2022, `ALTER DATABASE SCOPED CONFIGURATION SET ASYNC_STATS_UPDATE_WAIT_AT_LOW_PRIORITY = ON` lets the background update wait at low priority for its schema modification lock, avoiding blocking chains.
- **Incremental statistics** keep per-partition histograms and merge them into one table-level histogram (still max 200 steps). The optimizer uses the merged one; the benefit is maintenance cost, not finer estimates. Not supported for every kind of statistic (e.g. filtered stats on non-aligned partitions).
- **Columnstore indexes** have no histogram-based index statistic of their own; queries rely on column statistics created on the table.
- `sp_updatestats` uses the default sample unless told `'resample'`; it skips stats with zero modifications, so it won't undo a fresh FULLSCAN from an index rebuild.

### 6. Performance and design implications
- **Keep auto-create and auto-update ON.** Their benefit almost always outweighs the cost.
- **Async ON for high-throughput OLTP**, where a sync update stall hurts more than one compile on slightly old stats. Leave OFF for batch/reporting databases where the first query after a load must get the right plan.
- **Update after loads.** Add `UPDATE STATISTICS` (for the loaded tables) as the last step of ETL.
- **FULLSCAN selectively**: on skewed columns and critical tables, off-hours. FULLSCAN on everything every night wastes I/O.
- **Don't double up**: an index rebuild already full-scanned that index's stats. Follow it with an update of *column* statistics only.
- **Measure**: `sys.dm_db_stats_properties` (modifications, sample), estimated vs actual rows, `auto_stats` Extended Event for how often/long auto updates run.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE ... SET AUTO_UPDATE_STATISTICS_ASYNC ON` | Background stat updates | Busy OLTP |
| `UPDATE STATISTICS t WITH FULLSCAN` | Exact stats for all stats on a table | Skewed/critical tables |
| `UPDATE STATISTICS t s WITH SAMPLE 25 PERCENT, PERSIST_SAMPLE_PERCENT = ON` | Fixed sample for future auto-updates | Default sample too small |
| `UPDATE STATISTICS t WITH RESAMPLE` | Reuse last sample rate | Routine refresh |
| `UPDATE STATISTICS t (s) WITH RESAMPLE ON PARTITIONS (n)` | Refresh one partition's incremental stats | Partitioned tables |
| `EXEC sp_updatestats` | Update all modified stats in the DB | Simple nightly job / after ETL |
| `STATS_DATE(object_id, stats_id)` | Last update date | Quick check |
| XE `sqlserver.auto_stats` | Logs automatic creates/updates and duration | Investigate compile stalls |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Sync auto-update (default) for reporting/batch | OLTP where a 2-second stall on a hot query matters |
| Async auto-update for OLTP | The first query after a big change must use new stats |
| FULLSCAN for skewed or critical stats | Very large tables in business hours; everything every night |
| `sp_updatestats` for simplicity | You need precise control of sample size per stat |
| Incremental stats on big partitioned tables | Expecting better estimates (same 200-step merged histogram) |
| `NORECOMPUTE` with a dedicated job | Nobody owns the job; stats go stale forever |

### 9. Common mistakes
- Turning auto-update OFF because updates paused queries → silent stale stats → use async instead.
- Assuming index rebuilds cover everything → column stats stay stale → update column stats too.
- Running `UPDATE STATISTICS ... WITH FULLSCAN` on all tables nightly → hours of I/O → target by modification counter and skew.
- Running a sampled update right after a rebuild → replaces the rebuild's full-scan stats with sampled ones → skip stats with no modifications.
- Ignoring ascending keys → "today" estimates 1 row → update stats after each load.
- Forgetting async requires auto-update ON → setting has no effect.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server auto-creates single-column stats and auto-updates stats when changes cross a threshold, which since 2016 is the smaller of 500 + 20 % of rows and √(1000 × rows). Updates are synchronous by default, async if enabled. I keep both on, enable async on busy OLTP, update stats after big loads and use FULLSCAN or persisted samples for skewed columns. Index rebuilds refresh index stats with a full scan; column stats need separate updates.

**Structure a longer answer:** clarify → auto rules and threshold numbers → sync vs async → blind spots (ascending keys, skew, sampling) → manual strategy (targeted job, FULLSCAN where needed, incremental for partitions) → a stale-stats incident.

**Likely follow-ups:**
- *A 5M-row table: how many changes trigger auto-update?* → ≈ 70,711 (compat 130+).
- *Does REORGANIZE update stats?* → no; REBUILD does for that index.
- *When would you use async?* → OLTP with compile-sensitive hot queries.
- *How do you detect stale stats?* → `modification_counter`, last_updated, estimated vs actual.

**Signals interviewers look for:** knows the actual threshold formula; distinguishes index vs column stats in maintenance; targeted, not blanket, FULLSCAN; knows plans recompile after stats update.

### 11. Check yourself
- [ ] Compute the auto-update threshold for 2,000 and 50,000,000 rows under old and new rules.
- [ ] Write a query that lists stats whose `modification_counter` exceeds their threshold.
- [ ] Explain why a stale statistic may never be updated automatically.
- [ ] Design a post-ETL statistics step for a partitioned fact table.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Automating Statistics Maintenance
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Default Values › Automatically Maintaining Statistics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Best Practices for Statistics Management
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Automatic Updating
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Async Statistics Updates
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Statistics and Index Maintenance
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Automatic Updating › Manually Maintaining Statistics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Auto Update Statistics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Advanced Best Practices Recap
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality

