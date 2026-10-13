---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL"]
---

# NULL Handling

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `NULL` means "unknown / not applicable", not zero or empty — any comparison with it yields UNKNOWN (three-valued logic), so filters, joins, aggregates, `NOT IN` and uniqueness all behave differently around NULL and must be handled deliberately.

### 1. The big picture
SQL uses **three-valued logic**: a predicate is TRUE, FALSE or UNKNOWN. A row passes `WHERE` only when the predicate is TRUE, so rows with UNKNOWN are silently dropped. Most NULL bugs come from forgetting that UNKNOWN is neither true nor false: "not equal to 5" does *not* include rows where the value is NULL, and `NOT IN (…NULL…)` returns nothing at all.

This lesson covers how NULL behaves in comparisons, boolean logic, aggregates, joins, set operators, constraints and indexes, and the standard tools (`IS NULL`, `COALESCE`, `NULLIF`, null-safe comparison).

### 2. Core concepts and vocabulary
- **NULL** — marker for a missing/unknown value; not a value itself.
- **Three-valued logic (3VL)** — TRUE / FALSE / UNKNOWN.
- **`IS NULL` / `IS NOT NULL`** — the only correct tests for NULL.
- **`COALESCE(a, b, …)`** — first non-NULL argument (ANSI standard).
- **`ISNULL(a, b)` / `IFNULL`/`NVL`** — vendor two-argument forms.
- **`NULLIF(a, b)`** — returns NULL if `a = b`, else `a`; classic divide-by-zero guard.
- **`NOT NULL` constraint** — column must always have a value.
- **Null-safe equality** — `IS NOT DISTINCT FROM` (PG/ANSI), `<=>` (MySQL).
- **Anti-join** — rows with no match; typical place NULL bites (`NOT IN`).

### 3. How it works, step by step
**Truth tables (the essentials)**

| Expression | Result |
|---|---|
| `NULL = NULL` | UNKNOWN |
| `NULL <> 1` | UNKNOWN |
| `x = NULL` | UNKNOWN (never TRUE) |
| `TRUE AND UNKNOWN` | UNKNOWN |
| `FALSE AND UNKNOWN` | FALSE |
| `TRUE OR UNKNOWN` | TRUE |
| `NOT UNKNOWN` | UNKNOWN |

**Where NULL shows up and what happens**
1. **`WHERE`/`HAVING`/`ON`** keep only TRUE rows. `WHERE col <> 'A'` drops NULL rows.
2. **`CHECK` constraints** accept UNKNOWN (opposite of `WHERE`!): `CHECK (price > 0)` allows NULL.
3. **Aggregates** ignore NULLs, except `COUNT(*)`: `COUNT(col)` counts non-NULL; `SUM`, `AVG`, `MIN`, `MAX` skip NULL; all-NULL input yields NULL (`COUNT` yields 0). `AVG(col)` = `SUM/COUNT(col)`, not `/COUNT(*)`.
4. **`GROUP BY` / `DISTINCT` / `UNION` / `INTERSECT` / `EXCEPT`** treat NULLs as equal to each other (one NULL group).
5. **`ORDER BY`** places NULLs first or last depending on the engine (SQL Server, MySQL: first ascending; PG, Oracle: last ascending) — specify `NULLS FIRST/LAST` where supported.
6. **Joins**: `ON a.x = b.x` never matches NULL to NULL, so NULL keys don't join. Outer joins generate NULLs for unmatched side.
7. **`UNIQUE` constraints**: ANSI/PG/MySQL allow multiple NULLs; SQL Server allows only one NULL in a unique index (use a filtered unique index `WHERE col IS NOT NULL`).
8. **String concatenation**: `'a' + NULL` is NULL in T-SQL by default (use `CONCAT` or `CONCAT_WS`), and Oracle treats `''` as NULL.

