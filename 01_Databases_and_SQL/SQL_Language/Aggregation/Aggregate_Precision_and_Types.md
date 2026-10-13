---
id: "01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Aggregation"
tools: ["MySQL"]
---

# Aggregate Precision and Types

**L4 Advanced** · [↑ Aggregation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An aggregate's result depends on NULL handling and on the data type it accumulates in; integer division, approximate floating-point and wrong precision/scale quietly produce wrong averages and sums unless you choose types and NULL treatment on purpose.

### 1. The big picture
The standard aggregates (`SUM`, `AVG`, `MIN`, `MAX`, `COUNT`) look simple, but each hides two decisions:

1. **What happens to NULLs?** All of them except `COUNT(*)` skip NULL inputs. Whether that is *right* depends on your requirement ("average over orders that had a discount" vs. "average discount over all orders").
2. **What type is the result?** The engine picks an accumulator type from the input type. Summing integers can overflow; averaging integers can truncate; summing floats loses exactness; `NUMERIC(p,s)` results have derived precision and scale.

This lesson covers both, plus weighted averages, the "average of averages" trap, MIN/MAX on text, and nesting aggregates.

### 2. Core concepts and vocabulary
- **NULL-skipping** — aggregates ignore NULL inputs; a group with only NULLs yields NULL (for everything but `COUNT`).
- **Exact numeric (`DECIMAL`/`NUMERIC`, `MONEY`, integers)** — stored exactly; arithmetic is exact within precision.
- **Approximate numeric (`FLOAT`, `REAL`, `DOUBLE`)** — binary floating-point; many decimal fractions (0.1) can't be represented, errors accumulate.
- **Precision / scale** — total digits / digits after the decimal point.
- **Integer division** — dividing two integers yields an integer in many engines (T-SQL, SQL Server) and truncates the fraction.
- **Overflow** — the result exceeds the type's range (e.g. `SUM` of `INT` exceeding 2³¹−1).
- **Weighted average** — `SUM(value × weight) / SUM(weight)`.
- **Average of averages** — averaging pre-computed averages; wrong unless group sizes are equal.
- **Collation / lexicographic order** — how `MIN`/`MAX` compare text.

### 3. How it works, step by step
**NULL rules**

| Function | Input NULLs | All inputs NULL / empty |
|---|---|---|
| `SUM(x)` | skipped | `NULL` |
| `AVG(x)` | skipped (row not in denominator) | `NULL` |
| `MIN(x)`, `MAX(x)` | skipped | `NULL` |
| `COUNT(x)` | skipped | `0` |
| `COUNT(*)` | n/a | `0` |

If NULL should mean zero for the calculation, say so explicitly: `AVG(COALESCE(x,0))`. If a sentinel like 0 should mean "unknown", do the opposite: `AVG(NULLIF(x,0))`.

**Types of results (typical behaviour; check your engine)**
1. `SUM(INT)` → returns `INT` or `BIGINT` depending on engine (T-SQL: `INT` stays `INT` and can overflow; use `SUM(CAST(x AS BIGINT))`).
2. `AVG(INT)` → in SQL Server returns `INT` (truncated); PostgreSQL returns `NUMERIC`; MySQL returns a decimal. Cast to decimal to be portable.
3. `SUM(NUMERIC(p,s))` → `NUMERIC` with the same scale and (engine-defined) larger precision.
4. `SUM(FLOAT)` → `FLOAT`: order-dependent and non-exact.
5. `SUM(VARCHAR)` → implicit conversion in some engines (errors on non-numeric strings), error in others. Don't rely on it.

**Money rules**: Use `NUMERIC/DECIMAL` for financial amounts, with precision large enough for the *total* (not the single value), and scale matched to currency (usually 2–4). Round at defined points, not implicitly.

### 4. Worked example
```sql
-- Discount table: some orders have no discount recorded (NULL)
CREATE TABLE order_line (order_id INT, qty INT, price DECIMAL(10,2), discount DECIMAL(10,2) NULL);
INSERT INTO order_line VALUES (1,2,10.00,1.00),(2,1,20.00,NULL),(3,5, 4.00,NULL),(4,1, 6.00,3.00);

SELECT AVG(discount)                       AS avg_of_discounted,   -- (1+3)/2 = 2.00
       AVG(COALESCE(discount,0))           AS avg_all_orders,      -- (1+0+0+3)/4 = 1.00
       SUM(discount)                       AS total_discount,      -- 4.00
       SUM(discount) / COUNT(*)            AS sum_over_all_rows,   -- 1.00 (same as avg_all_orders)
       COUNT(discount)                     AS discounted_orders    -- 2
FROM order_line;
```
Neither average is "wrong": they answer different questions. Write the requirement into the query alias or a comment.

**Integer division**:

```sql
SELECT AVG(qty)                      FROM order_line;          -- SQL Server: 2 (INT), not 2.25
SELECT AVG(CAST(qty AS DECIMAL(12,4))) FROM order_line;         -- 2.2500
SELECT SUM(qty) * 1.0 / COUNT(*)     FROM order_line;           -- 2.250000
```

**Weighted average** (price weighted by quantity, with NULL/zero safety):

```sql
SELECT SUM(price * qty) / NULLIF(SUM(qty), 0) AS weighted_avg_price
FROM order_line;           -- (20+20+20+6)/9 = 7.33
```
`NULLIF(…,0)` turns division by zero into NULL instead of an error. Rows with NULL price or qty are dropped from the numerator but *not* automatically from the denominator, so filter both consistently (`WHERE price IS NOT NULL AND qty IS NOT NULL`).

**Floating-point drift**:

```sql
-- Engine-dependent, but typical: sum of 0.1 ten times in FLOAT is not exactly 1.0
SELECT SUM(CAST(0.1 AS FLOAT)) FROM (VALUES (1),(2),(3),(4),(5),(6),(7),(8),(9),(10)) v(n);  -- 0.9999999999999999
SELECT SUM(CAST(0.1 AS DECIMAL(5,2))) FROM (VALUES (1),(2),(3),(4),(5),(6),(7),(8),(9),(10)) v(n); -- 1.00
```

**Average of averages**: groups of size 1 (avg 100) and 99 (avg 10) → true overall ≈ 10.9, average of averages = 55. Keep `SUM` and `COUNT` per group (additive measures) and divide at the end.

### 5. Under the hood
- **Accumulator types**: the engine implements `SUM` as a running accumulator whose type is fixed at compile time from the input type. That's why overflow raises arithmetic errors (SQL Server error 8115 "Arithmetic overflow error converting expression to data type int") rather than promoting automatically.
- **Decimal arithmetic**: `SUM` of `DECIMAL(p,s)` keeps scale `s`; the result precision is raised (often to the maximum, e.g. 38) to hold the sum. Division and multiplication derive result scale by engine-specific rules, and can *reduce* scale, leading to rounding (e.g. SQL Server's `DECIMAL` division rule can truncate to a few decimals when precision is exhausted).
- **Floating-point** is IEEE-754 binary: associativity doesn't hold, so `SUM` over parallel workers (which add in different orders) can produce slightly different results run-to-run. That is a reason to avoid floats for accounting and for reproducible reports.
- **`MIN`/`MAX` on strings** use the column collation, i.e. lexicographic (and often case-insensitive), so `'10' < '9'`. On dates/numerics they use natural order.
- **Nested aggregates**: `AVG(SUM(salary))` is not valid in plain `SELECT … FROM` in most engines. Do the inner aggregate in a subquery/CTE first (`SELECT AVG(s) FROM (SELECT SUM(salary) s FROM e GROUP BY dept) t`). Some engines allow it only with `GROUP BY` as an extension (e.g. Oracle).
- **Aggregate vs. window**: `SUM() OVER (PARTITION BY …)` computes the same value but keeps every input row; use grouped aggregates to reduce rows and window aggregates when each row needs its group's total.

### 6. Performance and design implications
- **Choose the narrowest type that cannot overflow the total**; wider accumulators (BIGINT/DECIMAL) cost little but protect correctness.
- **Decimals are slower than integers/floats**, but correctness outweighs the difference for money.
- **Store money as integers of the minor unit** (cents) or `DECIMAL`; never `FLOAT`.
- **Implicit conversions** in aggregates (`SUM(varchar_col)`) prevent index use and risk errors; store numbers as numbers.
- **Additive measures** (sum, count) can be pre-aggregated and rolled up; non-additive ones (average, distinct count, median) need their components or the base data.
- **Audit control totals**: reconcile aggregate reports against source with counts, sums and checksums.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COALESCE(x,0)` | Replace NULL before aggregating | NULL means zero |
| `NULLIF(x,0)` | Turn a sentinel into NULL | Avoid division by zero, exclude zeros |
| `CAST(x AS DECIMAL(p,s))` | Force exact type/scale | Avoid integer division |
| `ROUND(x, n)`, `FORMAT` | Round for presentation | At the end, not in the middle |
| `SUM(CAST(x AS BIGINT))` | Prevent INT overflow | Large totals |
| `FILTER (WHERE …)` / `SUM(CASE …)` | Conditional aggregation | Pivots |
| `STRING_AGG`, `ARRAY_AGG`, `JSON_AGG` | Aggregate to a list/document | Nested results |
| `ONLY_FULL_GROUP_BY` / strict mode | Enforce valid grouping | Always |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `NUMERIC/DECIMAL` for money and exact totals | `FLOAT/DOUBLE` for currency |
| `FLOAT` for scientific measurements, ML features | Needs exact equality or audit reproducibility |
| `AVG(col)` over non-NULL rows | NULL really means 0 and the denominator must include them |
| Weighted average | Equal-weight average of already-averaged groups |
| Store sum+count and divide at read time | Storing averages in summary tables |

### 9. Common mistakes
- **Assuming `SUM(NULL)` is 0** → NULL propagates into reports → `COALESCE`.
- **Integer `AVG`** → truncated results → cast to decimal.
- **`INT` overflow in `SUM`** → runtime error at scale → `BIGINT`/`DECIMAL`.
- **Float for money** → cents drift, totals don't reconcile → `DECIMAL`.
- **Average of averages** → biased numbers → carry sum and count.
- **`MIN/MAX` on numeric strings** → `'9' > '10'` → store as numbers or cast.
- **Dividing by `COUNT(*)` vs. `COUNT(col)`** → wrong denominator → decide and document.
- **Rounding before aggregating** → accumulated rounding error → round at the end.

### 10. 🎯 Interview kit
**30-second answer:** Aggregates skip NULLs, so `AVG` divides by the non-NULL count and an all-NULL group gives NULL. The result type follows the input: integer averages can truncate, integer sums can overflow, and floats accumulate error, so use `DECIMAL` for money, cast where needed, and use weighted averages (sum of value×weight over sum of weight) rather than averaging averages.

**Structure a longer answer:** NULL semantics → type/accumulator rules → money and precision/scale → weighted and average-of-averages → engine-specific division/overflow → how to validate (control totals).

**Likely follow-ups:**
- *Why NUMERIC over DOUBLE for sums of money?* → Exact decimal vs. binary approximation; errors accumulate and totals fail to reconcile.
- *How do you choose precision/scale?* → Scale from the currency/business rule; precision from the maximum *total* you'll ever sum.
- *What does `SUM` return on an empty set?* → NULL (use `COALESCE`); `COUNT` returns 0.
- *Can you nest `AVG(SUM(x))`?* → Not directly; use a subquery or CTE.
- *Does MIN/MAX work on text?* → Yes, by collation order.

**Signals interviewers look for:** you reason about the denominator and the type before writing the query, and you know the engine-specific traps.

### 11. Check yourself
- [ ] Predict `AVG(x)` for values `1, 2, NULL, 5` and for `AVG(COALESCE(x,0))`.
- [ ] Explain why `AVG` of an `INT` column may return `2` for data averaging `2.25`.
- [ ] Choose precision and scale for summing up to 50 million invoices of at most 99,999.99 each.
- [ ] Write a weighted average with division-by-zero protection.
- [ ] Explain why average-of-averages is wrong and how to fix a dashboard that does it.
- [ ] Show how to compute the average of per-department salary sums.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How would you find the highest and lowest salary in the employees table using aggregate functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00736_How_would_you_find_the_highest_and_lowest_salary_in_the_empl.md>)
- [Basic] [Can you combine multiple aggregate functions in a single SELECT statement? Give an example.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00833_Can_you_combine_multiple_aggregate_functions_in_a_single_SEL.md>)
- [Basic] [Do aggregate functions ignore NULL values? Explain which ones do and which do not.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00847_Do_aggregate_functions_ignore_NULL_values_Explain_which_ones.md>)
- [Basic] [Write a query to find the average salary of employees in each department.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00880_Write_a_query_to_find_the_average_salary_of_employees_in_eac.md>)
- [Intermediate] [Write a query to compute the average rating per movie and only show movies with at least 50 ratings.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00881_Write_a_query_to_compute_the_average_rating_per_movie_and_on.md>)
- [Intermediate] [Explain how weighted averages differ from simple AVG in SQL, and write the general pattern for SUM(value*weight)/SUM(weight) with correct NULL/zero handling.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00771_Explain_how_weighted_averages_differ_from_simple_AVG_in_SQL_.md>)
- [Deep] [What happens when you apply SUM to a column of type VARCHAR that contains numeric-looking strings?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00843_What_happens_when_you_apply_SUM_to_a_column_of_type_VARCHAR_.md>)
- [Deep] [Write a query to find departments where the average salary exceeds the company-wide average salary.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00864_Write_a_query_to_find_departments_where_the_average_salary_e.md>)

## Related concepts

- [NULL vs Zero in Aggregates](<../SQL_Functions/NULL_vs_Zero_in_Aggregates.md>) *(SQL Functions)*
- [Aggregation Query Practice](<../SQL_Fundamentals/Aggregation_Query_Practice.md>) *(SQL Fundamentals)*
- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Aggregate Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Aggregate Functions and NULL
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Summarizing Results with Window Aggregates
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › Aggregate Functions
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › SUM
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Aggregation with NULLs
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Perform an aggregation to calculate the average salary › Perform an aggregation to calculate the average salary\
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Display the results › Aggregating data
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Putting It All Together
- *SQL Server Tutorial* (YouTube series) — Part 101: Grouping Sets in SQL Server ([watch](https://www.youtube.com/watch?v=Tc6-X6mNv7k))

