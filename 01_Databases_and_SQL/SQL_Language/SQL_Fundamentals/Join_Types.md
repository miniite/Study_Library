---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL"]
---

# Join Types

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Joins combine tables side by side (INNER, LEFT/RIGHT/FULL OUTER, CROSS, plus semi/anti via `EXISTS`), set operators stack results on top of each other (`UNION [ALL]`, `INTERSECT`, `EXCEPT`) — pick by whether you need unmatched rows, extra columns, or only a membership test.

### 1. The big picture
Relational data is split across tables; queries put it back together. Two families exist:
- **Joins** — horizontal: add *columns* from another table by matching rows on a condition.
- **Set operators** — vertical: add *rows* from another query with the same shape.

Choosing the wrong one gives silently wrong answers — dropped rows (INNER where LEFT was needed), duplicated rows (JOIN where EXISTS was right), or wasted sorts (`UNION` where `UNION ALL` was fine). This lesson is the decision guide; fan-out, NULL behavior and type‑2 dimensions build on it.

### 2. Core concepts and vocabulary
- **INNER JOIN** — only rows with a match on both sides.
- **LEFT / RIGHT OUTER JOIN** — all rows from one side, plus matches; unmatched columns become `NULL`.
- **FULL OUTER JOIN** — all rows from both sides, matched where possible.
- **CROSS JOIN** — every row × every row (Cartesian product); no condition.
- **Self join** — a table joined to itself under different aliases (employee → manager).
- **Semi-join** — "rows in A that have a match in B" (`EXISTS`/`IN`), returns A's columns only, never duplicates.
- **Anti-join** — "rows in A with *no* match in B" (`NOT EXISTS`, or `LEFT JOIN … IS NULL`).
- **`UNION ALL`** — concatenate result sets, keep duplicates (multiset).
- **`UNION`** — concatenate and remove duplicates (set).
- **`INTERSECT` / `EXCEPT`** — rows in both / rows in the first but not the second.

### 3. How it works, step by step
Logical processing of a join query: `FROM`/`JOIN` (build the combined row set) → `ON` filters matches → outer rows are added back → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`.

Key consequence for outer joins: a predicate in `ON` decides *what matches*; a predicate in `WHERE` filters *after* unmatched rows were added back.

| Need | Use |
|---|---|
| Columns from both, only matches | `INNER JOIN` |
| Keep every row of the main table even without a match | `LEFT JOIN` (put the main table on the left) |
| Rows in A not in B | `NOT EXISTS` (or `LEFT JOIN … WHERE b.key IS NULL`) |
| Just test existence, avoid duplicates | `EXISTS` |
| Combine two similarly-shaped tables | `UNION ALL` |
| Combine and deduplicate | `UNION` |
| Compare two sets | `INTERSECT` / `EXCEPT` |

Set operator rules: same number of columns, compatible types by position, column names come from the first query, `ORDER BY` only at the very end. A column present on one side only is filled with `NULL` (or a literal) on the other: `SELECT id, name, NULL AS region FROM a UNION ALL SELECT id, name, region FROM b`.

### 4. Worked example
```sql
-- Customers with their order count, including customers with no orders
SELECT c.customer_id, c.name, COUNT(o.order_id) AS orders
FROM   dbo.Customers c
LEFT   JOIN dbo.Orders o
       ON o.customer_id = c.customer_id
      AND o.order_date >= '2026-01-01'      -- filter in ON keeps the outer rows
GROUP  BY c.customer_id, c.name;

-- Same filter in WHERE turns it back into an inner join:
--   WHERE o.order_date >= '2026-01-01'   (drops customers whose o.* is NULL)

-- Products never ordered
SELECT p.product_id
FROM   dbo.Products p
WHERE  NOT EXISTS (SELECT 1 FROM dbo.OrderLines l WHERE l.product_id = p.product_id);

-- Stack current and archived orders
SELECT order_id, customer_id, order_date FROM dbo.Orders
UNION ALL
SELECT order_id, customer_id, order_date FROM dbo.OrdersArchive;
```
Reading it: `COUNT(o.order_id)` counts only non-NULL values, so customers with no orders correctly get 0 (`COUNT(*)` would give 1). The `NOT EXISTS` form returns each product once and handles NULLs safely; `NOT IN` would return nothing if the subquery produced even one `NULL`.

### 5. Under the hood
- The optimizer treats join order as logical, not physical: it picks among **nested loops** (small outer, indexed inner), **merge join** (both inputs sorted on the key), and **hash join** (large unsorted inputs, builds a hash table on the smaller one). Outer joins constrain reordering, which is why moving a `WHERE` predicate across an outer join changes semantics *and* plans.
- `EXISTS`, `IN (subquery)` and an inner join may compile to the same **semi-join** plan when the inner key is unique; differences appear when the inner side has duplicates (join would multiply rows, semi-join won't) or contains NULLs (`NOT IN`).
- `UNION` adds a duplicate-removing operator (sort/distinct or hash aggregate) over the entire concatenated output; `UNION ALL` is a simple concatenation. If duplicates can't occur, `UNION ALL` returns the same result at a fraction of the cost.
- `INTERSECT`/`EXCEPT` treat `NULL`s as equal for comparison (unlike `=`), which is why they work for "find rows that differ" checks.
- `RIGHT JOIN` is just a mirror of `LEFT JOIN`; most teams standardize on LEFT for readability.

### 6. Performance and design implications
- Index the join keys on the inner/probe side; foreign keys are not indexed automatically in most engines.
- `EXISTS` can stop at the first match; a join must produce every match. Prefer it for existence tests on large child tables.
- A `LEFT JOIN` that becomes inner because of a `WHERE` predicate on the right table is both a bug and a hint that the plan wasn't what you expected.
- Don't join on expressions (`ON YEAR(a.d) = b.y`): it blocks index seeks and degrades estimates.
- Check actual plans: row counts per operator and warnings (spills, missing join predicate).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INNER/LEFT/FULL/CROSS JOIN` | Row matching | Everyday modeling |
| `EXISTS` / `NOT EXISTS` | Semi/anti-join | Presence / absence tests |
| `UNION ALL` | Concatenate | Partitions, archives, multi-source |
| `UNION` / `INTERSECT` / `EXCEPT` | Set logic with dedupe | Reconciliation, diff |
| `OPTION (HASH JOIN)` / `/*+ */` hints | Force algorithm | Last resort for diagnosis |
| `SET STATISTICS IO ON` | Compare logical reads | Pick between JOIN/EXISTS/IN |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `INNER JOIN` | Missing matches must still appear |
| `LEFT JOIN` | Right-table filters placed in `WHERE` |
| `EXISTS` | You need child columns |
| `IN (list)` | Large or NULL-containing subqueries with `NOT IN` |
| `UNION ALL` | You truly need dedupe |
| `FULL OUTER JOIN` | Large tables on non-indexed keys |

