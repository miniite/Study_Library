---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["PostgreSQL", "MySQL"]
---

# Index Access Paths

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An *access path* is how the engine reaches the rows a query needs — table/index scan, index seek, key/RID lookup, covering index, range scan — and the optimizer picks one by comparing estimated cost, which depends on predicates, selectivity, index design and statistics.

### 1. The big picture
For any `WHERE` clause there are several physical ways to find rows. Reading the whole table is always possible; an index offers shortcuts. Choosing between them is the optimizer's core job, and your job as a developer is to give it good options (indexes), accurate information (statistics) and *sargable* predicates it can use.

The fundamental trade-off: an **index seek + lookups** costs roughly one random I/O per matching row, whereas a **scan** reads everything sequentially. Below some selectivity threshold (often a few percent of rows, far less for wide tables) seeks win; above it scans win. A **covering index** removes the lookup and shifts the threshold dramatically.

### 2. Core concepts and vocabulary
- **Access path** — the method used to read a table: heap/clustered scan, index scan, index seek, lookup.
- **Scan** — read all pages of the structure in order.
- **Seek** — navigate the B-tree from root to the first matching leaf entry, then read the range.
- **Range scan** — seek to the start of a key range and read sequentially until the end condition.
- **Lookup (key lookup / RID lookup / bookmark lookup)** — fetch the remaining columns from the clustered index (by key) or heap (by RID) for each index hit.
- **Covering index** — contains all columns the query needs (key + `INCLUDE`), so no lookup.
- **Sargable predicate** — can be used as a seek predicate (`col = @x`, `col >= @a AND col < @b`, `col LIKE 'abc%'`).
- **Selectivity / cardinality** — fraction / number of rows matching; estimated from statistics (histogram).
- **Leading column** — first key column; seeks require predicates on a prefix of the key.
- **Residual predicate** — evaluated after rows are fetched because it could not be used for seeking.
- **Index intersection / union** — combining multiple indexes (more common in MySQL/PostgreSQL bitmap scans; in SQL Server via joins of index seeks).
- **Index-only scan** — PostgreSQL's covering read, requires visibility map.

### 3. How it works, step by step
**Seek on a B-tree (height 3–4 even for billions of rows):**
1. Start at the root page; binary-search keys to pick the child pointer.
2. Descend to the leaf level (each level = one page read, mostly cached for upper levels).
3. At the leaf, find the first entry ≥ the seek key.
4. Read forward along the leaf-level linked list until the range ends.
5. For each entry, if columns are missing, perform a lookup into the clustered index/heap.

**How the optimizer chooses:**
1. Enumerate candidate paths for each table from predicates and available indexes.
2. Use statistics to estimate rows for each predicate.
3. Cost each path (I/O pages × cost + CPU).
4. Pick the cheapest overall plan including join order and sort needs (an index can supply order).

| Predicate shape | Path |
|---|---|
| `WHERE id = 42` (unique) | Singleton seek |
| `WHERE date >= '2026-01-01' AND date < '2026-02-01'` | Range seek |
| `WHERE LEFT(name,3) = 'Smi'` | Scan (non-sargable) — rewrite as `name LIKE 'Smi%'` |
| `WHERE col2 = 5` on index `(col1, col2)` | Scan of the index (or skip scan in Oracle/some engines) — leading column missing |
| `WHERE status = 'A'` returning 40% rows | Scan |
| `SELECT a,b … WHERE k = 1`, index `(k) INCLUDE (a,b)` | Covering seek, no lookup |

