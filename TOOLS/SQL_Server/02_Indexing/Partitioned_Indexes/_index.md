---
id: "TOOLS/SQL_Server/02_Indexing/Partitioned_Indexes"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Partitioned Tables and Indexes

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Partitioning splits one table and its indexes into independent pieces by ranges of one column. It is mainly a *manageability* feature (fast loads/purges via SWITCH and TRUNCATE, per-partition maintenance and compression), and it helps queries only when partition elimination actually happens.

### 1. The big picture
Large tables grow painful in operations long before they become slow to query. Rebuilding a 2-billion-row index takes hours. Deleting last year's data generates enormous log and blocking. Loading a new month competes with readers. **Partitioning** divides a table, and each of its indexes, into **partitions**: separate B-trees (or columnstore rowgroup sets), one per range of a chosen **partitioning column**, usually a date. To queries the table looks like one object. To maintenance it is many small ones.

A common misconception is that partitioning makes queries faster. Often it doesn't: a well-indexed non-partitioned table seeks just as fast. Queries benefit only when the predicate lets SQL Server **eliminate** partitions, and some query shapes can even get slower (e.g. TOP/MIN/MAX across partitions on a non-aligned order). The real wins are **partition switching** (moving millions of rows by changing metadata), **TRUNCATE … WITH (PARTITIONS)**, **per-partition rebuilds, compression and statistics**, and, for columnstore, combining partition elimination with rowgroup elimination.

### 2. Core concepts and vocabulary
- **Partition function**: defines boundary values and how the column's range is divided. `RANGE RIGHT` means each boundary belongs to the partition on its right (the usual choice for dates). `RANGE LEFT` means each boundary belongs to the partition on its left.
- **Partition scheme**: maps each partition of a function to a filegroup.
- **Partitioning column**: the single column whose value decides a row's partition. For a clustered index or unique index, it must be part of the key.
- **Aligned index**: an index partitioned on the same function and column as its table. Required for SWITCH.
- **Non-aligned index**: an index on a different scheme or not partitioned at all. It blocks switching, but can serve queries that need a global order or a global uniqueness check.
- **Partition elimination**: the optimizer or executor reads only partitions that can contain qualifying rows. **Static** elimination is decided at compile time from literals. **Dynamic** elimination is decided at runtime from parameters or join values.
- **SWITCH**: `ALTER TABLE … SWITCH PARTITION n TO …`, a metadata-only move of a partition between tables with identical structure.
- **SPLIT / MERGE RANGE**: add or remove a boundary. Cheap on *empty* partitions, very expensive when data must move.
- **Sliding window**: a routine that adds new empty partitions at the front and switches out/purges old ones at the back.
- **$PARTITION**: a function that returns the partition number for a value.
- **Incremental statistics**: per-partition statistics that are merged into a table-level histogram (2014+).

### 3. How it works, step by step
**Setting up:**
1. Create a partition function on the column's type, with boundaries (e.g. the first of each month).
2. Create a partition scheme mapping partitions to filegroups (often all to one filegroup, for simplicity).
3. Create the clustered index (or CCI) on the scheme: `ON ps_Month(OrderDate)`. The table is now partitioned.
4. Create nonclustered indexes on the same scheme so they are **aligned**. SQL Server adds the partitioning column to the index silently if needed.

**Query execution:**
1. The optimizer checks the predicate on the partitioning column.
2. With a literal range (`OrderDate >= '2026-01-01' AND < '2026-02-01'`), it computes the partitions at compile time (static elimination). The plan's *Actual Partition Count* shows 1.
3. With parameters, it adds a runtime filter on `PtnId`, which yields dynamic elimination with the same effect.
4. Within each remaining partition, the normal B-tree seek or scan runs.

**Elimination fails when:**
- the predicate is not on the partitioning column (e.g. on `CustomerID`), so all partitions are searched, one seek per partition;
- the column is wrapped in a function or converted (`YEAR(OrderDate) = 2026`, a `datetime` vs `date` mismatch causing an implicit conversion);
- the filter comes only through a join from a dimension table, and the plan doesn't apply dynamic elimination;
- `OR` conditions across columns spoil sargability.

