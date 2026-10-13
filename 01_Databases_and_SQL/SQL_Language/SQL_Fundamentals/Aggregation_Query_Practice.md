---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Aggregation Query Practice

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Most reporting questions reduce to one pattern — *filter rows (`WHERE`) → group (`GROUP BY`) → aggregate (`SUM/COUNT/AVG/MIN/MAX`) → filter groups (`HAVING`) → order/limit* — and a small set of recipes (top-N per group, percent of total, buckets, running totals, safe ratios, distinct counts) covers nearly every practice question.

### 1. The big picture
An **aggregate function** collapses many rows into one value. Combined with `GROUP BY` it produces one row per group. This is the core of dashboards, marts and KPI queries; it is also the most common interview coding topic. The Grokking Relational Database Design intro shows the minimum idea — `SELECT SUM(Price) FROM my_table;` adds up a column — and everything else is a variation: *which rows go in*, *how they're grouped*, *what is computed*, and *which groups survive*.

Data-quality work uses the same tools as assertions. Data Quality Fundamentals frames tests as "assertions" — e.g. `SELECT * FROM Customers WHERE City IS NULL` or a range test on `Price` — and counts/aggregates are how you turn those into metrics (null rate, out-of-range count).

### 2. Core concepts and vocabulary
- **Aggregate function** — `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` (plus `STDDEV`, `STRING_AGG`, …); ignores NULLs except `COUNT(*)`.
- **Group** — rows sharing the same values in the `GROUP BY` columns.
- **`GROUP BY`** — defines groups; every non-aggregated `SELECT` column must be in it (otherwise there's no single value per group).
- **`WHERE` vs `HAVING`** — `WHERE` filters *rows before* grouping; `HAVING` filters *groups after* aggregation.
- **Grain** — what one output row represents ("one row per customer per day"). Decide it first.
- **Conditional aggregation** — an aggregate over a `CASE` (or `FILTER`) to count/sum only some rows.
- **Fan-out** — a join that multiplies rows, inflating `SUM`/`COUNT`.
- **Logical processing order** — `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` (aliases defined here) → `ORDER BY` → `LIMIT/TOP`.

### 3. How it works, step by step
For:
```sql
SELECT category_id, COUNT(*) AS orders, SUM(amount) AS revenue
FROM   orders
WHERE  order_date >= '2025-01-01'
GROUP  BY category_id
HAVING COUNT(*) > 100
ORDER  BY revenue DESC;
```
1. `FROM`: read `orders`.
2. `WHERE`: drop rows before 2025.
3. `GROUP BY`: bucket remaining rows by `category_id`.
4. Aggregates are computed per bucket.
5. `HAVING`: drop buckets with 100 or fewer orders.
6. `SELECT` produces columns; `ORDER BY` can use the alias `revenue` (allowed because it runs after `SELECT`).

`WHERE` cannot reference `COUNT(*)` — it runs before groups exist; and `HAVING` should not be used for plain row filters (it works, but filters late and prevents index use).

### 4. Worked example — the recipe book
Assume `orders(order_id, customer_id, category_id, amount, order_date)` and `employees(id, name, salary, department)`.

```sql
-- 1. Orders per customer, only customers with more than five
SELECT customer_id, COUNT(*) AS n
FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;

-- 2. Highest and lowest salary per department
SELECT department, MAX(salary) AS hi, MIN(salary) AS lo FROM employees GROUP BY department;

-- 3. Revenue per category, highest first
SELECT category_id, SUM(amount) AS revenue FROM orders GROUP BY category_id ORDER BY revenue DESC;

-- 4. Products above the overall average price (scalar subquery)
SELECT * FROM products WHERE price > (SELECT AVG(price) FROM products);

-- 5. Percent of total in one query (window over the aggregate)
SELECT product_id, SUM(amount) AS sales,
       100.0 * SUM(amount) / SUM(SUM(amount)) OVER () AS pct_of_total
FROM sales GROUP BY product_id;

-- 6. Buckets (0-100, 100-500, 500+)
SELECT CASE WHEN amount < 100 THEN '0-99'
            WHEN amount < 500 THEN '100-499'
            ELSE '500+' END AS bucket,
       COUNT(*) AS orders
FROM orders GROUP BY CASE WHEN amount < 100 THEN '0-99' WHEN amount < 500 THEN '100-499' ELSE '500+' END;

-- 7. Safe ratio (conversion rate)
SELECT day, 1.0 * SUM(conversions) / NULLIF(SUM(visits), 0) AS conv_rate FROM kpi GROUP BY day;

-- 8. Distinct products per store per month
SELECT store_id, DATE_TRUNC('month', sold_at) AS month, COUNT(DISTINCT product_id) AS n_products
FROM sales GROUP BY store_id, DATE_TRUNC('month', sold_at);

-- 9. Daily mart table: one row per store per day
INSERT INTO mart.store_daily (store_id, day, sales, tx_count)
SELECT store_id, CAST(sold_at AS date), SUM(amount), COUNT(*)
FROM raw.transactions GROUP BY store_id, CAST(sold_at AS date);

-- 10. Top 3 paid per department (window, then filter)
SELECT * FROM (
  SELECT e.*, DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS r FROM employees e) x
WHERE r <= 3;

-- 11. Most frequent category
SELECT category FROM items GROUP BY category ORDER BY COUNT(*) DESC, category LIMIT 1;

-- 12. 10 newest users
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;      -- TOP (10) in SQL Server, FETCH FIRST 10 ROWS in ANSI
```

**Reading the results.** Recipe 5 nests an aggregate inside a window: `SUM(amount)` is the group sum, and `SUM(SUM(amount)) OVER ()` is the grand total over those group rows — legal because windows are evaluated after grouping. Recipe 7 multiplies by `1.0` to avoid integer division and uses `NULLIF` to avoid divide-by-zero (a zero-visit day gives NULL, not an error). Recipe 6 repeats the `CASE` in `GROUP BY` because many engines don't allow an alias there (MySQL, PostgreSQL, Snowflake do; SQL Server does not) — or compute it in a CTE.

### 5. Under the hood
- **Hash vs stream aggregate.** The engine either builds a hash table keyed by group (Hash Match Aggregate; memory proportional to number of groups) or reads pre-sorted input and aggregates as the key changes (Stream/Group Aggregate; needs an index or a sort on the group keys). An index on `(group_cols) INCLUDE (agg_cols)` enables a cheap stream aggregate.
- **Partial aggregation.** Parallel and distributed engines aggregate locally first (partial sums per node), shuffle by group key, then combine — so `SUM`, `COUNT`, `MIN`, `MAX` are cheap, while `COUNT(DISTINCT)` and medians need the full value set.
- **Skew.** One group with 100× more rows becomes one overloaded reducer (Spark/BigQuery); mitigate by salting or pre-aggregating.
- **NULL in groups.** `GROUP BY` puts all NULLs in one group even though `NULL = NULL` is unknown.
- **Why the "non-aggregated column" error exists.** A group has many `name` values; the engine can't pick one deterministically. MySQL's legacy mode silently picked an arbitrary row (now `ONLY_FULL_GROUP_BY`); PostgreSQL permits columns functionally dependent on the primary key that is grouped.
- **Fan-out.** Joining orders (1) to order_lines (N) before `SUM(order.amount)` repeats the amount N times. Aggregate before joining, or sum at the right grain.

### 6. Performance and design implications
- Filter early in `WHERE`; aggregate on the smallest columns needed; avoid `SELECT *` in grouped queries.
- Pre-aggregate large facts into **mart/summary tables** at the grain you query most (daily per store), then report from those.
- Use a supporting index (or clustered columnstore / partition pruning) on date and group keys.
- `COUNT(DISTINCT x)` is expensive: needs a sort or hash on x per group; use approximate versions on dashboards if ±2% is acceptable.
- Wrap-around functions on grouping keys (`DATE_TRUNC`, `YEAR()`) prevent index-ordered streaming; persist a `date_key`/month column if you group by it constantly.
- Prefer one pass with `GROUPING SETS`/`ROLLUP` over several `UNION ALL`ed group-bys.
- Cache-like designs (materialized views, summary tables) trade freshness for speed: define refresh cadence and verify counts against the source.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COUNT(*)` / `COUNT(col)` / `COUNT(DISTINCT col)` | Rows / non-NULL / distinct non-NULL | Volume, completeness, cardinality |
| `SUM`, `AVG`, `MIN`, `MAX` | Numeric/ordered aggregates (MIN/MAX also work on text/dates) | KPIs |
| `GROUP BY ROLLUP / CUBE / GROUPING SETS` | Subtotals and multiple groupings | Report totals |
| `HAVING` | Filter groups | After aggregation only |
| `FILTER (WHERE …)` / `SUM(CASE …)` | Conditional aggregate | Pivot-style counts |
| `NULLIF(x,0)` | Safe divisor | Ratios |
| `ORDER BY … LIMIT / TOP / FETCH FIRST` | Top-N | Leaderboards |
| `ONLY_FULL_GROUP_BY` (MySQL) | Enforce standard grouping | Always on |
| `STRING_AGG`/`GROUP_CONCAT` | Concatenate per group | Lists |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `GROUP BY` + `HAVING` for group filters | Row filters belong in `WHERE` |
| Window aggregate for percent-of-total/detail rows | Pure summary — plain `GROUP BY` is cheaper |
| `ROW_NUMBER`/`DENSE_RANK` window for top-N per group | Global top-N — `ORDER BY … LIMIT` |
| Pre-built mart tables | Ad-hoc, always-fresh requirement |
| Exact `COUNT(DISTINCT)` | Huge cardinalities on interactive dashboards |
| Conditional aggregation (`SUM(CASE…)`) for pivots | Dynamic column lists (`PIVOT` or app-side) |

### 9. Common mistakes
- Using `WHERE COUNT(*) > 5` → error → `HAVING`.
- Selecting a non-grouped column → error or arbitrary value → group or aggregate it.
- Joining before aggregating (fan-out) → inflated totals → aggregate in a subquery/CTE first, then join.
- Integer division in ratios → 0 or truncated → multiply by `1.0`/cast.
- `COUNT(col)` when you meant rows → under-count with NULLs → `COUNT(*)`.
- Missing tie-break in "top 1 per group" → nondeterministic result → add unique `ORDER BY` key.
- Grouping by a high-cardinality timestamp instead of truncating → every row is its own group → truncate to day/hour.
- Averaging averages → wrong weighting → `SUM(x)/SUM(n)`.

### 10. 🎯 Interview kit
**30-second answer:** "Aggregation collapses rows: `GROUP BY` defines groups, aggregate functions compute per group, `WHERE` filters rows before and `HAVING` filters groups after. I decide the output grain first, watch for join fan-out and NULLs, and use window functions when I need detail rows plus group context."

**Structure a longer answer:** state grain → write the grouped query → add filters in the right clause → check NULLs, ties and fan-out → discuss performance (index, pre-aggregation).

**Likely follow-ups:**
- *`WHERE` vs `HAVING`?* → row vs group filter, evaluation order.
- *`COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`?*
- *Top-3 per department?* → `DENSE_RANK`/`ROW_NUMBER` in a subquery.
- *Percent of total?* → `SUM(x) OVER ()` or a scalar subquery.
- *Customers with >5 orders?* → `GROUP BY … HAVING COUNT(*) > 5`.
- *Why is my total double what it should be?* → fan-out from the join.
- *Build a daily mart?* → `INSERT … SELECT … GROUP BY store, day`, with idempotent reload (delete partition then insert, or `MERGE`).

**Signals interviewers look for:** clean grain definition, correct clause placement, NULL/zero/duplicate awareness, and ability to propose a mart for repeated queries.

### 11. Check yourself
- [ ] Write the query for "customers whose total spend exceeds twice the average customer's spend."
- [ ] Explain the logical order of `FROM/WHERE/GROUP BY/HAVING/SELECT/ORDER BY` and why `HAVING alias` fails in some engines.
- [ ] Rewrite a double-counting join so totals are correct.
- [ ] Produce percent-of-total per category with one scan.
- [ ] Build a daily per-store summary table load that can safely be re-run.
<!-- explainer:end -->

## Practice questions

- [Basic] [How would you write a query to return all columns for the 10 most recently created users?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39773_How_would_you_write_a_query_to_return_all_columns_for_the_10.md>)
- [Basic] [Write a query to find the most frequent category value in a column so you can decide on encoding.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40517_Write_a_query_to_find_the_most_frequent_category_value_in_a_.md>)
- [Intermediate] [How would you compute what percentage each product's sales are of the total sales in one query?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39845_How_would_you_compute_what_percentage_each_product_s_sales_a.md>)
- [Intermediate] [Why might you prefer to compute aggregate features in SQL rather than pulling raw rows into pandas?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40537_Why_might_you_prefer_to_compute_aggregate_features_in_SQL_ra.md>)
- [Intermediate] [Write a query to find users who hit their token quota by summing tokens per user and comparing against a quota column.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40768_Write_a_query_to_find_users_who_hit_their_token_quota_by_sum.md>)
- [Intermediate] [How would you write SQL to compute the fraction of production requests where a required feature was defaulted/imputed (because the real value was missing), as a serving-quality metric?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40714_How_would_you_write_SQL_to_compute_the_fraction_of_productio.md>)
- [Deep] [How do you compute the median of a feature in SQL when there is no built-in MEDIAN function, and why does the median matter for imputation?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40540_How_do_you_compute_the_median_of_a_feature_in_SQL_when_there.md>)
- [Deep] [Explain how you'd design a table and query to answer 'which feature flags is this user in?' where flags target by percentage rollout using a deterministic hash of user_id. How do you keep the evaluation stable as the percentage changes?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40474_Explain_how_you_d_design_a_table_and_query_to_answer_which_f.md>)

## Related concepts

- [Counting with Joins](<../Aggregation/Counting_with_Joins.md>) *(Aggregation)*
- [Basic Query Practice](<../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*
- [First Last per Group and Running Totals](<../Window_Functions/First_Last_per_Group_and_Running_Totals.md>) *(Window Functions)*
- [Classic SQL Query Problems](<../../Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Window Functions Overview](<../SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*
- [Time-based Windows](<../Window_Functions/Time-based_Windows.md>) *(Window Functions)*

## References

- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Architecting for Data Reliability › Measuring and Maintaining High Data Reliability at Ingestion
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › � G
- *Expert T-SQL Window Functions in SQL Server* — Kathi Kellenberger, Clayton Groom, Ed Pollack — Summary › Contents
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *Data Pipelines Pocket Reference* — James Densmore — Transforming Data › Modeling Change Capture Data