### 4. Worked example
```sql
CREATE TABLE Sales.Orders (
  OrderID INT IDENTITY PRIMARY KEY,           -- clustered
  CustomerID INT NOT NULL,
  OrderDate DATE NOT NULL,
  Status CHAR(1) NOT NULL,
  Total DECIMAL(12,2) NOT NULL,
  Notes NVARCHAR(500) NULL);
CREATE INDEX IX_Orders_Customer ON Sales.Orders (CustomerID);                       -- narrow
CREATE INDEX IX_Orders_Date_Incl ON Sales.Orders (OrderDate) INCLUDE (CustomerID, Total);

SET STATISTICS IO ON;

-- A: selective lookup, needs Status/Total -> seek + key lookup
SELECT OrderID, Status, Total FROM Sales.Orders WHERE CustomerID = 1234;
-- B: covering range seek, no lookups
SELECT CustomerID, SUM(Total) FROM Sales.Orders
WHERE OrderDate >= '2026-03-01' AND OrderDate < '2026-04-01' GROUP BY CustomerID;
-- C: non-sargable, forces a scan
SELECT * FROM Sales.Orders WHERE YEAR(OrderDate) = 2026;
-- D: low selectivity, optimizer should scan
SELECT * FROM Sales.Orders WHERE Status = 'C';   -- 90% 'C'
-- E: implicit conversion kills the seek (column is VARCHAR, param is NVARCHAR)
DECLARE @n NVARCHAR(50) = N'Smith';
SELECT * FROM Person.Person WHERE LastName = @n;  -- CONVERT_IMPLICIT in plan
```
**Reading plans:** in A you see `Index Seek (IX_Orders_Customer)` → `Key Lookup (Clustered)` with a nested-loops join: reads ≈ index depth + rows × clustered depth. B shows only `Index Seek`. C shows `Clustered Index Scan` with a predicate on `YEAR()`; rewrite as the range in B. D proves seeks are not always better: scanning wins when lookups would touch most pages. In E, the conversion also distorts cardinality estimates (the optimizer cannot use the histogram well), which can produce *worse plans*, not only scans — compare logical reads rather than plan cost percentages.

### 5. Under the hood
- **Lookup cost model.** Each lookup is a random read in the clustered index: ~3 page reads for a 3-level tree, mostly cache hits for upper levels. A rule of thumb: the "tipping point" where scan beats seek+lookup is around 25–33% *of pages* — i.e. often 0.5–5% of rows depending on row width.
- **Included columns.** Stored only at leaf level (not in internal nodes), so they widen leaves but not the tree depth; covering for hot queries is usually worth it. The more included columns, the fewer rows per page and the more writes touch the index.
- **Statistics and the histogram.** Up to 200 steps; the estimate for `col = @v` uses the step containing `@v` (or density for unknown variables). Stale stats produce wrong access paths; sniffed parameter values produce plan-per-value effects.
- **Order and multi-column keys.** Index `(a, b)` supports seeks on `a`, `a,b`; ordering by `a, b`; and `a = ? ORDER BY b` without a sort. Column order: equality predicates first, then range/sort.
- **Ever-increasing keys & append paths.** Sequential keys target the rightmost leaf (PostgreSQL "fastpath", SQLite "quickbalance"), cheap inserts, but make last-page contention possible.
- **Data conversion.** Type precedence decides which side is converted; converting the *column* disables seeks. Matching parameter types to column types restores the seek and correct estimates.
- **Other access methods.** Hash/bitmap indexes, columnstore (segment elimination, batch mode), partition elimination, full-text, spatial, filtered indexes — all alter the set of available paths.