### 9. Common mistakes
- Filtering the right table of a LEFT JOIN in `WHERE` → unmatched rows vanish → move to `ON`.
- `NOT IN` with a nullable column → empty result → `NOT EXISTS`.
- `UNION` by habit → needless sort, possibly dropping legitimately duplicate rows → `UNION ALL`.
- Mismatched column order in a set operator → values land in the wrong columns → name columns explicitly.
- Joining on non-unique keys → fan-out → see the Join Fan-out lesson.

### 10. 🎯 Interview kit
**30-second answer:** INNER keeps matches only; LEFT keeps all left rows with NULLs for no match; FULL keeps both; CROSS multiplies. `UNION ALL` stacks rows, `UNION` also dedupes. For "does a related row exist" use `EXISTS`; for "rows with no match" use `NOT EXISTS`.
**Structure a longer answer:** clarify what must be preserved → pick the join/operator → note the NULL and duplicate behaviors → mention performance (index, semi-join, UNION ALL).
**Likely follow-ups:**
- *LEFT JOIN vs INNER in a dashboard?* → LEFT avoids dropping entities lacking related data; count the right key, not `*`.
- *UNION vs UNION ALL?* → dedupe sort cost vs preserving duplicates.
- *How can IN, JOIN and EXISTS give different plans?* → inner-set size and uniqueness drive semi-join vs join; estimates choose algorithm.
- *Find rows with no match?* → `NOT EXISTS` or `LEFT JOIN … IS NULL`.
**Signals interviewers look for:** ON vs WHERE awareness, NULL traps, anti-join patterns, cost reasoning.

### 11. Check yourself
- [ ] Draw which rows each join type returns for two small tables.
- [ ] Explain why a `WHERE` predicate on the right table breaks a LEFT JOIN.
- [ ] Write an anti-join three ways and name the NULL trap in one.
- [ ] State when `UNION` and `UNION ALL` give identical results and which is cheaper.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What is the difference between a WHERE clause using AND versus OR when combining conditions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39768_What_is_the_difference_between_a_WHERE_clause_using_AND_vers.md>)
- [Basic] [What are the different types of JOINs in SQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40708_What_are_the_different_types_of_JOINs_in_SQL.md>)
- [Basic] [What is the difference between UNION and UNION ALL when combining data from two source tables into one training set?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40577_What_is_the_difference_between_UNION_and_UNION_ALL_when_comb.md>)
- [Basic] [Why might a LEFT JOIN be safer than an INNER JOIN when joining an alerts table to a hosts table for a dashboard?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40963_Why_might_a_LEFT_JOIN_be_safer_than_an_INNER_JOIN_when_joini.md>)
- [Intermediate] [How do you retrieve rows from one table that have no matching row in another, using a LEFT JOIN?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39832_How_do_you_retrieve_rows_from_one_table_that_have_no_matchin.md>)
- [Intermediate] [Why must columns in a UNION match in count and type, and how do you handle a column that only one side has?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40222_Why_must_columns_in_a_UNION_match_in_count_and_type_and_how_.md>)
- [Intermediate] [What is the difference between a subquery in the WHERE clause and a JOIN when selecting a positive-label cohort, and which is clearer?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40590_What_is_the_difference_between_a_subquery_in_the_WHERE_claus.md>)
- [Deep] [Explain how a large IN-subquery, a JOIN, and an EXISTS can produce three different plans for 'orders whose customer is premium', and how inner-set row estimates decide which wins.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39971_Explain_how_a_large_IN-subquery_a_JOIN_and_an_EXISTS_can_pro.md>)

## Related concepts

- [WHERE vs HAVING](<../../Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [Joins vs Subqueries and Join Types](<../Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>) *(Joins and Subqueries)*
- [CROSS APPLY and OUTER APPLY](<../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>) *(T-SQL)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › EXISTS versus IN versus JOIN -- Comparison
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Set Operators: UNION, INTERSECT, EXCEPT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › UNION Operator
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Example: Employee Union, Intersect, Except
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Set operators
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Writing UNION Queries
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using EXCEPT and INTERSECT
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.6 #19 Using UNION unnecessarily
- *SQL Server Tutorial* (YouTube series) — Part 17: Union and union all in sql server ([watch](https://www.youtube.com/watch?v=9w5uRCFOiTo))
- *SQL Server Tutorial* (YouTube series) — Part 90: Difference between union intersect and except in sql server ([watch](https://www.youtube.com/watch?v=0Yu9-OZEv7A))

