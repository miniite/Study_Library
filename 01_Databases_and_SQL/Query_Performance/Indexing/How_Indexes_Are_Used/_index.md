---
id: "01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# How Indexes Are Used

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An index is a separate, sorted lookup structure (usually a B-tree) that lets the engine jump to matching rows instead of reading every row — but the optimizer only uses it when the predicate is searchable and the estimated saving beats a scan.

### 1. The big picture
Without an index, a `WHERE` clause forces a **full table scan**: read every page, test every row. That's linear in table size. A B-tree index turns lookup into logarithmic: a few page reads to find the first match, then a contiguous read of the rest. The book analogy is exact — instead of flipping through every page, jump to the entry in the index at the back.

Two things are often confused:
- *Having* an index (a structure that exists and must be maintained on every write).
- *Using* an index (the optimizer picked it for this query's plan).

This lesson is the foundation for the rest of the Indexing topic: it explains what the structure is, how a query "hits" it, why sometimes a scan wins, what it costs, and how to see what happened.

### 2. Core concepts and vocabulary
- **Table scan / sequential scan** — read the whole table (heap or clustered index) in storage order.
- **Index seek** — navigate the B-tree to a specific key or range start.
- **Index scan** — read the index's leaf level in order (partial or full).
- **Clustered index** — the table's rows themselves are stored in key order (SQL Server clustered, InnoDB primary key). One per table.
- **Nonclustered (secondary) index** — separate structure of key + locator pointing to the row.
- **Heap** — a table with no clustered index; rows are located by a physical row id (RID/TID).
- **Lookup (key/RID/heap fetch)** — hop from a secondary index entry to the base row for columns not in the index.
- **Selectivity** — fraction of rows matching a predicate; high selectivity = few rows.
- **Cardinality** — number of distinct values (or, in plan context, row count).
- **Statistics** — histograms/density the optimizer uses to estimate rows.
- **Sargable** — predicate form an index can serve (`col = ?`, `col > ?`, `col BETWEEN`, `col LIKE 'abc%'`).

### 3. How it works, step by step
**Lookup by a nonclustered index:**
1. Start at the **root page**; compare the search key with separator keys to pick a child.
2. Descend through **intermediate pages** (typically 2–4 levels for millions or billions of rows; each page ≈ 8 KB holding hundreds of keys).
3. Reach the **leaf page** holding the first matching key.
4. Read forward along leaf pages (they are linked) while keys still match (range scans).
5. For each matching entry, if the query needs more columns, do a **lookup** into the clustered index / heap.

**Choosing the access path (optimizer):**
1. Parse predicates, identify sargable ones and candidate indexes.
2. Estimate rows each predicate returns using statistics.
3. Cost alternatives: table scan (sequential I/O, cheap per page) vs index seek + lookups (random I/O, expensive per row).
4. Pick the cheapest plan; "faster than a scan" is not guaranteed.

Rule of thumb: when a predicate returns more than a few percent of the table (the threshold varies, often ~1–10 %), a scan beats seek+lookup. That's why `country = 'IN'` returning 80 % of rows won't use an index on `country`, and why a 10-row test table tells you nothing.

### 4. Worked example
```sql
CREATE TABLE Product (ProductID int PRIMARY KEY, Name nvarchar(100), ListPrice money);
-- no other indexes

SET STATISTICS IO ON;

-- 1) no usable predicate: scan is the only option
SELECT Name FROM Product;                       -- Clustered Index Scan

-- 2) unique key predicate: seek
SELECT Name FROM Product WHERE ProductID = 707; -- Clustered Index Seek, ~2 logical reads

-- 3) predicate on unindexed column: scan + filter
SELECT ProductID FROM Product WHERE Name = N'Mountain Bike'; -- Scan, reads every page

CREATE INDEX IX_Product_Name ON Product (Name);

-- 4) now seek on IX_Product_Name; ProductID comes free (clustered key rides in the index)
SELECT ProductID FROM Product WHERE Name = N'Mountain Bike';

-- 5) ListPrice isn't in the index -> Key Lookup per matching row
SELECT ListPrice FROM Product WHERE Name = N'Mountain Bike';
```
Read the plan: operators (Seek vs Scan), thickness of arrows (rows), `Actual vs Estimated Rows`, and `logical reads` in the Messages tab. In the cited timing example, adding an unambiguous key predicate cut reads to 2 and duration from ~534 µs to ~132 µs. Note two measuring tips: don't capture `STATISTICS IO` and the actual plan *while* timing (they distort duration), and a table that fits on one page may show a scan even with a perfect index, because a scan is as cheap as a seek.

### 5. Under the hood
- **B-tree shape.** Pages are fixed size (8 KB SQL Server, 8 KB PostgreSQL, 16 KB InnoDB). With ~500 keys per page, 3 levels address ~125 M rows; a lookup is ~3 page reads, usually all cached for upper levels.
- **Clustered vs heap.** In a clustered table the leaf level *is* the data. In a heap, secondary-index leaves store a RID/TID; in PostgreSQL every table is a heap and the primary key index is just another B-tree.
- **Secondary index locators.** SQL Server and InnoDB append the clustering/primary key to every secondary index entry (so wide primary keys inflate every index; "use narrow indexes"). PostgreSQL stores a 6-byte TID.
- **Ordered scans.** A B-tree leaf chain supports `ORDER BY`, `MIN/MAX` (read the first/last leaf entry — the book's `MIN(UnitPrice)` went from ~1,248 reads to a few after adding an index), merge joins and `GROUP BY` with streaming aggregation.
- **Hash indexes** exist (PostgreSQL, SQL Server memory-optimized tables) but support only equality; bucket count must be sized to rows (about 2–5× for in-memory hash indexes).
- **Page cache.** Index pages that fit in memory (buffer pool/shared buffers) eliminate physical I/O; the "hot" upper levels stay cached by LRU-like policies. An index that doesn't fit forces disk reads on each probe — why index size/width matters.
- **Filtered (partial) indexes** include only rows matching a predicate, shrinking size and maintenance and giving a high-selectivity structure out of a low-selectivity column such as `status = 'open'`.

### 6. Performance and design implications
- **Reads get faster, writes slower.** Every `INSERT`/`DELETE` touches every index; an `UPDATE` touches every index containing a modified column. Over-indexing slows OLTP and bloats storage.
- **Index what you filter, join and sort on.** Foreign keys deserve an index (the child side), otherwise parent `DELETE`/`UPDATE` scans the child table — a pattern that turned 4,513 reads into 2 in the cited example.
- **Narrow keys** (int over varchar(50)) → more entries per page → shallower, smaller, cached.
- **Selectivity matters.** Low-cardinality columns (booleans, country) are poor *leading* keys alone; fine as filtered-index predicates or trailing columns.
- **Don't index everything on analytic tables.** Wide 100+ column warehouse/staging tables loaded fully and read by scans are better served by columnar storage, partition pruning or min/max statistics; for bulk loads, drop/disable indexes, load, rebuild.
- **MySQL EXPLAIN `type`:** `system/const` (single row) > `eq_ref` > `ref` > `range` > `index` (full index scan) > `ALL` (full table scan).
- **Monitoring:** `sys.dm_db_index_usage_stats` (seeks, scans, lookups, updates), `pg_stat_user_indexes` (`idx_scan`, `idx_tup_read`, `idx_tup_fetch`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [UNIQUE] INDEX ix ON t (cols)` | Builds a B-tree; `UNIQUE` also enforces uniqueness | Speed lookups / enforce rule |
| `DROP INDEX ix ON t` / `DROP INDEX ix` | Removes the structure; table data unchanged | Pruning unused indexes |
| `EXPLAIN` / `EXPLAIN ANALYZE` / `SET STATISTICS IO ON` | See plan and I/O | Always, before and after |
| `SET STATISTICS TIME ON` | CPU/elapsed | Timing comparisons |
| `UPDATE STATISTICS` / `ANALYZE` | Refresh selectivity data | After big loads |
| `CREATE INDEX … WHERE …` / filtered index | Partial index | Skewed or soft-delete data |
| `sys.dm_db_index_usage_stats`, `pg_stat_user_indexes` | Usage counters | Index review |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Predicate returns a small fraction of a large table | Table is tiny (a page or two) or the predicate returns most rows |
| Read-heavy table, selective filters/joins | Write-heavy table where each extra index adds latency |
| Join/FK columns on the child side | Column is rarely queried or has 2 distinct values |
| `ORDER BY … LIMIT n` served by index order | Wide table scanned fully by reports (columnar/aggregate instead) |
| Staging table loaded nightly and read once | Index maintained during load — drop and recreate instead |

### 9. Common mistakes
- **"Index everything"** → write slowdown, storage bloat → index measured access paths.
- **Function on the indexed column** (`WHERE YEAR(d) = 2025`) → scan → keep the column bare or use an expression index.
- **Testing on 10 rows** → "index doesn't matter" → test on production-sized data; stats and cost thresholds change plans.
- **Assuming the index is used** → the plan says scan → read the plan, check estimates and sargability.
- **Implicit conversion** (`varchar` column compared to `nvarchar` parameter) → index can't seek → match types.
- **Forgetting FK indexes** → parent deletes scan child → index FK columns.
- **Confusing PK and unique index** → a primary key is a unique, non-null constraint *implemented by* an index; other indexes are not constraints.

### 10. 🎯 Interview kit
**30-second answer:** An index is a sorted lookup structure, typically a B-tree, stored separately from the data. It lets the database descend a few pages to find matching keys instead of reading the whole table, so reads on selective predicates go from linear to logarithmic. The trade-off is extra storage and slower writes, and the optimizer may still choose a scan if the predicate isn't selective.

**Structure a longer answer:** problem (scan cost) → B-tree mechanics (root→leaf, ordered leaves) → clustered vs nonclustered + lookups → optimizer decision using statistics → costs (writes, space) → how you verify (plan, reads) → example.

**Likely follow-ups:**
- *Why can a full scan beat an index?* → Random lookups per row cost more than sequential page reads once a sizeable fraction of the table matches, or when the table is tiny.
- *What does PK index vs regular index mean?* → PK = unique + not null constraint, usually the clustered index; a regular index has no constraint.
- *Does every table have an index?* → Not necessarily (heap); a PK/unique constraint creates one automatically; foreign keys don't (except in MySQL InnoDB).
- *What's index selectivity and how to check it?* → `COUNT(DISTINCT col)/COUNT(*)`; more important is per-value selectivity from the histogram.
- *Why are in-memory indexes faster?* → No physical I/O; cost becomes CPU only.

**Signals interviewers look for:** correct B-tree explanation, understanding of the scan-vs-seek cost trade-off, awareness of write costs, evidence-based tuning.

### 11. Check yourself
- [ ] Sketch a B-tree lookup for 100 M rows and count page reads.
- [ ] Explain Seek vs Scan vs Lookup in a plan.
- [ ] Give two situations where a scan is correct even though an index exists.
- [ ] Explain why a narrow clustered key helps every nonclustered index.
- [ ] State what happens to table data when an index is dropped.
<!-- explainer:end -->

## Go deeper

- [Covering Indexes and Index Strategy for Sharding](<Covering_Indexes_and_Index_Strategy_for_Sharding.md>)

## Practice questions

- [Basic] [You wrote a Python script that reads a table row by row using pandas.read_sql with a filter. How does a missing index on the filter column hurt it?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20283_You_wrote_a_Python_script_that_reads_a_table_row_by_row_usin.md>)
- [Basic] [Does every table automatically have any index by default? If so, which one?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19974_Does_every_table_automatically_have_any_index_by_default_If_.md>)
- [Basic] [Explain why SELECT * can be slower than selecting only the columns you need.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19999_Explain_why_SELECT_can_be_slower_than_selecting_only_the_col.md>)
- [Intermediate] [What happens to indexes during INSERT, UPDATE, and DELETE operations?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19481_What_happens_to_indexes_during_INSERT_UPDATE_and_DELETE_oper.md>)
- [Intermediate] [How would you optimize a query that must scan a wide table but only three columns, using columnar projection, and quantify the bytes-scanned savings?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20454_How_would_you_optimize_a_query_that_must_scan_a_wide_table_b.md>)
- [Deep] [What is a functional (expression) index and when would you use it?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19489_What_is_a_functional_expression_index_and_when_would_you_use.md>)
- [Deep] [How does the database decide whether to use your index or ignore it, in simple terms?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20053_How_does_the_database_decide_whether_to_use_your_index_or_ig.md>)
- [Deep] [A slow query returns few rows but reads many index pages. What does a high index-tuples-read-to-rows-returned ratio imply about your index design?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20152_A_slow_query_returns_few_rows_but_reads_many_index_pages_Wha.md>)

## Related concepts

- [Query Rewrites and Plan Stability](<../../Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Leftmost Prefix Rule](<../../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Understanding Scan vs. Seek
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Introduction to Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Data Access Operators
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Clustered Indexes › Nonclustered Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hash and Range Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Filtered Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.indexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — load customer names › Chapter 23 A Query Tuning Methodology
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Understanding Indexes
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Storing values within the index
- *SQL Server Tutorial* (YouTube series) — Part 35: Indexes in sql server ([watch](https://www.youtube.com/watch?v=i_FwqzYMUvk))
- *SQL Server Tutorial* (YouTube series) — Part 38: Advantages and disadvantages of indexes in sql server ([watch](https://www.youtube.com/watch?v=jYS4PwKY6EM))

