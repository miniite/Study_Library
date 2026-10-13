---
id: "TOOLS/SQL_Server/03_Statistics_and_Cardinality/Statistics_Objects"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Statistics Objects

**L4 Advanced** · [↑ Statistics and Cardinality](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A statistics object is a header (metadata), a density vector (one uniqueness figure per column prefix) and a histogram (up to 200 steps on the first column); you inspect it with `DBCC SHOW_STATISTICS`, the `sys.stats` views and `sys.dm_db_stats_properties`.

### 1. The big picture
The parent lesson explained *why* the optimizer needs statistics. This lesson explains *what one is* and *how to look inside it*. Every index carries one statistics object with the same name. Columns used in predicates get their own single-column statistics when auto-create is on. You can also create statistics yourself: multi-column, filtered or incremental.

Knowing how to read a statistics object is a core tuning skill. When a plan estimates 5,001 rows and gets 1, the answer is almost always visible in the header (old update date, low sample), the histogram (value missing or averaged), or the density vector (correlation between columns). The child lesson *Histogram and Density* then shows the exact estimation math.

### 2. Core concepts and vocabulary
- **Stats header**: metadata: name, last update time, rows, rows sampled, number of steps, average key length, filter.
- **Rows / Rows Sampled**: total rows at last update vs rows actually read. If sampled < rows, the histogram and densities are *estimates*.
- **Density vector**: one row per *left-based prefix* of the key columns. For a statistic on (A, B): one row for A, one for (A, B).
- **All density**: `1 / distinct values` of that prefix.
- **Histogram**: up to 200 steps on the **first** key column only. Each step has `RANGE_HI_KEY`, `RANGE_ROWS`, `EQ_ROWS`, `DISTINCT_RANGE_ROWS`, `AVG_RANGE_ROWS` (details in the child lesson).
- **String index (string summary)**: an extra structure for string columns that improves estimates for `LIKE` predicates.
- **Filtered statistics**: statistics built only on rows that match a `WHERE` clause.
- **Incremental statistics**: per-partition statistics merged into one table-level statistic, so you can update one partition at a time.
- **`NORECOMPUTE`**: tells SQL Server never to auto-update this statistic.
- **`modification_counter`**: count of changes to the leading column since the last update.

### 3. How it works, step by step
**Reading `DBCC SHOW_STATISTICS`.** Syntax: `DBCC SHOW_STATISTICS (table_or_view, target) [WITH NO_INFOMSGS, STAT_HEADER | DENSITY_VECTOR | HISTOGRAM]`. Without options it returns three result sets.

1. **Header** (one row):

| Column | Meaning |
|---|---|
| Name | Statistic name (index name for index stats) |
| Updated | Last update date and time |
| Rows | Rows in the table (or matching the filter) at last update |
| Rows Sampled | Rows read to build the stat |
| Steps | Histogram steps (max 200) |
| Density | Old first-column density; not used by the optimizer since SQL Server 2008 |
| Average key length | Average bytes per key value across all key columns |
| String Index | YES if a string summary exists (helps `LIKE`) |
| Filter Expression | Filter predicate for filtered stats, else NULL |
| Unfiltered Rows | Rows in the table before the filter |

2. **Density vector** (one row per column prefix). Example for the primary key `(SalesOrderID, SalesOrderDetailID)` on `Sales.SalesOrderDetail`:

| All density | Average length | Columns |
|---|---|---|
| 3.178e-05 | 4 | SalesOrderID |
| 8.24e-06 | 8 | SalesOrderID, SalesOrderDetailID |

`3.178e-05 ≈ 1/31,465` distinct orders. The second row is 1 / number of distinct (order, line) pairs, which equals 1 / row count because the key is unique. The optimizer uses these rows for equality predicates on the prefix when it doesn't know the value, and for `GROUP BY` / `DISTINCT` estimates.

3. **Histogram**: up to 200 rows, one per step, on `SalesOrderID` only. Nothing about `SalesOrderDetailID` is in the histogram. That is why the *order of key columns* matters for statistics as well as for seeks.

**Where statistics come from:**
1. `CREATE INDEX` builds index statistics with a full scan (because the index build reads every row anyway).
2. Auto-create builds a **single-column** statistic when a predicate column has none. It never creates multi-column statistics.
3. `CREATE STATISTICS` builds any shape you want: multiple columns, filters, sample sizes, `INCREMENTAL`, `NORECOMPUTE`.

### 4. Worked example
```sql
-- 1) All statistics on a table, with their columns and freshness
SELECT s.stats_id, s.name, s.auto_created, s.user_created, s.no_recompute,
       s.has_filter, s.filter_definition, s.is_incremental,
       STRING_AGG(c.name, ', ') WITHIN GROUP (ORDER BY sc.stats_column_id) AS columns,
       sp.last_updated, sp.rows, sp.rows_sampled, sp.steps, sp.modification_counter
FROM sys.stats s
JOIN sys.stats_columns sc ON sc.object_id = s.object_id AND sc.stats_id = s.stats_id
JOIN sys.columns c        ON c.object_id = sc.object_id AND c.column_id = sc.column_id
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE s.object_id = OBJECT_ID('Production.Product')
GROUP BY s.stats_id, s.name, s.auto_created, s.user_created, s.no_recompute, s.has_filter,
         s.filter_definition, s.is_incremental, sp.last_updated, sp.rows, sp.rows_sampled,
         sp.steps, sp.modification_counter;

-- 2) A query on two non-indexed columns
SELECT p.Name, p.Class FROM Production.Product p
WHERE p.Color = 'Red' AND p.DaysToManufacture > 15;
-- Rerun query 1: TWO new single-column _WA_Sys_ stats, not one two-column stat.

-- 3) Build a multi-column statistic yourself
CREATE STATISTICS st_Color_Days ON Production.Product (Color, DaysToManufacture) WITH FULLSCAN;
DBCC SHOW_STATISTICS ('Production.Product', st_Color_Days) WITH DENSITY_VECTOR;

-- 4) Filtered statistic for a skewed slice
CREATE STATISTICS st_PO_NotNull ON Sales.SalesOrderHeader (PurchaseOrderNumber)
WHERE PurchaseOrderNumber IS NOT NULL WITH FULLSCAN;

-- 5) Watch modification_counter grow
UPDATE TOP (500) Production.Product SET Color = Color;
SELECT name, sp.modification_counter
FROM sys.stats s CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) sp
WHERE s.object_id = OBJECT_ID('Production.Product');
```
**How to read it:**
- After query 2, the optimizer created one statistic per column. Auto-create cannot capture correlation between `Color` and `DaysToManufacture`. Query 3's density vector gives a combined density for the pair, which the optimizer can use for equality on both.
- The filtered statistic in query 4 has `Rows` equal to non-NULL rows and `Unfiltered Rows` equal to the table count. Its 200 steps are spent only on real values, so estimates are sharper.
- `modification_counter` counts changes to the leading column of each statistic. Statistics led by other columns may stay at 0.

### 5. Under the hood
- Statistics are stored as a binary large object (the *stats blob*) in an internal system table. The `STATS_STREAM` option of `DBCC SHOW_STATISTICS` exposes it (used to clone stats; not for normal use).
- `stats_id` equals `index_id` for index statistics. Column statistics get higher IDs.
- Auto-created names follow `_WA_Sys_<column_id in hex>_<object_id in hex>`.
- `sys.dm_db_stats_histogram(object_id, stats_id)` (SQL Server 2016 SP1 CU2+) returns the histogram as rows you can query and join, which is easier than DBCC output.
- The header `Density` column is legacy. The optimizer uses the density vector's *All density* instead.
- For a filtered statistic, the optimizer only uses it when the query predicate matches (is covered by) the filter.
- Statistics are created with `FULLSCAN` when built as part of an index; auto-created and auto-updated stats use a sample.

### 6. Performance and design implications
- **Check `Rows Sampled` vs `Rows`.** A 1 % sample on a skewed column can miss rare values entirely.
- **Check `Updated` and `modification_counter`.** Many changes since the last update mean the histogram may not cover new values (for example, new dates beyond the last step).
- **Key order matters.** A histogram exists only for the first column. A statistic on `(CustomerID, OrderDate)` gives no histogram on `OrderDate`.
- **Correlated columns need help.** Multi-column statistics or indexes give a combined density; filtered statistics handle skewed subsets.
- Statistics are small; creating a few targeted ones is cheap. Too many statistics only add update time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DBCC SHOW_STATISTICS (t, s) WITH STAT_HEADER` | Header only | Quick freshness check |
| `sys.stats` / `sys.stats_columns` | Inventory of statistics and their columns | Find auto vs user stats |
| `sys.dm_db_stats_properties(obj, stat)` | Rows, sampled, steps, last update, modifications | Staleness monitoring |
| `sys.dm_db_stats_histogram(obj, stat)` | Histogram as a rowset | Scripted analysis |
| `STATS_DATE(obj, stat)` | Last update date | Simple checks |
| `CREATE STATISTICS n ON t (a, b) WHERE ... WITH FULLSCAN` | Multi-column / filtered stats | Correlated or skewed data |
| `CREATE STATISTICS ... WITH INCREMENTAL = ON` | Per-partition statistics | Large partitioned tables |
| `CREATE STATISTICS ... WITH NORECOMPUTE` | Disables auto-update for that stat | Rarely; you own its maintenance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Multi-column statistics for correlated equality predicates | Range predicates on the second column (no histogram there) |
| Filtered statistics for skewed subsets (e.g. active rows) | Queries with parameters that may not match the filter |
| Incremental statistics on big partitioned tables | Expecting per-partition estimates; the merged stat still has 200 steps |
| `NORECOMPUTE` for a hand-tuned stat | Forgetting it exists; it then goes stale forever |

### 9. Common mistakes
- Assuming auto-create builds multi-column stats → correlated predicates underestimated → create them manually or via an index.
- Reading the header `Density` → it is unused → use the density vector.
- Ignoring `Rows Sampled` → estimates built on a tiny sample → update with a larger sample or FULLSCAN.
- Dropping `_WA_Sys_` stats because they "look like junk" → recreated at next compile, with compile-time cost → leave them or replace with designed stats.

### 10. 🎯 Interview kit
**30-second answer:** A statistics object has three parts. The header tells you when it was updated and how many rows were sampled. The density vector stores 1/distinct values for each left prefix of the key columns, used when the exact value is unknown. The histogram has up to 200 steps on the first column, each with the rows equal to the step's top value and the rows inside the range. You read it with `DBCC SHOW_STATISTICS` or `sys.dm_db_stats_histogram`, and check staleness with `sys.dm_db_stats_properties`.

**Structure a longer answer:** clarify → three parts → how they come into being (index, auto, manual) → what to check during a misestimate (date, sample, modifications, key order) → example with a correlated pair fixed by a multi-column stat.

**Likely follow-ups:**
- *Is there a histogram on the second column of an index?* → no, only density for the prefix.
- *How do you know stats are stale?* → `modification_counter` vs rows, `last_updated`.
- *What is a filtered statistic for?* → better histogram for a skewed subset.

**Signals interviewers look for:** knows the three components and what each is used for; knows histogram = first column only; uses DMVs, not just the GUI.

### 11. Check yourself
- [ ] Run `DBCC SHOW_STATISTICS` on a two-column index and explain each density vector row.
- [ ] Write the query that lists all stats on a table with their `modification_counter`.
- [ ] Explain why two `_WA_Sys_` stats are not the same as one two-column statistic.
- [ ] Create a filtered statistic and verify `Rows` vs `Unfiltered Rows`.
<!-- explainer:end -->

## In this section (study in order)

1. [Histogram and Density Vector](<Histogram_and_Density/_index.md>) (L5 Expert)

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DBCC SHOWSTATISTICS
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Stats Header
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Density Vector › Catalog Views
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbstatsproperties
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Statistics
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Density Vector
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Header Columns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Density Vector › sys.stats
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index Operational Stats Summary › Header Columns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Row Statistics

