---
id: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries"
tools: []
---

# Joins vs Subqueries and Join Types

**L4 Advanced** · [↑ Joins and Subqueries](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Joins combine tables side by side (inner, left, right, full, cross, self), set operators stack results (`UNION`, `INTERSECT`, `EXCEPT`), and subqueries nest one query inside another — choose by what you need to *return* (columns from both vs only a filter) and how NULLs and duplicates should behave.

### 1. The big picture
Relational databases split information across tables to avoid redundancy: customer details live once in `Customers` and orders reference them by key. To get the full picture you need `JOIN`s. Alternatively you can express the same intent with subqueries. In many cases the optimizer produces the same plan for equivalent forms; the real differences are (a) which columns you can return, (b) duplicate behaviour, and (c) NULL handling. This lesson gives the full vocabulary of join types, set operations and subquery kinds, and the rules of thumb for choosing.

### 2. Core concepts and vocabulary
- **`INNER JOIN`** — only rows with matching keys in both tables.
- **`LEFT [OUTER] JOIN`** — all left rows; unmatched right columns are NULL.
- **`RIGHT [OUTER] JOIN`** — mirror image; rarely needed (swap table order and use `LEFT`).
- **`FULL [OUTER] JOIN`** — unmatched rows from both sides, NULL-padded.
- **`CROSS JOIN`** — Cartesian product (n × m rows); no predicate.
- **Self join** — a table joined to itself via two aliases; there is no keyword.
- **`UNION` / `UNION ALL`** — append result sets; `UNION` removes duplicates (sort/hash cost), `UNION ALL` keeps them.
- **`INTERSECT`** — rows in both; **`EXCEPT`** — rows in the first but not the second. Both de-duplicate and treat NULLs as equal.
- **Non-correlated subquery** — independent; evaluated once; good for global aggregates.
- **Correlated subquery** — references the outer row; conceptually once per outer row.
- **Scalar / row / table subquery** — returns one value / one row / a set; allowed in `SELECT`, `FROM`, `WHERE`.
- **Table alias** — short name for a table in a query; mandatory for self-joins and derived tables, recommended always for readability.

### 3. How it works, step by step
1. **Join semantics (logical)**: form the Cartesian product, keep pairs satisfying `ON`; for outer joins add back the unmatched rows of the preserved side with NULLs.
2. **Fan-out**: if the right side has several matches, the left row repeats.
3. **Subquery semantics**: a subquery in `WHERE` produces a value or set used for comparison (`=`, `IN`, `EXISTS`, `> ALL`); in `FROM` it is a derived table; in `SELECT` it must return exactly one value per outer row.
4. **Join vs subquery decision**:
   - Need columns from both tables → join.
   - Need only to filter by existence → `EXISTS` / `IN` (no duplicates).
   - Need a global aggregate as a filter (`salary > (SELECT AVG(salary) …)`) → non-correlated subquery.
   - Need per-row comparison (latest order per customer) → correlated subquery, `APPLY` or window function.
5. **Set operations** require the same number of columns and compatible types; column names come from the first query; `ORDER BY` goes once at the end.

### 4. Worked example
```sql
-- Inner vs left: all orders, customer shown if present
SELECT o.order_id, c.name
FROM orders o LEFT JOIN customers c ON c.cust_id = o.cust_id;

-- Self join: employee and manager (preserve the CEO with LEFT)
SELECT e.EmployeeName AS Employee, m.EmployeeName AS Manager
FROM Employees e LEFT JOIN Employees m ON e.ManagerID = m.EmployeeID;
-- Alice NULL / Bob Alice / Carol Alice / Dave Bob

-- Set difference: employees present in 2024 but not in 2025
SELECT EmployeeID FROM Employees_2024
EXCEPT
SELECT EmployeeID FROM Employees_2025;

-- Non-correlated subquery: above-average salaries
SELECT name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated subquery: above the average of own department
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (SELECT AVG(x.salary) FROM employees x WHERE x.dept_id = e.dept_id);

-- Same intent as a join to a pre-aggregated derived table (often faster, one pass)
SELECT e.name, e.salary
FROM employees e
JOIN (SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id) d
  ON d.dept_id = e.dept_id
WHERE e.salary > d.avg_sal;
```
Compare outputs: the last two return the same rows. The correlated form is logically per-row; the optimizer normally decorrelates it into the aggregate-then-join plan.

### 5. Under the hood   *(optional at L4)*
- The optimizer is free to transform subqueries into joins ("subquery unnesting"), and `IN`/`EXISTS` into semi-joins; the textual form is mostly a readability choice. Exceptions: `NOT IN` (NULL semantics prevent the anti-join rewrite unless the column is non-nullable), subqueries with `TOP`/`LIMIT`, and correlated scalar subqueries in the select list that can't be flattened.
- Physical join algorithms: nested loops (small × indexed), hash (large, unsorted), merge (both sorted). See *EXISTS, Semi-joins and Predicate Placement*.
- `UNION` adds a distinct step (sort or hash aggregate) over `UNION ALL`; `UNION ALL` is simply concatenation.
- `FULL OUTER JOIN` is implemented with merge or hash join variants; it is not rewritten into two left joins in general.

### 6. Performance and design implications
- Use `UNION ALL` unless de-duplication is required.
- Index join keys (especially foreign keys); keep join columns of identical type to avoid implicit conversions that kill seeks.
- A cross join with no limiting predicate on large tables is almost always a bug.
- Measure alternatives with `SET STATISTICS IO, TIME ON`; the book's advice is to test `IN`, `EXISTS` and `JOIN` forms rather than assume.
- Aliases do not change performance; they only disambiguate and shorten.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INNER JOIN … ON` | Matching rows | Normal relationships |
| `LEFT JOIN … ON` | Preserve left | Optional relationships |
| `FULL OUTER JOIN` | Reconcile two sets | Diffing two tables |
| `CROSS JOIN` | All pairs | Calendars/grids |
| `UNION ALL` | Concatenate | Partitioned/archived tables |
| `EXCEPT` / `INTERSECT` | Set difference/overlap (NULLs equal) | Comparing sets |
| `EXISTS`, `IN`, `ANY/ALL` | Subquery predicates | Filtering |
| `CROSS/OUTER APPLY` | Correlated table expression per row (SQL Server) | Top-N per group |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Join for columns from both | Only a filter is needed and fan-out is possible |
| `EXISTS`/`IN` for filters | `NOT IN` on nullable columns |
| Subquery for readability | Correlated scalar subquery in `SELECT` over millions of rows (hidden nested loops) |
| `UNION ALL` | Duplicates must be removed (then `UNION`) |
| `EXCEPT` | Columns include NULL semantic expectations differ from `=`; it also de-duplicates |

### 9. Common mistakes
- Forgetting the `ON` clause (or having an always-true one) → Cartesian product → verify row counts.
- Using `WHERE` on the optional table in an outer join → inner-join behaviour → put the predicate in `ON`.
- `NOT IN` with NULLs → empty result → `NOT EXISTS`.
- Mixing `UNION` and `UNION ALL` without understanding duplicates → missing or extra rows.
- Assuming a subquery in `SELECT` may return several rows → error "subquery returned more than 1 value" → aggregate or `TOP 1`/`MAX`.
- Assuming `RIGHT JOIN` differs semantically from a swapped `LEFT JOIN`.

### 10. 🎯 Interview kit
**30-second answer:** "A join combines columns from several tables by a key; inner keeps matches, left/right/full preserve one or both sides with NULLs, cross gives all pairs. `UNION` stacks results with or without de-dup; subqueries nest queries — correlated ones depend on the outer row. I use joins when I need columns from both, `EXISTS` when I only filter, and I check NULL semantics."

**Structure a longer answer:** list join types with one-liner each → set ops → subquery kinds → how to choose → optimizer rewrites and NULL traps → example.

**Likely follow-ups:**
- *Join vs `UNION`?* → join widens rows by key; `UNION` lengthens the result by appending compatible row sets.
- *Why aliases?* → readability, required for self-joins and derived tables, disambiguation of column names.
- *Join vs subquery performance?* → usually the same after optimization; measure.
- *`UNION` vs `UNION ALL` cost?* → de-dup requires sort/hash.

**Signals interviewers look for:** accurate semantics of every join type, awareness of NULLs and duplicates, evidence-based performance claims.

### 11. Check yourself
- [ ] Draw the result sets of inner, left, right, full and cross join for two 3-row tables.
- [ ] Write the "employees in 2024 not in 2025" query three ways.
- [ ] Explain why `EXCEPT` and `NOT IN` can return different results with NULLs.
- [ ] Rewrite a correlated subquery as a join to an aggregate.
- [ ] Say when you'd pick `CROSS APPLY` over a join.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between an implicit join written with a comma in the FROM clause and an explicit JOIN with an ON clause?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30393_What_is_the_difference_between_an_implicit_join_written_with.md>)
- [Basic] [What is the difference between joining tables and using UNION to combine results?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30422_What_is_the_difference_between_joining_tables_and_using_UNIO.md>)
- [Basic] [How do you handle a case where you need data from two tables but only some rows match — which join do you pick?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30475_How_do_you_handle_a_case_where_you_need_data_from_two_tables.md>)
- [Intermediate] [How do you join a table to a subquery? Show a small example.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30426_How_do_you_join_a_table_to_a_subquery_Show_a_small_example.md>)
- [Intermediate] [How do you diagnose a join returning duplicate keys because a supposed unique column isn't actually unique in prod data?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30355_How_do_you_diagnose_a_join_returning_duplicate_keys_because_.md>)
- [Deep] [What does the ANY (or SOME) operator do with a subquery? Give an example.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30406_What_does_the_ANY_or_SOME_operator_do_with_a_subquery_Give_a.md>)
- [Deep] [What is the difference between GROUP BY with a JOIN versus using a subquery to aggregate first then join?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30431_What_is_the_difference_between_GROUP_BY_with_a_JOIN_versus_u.md>)
- [Deep] [What is the difference between DISTINCT and GROUP BY when removing join duplicates?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30457_What_is_the_difference_between_DISTINCT_and_GROUP_BY_when_re.md>)

## Related concepts

- [PostgreSQL Query Basics](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>) *(PostgreSQL Platform)*
- [Join Types](<../SQL_Fundamentals/Join_Types.md>) *(SQL Fundamentals)*
- [CROSS APPLY and OUTER APPLY](<../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>) *(T-SQL)*
- [WHERE vs HAVING](<../../Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Explain Complex Logic
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › JOIN Filters
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › CROSS JOIN
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › SELF JOIN
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Joining Tables
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Joining Two Tables
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Joining on a Different Column Name
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Joining on More Than One Column
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Using OUTER JOIN
- *Hadoop: The Definitive Guide* — Tom White — Hive › Note
- *Hadoop: The Definitive Guide* — Tom White — Hive › Outer joins
- *Hadoop: The Definitive Guide* — Tom White — Hive › Semi joins
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Joins
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *SQL Server Tutorial* (YouTube series) — Part 13: Advanced or intelligent joins in sql server ([watch](https://www.youtube.com/watch?v=GKGtOABAO9s))
- *SQL Server Tutorial* (YouTube series) — Part 14: Self join in sql server ([watch](https://www.youtube.com/watch?v=qnYSN_7qwgg))
- *SQL Server Tutorial* (YouTube series) — Part 91: Cross apply and outer apply in sql server ([watch](https://www.youtube.com/watch?v=kVogo0AbatM))