**Sliding window, monthly:**
1. Before the new month: `ALTER PARTITION SCHEME … NEXT USED fg;` then `ALTER PARTITION FUNCTION … SPLIT RANGE ('2026-11-01')` on an **empty** trailing partition, which is instant.
2. Load the new data into a staging table with an identical structure, on the same filegroup, with a check constraint matching the boundary. Then `SWITCH` it in.
3. Archive the oldest month: `SWITCH` it out to an archive table, or `TRUNCATE TABLE … WITH (PARTITIONS (1))`.
4. `MERGE RANGE` the now-empty oldest boundary.

### 4. Worked example
```sql
CREATE PARTITION FUNCTION pf_Month (date)
AS RANGE RIGHT FOR VALUES ('2026-01-01','2026-02-01','2026-03-01' /* ... */);

CREATE PARTITION SCHEME ps_Month AS PARTITION pf_Month ALL TO ([PRIMARY]);

CREATE TABLE Sales.OrderLines (
    OrderDate  date   NOT NULL,
    OrderID    bigint NOT NULL,
    LineNo     int    NOT NULL,
    CustomerID int    NOT NULL,
    Amount     decimal(12,2) NOT NULL,
    CONSTRAINT PK_OrderLines PRIMARY KEY CLUSTERED (OrderDate, OrderID, LineNo)
) ON ps_Month(OrderDate);

CREATE NONCLUSTERED INDEX IX_OrderLines_Customer
ON Sales.OrderLines (CustomerID) INCLUDE (Amount)
ON ps_Month(OrderDate);          -- aligned

-- Where do rows live?
SELECT $PARTITION.pf_Month(OrderDate) AS p, COUNT(*) AS rows
FROM Sales.OrderLines GROUP BY $PARTITION.pf_Month(OrderDate) ORDER BY p;

-- Elimination works: Actual Partition Count = 1
SELECT SUM(Amount) FROM Sales.OrderLines
WHERE OrderDate >= '2026-02-01' AND OrderDate < '2026-03-01';

-- No elimination: one seek per partition (Actual Partition Count = all)
SELECT SUM(Amount) FROM Sales.OrderLines WHERE CustomerID = 42;

-- Purge January instantly
TRUNCATE TABLE Sales.OrderLines WITH (PARTITIONS (2));

-- Rebuild only the current month
ALTER INDEX PK_OrderLines ON Sales.OrderLines REBUILD PARTITION = 12 WITH (ONLINE = ON);
```
What to observe: in the actual plan, the seek or scan properties show **Actual Partition Count** and **Actual Partitions Accessed**. The `CustomerID` query reads every partition's IX_OrderLines_Customer B-tree. With 120 monthly partitions, that is 120 seeks where a non-partitioned index needs one. This is the hidden cost of aligned indexes for queries that don't filter on the partitioning column.

### 5. Under the hood
- **Each partition is its own HoBT.** A heap or B-tree, with its own allocation units, appears as a row in `sys.partitions`. An index with 120 partitions is 120 separate trees. Each tree is shallower, but a query without elimination must probe all of them.
- **Uniqueness.** A unique aligned index must include the partitioning column in its key, because SQL Server can only check uniqueness within a partition. To enforce `OrderID` unique across all dates, you need a **non-aligned** unique index, which disables switching while it exists.
- **SWITCH requirements.** Identical columns, types, nullability and indexes. Same filegroup. The target partition/table must be empty. The source must have a check constraint proving its rows fit the target boundary (when switching into a partitioned table). No non-aligned indexes. With these met, the switch is a metadata change and takes milliseconds.
- **SPLIT/MERGE on non-empty partitions** physically moves rows, which is fully logged and can take hours. That's why the sliding window always splits empty ranges and merges empty ones.
- **Lock escalation.** `ALTER TABLE … SET (LOCK_ESCALATION = AUTO)` allows escalation to the partition (HoBT) level instead of the whole table. This can reduce blocking, but also creates deadlock risk across partitions.
- **Statistics.** By default statistics stay table-level (one 200-step histogram for all partitions). `STATISTICS_INCREMENTAL = ON` keeps per-partition statistics, so updating one partition's statistics is cheap. The optimizer still uses the merged histogram.
- **Limits.** Up to 15,000 partitions per table. Too many partitions inflate metadata, compile time and memory. For columnstore, tiny partitions mean trimmed rowgroups.
- **Columnstore + partitioning.** Rowgroups never cross partitions, so partition elimination removes whole sets of rowgroups before rowgroup elimination runs. Per-partition `COLUMNSTORE_ARCHIVE` compression suits cold history.

