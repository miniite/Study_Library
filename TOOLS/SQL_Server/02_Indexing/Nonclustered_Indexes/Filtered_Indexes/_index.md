---
id: "TOOLS/SQL_Server/02_Indexing/Nonclustered_Indexes/Filtered_Indexes"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Filtered Indexes

**L5 Expert** · [↑ Nonclustered Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A filtered index is a nonclustered index with a `WHERE` clause. It stores only the rows that match, so it is smaller, cheaper to maintain and has more accurate statistics, but the optimizer can only use it when it can *prove* at compile time that the query's rows fall inside the filter.

### 1. The big picture
Many tables have a small "interesting" subset that queries focus on: open orders among millions of closed ones, active users, rows where `ProcessedAt IS NULL`, non-NULL values of a sparse column. A normal index on such a column indexes *every* row, including the 99% nobody searches. A **filtered index** (SQL Server 2008+) indexes only the rows matching its predicate. The result is a tiny B-tree that seeks fast, fits in memory, costs nothing when non-matching rows change, and comes with its own **filtered statistics** whose histogram describes only the subset, which is far more precise than a table-wide histogram.

Filtered indexes also solve a classic modelling problem: **unique-where-not-null**. A unique constraint allows only one NULL, but a unique filtered index `WHERE col IS NOT NULL` enforces uniqueness among real values only.

The catch is **matching**. The optimizer must be sure that every row the query needs is inside the filter. That works with literals, but often fails with parameters and with simple or forced parameterization. It also requires specific SET options. Knowing these rules is what separates using filtered indexes well from being puzzled why they're ignored.

### 2. Core concepts and vocabulary
- **Filter predicate**: the index's `WHERE` clause. Only simple comparisons are allowed: `=, <>, !=, >, >=, <, <=, IS NULL, IS NOT NULL, IN (…)`, combined with `AND`. Not allowed: `OR`, `NOT IN`, `LIKE`, `BETWEEN` (write `>= AND <=`), functions, computed columns, UDT, spatial or hierarchyid columns, and references to other tables.
- **Filtered statistics**: statistics built only on the rows matching a filter. They are created automatically with the filtered index, or created manually with `CREATE STATISTICS … WHERE …`.
- **Predicate matching / subsumption**: the optimizer's proof that the query predicate implies the index filter (e.g. `Status = 'Open'` implies `Status IN ('Open','Pending')`).
- **Parameter sniffing**: the plan is compiled for the first parameter values, but must be valid for *any* value. That is why parameters often prevent filtered-index use.
- **Unique filtered index**: enforces uniqueness only among rows in the filter.
- **Required SET options**: `ANSI_NULLS, ANSI_PADDING, ANSI_WARNINGS, ARITHABORT, CONCAT_NULL_YIELDS_NULL, QUOTED_IDENTIFIER` ON, and `NUMERIC_ROUNDABORT` OFF. These are the same as for indexed views and computed-column indexes.

### 3. How it works, step by step
1. **Create.** `CREATE INDEX ix ON t(cols) [INCLUDE (…)] WHERE <filter>`. SQL Server scans the table and builds a B-tree with only the matching rows, plus filtered statistics.
2. **Maintain.** On every INSERT, UPDATE or DELETE, the engine evaluates the filter for the old and new row versions. A row entering the filter is inserted into the index, a row leaving it is deleted, and rows outside it are ignored. Updates that change a filter column can move rows in or out.
3. **Compile a query.** The optimizer considers the filtered index only if the query's WHERE clause logically implies the filter **for every possible execution of this plan**:
   - `WHERE Status = 'Open'` (a literal) against `WHERE Status = 'Open'` → match.
   - `WHERE Status = @s` (a parameter) → no match in general, because the next execution could pass `'Closed'`.
   - `WHERE Status = @s OPTION (RECOMPILE)` → match, because the plan is compiled for the actual value and not reused.
4. **Cover the filter column.** If the filter column isn't in the index (key or INCLUDE), the optimizer sometimes adds a key lookup to re-check it, or can't use the index for certain shapes. Including the filter column (cheaply, in INCLUDE) avoids this. The book recommends it as a habit.
5. **Statistics.** The filtered statistics' auto-update threshold is based on the *table's* modification counter, not the subset's. A small hot subset can change completely before its statistics refresh, so schedule manual updates for volatile filtered indexes.

**Where filtered indexes shine:**

| Scenario | Example filter |
|---|---|
| Hot subset of a large table | `WHERE Status IN ('Open','Pending')` |
| Queue / work table | `WHERE ProcessedAt IS NULL` |
| Sparse column | `WHERE DiscontinuedDate IS NOT NULL` |
| Unique-where-not-null | `UNIQUE … WHERE TaxId IS NOT NULL` |
| Skewed data with bad estimates | filtered statistics on the skewed slice |
| Partition-like slicing without partitioning | `WHERE OrderDate >= '2026-01-01'` (static) |

### 4. Worked example
```sql
-- 10M orders, ~0.5% open
CREATE NONCLUSTERED INDEX IX_Orders_Open
ON Sales.Orders (CustomerID, OrderDate)
INCLUDE (Status, TotalDue)                -- include the filter column
WHERE Status = 'Open';

SELECT CustomerID, OrderDate, TotalDue
FROM Sales.Orders WHERE Status = 'Open' AND CustomerID = 42;     -- uses IX_Orders_Open

-- Parameterized: index NOT used (plan must work for any @s)
DECLARE @s varchar(10) = 'Open';
SELECT CustomerID, OrderDate, TotalDue
FROM Sales.Orders WHERE Status = @s AND CustomerID = 42;

-- Fixes: literal in code path, OPTION (RECOMPILE), or a dedicated procedure branch
SELECT CustomerID, OrderDate, TotalDue
FROM Sales.Orders WHERE Status = @s AND CustomerID = 42
OPTION (RECOMPILE);

-- Unique where not null
CREATE UNIQUE NONCLUSTERED INDEX UX_Customers_TaxId
ON Sales.Customers (TaxId) WHERE TaxId IS NOT NULL;

-- Size comparison
SELECT i.name, SUM(ps.used_page_count) AS pages, SUM(ps.row_count) AS rows
FROM sys.dm_db_partition_stats ps JOIN sys.indexes i
  ON i.object_id = ps.object_id AND i.index_id = ps.index_id
WHERE ps.object_id = OBJECT_ID('Sales.Orders') GROUP BY i.name;

-- List filtered indexes
SELECT OBJECT_NAME(object_id) AS tbl, name, filter_definition
FROM sys.indexes WHERE has_filter = 1;
```
What to observe: the filtered index has about 50,000 rows instead of 10M, a few hundred pages instead of tens of thousands, and its statistics histogram covers only open orders. In the parameterized version, the plan uses a different index or a scan. Often the plan shows an *UnmatchedIndexes* warning in its XML, which tells you a filtered index existed but couldn't be matched. A session with the wrong SET options (e.g. an old client with `ANSI_NULLS OFF`) gets error 1934 when it modifies the table: *"INSERT failed because the following SET options have incorrect settings"*.

### 5. Under the hood
- **Matching is purely logical.** The optimizer does no runtime check. It only proves implication from the query text and constants. Simple parameterization (auto-parameterizing trivial queries) and `FORCED` parameterization can turn your literal into a parameter and silently disable matching. Statements that hit the filtered index frequently need to stay out of forced parameterization (e.g. via a plan guide), or use `OPTION (RECOMPILE)`.
- **Leading-column rule still applies.** A myth the indexing book debunks is that any column of a multi-column index can be used to filter efficiently. Only a left prefix of the key can be seeked, and only the leading column has a histogram. A filtered index doesn't change that. The filter selects *rows*; the key order still governs *seeks*.
- **Statistics staleness.** The modification counter for the table drives auto-update. With 10M rows, the dynamic threshold (about √(1000 × rows) on 2016+, about 100K) may never be reached by churn in a 50K-row subset, so the filtered stats can describe yesterday's open orders. Add `UPDATE STATISTICS Sales.Orders IX_Orders_Open` to frequent maintenance.
- **MERGE statements.** Historically, `MERGE` with a unique filtered index on the target could raise spurious duplicate-key errors in some versions. Test MERGE logic against filtered unique indexes, or use separate INSERT/UPDATE statements.
- **Where they can't be used.** Filtered indexes can't be clustered, can't be created on views, and the filter can't reference computed columns. Filtered *columnstore* indexes (NCCI with WHERE) exist separately; they suit the hot/cold pattern in *Columnstore with Rowstore*.
- **OR rewriting.** Because filters can't contain `OR`, a "status is A or B" requirement uses `IN ('A','B')`. Queries with `OR` across *different* columns can't match a single filtered index; rewriting them as `UNION ALL` branches can let each branch match.

### 6. Performance and design implications
- The best payoff is high selectivity (a small fraction of rows) plus high query frequency on that slice.
- Write savings: modifications to rows outside the filter cost nothing for this index. That's ideal for big history tables with a small active window.
- Verify use in the actual plan and watch for `UnmatchedIndexes` warnings. Track `user_seeks` in usage stats. A filtered index with zero seeks usually means parameterization defeated it.
- Plan statistics maintenance explicitly for volatile subsets.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … WHERE <predicate>` | Filtered index | Hot or sparse subsets |
| `CREATE UNIQUE INDEX … WHERE col IS NOT NULL` | Unique among non-NULLs | Optional natural keys |
| `CREATE STATISTICS … WHERE …` | Filtered statistics without an index | Skewed estimates |
| `OPTION (RECOMPILE)` | Compile with actual values | Parameterized queries on a filtered index |
| `sys.indexes.has_filter / filter_definition` | Lists filters | Inventory |
| `UPDATE STATISTICS t ix WITH FULLSCAN` | Refresh filtered stats | Volatile subsets |
| Required SET options (`ANSI_NULLS`, `QUOTED_IDENTIFIER`, …) | Must be ON for DML | Avoid error 1934 |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Queries target a small, well-defined subset with literals | The subset value is always a parameter (and RECOMPILE is unacceptable) |
| Unique-where-not-null rules | Clients with legacy SET options modify the table |
| Large table, small active window | The subset is a large fraction of the table (little gain) |
| Filtered stats to fix skewed estimates | Forced parameterization is on for the database |

### 9. Common mistakes
- Expecting `WHERE Status = @s` to use the index → it's ignored → use a literal branch or `OPTION (RECOMPILE)`.
- Writing `BETWEEN` or `OR` in the filter → creation error → use `>= AND <=`, or `IN`.
- Not including the filter column → extra lookups, or the index isn't chosen → INCLUDE it.
- Never updating filtered statistics → bad estimates on the hot subset → schedule manual updates.
- Deploying without checking client SET options → DML fails with error 1934 → standardize the connection settings.

### 10. 🎯 Interview kit
**30-second answer:** "A filtered index is a nonclustered index with a WHERE clause, so it holds only the matching rows. It's smaller, cheaper to maintain, and has precise statistics for the subset. It's great for hot subsets, queue tables, sparse columns and unique-where-not-null. The catch is that the optimizer must prove the query falls inside the filter at compile time, so parameterized queries often can't use it without RECOMPILE, and it needs specific SET options."
**Structure a longer answer:** what it stores → use cases → maintenance behaviour → matching rules (literals vs parameters, forced parameterization) → statistics staleness → SET options.
**Likely follow-ups:** *Why isn't my filtered index used?* → parameters / forced parameterization / filter column not covered. *How do you allow multiple NULLs in a unique column?* → a unique filtered index WHERE col IS NOT NULL. *What can't a filter contain?* → OR, LIKE, BETWEEN, functions, computed columns.
**Signals interviewers look for:** matching rules, the stats auto-update caveat, practical use cases, and the unique-where-not-null pattern.

### 11. Check yourself
- [ ] Create a filtered index for a queue table and compare its size with a full index.
- [ ] Show a parameterized query that fails to use it, and fix it two ways.
- [ ] Build a unique-where-not-null constraint and test it with several NULLs.
- [ ] Explain why filtered statistics can go stale on a large table.
- [ ] Rewrite a filter that uses `BETWEEN` and `OR` into a legal one.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Filtered Indexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 10 Index Behaviors
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Filtered Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › OR Conditions and IN Clauses
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Optimizing OR Predicates
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 4 Any Column Can Be Filtered in Multicolumn Indexes › Myth 4: Any Column Can Be Filtered in Multicolumn Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Scalar Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Order of Operations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Predicates in the WHERE Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › The WHERE Clause