### 4. Worked example
```sql
-- Data: orders(order_id, customer_id, discount, ship_date)
-- 1) Not-equal that keeps NULL rows
SELECT * FROM orders
WHERE discount <> 0 OR discount IS NULL;        -- not just <> 0

-- 2) COUNT variants
SELECT COUNT(*)            AS all_rows,
       COUNT(ship_date)    AS shipped,
       COUNT(*) - COUNT(ship_date) AS not_shipped
FROM orders;

-- 3) NULLs per column (data profiling)
SELECT SUM(CASE WHEN discount  IS NULL THEN 1 ELSE 0 END) AS discount_nulls,
       SUM(CASE WHEN ship_date IS NULL THEN 1 ELSE 0 END) AS ship_nulls
FROM orders;

-- 4) Safe division and defaults
SELECT order_id,
       total / NULLIF(qty, 0)           AS unit_price,   -- NULL instead of error
       COALESCE(discount, 0)            AS discount_eff,
       AVG(COALESCE(discount, 0)) OVER () AS avg_incl_nulls_as_zero
FROM orders;

-- 5) NOT IN trap vs NOT EXISTS
SELECT c.customer_id FROM customers c
WHERE  c.customer_id NOT IN (SELECT customer_id FROM orders);   -- returns 0 rows if any orders.customer_id IS NULL
SELECT c.customer_id FROM customers c
WHERE  NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);

-- 6) Customers with no orders via LEFT JOIN
SELECT c.customer_id
FROM customers c LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL;                       -- test a NOT NULL column of the right table
```
Why #5 fails: `x NOT IN (1, 2, NULL)` expands to `x<>1 AND x<>2 AND x<>NULL`; the last term is UNKNOWN, so the whole conjunction is never TRUE. `NOT EXISTS` only checks for matching rows, so NULLs are harmless.

### 5. Under the hood
- **Storage:** engines keep a *null bitmap* per row (SQL Server: null bitmap in the row header; PG: `t_bits`), so NULLs in nullable columns cost almost no space — and `NOT NULL` fixed-width columns avoid the bitmap check.
- **Optimizer:** a `NOT NULL` constraint lets the planner prove join eliminations, convert `NOT IN` to an anti-join safely, and skip null checks; declaring nullable columns "just in case" costs plan quality.
- **Index behavior:** B-tree indexes in SQL Server, PG and MySQL store NULLs, so `IS NULL` can seek; Oracle B-trees omit entirely-NULL keys. Statistics treat NULL fraction as a separate density.
- **`COALESCE` vs `ISNULL`:** `ISNULL` takes the data type of the first argument (may truncate); `COALESCE` follows type precedence and evaluates arguments lazily-ish. In a `WHERE`, wrapping an indexed column in `COALESCE` is non-sargable — instead write `col = @x OR (col IS NULL AND @x IS NULL)`, or use `IS NOT DISTINCT FROM`.
- **Aggregates and ML features:** `AVG` over data with NULLs silently averages only the present values; whether NULL means "zero" or "missing" is a *business* decision, so impute explicitly and keep an `is_missing` flag.
- **Set operators** (`EXCEPT`, `INTERSECT`) use null-safe comparison, making them handy for diffing tables without `IS NULL` boilerplate.

### 6. Performance and design implications
- Prefer `NOT NULL` with a sensible default where the domain permits; it simplifies queries and helps the optimizer.
- Don't use sentinel values (`0`, `''`, `'1900-01-01'`) in place of NULL for "unknown" — they corrupt aggregates and join semantics.
- Add `NOT NULL` early; retrofitting on large tables requires a validate/backfill and a lock.
- Check NULL rates as part of data-quality tests (per column, per load).
- For optional FKs, remember a NULL FK is not checked and doesn't join; model "none" intentionally.
- Use filtered/partial indexes (`WHERE col IS NOT NULL`) when most rows are NULL.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `IS [NOT] NULL` | Correct NULL test | Always instead of `= NULL` |
| `COALESCE` | First non-NULL | Defaults, imputation |
| `NULLIF(a,b)` | NULL when equal | Divide-by-zero, blank→NULL |
| `IS NOT DISTINCT FROM` | Null-safe `=` | Comparing nullable columns |
| `NULLS FIRST/LAST` | Sort placement | Deterministic ordering |
| `SET ANSI_NULLS ON` | Standard NULL comparison (T-SQL) | Keep ON (OFF is deprecated) |
| `CONCAT` / `CONCAT_WS` | NULL-tolerant concat | String building |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| NULL for unknown | A real default exists (use NOT NULL + default) |
| `COALESCE` in SELECT | In `WHERE` on indexed columns |
| `NOT EXISTS` | Needing NULLs to count as "match" |
| Sentinel date (`9999-12-31`) | Reporting tools show it as real data |
| Nullable FK | Business rule says relationship is mandatory |

### 9. Common mistakes
- `WHERE col = NULL` → never TRUE → `IS NULL`.
- `NOT IN` with nullable subquery → empty result → `NOT EXISTS` or filter NULLs.
- `COUNT(*)` vs `COUNT(col)` confusion → wrong "number of valid observations" → choose deliberately.
- `AVG` ignoring NULLs unexpectedly → biased metric → `COALESCE` if NULL means zero.
- `col <> 'X'` dropping NULL rows → missing records → `OR col IS NULL`.
- Concatenating with NULL → whole string NULL → `CONCAT`/`COALESCE`.
- Unique index with multiple NULLs in SQL Server → violation → filtered unique index.

