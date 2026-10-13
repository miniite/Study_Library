---
id: "01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Aggregation"
tools: []
---

# GROUP BY and HAVING

**L4 Advanced** · [↑ Aggregation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `GROUP BY` collapses rows into one row per distinct key combination, `WHERE` filters rows *before* that and `HAVING` filters groups *after* it; every column you select must be a grouping key or sit inside an aggregate.

### 1. The big picture
Plain aggregates (`SUM`, `COUNT`, `AVG`) summarise the whole result set into a single row. `GROUP BY` adds *grouping levels*: the same aggregate is computed once per group instead. It is how you answer "per customer", "per month", "per department and status". `HAVING` then lets you keep or drop whole groups based on their aggregate value ("customers who spent over 10,000").

Most trouble comes from two things: (a) not knowing the logical order in which the clauses run, which explains why `WHERE` cannot see aggregates and why select-list aliases cannot be used in `HAVING`, and (b) the single-value-per-group rule, which engines enforce differently.

### 2. Core concepts and vocabulary
- **Group** — all rows that share the same values for every `GROUP BY` expression.
- **Grouping set** — the list of expressions you group by; `GROUP BY a, b` is one grouping set `(a, b)`.
- **Aggregate function** — collapses the rows of a group to one value (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`, `STRING_AGG`…); NULLs are ignored except by `COUNT(*)`.
- **`WHERE`** — row-level filter, evaluated before grouping; cannot contain aggregates.
- **`HAVING`** — group-level filter, evaluated after grouping; may contain aggregates and grouping columns.
- **Functional dependency** — column B is determined by column A (e.g. `name` by primary key `id`); PostgreSQL and MySQL 5.7+ allow selecting B when A is the group key, SQL Server does not.
- **`GREATEST` / `LEAST`** — row-level max/min across columns (SQL Server 2022+); the *horizontal* counterpart of `MAX`/`MIN`.
- **`GROUPING SETS`, `ROLLUP`, `CUBE`** — compute several grouping levels (subtotals, grand total) in one statement.

### 3. How it works, step by step
Logical processing order (this is *logical*; the optimizer may physically reorder):

1. `FROM` / `JOIN` — build the input rows.
2. `WHERE` — drop rows not matching the row predicate.
3. `GROUP BY` — partition the surviving rows into groups.
4. Aggregates are computed for each group.
5. `HAVING` — drop groups whose aggregate/key predicate is false.
6. `SELECT` — compute the select list and aliases.
7. `DISTINCT`, `ORDER BY` (the only clause that may use select aliases), `TOP`/`OFFSET`.

Consequences:
- An aggregate in `WHERE` is an error, because groups do not exist yet.
- An alias defined in `SELECT` is not visible in `HAVING` in SQL Server and PostgreSQL (MySQL allows it). Repeat the expression.
- After step 3 each group yields **one** row, so every non-aggregated select-list expression must be one value per group — i.e. in `GROUP BY`. SQL Server error 8120: *"Column … is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause."*
- Non-deterministic order: grouping may be done by hashing or by sorting, so the output order is unspecified. Add `ORDER BY` — it costs nothing for the aggregate itself.

### 4. Worked example
```sql
CREATE TABLE orders(order_id INT, empid INT, custid INT, orderdate DATE, freight DECIMAL(10,2), status VARCHAR(10));

-- one row per employee and year, with sum and count
SELECT empid, YEAR(orderdate) AS orderyear,
       SUM(freight) AS totalfreight, COUNT(*) AS numorders
FROM orders
WHERE custid = 71                 -- row filter first
GROUP BY empid, YEAR(orderdate);  -- expression in GROUP BY must match the select expression

-- customers with >= 10 orders AND spending >= 10,000
SELECT custid, COUNT(*) AS n, SUM(freight) AS spent
FROM orders
WHERE status <> 'CANCELLED'       -- cheap row filter BEFORE grouping
GROUP BY custid
HAVING COUNT(*) >= 10 AND SUM(freight) >= 10000   -- two aggregate conditions are fine
ORDER BY spent DESC;              -- ORDER BY may use the alias and sort by an aggregate

-- fails: freight neither grouped nor aggregated
SELECT empid, YEAR(orderdate), freight FROM orders GROUP BY empid, YEAR(orderdate);  -- Msg 8120
```
Walk-through of query 2: rows with status `CANCELLED` are removed; remaining rows are bucketed by `custid`; `COUNT(*)` and `SUM(freight)` are computed per bucket; buckets failing either condition are dropped; results sorted. Putting `status <> 'CANCELLED'` in `HAVING` would be an error here (`status` is neither grouped nor aggregated); even where such a predicate is legal because the column is a grouping key, write row predicates in `WHERE`, where the engine can use an index and shrink the input.

### 5. Under the hood   *(optional at L4)*
- **Two physical strategies**: *Stream Aggregate* (input sorted on the group key, e.g. via an index; low memory, output sorted) and *Hash Match (Aggregate)* (builds a hash table keyed by group; unsorted output; may spill to tempdb when the memory grant is too small).
- **Predicate pushdown**: the book notes the engine may move non-aggregate `HAVING` criteria down into `WHERE`; still, write them in `WHERE` yourself.
- **Functional dependency**: grouping by a primary key lets PostgreSQL/MySQL accept `SELECT id, name … GROUP BY id` because `name` is determined by `id`. SQLite and old MySQL accept *any* extra column and return the value from an arbitrary row of the group — a silent wrong answer (the books' "Atomic Nose Hair Trimmer shows 1 instead of 3" example). MySQL's `ONLY_FULL_GROUP_BY` mode turns this into an error.
- **Expression grouping**: `GROUP BY LOWER(email)` must evaluate the function per row. A computed column with an index, or a functional/expression index (PostgreSQL), turns this into an index scan with a stream aggregate.
- **`GROUPING SETS`** computes all requested levels from one pass over the data (SQL Server uses a *Table Spool* or repeated hash aggregates internally), versus N scans for `UNION ALL` of N queries.

### 6. Performance and design implications
- Filter in `WHERE` first; the fewer rows reach the aggregate, the less memory/CPU it needs.
- An index whose leading columns are the `GROUP BY` columns (plus included aggregated columns) enables a stream aggregate with no sort.
- High-cardinality grouping (e.g. per order) produces almost as many groups as rows: the hash table is huge and spills. Low cardinality is cheap — but low cardinality is not a guarantee of speed if the scan is huge.
- Measure: actual execution plan (look for *Sort*, *Hash Match* with a warning icon), `SET STATISTICS IO, TIME ON`.
- Use `ROLLUP`/`GROUPING SETS` for "per group + grand total" instead of a `UNION ALL`: one scan, same result.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GROUP BY a, b` | One group per distinct `(a, b)` | Any per-key aggregate |
| `HAVING agg(...) > n` | Filter groups | Conditions on aggregates |
| `COUNT(DISTINCT x)` | Counts unique non-NULL values | De-duplicated counts |
| `ROLLUP(a,b)` | Subtotals `(a,b)`, `(a)`, `()` | Hierarchical totals |
| `CUBE(a,b)` | All combinations of the columns | Cross-tab totals |
| `GROUPING SETS((a),(b),())` | Exactly the sets you list | Custom subtotals |
| `GROUPING(col)` | 1 if the NULL is a subtotal marker | Tell real NULLs from rollup NULLs |
| `GREATEST` / `LEAST` | Per-row max/min across columns (2022+) | Not an aggregate; no `GROUP BY` needed |
| `ONLY_FULL_GROUP_BY` (MySQL) | Rejects ambiguous select lists | Keep ON; fix the query |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `WHERE` for non-aggregate predicates | Writing them in `HAVING` — works but wastes work and hides intent |
| `GROUP BY` all non-aggregated columns | Relying on lenient engines that pick an arbitrary row |
| `ROLLUP`/`GROUPING SETS` | Cross-engine portability matters; support differs (`WITH ROLLUP` in MySQL) |
| Window aggregate (`SUM() OVER`) | You really want fewer rows — windows keep every row |
| `PIVOT` / conditional aggregates | The column list is unknown at write time (needs dynamic SQL) |

### 9. Common mistakes
- Aggregate in `WHERE` → error → move to `HAVING` or pre-filter in a subquery.
- Using a select alias in `HAVING` → "Invalid column name" → repeat the expression or wrap in a CTE.
- Selecting an extra column "just to show it" → forces another group key and splits groups, or errors → aggregate it (`MAX(name)`) or group by the key and join to the dimension afterwards.
- Assuming `GROUP BY` returns sorted output → random order after a plan change → add `ORDER BY`.
- `HAVING COUNT(*) > 1` to find duplicates but counting after a join → inflated counts → group at the correct grain.
- `NULL` grouping: all NULLs form *one* group; a rollup NULL looks identical → use `GROUPING()`.

### 10. 🎯 Interview kit
**30-second answer:** "`GROUP BY` partitions rows into groups and computes an aggregate per group. `WHERE` filters rows before grouping, `HAVING` filters groups after, so only `HAVING` can use aggregates. Everything in `SELECT` must be a group key or inside an aggregate."

**Structure a longer answer:** logical processing order → single-value-per-group rule → `WHERE` vs `HAVING` → physical operators (stream vs hash) and indexing → subtotals with `ROLLUP`/`GROUPING SETS` → pitfalls.

**Likely follow-ups:**
- *Is GROUP BY output ordered?* → no, add `ORDER BY`; hash aggregation has no order.
- *Can you `ORDER BY` an aggregate?* → yes, `ORDER BY` runs last and may reference aggregates and aliases.
- *Two aggregate conditions in `HAVING`?* → yes, combine with `AND`/`OR`.
- *Alias in `HAVING`?* → not in SQL Server/PostgreSQL; `HAVING` is evaluated before `SELECT`.
- *Why do some engines let you select non-grouped columns?* → functional dependency on the grouping key (PK), or a lenient/unsafe mode.
- *`GROUPING SETS` vs `UNION ALL`?* → one scan vs N scans; same rows; roughly N× cheaper on I/O.

**Signals interviewers look for:** you recite the logical order without prompting, know alias visibility, and mention the 8120/ONLY_FULL_GROUP_BY behaviour and ordering non-guarantee.

### 11. Check yourself
- [ ] Write the logical order of clauses and say where aliases become visible.
- [ ] Explain why `HAVING salary > 100` on a non-grouped column is wrong or pointless.
- [ ] Fix a query that selects `name` while grouping by `manufacturer`.
- [ ] Produce per-category counts and a grand total in one statement with `ROLLUP`, labelling the total row with `GROUPING()`.
- [ ] Find which index would let `GROUP BY custid, YEAR(orderdate)` run as a stream aggregate.
<!-- explainer:end -->

## Practice questions

- [Basic] [Which SQL keyword do you use to remove duplicate rows before an aggregate counts or sums them, and where does it go in the function call?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00826_Which_SQL_keyword_do_you_use_to_remove_duplicate_rows_before.md>)
- [Intermediate] [Can you use ORDER BY with a GROUP BY query to sort by an aggregated value?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00827_Can_you_use_ORDER_BY_with_a_GROUP_BY_query_to_sort_by_an_agg.md>)
- [Intermediate] [Can you filter on two different aggregate conditions in the same HAVING clause? Give an example.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00874_Can_you_filter_on_two_different_aggregate_conditions_in_the_.md>)
- [Intermediate] [A user complains a grouped report is non-deterministic in row order. Why does GROUP BY not guarantee ordering, and how do you make output stable without hurting the aggregate?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00756_A_user_complains_a_grouped_report_is_non-deterministic_in_ro.md>)
- [Deep] [What is the logical order of execution of FROM, WHERE, GROUP BY, HAVING, SELECT, and ORDER BY?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00829_What_is_the_logical_order_of_execution_of_FROM_WHERE_GROUP_B.md>)
- [Deep] [What happens if you use HAVING without a GROUP BY clause?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00884_What_happens_if_you_use_HAVING_without_a_GROUP_BY_clause.md>)
- [Deep] [How does the GROUPING() function let you distinguish a real NULL in the data from a subtotal NULL generated by ROLLUP?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00728_How_does_the_GROUPING_function_let_you_distinguish_a_real_NU.md>)
- [Deep] [Explain how the ordering clause inside an aggregate — ARRAY_AGG(x ORDER BY y) or STRING_AGG(x, ',' ORDER BY y) — works and why forgetting it produces unstable output.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00762_Explain_how_the_ordering_clause_inside_an_aggregate_ARRAY_AG.md>)

## Related concepts

- [GROUP BY vs PARTITION BY](<../SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>) *(SQL Fundamentals)*
- [WHERE vs HAVING](<../../Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [UDF Performance](<../SQL_Functions/UDF_Performance.md>) *(SQL Functions)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › The GROUP BY Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Grouping on Expressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › The ORDER BY Clause
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › The WHERE Clause
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Adding Window Aggregates to Aggregate Queries
- *SQL Server Tutorial* (YouTube series) — Part 11: Group by in sql server ([watch](https://www.youtube.com/watch?v=FKSSOpQe5Jc))
- *SQL Server Tutorial* (YouTube series) — Part 105: Grouping function in SQL Server ([watch](https://www.youtube.com/watch?v=h25K5S998fA))
- *SQL Server Tutorial* (YouTube series) — Part 106: GROUPING ID function in SQL Server ([watch](https://www.youtube.com/watch?v=8NIXZL-EROg))

