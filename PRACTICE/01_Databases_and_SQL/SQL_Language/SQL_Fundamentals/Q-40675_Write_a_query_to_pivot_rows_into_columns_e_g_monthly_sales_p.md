---
type: "practice"
difficulty: "Deep"
---

# Write a query to pivot rows into columns (e.g. monthly sales per product).

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Expanding Date Ranges](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Expanding_Date_Ranges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Write SQL that pivots — turns row values into columns. The canonical example: a sales table with one row per (product, month, amount) and I need one row per product with a column per month (Jan, Feb, Mar…) holding that month's total. The interviewer at a deep-understanding level is probing whether I know (a) the portable conditional-aggregation idiom that works on any engine, (b) engine-specific pivot operators (PIVOT in SQL Server / Oracle, crosstab/FILTER in Postgres), (c) the trade-off that the columns must be known in advance (static SQL), which forces dynamic SQL when the pivot keys are open-ended, and (d) correctness details: choosing the right aggregate, handling NULLs vs 0, and the GROUP BY grain.

Clarifying questions I'd ask: How many distinct pivot values (12 months → fixed, fine; arbitrary products → dynamic)? Which database engine? Do empty cells want NULL or 0? Do I need a total column?

Plan: Start from the source shape → give the portable CASE/SUM solution (the answer that always works) → show the engine-native PIVOT and Postgres FILTER variants → explain the "columns must be static" limitation and the dynamic-SQL escape hatch → cover correctness pitfalls → summarize.

The starting shape (long / narrow) and the target (wide)

Source sales is in long format: product, sale_month, amount, many rows per product. The target is wide format: one row per product, one column per month. Pivoting is fundamentally: group by the row key, and for each target column, aggregate only the rows belonging to that column.

Portable solution — conditional aggregation (works everywhere)

This is the idiom every senior engineer reaches for because it runs on MySQL, PostgreSQL, Oracle, SQL Server, Snowflake, and BigQuery unchanged:

SQL
SELECT
  product,
  SUM(CASE WHEN sale_month = 1 THEN amount ELSE 0 END) AS jan,
  SUM(CASE WHEN sale_month = 2 THEN amount ELSE 0 END) AS feb,
  SUM(CASE WHEN sale_month = 3 THEN amount ELSE 0 END) AS mar
FROM sales
GROUP BY product;

Why it works: GROUP BY product collapses to one row per product. Inside each group, each SUM(CASE …) sums amount only for the rows matching that month and contributes 0 (or NULL if you drop ELSE 0) otherwise. So each expression becomes exactly that product-month's total, laid out side by side as columns. If more than one row can share a (product, month), SUM correctly aggregates them; use MAX/COUNT/AVG if the semantics differ.

Engine-native pivot operators

SQL Server / Oracle — PIVOT: cleaner syntax but you must still enumerate the column values:

SQL
SELECT product, [1] AS jan, [2] AS feb, [3] AS mar
FROM (SELECT product, sale_month, amount FROM sales) src
PIVOT (SUM(amount) FOR sale_month IN ([1],[2],[3])) pvt;

PostgreSQL — FILTER clause (my preferred Postgres form, cleaner than crosstab from the tablefunc extension):

SQL
SELECT product,
  SUM(amount) FILTER (WHERE sale_month = 1) AS jan,
  SUM(amount) FILTER (WHERE sale_month = 2) AS feb,
  SUM(amount) FILTER (WHERE sale_month = 3) AS mar
FROM sales
GROUP BY product;

FILTER is standard SQL and reads better than CASE, but is Postgres/SQLite/DuckDB territory; older engines need the CASE form.

The key limitation: columns must be known at query-compile time

SQL is statically typed — the number and names of output columns must be fixed before the query runs. Twelve months is fine (hard-code 12 columns). But if the pivot values are open-ended ("a column per product", where products change), you can't write the columns by hand. Two options:

Dynamic SQL: query the distinct pivot values, build the SUM(CASE…) list as a string, and EXECUTE it. This is how frameworks/reporting tools pivot arbitrary data. It's powerful but carries SQL-injection risk — always whitelist/quote the interpolated identifiers.
Return long format and let the application (or a BI tool) pivot in the presentation layer — often the cleaner architectural choice for a truly variable column set.
Correctness pitfalls to call out
Aggregate choice: SUM for money totals, but MAX/MIN when the cell is a single expected value (a pure "reshape" with one row per cell) — using SUM there still works but signals intent poorly; COUNT for occurrence counts.
NULL vs 0 in empty cells: ELSE 0 yields 0; omitting it yields NULL. For "no sales that month," 0 usually reads better in a report; NULL is correct if you must distinguish "no data" from "zero."
The GROUP BY grain must match the row key — group by exactly the columns you want as rows, and every non-aggregated SELECT column must be in GROUP BY.
Ordering / totals: add ORDER BY product for stable output, and a SUM(amount) AS total column is a common freebie.
Summary — what a strong answer emphasizes

Lead with conditional aggregation (SUM(CASE WHEN key = v THEN val END) + GROUP BY row_key) as the portable answer, mention native PIVOT/FILTER as cleaner engine-specific sugar, and — the mark of depth — explain that pivot columns must be static, so an unbounded key set requires dynamic SQL (with injection safety) or pivoting in the app. Close on aggregate choice and NULL-vs-0.

**Comparison:**

```
Approach	Engines	Columns static?	Pros	Cons
SUM(CASE WHEN…)	All (MySQL, PG, Oracle, MSSQL, Snowflake, BQ)	Yes	Fully portable, explicit	Verbose
SUM(...) FILTER (WHERE…)	Postgres, SQLite, DuckDB	Yes	Cleanest, standard SQL	Not on MySQL/old Oracle
PIVOT operator	SQL Server, Oracle	Yes	Concise, purpose-built	Non-portable, still must list values
Dynamic SQL	All (built at runtime)	No (runtime)	Handles unknown column set	Injection risk, harder to debug
Pivot in app / BI layer	Any	No	Flexible, keeps SQL simple	Moves logic out of DB
```

**Code:**

```sql
-- 1) Portable: conditional aggregation (any SQL engine)
SELECT
  product,
  SUM(CASE WHEN sale_month = 1 THEN amount ELSE 0 END) AS jan,
  SUM(CASE WHEN sale_month = 2 THEN amount ELSE 0 END) AS feb,
  SUM(CASE WHEN sale_month = 3 THEN amount ELSE 0 END) AS mar,
  SUM(amount)                                          AS total
FROM sales
GROUP BY product
ORDER BY product;

-- 2) PostgreSQL FILTER clause (standard SQL, clean)
SELECT product,
  SUM(amount) FILTER (WHERE sale_month = 1) AS jan,
  SUM(amount) FILTER (WHERE sale_month = 2) AS feb,
  SUM(amount) FILTER (WHERE sale_month = 3) AS mar
FROM sales
GROUP BY product;

-- 3) SQL Server / Oracle PIVOT operator
SELECT product, [1] AS jan, [2] AS feb, [3] AS mar
FROM (SELECT product, sale_month, amount FROM sales) src
PIVOT (SUM(amount) FOR sale_month IN ([1],[2],[3])) AS pvt;

-- 4) Dynamic SQL sketch (SQL Server) when month/column set is unknown
DECLARE @cols NVARCHAR(MAX), @sql NVARCHAR(MAX);
SELECT @cols = STRING_AGG(QUOTENAME(sale_month), ',')
FROM (SELECT DISTINCT sale_month FROM sales) m;      -- QUOTENAME guards injection
SET @sql = N'SELECT product, ' + @cols + N'
FROM (SELECT product, sale_month, amount FROM sales) s
PIVOT (SUM(amount) FOR sale_month IN (' + @cols + N')) p;';
EXEC sp_executesql @sql;
```

**Example input / output:**

```text
Input: the sales table above.
Query: portable conditional-aggregation pivot.

Output (wide format, ELSE 0 fills gaps):
+---------+-----+-----+-----+-------+
| product | jan | feb | mar | total |
+---------+-----+-----+-----+-------+
| Gadget  | 300 |   0 | 400 |  700  |
| Widget  | 150 | 200 |   0 |  350  |   (Jan = 100 + 50 aggregated by SUM)
+---------+-----+-----+-----+-------+

Note: Widget Jan = 150 shows SUM correctly folding the two Jan rows;
Gadget Feb = 0 shows the ELSE 0 default for a missing month.
```

**Data:**

```text
-- Source: sales (long / narrow format)
+----------+------------+--------+
| product  | sale_month | amount |
+----------+------------+--------+
| Widget   | 1          | 100    |
| Widget   | 1          | 50     |   <- two Jan rows for Widget -> SUM aggregates
| Widget   | 2          | 200    |
| Gadget   | 1          | 300    |
| Gadget   | 3          | 400    |   <- Gadget has no Feb row
+----------+------------+--------+
```

**Flow:**

1. Yes, e.g. 12 months
2. No, open-ended keys
3. MySQL / portable
4. Postgres
5. SQL Server / Oracle
6. Is the set of pivot valuesknown & fixed?
7. Static pivot
8. Dynamic
9. Which engine?
10. SUM CASE WHEN key=v THENval + GROUP BY
11. SUM ... FILTER WHERE key=v
12. PIVOT ... FOR key IN (...)
13. Query DISTINCT keys -> buildcolumn list string
14. EXECUTE dynamic SQL -whitelist/quote identifiers
15. Or return long format, pivot inapp/BI