### 6. Performance and design implications
- Make predicates sargable: no functions/expressions on the column, matching types, ranges instead of `YEAR()`/`CONVERT`.
- Index for the *workload*: equality columns, then range, then include columns for coverage; avoid dozens of overlapping indexes (write cost).
- Keep statistics current (`AUTO_UPDATE_STATISTICS`, scheduled updates with adequate sampling).
- Use filtered indexes for skewed columns (`WHERE Status = 'O'`).
- Check `sys.dm_db_missing_index_details` as hints, not commands; check `sys.dm_db_index_usage_stats` for unused indexes.
- Measure with `SET STATISTICS IO/TIME`, actual plans, Query Store; compare logical reads.
- Avoid `SELECT *` — it defeats covering indexes and widens lookups.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … INCLUDE (…)` | Covering nonclustered index | Remove key lookups |
| `CREATE INDEX … WHERE …` | Filtered index | Skewed/low-cardinality predicates |
| `SET STATISTICS IO, TIME ON` | Reads/time per query | Compare access paths |
| `SET SHOWPLAN_XML / actual plan` | See operators | Diagnose scan vs seek |
| `UPDATE STATISTICS` | Refresh histogram | After big data changes |
| `WITH (INDEX(ix))` hint | Force an index | Experiments only |
| `sys.dm_db_index_usage_stats` | Seeks/scans/lookups/updates per index | Prune and validate |
| `EXPLAIN` / `EXPLAIN ANALYZE` (PG/MySQL) | Show chosen path | Non-SQL Server engines |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Narrow index + lookups | Many rows match or table is wide |
| Covering index | Query list changes often; writes are heavy |
| Clustered range on date | Date column isn't the dominant filter |
| Filtered index | Parameterized queries where filter predicate isn't guaranteed to match |
| Table scan | Table small or most rows needed — it's the right answer |

### 9. Common mistakes
- Functions on indexed columns → scan → rewrite as range.
- Type mismatch → implicit conversion, scan/bad estimates → match types.
- Wrong column order in composite index → index unusable for the common query → order by usage.
- Over-indexing → slower writes, bloated memory → drop unused/duplicate.
- Forcing indexes with hints permanently → plans rot as data changes → fix root cause.
- Judging by "cost %" in a batch → misleading → compare IO and time.
- Ignoring stale statistics → wrong path → update stats.

### 10. 🎯 Interview kit
**30-second answer:** An access path is how a query reaches its rows: scan, seek, or seek plus lookup. The optimizer estimates selectivity from statistics and chooses the cheapest. Seeks win for selective sargable predicates; scans win when many rows match; covering indexes avoid lookups.

**Structure a longer answer:** clarify (predicates, selectivity, columns needed) → B-tree seek mechanics → lookup cost and tipping point → design (key order, includes, filtered) → diagnostics (plans, IO, stats).

**Likely follow-ups:**
- *Why did the optimizer choose a scan though an index exists?* → low selectivity, non-sargable predicate, stale stats, missing leading column, or lookup cost.
- *Seek vs scan?* → predicate-driven navigation vs full read.
- *What is a key lookup and how to avoid it?* → include columns or reduce selected columns.
- *Column order in composite index?* → equality first, range next, sort needs.
- *How does data conversion affect plans?* → disables seeks, distorts cardinality.

**Signals interviewers look for:** you think in terms of I/O and selectivity, read plans, and fix queries and indexes together.

### 11. Check yourself
- [ ] Trace a B-tree seek and say how many page reads it takes for 100M rows.
- [ ] Rewrite `WHERE YEAR(d) = 2026` into a sargable form.
- [ ] Explain why a covering index can change the optimizer's decision.
- [ ] Choose key order for `WHERE a = ? AND b > ? ORDER BY c`.
- [ ] Diagnose a plan that scans despite an index on the filtered column.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What is a delimiter and why do CSV files break when a text field contains a comma or newline?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22614_What_is_a_delimiter_and_why_do_CSV_files_break_when_a_text_f.md>)
- [Intermediate] [What does the term "cardinality" mean for a column and how does it affect index selectivity?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22249_What_does_the_term_cardinality_mean_for_a_column_and_how_doe.md>)
- [Deep] [What is predicate pushdown in a columnar file format and how does it reduce the amount of data scanned?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22568_What_is_predicate_pushdown_in_a_columnar_file_format_and_how.md>)
- [Deep] [How would you paginate a large result set efficiently, and why is OFFSET slow for deep pages?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22240_How_would_you_paginate_a_large_result_set_efficiently_and_wh.md>)
- [Deep] [You added an index but writes got slower. Explain the write-amplification trade-off you are seeing.](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22255_You_added_an_index_but_writes_got_slower_Explain_the_write-a.md>)
- [Deep] [What is a descending index and when does adding DESC on a column let the planner avoid a sort for ORDER BY ... DESC?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22481_What_is_a_descending_index_and_when_does_adding_DESC_on_a_co.md>)
- [Deep] [How would you tune a query where an implicit data-type conversion (comparing a VARCHAR column to a number) silently disables the index?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22491_How_would_you_tune_a_query_where_an_implicit_data-type_conve.md>)
- [Expert] [You have predicates on col_a = ? and col_b > ? and an ORDER BY col_b. What composite index column order lets you both filter and sort without a separate sort?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22480_You_have_predicates_on_col_a_and_col_b_and_an_ORDER_BY_col_b.md>)

## Related concepts

- [Leftmost Prefix Rule](<../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Query Rewrites and Plan Stability](<../../Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Indexing Multi-join Queries](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>) *(PostgreSQL Platform)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Creating an Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Syntax
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 7 Fill Factor Is Applied to Indexes During Inserts › Myth 7: Fill Factor Is Applied to Indexes During Inserts
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — T-SQL Scripts › Fill Factor
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › Query Languages for Data
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Column Compression
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Writing to Column-Oriented Storage
- *Designing Data-Intensive Applications* — Martin Kleppmann — Snapshot Isolation and Repeatable Read › Indexes and snapshot isolation
- *Database Internals* — Alex Petrov — Implementing B-Trees › Right-Only Appends
- *Database Internals* — Alex Petrov — Log-Structured Storage › Leveled compaction
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Point queries
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › Range queries
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Glossary
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE › Non-clustered Columnstore Index Architecture