### 6. Performance and design implications
- Choose the partitioning column from **data lifecycle** (load and purge by date), and confirm that most heavy queries filter on it.
- Partition granularity: big enough that each partition holds meaningful data (for columnstore, many full rowgroups), small enough that maintenance windows fit. Monthly or yearly is common. Daily is rarely right except for huge volumes.
- Measure elimination via *Actual Partition Count*. Measure the overhead of non-eliminated queries by comparing logical reads with a non-partitioned copy.
- Keep all indexes aligned unless a specific query or uniqueness need justifies giving up SWITCH.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE PARTITION FUNCTION … RANGE RIGHT FOR VALUES (…)` | Defines boundaries | Setup |
| `CREATE PARTITION SCHEME … ALL TO ([PRIMARY])` | Maps partitions to filegroups | Setup |
| `ALTER TABLE … SWITCH PARTITION n TO t2 [PARTITION m]` | Metadata-only move | Loads, archiving |
| `TRUNCATE TABLE t WITH (PARTITIONS (n[, m TO k]))` | Instantly empties partitions | Purging (2016+) |
| `ALTER PARTITION FUNCTION … SPLIT / MERGE RANGE` | Add/remove a boundary | Sliding window (empty partitions only) |
| `ALTER INDEX … REBUILD PARTITION = n [WITH (ONLINE = ON)]` | Per-partition rebuild | Maintenance |
| `$PARTITION.pf(value)` | Partition number for a value | Diagnostics |
| `STATISTICS_INCREMENTAL = ON` | Per-partition statistics | Large partitioned tables |
| `LOCK_ESCALATION = AUTO` | Escalate to partition, not table | Concurrency on partitions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Huge tables with time-based load/purge | Partitioning just "for speed" without elimination |
| Aligned indexes (SWITCH, per-partition ops) | Frequent lookups not on the partitioning column (N seeks) |
| Non-aligned unique index for a global key | You need SWITCH on that table |
| Monthly/yearly partitions | Thousands of tiny partitions |

### 9. Common mistakes
- Partitioning to fix slow queries → no elimination, more seeks → fix the indexes; partition for manageability.
- `RANGE LEFT` with date boundaries like '2026-02-01' → the boundary day lands in the previous month → use `RANGE RIGHT` for dates.
- SPLIT on a full partition → hours of logged data movement → always keep an empty partition at the end and split it ahead of time.
- A data type mismatch in predicates → implicit conversion, no elimination → match parameter types to the column.
- Creating a non-aligned index casually → SWITCH fails at the next load → align by default.

### 10. 🎯 Interview kit
**30-second answer:** "Partitioning splits a table and its indexes into separate B-trees by ranges of one column, usually date. It's primarily for manageability: switch partitions in and out as metadata operations, truncate old partitions, and rebuild or compress per partition. Queries benefit only with partition elimination, meaning a sargable predicate on the partitioning column. Without it, an aligned index costs one seek per partition."
**Structure a longer answer:** function/scheme/aligned indexes → elimination rules and failure modes → SWITCH and sliding window → uniqueness and alignment trade-off → columnstore synergy.
**Likely follow-ups:** *Does partitioning improve performance?* → only with elimination; it's mainly for maintenance. *Why must the unique key include the partition column?* → uniqueness is checked per partition. *How do you load a month without blocking?* → stage, check constraint, SWITCH IN.
**Signals interviewers look for:** the "manageability first" message, knowledge of elimination pitfalls, sliding-window mechanics, and awareness of RANGE LEFT/RIGHT.

### 11. Check yourself
- [ ] Create a monthly partitioned table with aligned indexes.
- [ ] Show Actual Partition Count for an eliminated and a non-eliminated query.
- [ ] Write the full sliding-window script (split, switch in, switch out/truncate, merge).
- [ ] Explain why a unique index on `OrderID` alone breaks SWITCH.
- [ ] Decide a partition granularity for 50M rows/month in a CCI.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Partitioned Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › Partition Elimination Partition Elimination
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.7 #60 Not partitioning large tables
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.8 #61 Not understanding the limitations of partition elimination
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › CHAPTER 11
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Partitioning Large Tables
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Database Maintenance Database Maintenance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Partition Column Choice
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Using Window Aggregates to Solve Common Queries › The Partitioned Table Problem
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Using the DTA Utility