### 10. 🎯 Interview kit
**30-second answer:** NULL means unknown, and comparisons with it produce UNKNOWN, so `WHERE` drops those rows. Use `IS NULL`, `COALESCE`, `NULLIF`; aggregates ignore NULLs except `COUNT(*)`; `NOT IN` with a NULL returns nothing, so use `NOT EXISTS`.
**Structure a longer answer:** define 3VL → predicate/join/aggregate effects → classic traps (`NOT IN`, `<>`, `COUNT`) → design choices (NOT NULL, defaults) → data-quality checks.
**Likely follow-ups:**
- *`COUNT(*)` vs `COUNT(col)`?* → rows vs non-NULL values.
- *Find rows with no match?* → `LEFT JOIN … IS NULL` or `NOT EXISTS`.
- *Divide by zero → NULL?* → `NULLIF(denominator, 0)`.
- *Why is NULL ≠ NULL?* → unknown vs unknown can't be asserted equal.
- *Do GROUP BY / DISTINCT treat NULLs as equal?* → yes, one group.
**Signals interviewers look for:** 3VL fluency, the `NOT IN` trap, awareness of engine differences (unique indexes, concat, sort order).

### 11. Check yourself
- [ ] Fill in the 3VL truth table for AND/OR/NOT.
- [ ] Explain why `NOT IN` fails with a NULL and show two fixes.
- [ ] Predict outputs of `COUNT(*)`, `COUNT(col)`, `AVG(col)` on a column with NULLs.
- [ ] Write a null-safe comparison in your engine of choice.
- [ ] Explain how NULLs behave in `UNIQUE` constraints across SQL Server and PostgreSQL.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What happens if you INSERT a row without specifying a value for a NOT NULL column that has no default?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39765_What_happens_if_you_INSERT_a_row_without_specifying_a_value_.md>)
- [Basic] [What does a NULL value mean conceptually, and why is it not the same as zero or an empty string?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39813_What_does_a_NULL_value_mean_conceptually_and_why_is_it_not_t.md>)
- [Basic] [How do you write a SQL query to find rows in a source extract that violate a NOT NULL business rule before loading them into the warehouse?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39983_How_do_you_write_a_SQL_query_to_find_rows_in_a_source_extrac.md>)
- [Basic] [What is the difference between COUNT(*) and COUNT(column) when counting valid observations for a feature?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40528_What_is_the_difference_between_COUNT_and_COUNT_column_when_c.md>)
- [Basic] [How would you count NULLs per column in a metrics feature table to assess missing telemetry before modeling?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41028_How_would_you_count_NULLs_per_column_in_a_metrics_feature_ta.md>)
- [Intermediate] [How do you safely divide two columns so that a zero denominator returns NULL instead of an error?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39847_How_do_you_safely_divide_two_columns_so_that_a_zero_denomina.md>)
- [Intermediate] [How do NULL values behave in SQL comparisons and aggregate functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40265_How_do_NULL_values_behave_in_SQL_comparisons_and_aggregate_f.md>)
- [Intermediate] [Explain how you would use a LEFT JOIN plus IS NULL to find users who have never made a purchase for a negative training class.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40546_Explain_how_you_would_use_a_LEFT_JOIN_plus_IS_NULL_to_find_u.md>)

## Related concepts

- [MySQL Query Basics](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>) *(MySQL Platform)*
- [WHERE vs HAVING](<../../Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [NULL vs Zero in Aggregates](<../SQL_Functions/NULL_vs_Zero_in_Aggregates.md>) *(SQL Functions)*
- [CTEs in SQL Server](<../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [COUNT Semantics](<../Aggregation/COUNT_Semantics.md>) *(Aggregation)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Handling NULLs
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › NULL
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › NULL and Comparisons
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › NULL in Logical Expressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Working with NULL
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 4. Using Built-in Functions and Expressions › ISNULL and COALESCE
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Using Mathematical Functions › COALESCE
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using a Subquery Containing NULL with NOT IN
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *Hadoop: The Definitive Guide* — Tom White — Comparison with Databases › Validation and nulls
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.1 #14 Dealing incorrectly with NULL values
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Build Your First ETL Pipeline in ADF › Task 4: Filter Out NULLs
- *SQL Server Tutorial* (YouTube series) — Part 15: Different ways to replace NULL in sql server ([watch](https://www.youtube.com/watch?v=4ZoHY4RT1Fo))
- *SQL Server Tutorial* (YouTube series) — Part 137: How to check GUID is null or empty in SQL Server ([watch](https://www.youtube.com/watch?v=CbXQ3gDutXw))

