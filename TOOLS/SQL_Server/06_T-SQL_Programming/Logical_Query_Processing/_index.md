---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Logical_Query_Processing"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Logical Query Processing

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL clauses are written in one order (SELECT first) but are *logically* evaluated in another: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY. Each phase produces a virtual table for the next. Knowing this order explains which aliases you can use where, how NULLs behave, and why some queries give "surprising" results.

### 1. The big picture
English-like syntax makes SQL easy to start. It also hides how a query is actually *defined*. You type `SELECT` first, but the column list is one of the last things computed. **Logical query processing** is the conceptual model in the SQL standard: a sequence of phases, each taking a table as input and producing a table as output. The final table is the query's result.

This is different from **physical processing**, which is what the engine actually does (the execution plan). The optimizer is free to reorder, merge or skip steps, such as pushing a filter into an index seek before a join, as long as the result matches the logical definition. So logical order tells you *what a query means*. The plan tells you *how it ran*. This lesson is the foundation for joins, table expressions and window functions in the next lessons.

### 2. Core concepts and vocabulary
- **Logical query processing**: the conceptual evaluation order of a query's clauses, defined by the standard.
- **Virtual table**: the intermediate result each phase produces and passes to the next. It is a concept, not a stored object.
- **Predicate**: a logical expression evaluated per row: TRUE, FALSE or UNKNOWN.
- **Three-valued logic**: UNKNOWN is the result of comparing with NULL. `WHERE`, `HAVING` and `ON` keep only rows where the predicate is TRUE. `CHECK` constraints reject only FALSE.
- **Group**: the set of rows that share the same values in the `GROUP BY` list. After grouping, each group becomes one output row.
- **Aggregate function**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` and others. They work on a set of rows and ignore NULLs, except `COUNT(*)`.
- **Column alias**: a name given in the `SELECT` list (`expr AS name`).
- **All-at-once operation**: all expressions in the same logical phase are evaluated as if at the same instant. One expression can't see the result of another in the same phase.
- **Presentation ordering**: `ORDER BY` turns the result from a set into a cursor-like ordered result. Only the outermost `ORDER BY` guarantees order.
- **DISTINCT**: removes duplicate rows from the `SELECT` result.
- **TOP / OFFSET-FETCH**: row-limiting filters applied after ordering.

### 3. How it works, step by step
The logical phases for a single-table or join query:

| # | Phase | Input → output | What it does |
|---|---|---|---|
| 1 | `FROM` (with joins, APPLY, PIVOT) | tables → VT1 | Builds the source rows; joins are cartesian product → `ON` filter → add outer rows |
| 2 | `WHERE` | VT1 → VT2 | Keeps rows where the predicate is TRUE |
| 3 | `GROUP BY` | VT2 → VT3 | Arranges rows into groups, one per distinct grouping value |
| 4 | `HAVING` | VT3 → VT4 | Keeps *groups* where the predicate is TRUE |
| 5 | `SELECT` | VT4 → VT5 | Evaluates expressions, assigns aliases; then `DISTINCT` |
| 6 | `ORDER BY` | VT5 → cursor | Sorts for presentation |
| 7 | `TOP` / `OFFSET-FETCH` | → result | Limits rows, based on the `ORDER BY` |

Consequences you can derive from this table:
1. **Aliases defined in SELECT can't be used in WHERE, GROUP BY or HAVING.** Those phases run earlier. They *can* be used in `ORDER BY`, which runs later.
2. **Aggregates can't appear in WHERE.** Groups don't exist yet. Filter groups in `HAVING`.
3. **After GROUP BY, every expression in HAVING, SELECT and ORDER BY must be group-level.** Either the expression is in the `GROUP BY` list, or it is inside an aggregate. Each group returns one row, so a non-grouped column would have many possible values.
4. **All-at-once.** `SELECT col1 + 1 AS a, a * 2 AS b` fails, because `a` doesn't exist yet in the same phase. Likewise, `WHERE col <> 0 AND 10 / col > 1` is *not* guaranteed to check `col <> 0` first; the optimizer may evaluate either side first. Use `CASE` or `NULLIF` (`10 / NULLIF(col, 0)`) to control it.
5. **ORDER BY with DISTINCT** can only use columns in the SELECT list. After DISTINCT, one output row may stand for several source rows with different values in other columns.
6. **TOP without ORDER BY** returns arbitrary rows. `TOP` is a filter defined by an order.

**NULL behavior by phase:**
- `WHERE col = NULL` is never TRUE. Use `IS NULL`.
- `GROUP BY` puts all NULLs into one group. For grouping, NULLs are treated as equal.
- `ORDER BY` sorts NULLs first in ascending order in SQL Server.
- `DISTINCT` and set operators treat NULLs as equal.
- `COUNT(col)` skips NULLs; `COUNT(*)` counts rows.

### 4. Worked example
```sql
-- Sample: Sales.Orders(orderid, custid, empid, orderdate, freight)
SELECT   empid,
         YEAR(orderdate)  AS orderyear,
         COUNT(*)         AS numorders
