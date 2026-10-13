---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: ["MySQL", "Oracle", "SQL_Server"]
---

# WHERE vs HAVING

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `WHERE` filters individual rows *before* grouping; `HAVING` filters whole groups *after* aggregation — and that order of evaluation is why aggregates cannot appear in `WHERE`.

### 1. The big picture
SQL is written in one order (`SELECT … FROM … WHERE … GROUP BY … HAVING … ORDER BY`) but **logically evaluated** in another. Understanding that logical order explains most "why can't I use this here?" errors. `WHERE` and `HAVING` both filter, but at different stages and on different things: rows vs groups.

Using the right one is a correctness question (aggregates are only available after grouping) and a performance question (filtering rows early shrinks the data fed to the aggregation).

### 2. Core concepts and vocabulary
- **Logical query processing order** — the conceptual sequence of clauses: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `TOP/LIMIT`.
- **Predicate** — a boolean condition (`amount > 100`).
- **Aggregate function** — `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`; collapses a group of rows to one value.
- **Group** — the set of rows sharing the same `GROUP BY` values (the whole input is one group if there is no `GROUP BY` but an aggregate is used).
- **Three-valued logic** — predicates may be TRUE, FALSE or UNKNOWN; only TRUE rows pass.
- **Alias** — a name given in `SELECT`; not visible to earlier clauses.
- **Sargable** — a predicate the engine can answer using an index seek.

### 3. How it works, step by step
For this query:
```sql
SELECT customer_id, COUNT(*) AS orders, SUM(amount) AS total
FROM   orders
WHERE  order_date >= '2026-01-01'
GROUP  BY customer_id
HAVING SUM(amount) > 1000
ORDER  BY total DESC;
```
1. **FROM** — read the `orders` table.
2. **WHERE** — keep rows with `order_date >= '2026-01-01'`. Works per row; no aggregates available because groups do not exist yet.
3. **GROUP BY** — partition remaining rows by `customer_id`.
4. **Aggregate** — compute `COUNT(*)` and `SUM(amount)` for each group.
5. **HAVING** — discard groups whose `SUM(amount)` is not above 1000. Can reference aggregates and grouping columns.
6. **SELECT** — compute output expressions and aliases.
7. **ORDER BY** — sort; this is the first clause that can use the alias `total`.

Rules of thumb:

| Question | Use |
|---|---|
| Filter on a column value of each row | `WHERE` |
| Filter on `COUNT/SUM/AVG` of a group | `HAVING` |
| Filter on a grouping column | `WHERE` (preferred — earlier) |
| Filter windowed values (`ROW_NUMBER()`) | Outer query / CTE, or `QUALIFY` where supported |

### 4. Worked example
```sql
CREATE TABLE orders (order_id INT, customer_id INT, region VARCHAR(10),
                     amount DECIMAL(10,2), order_date DATE);
INSERT INTO orders VALUES
 (1,1,'EU',500,'2026-01-05'),(2,1,'EU',700,'2026-02-10'),
 (3,2,'US',300,'2026-01-20'),(4,3,'EU',1500,'2025-12-30'),
 (5,3,'EU',200,'2026-03-01');

-- A: customers with more than 1000 in 2026
SELECT customer_id, SUM(amount) AS total
FROM   orders
WHERE  order_date >= '2026-01-01'
GROUP  BY customer_id
HAVING SUM(amount) > 1000;
```
Result: only customer 1 (total 1200). Customer 3's 1500 order is in 2025, so `WHERE` removes it *before* summing; customer 3's 2026 total is 200 and fails `HAVING`. If you moved the date test into `HAVING` you would need to aggregate it, giving a different (wrong) meaning.

```sql
-- B: an error and its fix
-- SELECT customer_id FROM orders WHERE SUM(amount) > 1000 GROUP BY customer_id;  -- invalid
SELECT customer_id FROM orders GROUP BY customer_id HAVING SUM(amount) > 1000;

-- C: filtering the group by a non-aggregate: works but is slower
SELECT region, SUM(amount) FROM orders GROUP BY region HAVING region = 'EU';
-- Better: WHERE region = 'EU' before GROUP BY
```
In C both give the same rows, but `WHERE` lets the engine discard rows (and use an index on `region`) before aggregating.

### 5. Under the hood
- **Logical vs physical order.** The logical order is a *semantic* model. The optimiser may push a `HAVING` predicate that references only grouping columns down to a `WHERE` (predicate pushdown) automatically, but you cannot rely on that for all engines.
- **Why aliases fail.** `SELECT` is evaluated after `WHERE`/`GROUP BY`/`HAVING`, so an alias defined there does not exist yet (T-SQL, standard SQL). Some engines (MySQL, PostgreSQL for `GROUP BY`/`ORDER BY`) relax this as an extension. `ORDER BY` runs after `SELECT`, so aliases work there.
- **Aggregation operators.** The plan uses a hash aggregate or stream aggregate. `HAVING` becomes a filter operator after the aggregate; `WHERE` becomes a predicate on the scan/seek — visible in the plan as "Predicate" or "Seek Predicate".
- **Aggregates ignore NULLs** (except `COUNT(*)`); `AVG` divides by the non-NULL count. A `HAVING` predicate evaluating to UNKNOWN drops the group.
- **Windows.** Window functions evaluate after `HAVING` and before `ORDER BY`; hence you cannot use them in `WHERE`/`HAVING` and must wrap in a subquery or CTE (`QUALIFY` is a shortcut in Snowflake, BigQuery, Databricks).

