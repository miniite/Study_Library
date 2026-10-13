---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore/Segment_and_Rowgroup_Elimination"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Segment and Rowgroup Elimination

**L5 Expert** · [↑ Columnstore Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Columnstore has no seek. Its substitute is elimination: skipping columns the query doesn't need (segment elimination) and whole rowgroups whose min/max metadata cannot match the filter (rowgroup elimination). Data order decides whether it works.

### 1. The big picture
A B-tree finds rows by navigating a sorted key. A columnstore cannot do that, because the only read operator is a scan. Yet a well-built columnstore answers "sales for last week" from a 5-billion-row table in milliseconds. The trick is **metadata**. Every segment records the exact minimum and maximum value it contains. Before reading a rowgroup, SQL Server checks whether the filter could possibly match it, and skips it if not. Combined with reading only the needed columns, the work a query does depends on *how much data it needs*, not on *how big the table is*. A query for one week performs about the same against one month of data as against ten years.

But elimination only works when the min/max ranges are **narrow and non-overlapping** for the column you filter on, which means the data must be stored **in order**. Unordered data gives every rowgroup a range covering nearly all values, and then nothing can be skipped. This lesson is about getting and keeping that order: load order, ordered columnstore, partitioning, and avoiding changes that scramble it.

### 2. Core concepts and vocabulary
- **Segment elimination (column elimination)**: reading only the segments of columns the query references. It is automatic and always works, since a query on 4 of 30 columns reads 4 segments per rowgroup.
- **Rowgroup elimination**: skipping all segments of a rowgroup because a predicate column's min/max proves no row qualifies. Many documents call this "segment elimination" too. The book separates the two terms.
- **min_data_id / max_data_id**: per-segment metadata in `sys.column_store_segments`. For value-encoded numeric and date columns these are the (encoded) actual min/max values. For dictionary-encoded columns they are dictionary IDs, which is why string elimination historically didn't work.
- **Ordered data**: rows stored so that each rowgroup covers a narrow, distinct range of the filter column (usually a date).
- **Ordered columnstore index**: (2022+ for CCI, 2025 for NCCI) `CREATE CLUSTERED COLUMNSTORE INDEX ... ORDER (col)`. SQL Server sorts the data during the build.
- **Partition elimination**: skipping whole partitions based on the partitioning column. Each partition has its own rowgroups, so the two kinds of elimination stack.
- **Predicate pushdown**: evaluating the filter inside the columnstore scan, on compressed data where possible, instead of in a separate Filter operator.

### 3. How it works, step by step
1. The optimizer sees a predicate such as `[Invoice Date Key] >= '2016-03-01' AND < '2016-03-08'` on a columnstore scan.
2. At runtime, for each rowgroup, the scan reads the metadata of the predicate column's segment (min, max).
3. If the range `[min, max]` does not overlap the predicate's range, the whole rowgroup is skipped. No segment of any column in it is read.
4. For surviving rowgroups, only segments of the referenced columns are read and decompressed (segment elimination).
5. The predicate is applied inside the scan (pushdown), in batches.
6. `SET STATISTICS IO ON` reports the outcome per table: `Segment reads N, segment skipped M`. Here the "segments" counted are effectively rowgroups.

**Why order matters (illustration):**

| Rowgroup | Unordered: date range | Ordered: date range |
|---|---|---|
| 1 | 2013-01-01 … 2016-05-31 | 2013-01-01 … 2013-04-10 |
| 2 | 2013-01-01 … 2016-05-31 | 2013-04-10 … 2013-07-22 |
| 3 | 2013-01-02 … 2016-05-30 | 2013-07-22 … 2013-11-01 |
| … | … | … |

A query for one week in 2013-08 must read every unordered rowgroup, but only rowgroup 3 of the ordered layout.

**Ways to get order:**
1. **Load in order.** Insert data sorted by the main filter column, e.g. nightly loads of yesterday's rows. New rowgroups naturally cover new dates.
2. **Rebuild from a sorted clustered B-tree.** The classic pre-2022 trick: create a clustered rowstore index on the date, then `CREATE CLUSTERED COLUMNSTORE INDEX ... WITH (DROP_EXISTING = ON, MAXDOP = 1)`. `MAXDOP = 1` matters, because parallel threads would each take interleaved chunks and the ranges would overlap.
3. **Ordered CCI (2022+).** `ORDER ([Invoice Date Key])` sorts during the build (within memory limits, so the order is not always perfect). Check the result with segment metadata.
4. **Partitioning.** Partition by date. Even if order inside a partition decays, a query touches only the relevant partitions' rowgroups.

### 4. Worked example
```sql
SET STATISTICS IO ON;

-- 1) Unordered table
SELECT SUM(Quantity)
FROM Fact.Sale_CCI
WHERE [Invoice Date Key] >= '2016-01-01' AND [Invoice Date Key] < '2016-01-08';
-- Table 'Sale_CCI'. Segment reads 26, segment skipped 0.

-- Inspect why: every rowgroup spans (almost) all dates
SELECT s.segment_id, s.min_data_id, s.max_data_id, s.row_count
FROM sys.column_store_segments s
JOIN sys.partitions p ON p.hobt_id = s.hobt_id
JOIN sys.columns c ON c.object_id = p.object_id AND c.column_id = s.column_id
WHERE p.object_id = OBJECT_ID('Fact.Sale_CCI') AND c.name = 'Invoice Date Key'
ORDER BY s.segment_id;

-- 2) Build an ordered copy (pre-2022 method)
CREATE CLUSTERED INDEX CCI_fact_Sale_CCI_ORDERED
    ON Fact.Sale_CCI_ORDERED ([Invoice Date Key]);
CREATE CLUSTERED COLUMNSTORE INDEX CCI_fact_Sale_CCI_ORDERED
    ON Fact.Sale_CCI_ORDERED WITH (DROP_EXISTING = ON, MAXDOP = 1);

-- 2022+ equivalent
-- CREATE CLUSTERED COLUMNSTORE INDEX CCI_fact_Sale_CCI_ORDERED
--     ON Fact.Sale_CCI_ORDERED ORDER ([Invoice Date Key]) WITH (MAXDOP = 1);

SELECT SUM(Quantity)
FROM Fact.Sale_CCI_ORDERED
WHERE [Invoice Date Key] >= '2016-01-01' AND [Invoice Date Key] < '2016-01-08';
-- Table 'Sale_CCI_ORDERED'. Segment reads 1, segment skipped 25.
```
Interpretation: the second query reads about 4% of the rowgroups. The segment metadata of the ordered table shows min/max ranges that increase steadily from rowgroup to rowgroup. If you run the same check on another column, e.g. `Stock Item Key`, every rowgroup still spans about 1…227. Filters on that column get no rowgroup elimination, because you can only order by one leading dimension well. For frequent filters on such a column, the options are a nonclustered B-tree on the columnstore (see *Columnstore with Rowstore*) or accepting the full scan.

### 5. Under the hood
- **What can be eliminated.** Numeric, date and time types that use value encoding have meaningful min/max. Before SQL Server 2022, string columns and other dictionary-encoded columns stored dictionary IDs in min/max, so no elimination was possible on them. SQL Server 2022 extended elimination to string, binary, `uniqueidentifier` and high-scale `datetimeoffset` columns, so `WHERE CustomerCode = 'X'` can now skip rowgroups when the data is ordered. LOB columns (`varchar(max)` etc.) still get no elimination.
- **Predicate shape matters.** Elimination needs a predicate the engine can compare with min/max: `=`, `<`, `>`, `BETWEEN`, `IN`. Wrapping the column in a function (`YEAR(date) = 2016`), implicit conversions, or `OR` across different columns can prevent it. Write sargable ranges.
- **Joins.** In a star join with `WHERE d.CalendarYear = 2016` on the dimension, the fact table's date key is not filtered directly, so classic elimination needs a literal range on the fact column. Batch mode **bitmap filters** pushed into the scan can reduce rows, but they do not skip rowgroups as reliably. A common tuning step is to filter the fact table's own date key directly.
- **Order decays.** Updates (delete + insert into delta), late-arriving data, and merges of small rowgroups by `REORGANIZE` can mix old and new dates into the same rowgroup. Partitioning confines the damage. Periodic rebuilds of affected partitions (from sorted data) restore it.
- **Rowgroup sizing and partitions.** Rowgroups never cross partitions. Very small partitions (e.g. daily partitions with 200K rows) produce small rowgroups and hurt compression. The book recommends that each partition hold at least several full rowgroups, so size partitions by volume (often monthly or yearly), not by habit.
- **Multi-column ordering.** Ordering by date first and then by a second column helps only when each date's data spans many rowgroups (e.g. 25M rows per day). Then the second column gets narrow ranges within each day.

### 6. Performance and design implications
- Measure elimination with `STATISTICS IO` (`segment skipped`), and with the actual plan's columnstore scan properties (`Actual Segment Reads`, `Actual Segments Skipped`).
- Pick the order column from real query filters. Time is the most common choice for OLAP.
- Quantify "how ordered is it": for the order column, count how many rowgroups each value range overlaps. Overlap that grows over time means it is time to rebuild a partition.
- The best case combines partition elimination, rowgroup elimination and column (segment) elimination. A query can then touch one rowgroup's few columns out of thousands of rowgroups, with near-zero logical reads.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET STATISTICS IO ON` | Shows `Segment reads / skipped` | Every columnstore tuning session |
| `sys.column_store_segments` (`min_data_id`, `max_data_id`) | Per-segment range | Check ordering on a column |
| `CREATE CLUSTERED COLUMNSTORE INDEX ... ORDER (col)` | Ordered build (2022+) | Enable elimination on a column |
| `sys.index_columns.column_store_order_ordinal` | Shows the ORDER columns | Verify an index is ordered |
| `WITH (DROP_EXISTING = ON, MAXDOP = 1)` | Single-threaded rebuild preserving the B-tree order | Pre-2022 ordering trick |
| `ALTER INDEX ... REBUILD PARTITION = n` | Rebuild one partition | Restore order after decay |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Order by date when most queries filter on time | Filters are on many unrelated columns: only one column orders well |
| `MAXDOP = 1` builds for perfect order | Huge tables: single-threaded builds take long (do it per partition) |
| Partition by date to confine decay | Partitions so small that rowgroups are trimmed |
| Multi-column order for very large daily volumes | Small daily volumes: the second column gives no benefit |

### 9. Common mistakes
- Building the CCI in parallel from a sorted table → interleaved ranges → use `MAXDOP = 1` or `ORDER`.
- `WHERE YEAR([Invoice Date Key]) = 2016` → no elimination → use `>= '2016-01-01' AND < '2017-01-01'`.
- Expecting elimination on a string column on SQL 2019 → no skip → upgrade path (2022) or filter on a numeric/date key.
- Letting updates and late data scramble order for years → `segment skipped` drops toward 0 → rebuild affected partitions periodically.
- Assuming the dimension filter in a star join triggers elimination → it may not → add a direct range on the fact's date key.

### 10. 🎯 Interview kit
**30-second answer:** "Columnstore can't seek, so it skips data instead. It reads only the referenced columns, and it uses exact per-segment min/max metadata to skip whole rowgroups that can't match the filter. That only works if the data is ordered on the filter column. So we load in date order, use ordered columnstore in 2022+, or rebuild from a sorted B-tree with MAXDOP 1, and partition by date. We verify with STATISTICS IO's segment reads/skipped."
**Structure a longer answer:** why no seeks → metadata → column vs rowgroup elimination → order dependence → techniques → measurement → decay and maintenance.
**Likely follow-ups:** *Why MAXDOP 1?* → parallel threads produce overlapping ranges. *Strings?* → dictionary IDs pre-2022; real min/max from 2022. *What about a second filter column?* → only one dimension orders well; use partitioning or a nonclustered B-tree.
**Signals interviewers look for:** knowing that elimination depends on order, using the right measurement, writing sargable predicates, and version awareness.

### 11. Check yourself
- [ ] Explain the difference between segment (column) elimination and rowgroup elimination.
- [ ] Given a list of min/max per rowgroup, predict `segment reads/skipped` for a query.
- [ ] Rebuild a CCI in date order two ways (pre-2022 and 2022+).
- [ ] Rewrite a non-sargable date filter so elimination works.
- [ ] Explain how partitioning and rowgroup elimination combine.
<!-- explainer:end -->

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination › Combining Segment and Rowgroup Elimination
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Segment and rowgroup elimination
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI › CHAPTER 10
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Database Maintenance Database Maintenance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — partitioning › Partition and Rowgroup Sizing
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › Rowgroups and Segments Rowgroups and Segments
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Detailing Unordered Data
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › How Much Fragmentation Is Too Much?
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Removing Duplicates
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n

