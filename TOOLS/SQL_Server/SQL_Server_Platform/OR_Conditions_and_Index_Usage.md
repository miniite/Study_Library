---
id: "TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/SQL_Server_Platform"
tools: ["SQL_Server"]
---

# OR Conditions and Index Usage

**L4 Advanced** · [↑ SQL Server Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An `OR` across different columns forces SQL Server to find rows by two different paths, which often turns an index seek into a scan; you can restore seeks with indexes on each branch, `UNION ALL`, or by rewriting the predicate.

### 1. The big picture
Index seeks work by navigating a B-tree to a contiguous key range. Predicates joined with `AND` narrow that range; predicates joined with `OR` widen it, and when the branches reference different columns no single index range can cover both. The optimizer then chooses between a scan, an index union (seek each index and merge), or a rewritten plan. Understanding when it can do the union — and when it can't — is key to writing sargable queries.

### 2. Core concepts and vocabulary
- **Sargable** — a predicate form that lets the engine seek an index (`col = @x`, `col >= @a`, `col LIKE 'abc%'`).
- **Seek predicate vs predicate** — the seek predicate bounds the index range; a residual predicate filters rows afterward.
- **Index union** — plan that seeks two indexes and merges/de-duplicates row locators (Concatenation + Sort Distinct or Merge Join).
- **Residual predicate** — filter applied to rows already read.
- **`IN` list** — shorthand for ORs on one column; is sargable on that column.
- **`UNION ALL` rewrite** — one query per branch, combined.
- **Overlap / duplicates** — rows satisfying both branches; must be handled in rewrites.
- **Key lookup** — fetch of non-indexed columns after an NC seek.

### 3. How it works, step by step
1. The optimizer normalizes the `WHERE` into a boolean tree.
2. **Same column** ORs (`a = 1 OR a = 5`, or `IN`) become a set of seek ranges on one index — fine.
3. **Different columns** (`a = 1 OR b = 2`) need both an index on `a` and one on `b` for an index union; if either is missing, it must scan the clustered index/heap.
4. Even with both indexes, the optimizer compares cost: if the estimated rows are large or lookups expensive, it chooses a scan.
5. Functions on columns inside either branch (`ISNULL(a,0)=0 OR ...`) make that branch non-sargable and spoil the whole union.
6. You influence the result with indexes, rewrite, or hints, then verify in the plan.

### 4. Worked example
```sql
CREATE INDEX IX_Orders_CustomerID ON dbo.Orders (CustomerID) INCLUDE (OrderDate, Amount);
CREATE INDEX IX_Orders_SalesRepID ON dbo.Orders (SalesRepID) INCLUDE (OrderDate, Amount);

-- Often scans when ranges are wide
SELECT OrderID, OrderDate, Amount
FROM dbo.Orders
WHERE CustomerID = 42 OR SalesRepID = 7;

-- Rewrite: two seeks, no duplicates
SELECT OrderID, OrderDate, Amount FROM dbo.Orders WHERE CustomerID = 42
UNION ALL
SELECT OrderID, OrderDate, Amount FROM dbo.Orders
WHERE SalesRepID = 7 AND (CustomerID <> 42 OR CustomerID IS NULL);
```
Read the plans: the first may show a clustered index scan with the OR as a residual predicate, or an index union with a `Merge Join (Union)`/`Sort (Distinct Sort)`; the second shows two index seeks and a `Concatenation`. The extra condition on the second branch stops rows matching both from being returned twice (plain `UNION` also dedupes, but adds a sort). If `OrderID` isn't unique across the union you must remove duplicates differently.

### 5. Under the hood
- Row locators from each seek (RIDs or clustering keys) are merged; to avoid returning a row twice the engine must sort/hash distinct on the locator — this is why the union plan has a distinct sort step.
- Cost thresholds: reading roughly more than ~1/4–1/3 of the pages as lookups costs more than a scan, so large result sets prefer scans regardless of indexes.
- With a covering index (via `INCLUDE`) the lookup cost disappears and the union becomes much more attractive.
- Parameter sniffing interacts: `@a IS NULL OR col = @a` ("catch-all") compiles a plan for the first parameter set; the plan cannot seek when the parameter is null.
- `OPTION (RECOMPILE)` lets the optimizer embed the parameter value and simplify `@a IS NULL OR ...` away.

### 6. Performance and design implications
- Index every column that appears in an OR branch, and make indexes covering when queries are hot.
- Measure with `SET STATISTICS IO`, the actual plan (Seek Predicates vs Predicate), and `sys.dm_db_missing_index_details` (treat suggestions skeptically).
- For optional filters, build dynamic SQL via `sp_executesql` with only the active predicates, or use `OPTION (RECOMPILE)` for infrequent calls.
- Prefer `IN` or `EXISTS` forms over repeated OR when applicable; `JOIN` to a values list for large sets.
- `LIKE '%x'` and `OR` across a leading wildcard cannot seek regardless.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `UNION ALL` rewrite | Splits OR into seekable parts | Different-column ORs |
| `INCLUDE (cols)` | Makes index covering | Remove key lookups |
| `OPTION (RECOMPILE)` | Optimizes with actual values | Catch-all search procs |
| `sp_executesql` | Parameterized dynamic SQL | Optional filter queries |
| `SET STATISTICS IO ON` | Logical reads per table | Compare variants |
| `sys.dm_db_missing_index_details` | Optimizer suggestions | Hints, not orders |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `UNION ALL` rewrite for selective branches | Branches overlap heavily or are non-selective |
| Index each branch + keep `OR` | Table is write-heavy (extra index cost) |
| Dynamic SQL for optional filters | Strict no-dynamic-SQL policy; injection risk if unparameterized |
| `OPTION (RECOMPILE)` | Very frequent calls (compile CPU) |

### 9. Common mistakes
- Using `UNION` when `UNION ALL` is fine → extra sort/distinct → know your overlap.
- Using `UNION ALL` without excluding overlap → duplicate rows → add the exclusion predicate.
- Catch-all `(@x IS NULL OR col = @x)` in a stored proc → one cached bad plan → recompile or dynamic SQL.
- Creating an index on only one OR column and expecting a seek → still a scan → index both.
- Wrapping columns in `ISNULL`/`COALESCE` inside the OR → non-sargable → restructure.
- Trusting "OR is always slow" → measure; same-column ORs and small ranges seek fine.

### 10. 🎯 Interview kit
**30-second answer:** An `OR` on different columns can't be satisfied by one index range, so the optimizer may scan. Provide an index per branch, or rewrite with `UNION ALL`, taking care to prevent duplicates, then confirm in the plan.

**Structure a longer answer:** clarify (columns, selectivity, volume) → mechanism (seek ranges, index union, scan cost) → options → scenario (search procedure with optional filters).

**Likely follow-ups:**
- *Why is `IN` fine but OR across columns not?* → single column forms ranges on one B-tree.
- *UNION vs UNION ALL?* → dedupe cost vs duplicates.
- *How does parameter sniffing affect catch-all queries?* → plan built for first values; recompile fixes.
- *When is the scan actually correct?* → low selectivity, large fractions of the table.

**Signals interviewers look for:** reading plans, handling duplicates, awareness of parameter sniffing.

### 11. Check yourself
- [ ] Explain why `a=1 OR b=2` needs two indexes to avoid a scan.
- [ ] Rewrite an OR query with `UNION ALL` without duplicates.
- [ ] Fix a catch-all search procedure using `OPTION (RECOMPILE)` or dynamic SQL.
- [ ] Identify Seek Predicate versus residual Predicate in a plan.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between the WHERE clause and the HAVING clause in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41386_What_is_the_difference_between_the_WHERE_clause_and_the_HAVI.md>)
- [Basic] [What is the difference between UNION and UNION ALL in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41441_What_is_the_difference_between_UNION_and_UNION_ALL_in_SQL_Se.md>)
- [Basic] [How do you use TOP with ORDER BY to return the first N rows in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41467_How_do_you_use_TOP_with_ORDER_BY_to_return_the_first_N_rows_.md>)
- [Basic] [What is a subquery in SQL Server and where can you use one?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41506_What_is_a_subquery_in_SQL_Server_and_where_can_you_use_one.md>)
- [Intermediate] [What does a FULL OUTER JOIN return and how is it different from a CROSS JOIN?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41419_What_does_a_FULL_OUTER_JOIN_return_and_how_is_it_different_f.md>)
- [Intermediate] [What does the OVER() clause with PARTITION BY do in a window function?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41515_What_does_the_OVER_clause_with_PARTITION_BY_do_in_a_window_f.md>)
- [Intermediate] [Explain how OR conditions in a WHERE clause can defeat index usage, and how UNION ALL or a rewrite can restore seeks.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41421_Explain_how_OR_conditions_in_a_WHERE_clause_can_defeat_index.md>)
- [Deep] [What is a table scan versus an index seek in a SQL Server execution plan?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41535_What_is_a_table_scan_versus_an_index_seek_in_a_SQL_Server_ex.md>)

## Related concepts

- [WHERE vs HAVING](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [Join Types](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>) *(SQL Fundamentals)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Data Conversion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › WHERE Clause Predicates and Logical Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › The EXISTS Operator
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › INTERSECT Operator
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › OR Conditions and IN Clauses
- *SQL Server Tutorial* (YouTube series) — Part 36: Clustered and nonclustered indexes in sql server ([watch](https://www.youtube.com/watch?v=NGslt99VOCw))
- *SQL Server Tutorial* (YouTube series) — Part 87: SQL Server except operator ([watch](https://www.youtube.com/watch?v=wAj02hGQYs8))
- *SQL Server Tutorial* (YouTube series) — Part 88: Difference between except and not in sql server ([watch](https://www.youtube.com/watch?v=Vn_7hBPmw2o))
- *SQL Server Tutorial* (YouTube series) — Part 89: Intersect operator in sql server ([watch](https://www.youtube.com/watch?v=GfQD28ektk8))