FROM     Sales.Orders
WHERE    custid = 71
GROUP BY empid, YEAR(orderdate)
HAVING   COUNT(*) > 1
ORDER BY empid, orderyear;
```
Walking through the phases:
1. **FROM**: all rows of `Sales.Orders`.
2. **WHERE**: only customer 71's orders remain.
3. **GROUP BY**: one group per (employee, order year). We must repeat `YEAR(orderdate)`; the alias `orderyear` doesn't exist yet.
4. **HAVING**: keep groups with more than one order.
5. **SELECT**: compute the three output columns and assign the aliases.
6. **ORDER BY**: now the alias `orderyear` is visible, so we can sort by it.

Errors that this model predicts:
```sql
SELECT orderid, YEAR(orderdate) AS orderyear
FROM Sales.Orders
WHERE orderyear > 2021;           -- Msg 207: Invalid column name 'orderyear'

SELECT empid, orderid, COUNT(*)   -- Msg 8120: orderid is not in GROUP BY
FROM Sales.Orders
GROUP BY empid;
```
To reuse an expression without repeating it, define it in an earlier phase with `CROSS APPLY (VALUES …)` or a table expression:
```sql
SELECT o.orderid, y.orderyear
FROM Sales.Orders AS o
CROSS APPLY (VALUES (YEAR(o.orderdate))) AS y(orderyear)
WHERE y.orderyear > 2021;
```
**Sargability note:** `WHERE YEAR(orderdate) = 2022` gives the right answer but forces a scan, because the index on `orderdate` can't be used for a function result. The range form `orderdate >= '20220101' AND orderdate < '20230101'` means the same thing and allows a seek.

### 5. Under the hood
- **Logical vs. physical.** In the plan for the example above, the `WHERE` filter often becomes a *Seek Predicate* on an index on `custid`. Grouping becomes a *Stream Aggregate* (if the input is sorted) or a *Hash Match (Aggregate)*. `HAVING` becomes a *Filter* operator after the aggregate. Nothing in the plan is labeled "phase 3". The optimizer has rewritten the logical tree into the cheapest equivalent.
- **Predicate pushdown.** Because `WHERE` and the inner-join `ON` are logically equivalent filters, the optimizer moves predicates between them and pushes them down to the earliest table access. For **outer joins** it can't do that freely. A `WHERE` filter on the inner side of a LEFT JOIN removes the NULL-extended rows and effectively turns it into an inner join, and the optimizer will simplify it to one.
- **Contradiction detection.** With a trusted `CHECK (freight >= 0)` constraint, `WHERE freight < 0` produces a plan with a *Constant Scan* that reads no data.
- **Short-circuiting isn't guaranteed.** Expressions may be evaluated in any order or even before filtering, which is why `CAST(col AS int)` combined with `WHERE ISNUMERIC(col) = 1` can still fail. Use `TRY_CAST` / `TRY_CONVERT`.
- **ORDER BY in table expressions** (views, derived tables, CTEs) is not allowed unless `TOP`, `OFFSET` or `FOR XML` is present. Even then it only defines which rows `TOP` picks, not the order the outer query returns.

### 6. Performance and design implications
- Filter early and sargably. Logically, `WHERE` runs before grouping, and the optimizer pushes it down to an index access when the predicate allows it.
- Put row-level filters in `WHERE`, not `HAVING`. `HAVING custid = 71` is logically correct but filters after grouping; the optimizer often moves it, but don't rely on that.
- Sorting is expensive (memory grant, possible tempdb spill). Ask for `ORDER BY` only in the outer query and only when the caller needs order. An index whose key matches the order removes the Sort operator.
- `DISTINCT` adds a sort or hash. If you need it to hide duplicates from a bad join, fix the join instead.
- Check plans: Seek vs. Scan, the *Predicate* vs. *Seek Predicate* properties, and warnings such as `CONVERT_IMPLICIT`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `IS NULL` / `IS NOT NULL` | Correct NULL tests | Never `= NULL` |
| `IS [NOT] DISTINCT FROM` (SQL 2022+) | NULL-safe equality | Comparing nullable columns |
| `NULLIF(a, b)` | NULL if equal, else `a` | Avoid divide-by-zero |
| `COALESCE` / `ISNULL` | First non-NULL value | Display defaults |
| `TRY_CAST` / `TRY_CONVERT` | NULL instead of a conversion error | Dirty input |
| `OFFSET n ROWS FETCH NEXT m ROWS ONLY` | Standard paging, needs `ORDER BY` | Paging |
| `TOP (n) WITH TIES` | Includes rows tied with the last one | Fair "top N" |
| `CROSS APPLY (VALUES …)` | Defines reusable column aliases early | Avoid repeating expressions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `WHERE` for row filters | Never use `HAVING` for conditions that don't need aggregates |
| `HAVING` for conditions on aggregates | — |
| Repeat the expression or use APPLY/CTE to reuse aliases | Don't nest views just to reuse an alias |
| `OFFSET-FETCH` for paging | Deep pages (`OFFSET 1000000`) read and discard all earlier rows → keyset paging (`WHERE key > @last`) |
| `TOP … WITH TIES` | When the caller needs exactly n rows |

### 9. Common mistakes
- Using a SELECT alias in WHERE → error 207 → repeat the expression or use APPLY / a CTE.
- `WHERE col = NULL` → zero rows → `IS NULL`.
- Relying on `AND` order to protect a conversion → intermittent conversion errors → `TRY_CAST` or `CASE`.
- `ORDER BY` inside a view → order not honored → put `ORDER BY` in the query that reads the view.
- `TOP 10` without `ORDER BY` → different rows on different runs → always order.
- Functions on filtered columns → scans → rewrite as ranges.

### 10. 🎯 Interview kit
**30-second answer:** Logically, a query is evaluated FROM, WHERE, GROUP BY, HAVING, SELECT (then DISTINCT), ORDER BY, then TOP/OFFSET. That's why SELECT aliases work in ORDER BY but not in WHERE, why aggregates go in HAVING, and why everything after GROUP BY must be group-level. Physically, the optimizer can reorder anything as long as the result is the same.

**Structure a longer answer:** list the phases → explain one consequence (aliases) → NULL and three-valued logic → all-at-once evaluation → logical vs. physical, with a plan example.

**Likely follow-ups:**
- *Why does `WHERE orderyear > 2021` fail?* → the alias is created in SELECT, after WHERE.
- *WHERE vs. HAVING?* → rows before grouping vs. groups after grouping.
- *Is evaluation short-circuited?* → not guaranteed; use CASE or TRY_ functions.
- *Why does a view with ORDER BY return unordered data?* → table expressions are sets; only the outer ORDER BY counts.

**Signals interviewers look for:** you can recite the order *and* derive consequences; you separate logical meaning from the physical plan; you handle NULLs precisely.

### 11. Check yourself
- [ ] Write the logical phase order from memory and give one consequence for each phase.
- [ ] Explain why `SELECT DISTINCT a ORDER BY b` fails.
- [ ] Rewrite `WHERE YEAR(d) = 2024` as a sargable range and compare the plans.
- [ ] Show how `GROUP BY` and `COUNT(col)` treat NULLs differently.
- [ ] Explain why `WHERE x <> 0 AND 1/x > 0` can still divide by zero.
<!-- explainer:end -->

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › WHERE Clause Predicates and Logical Operators
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Practical Examples of WHERE Logic
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › READ (SELECT)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › NULL in ORDER BY and GROUP BY
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Solutions to Exercise 7-2: The GROUP BY Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › The ORDER BY Clause