### 6. Performance and design implications
- Put every row-level condition in `WHERE`: fewer rows reach the aggregate, memory for hash tables drops, and an index seek becomes possible.
- Keep `HAVING` for conditions that really need aggregates.
- A `HAVING` on a grouping column that cannot be pushed down forces a full aggregate first.
- Make `WHERE` predicates sargable: `order_date >= '2026-01-01'` not `YEAR(order_date) = 2026`.
- Check the plan: the filter on the base table should appear at the scan/seek, not above the aggregate.
- Large `GROUP BY` can spill to tempdb/disk; reduce rows early or pre-aggregate.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WHERE` | Row filter before grouping | Almost always first |
| `HAVING` | Group filter after aggregation | Aggregate conditions |
| `GROUP BY … WITH ROLLUP/CUBE/GROUPING SETS` | Subtotals | Reporting |
| `FILTER (WHERE …)` (Postgres) / `SUM(CASE WHEN …)` | Conditional aggregate per column | Multiple conditional sums in one pass |
| `QUALIFY` | Filter on window functions | Where supported |
| Execution plan | Shows where each predicate is applied | Verifying pushdown |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `WHERE` for any row-level condition | Condition depends on an aggregate |
| `HAVING` for group-level conditions | Using it as a lazy substitute for `WHERE` |
| Conditional aggregation for several measures | Different row sets need different `WHERE` — use subqueries |
| Subquery/CTE to filter on window results | Engines supporting `QUALIFY` give a shorter form |

### 9. Common mistakes
- Aggregate in `WHERE` → error ("aggregate may not appear in WHERE") → move to `HAVING`.
- Non-aggregate filter in `HAVING` → works but slower, harder to read → move to `WHERE`.
- Using a `SELECT` alias in `WHERE`/`HAVING` → "invalid column" → repeat the expression or use a subquery.
- Selecting a non-grouped, non-aggregated column → error (or arbitrary value in lenient MySQL modes) → add to `GROUP BY` or aggregate it.
- `COUNT(column)` vs `COUNT(*)` confusion with NULLs → wrong counts → pick deliberately.
- Filtering a left-joined table in `WHERE` → turns outer join into inner → put the condition in `ON`.

### 10. 🎯 Interview kit
**30-second answer:** `WHERE` filters rows before grouping and cannot use aggregates; `HAVING` filters groups after aggregation and can. Logically the order is FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY — and I put every row-level condition in `WHERE` so less data is aggregated.

**Structure a longer answer:** state the logical processing order → example showing both → performance reason → pitfalls (aliases, NULLs, outer joins).

**Likely follow-ups:**
- *Can HAVING be used without GROUP BY?* → Yes; the whole result is one group.
- *Why can't I use an alias in WHERE?* → SELECT is evaluated later.
- *Filter on a window function?* → CTE/subquery or `QUALIFY`.
- *Find duplicates?* → `GROUP BY cols HAVING COUNT(*) > 1`.

**Signals interviewers look for:** you recite the logical order without prompting and relate it to performance.

### 11. Check yourself
- [ ] Write the logical processing order of a SELECT statement.
- [ ] Rewrite a query that uses `HAVING region='EU'` to use `WHERE`.
- [ ] Explain why `WHERE SUM(x) > 10` fails.
- [ ] Write a query returning customers with at least 3 orders and total over 500.
- [ ] Find duplicate emails in a table.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [Oracle](<../../../TOOLS/Oracle/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [What is the difference between WHERE and HAVING clauses in SQL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22155_What_is_the_difference_between_WHERE_and_HAVING_clauses_in_S.md>)
- [Basic] [What is the difference between IS NULL and = NULL in a WHERE clause?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22176_What_is_the_difference_between_IS_NULL_and_NULL_in_a_WHERE_c.md>)
- [Basic] [What is the difference between DISTINCT and GROUP BY when you just want unique rows?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22281_What_is_the_difference_between_DISTINCT_and_GROUP_BY_when_yo.md>)
- [Basic] [What is the difference between an aggregate function and a scalar function in SQL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22436_What_is_the_difference_between_an_aggregate_function_and_a_s.md>)
- [Intermediate] [When would you use a JOIN versus a subquery, and which is generally faster?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22178_When_would_you_use_a_JOIN_versus_a_subquery_and_which_is_gen.md>)
- [Intermediate] [What is the difference between a NATURAL JOIN and an explicit INNER JOIN with an ON clause?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22368_What_is_the_difference_between_a_NATURAL_JOIN_and_an_explici.md>)
- [Intermediate] [What does GROUP BY do with rows that have NULL in the grouping column?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22437_What_does_GROUP_BY_do_with_rows_that_have_NULL_in_the_groupi.md>)
- [Deep] [Why can NOT IN with a subquery return zero rows unexpectedly when NULLs are involved?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22280_Why_can_NOT_IN_with_a_subquery_return_zero_rows_unexpectedly.md>)

## Related concepts

- [Join Types](<../../SQL_Language/SQL_Fundamentals/Join_Types.md>) *(SQL Fundamentals)*
- [OR Conditions and Index Usage](<../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>) *(SQL Server Platform)*
- [Scalar Subqueries](<../../SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Order of Operations
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Advanced WHERE Clauses › Using SOME, ANY, and ALL
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Predicates in the WHERE Clause
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › EXCEPT Operator
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Filtering with WHERE
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *SQL Server Tutorial* (YouTube series) — Part 98: Difference between where and having in sql server ([watch](https://www.youtube.com/watch?v=15ndzPfa6-E))

