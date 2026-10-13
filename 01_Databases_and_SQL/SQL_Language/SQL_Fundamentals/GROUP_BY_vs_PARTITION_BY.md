---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# GROUP BY vs PARTITION BY

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `GROUP BY` *collapses* rows into one row per group; `PARTITION BY` (inside `OVER`) computes the same kind of aggregate but *keeps every row* and attaches the group's result to each — so use `GROUP BY` for summaries and `OVER (PARTITION BY …)` when you need detail and group context side by side.

### 1. The big picture
Both clauses answer "per what?" — per customer, per region, per day — but they change the *shape* of the result differently. `GROUP BY` reduces N rows to G rows (one per group). A window function with `PARTITION BY` returns N rows, each decorated with a per-group value. That difference decides everything: percent-of-total, "row vs group average", top-N per group, running totals and de-duplication all need the detail rows retained, so they use windows; dashboards and KPI tables need one row per group, so they use `GROUP BY`.

They compose: you can `GROUP BY` first and then apply a window over the grouped result (e.g. rank regions by their monthly total).

### 2. Core concepts and vocabulary
- **Aggregate function** — `COUNT/SUM/AVG/MIN/MAX…` combining many values to one.
- **`GROUP BY`** — partitions rows into groups and returns one row per group; every selected column must be grouped or aggregated.
- **Window function** — a function evaluated over a *window* of rows related to the current row; `OVER (…)` introduces it.
- **`PARTITION BY`** — divides the window's rows into independent partitions (like groups, but non-collapsing).
- **`ORDER BY` in `OVER`** — orders rows within a partition; required for ranking, `LAG/LEAD`, running totals.
- **Frame** — the subset of the partition around the current row (`ROWS BETWEEN …`) used by aggregates with `ORDER BY`.
- **Ranking functions** — `ROW_NUMBER` (unique), `RANK` (ties share, gaps), `DENSE_RANK` (ties share, no gaps).
- **Grouping key NULL** — `GROUP BY` treats all NULLs as one group.

### 3. How it works, step by step
Logical evaluation order of a `SELECT`:
1. `FROM` / `JOIN` — build the rows.
2. `WHERE` — filter rows.
3. `GROUP BY` — form groups.
4. `HAVING` — filter groups.
5. **Window functions** — computed on the result of steps 1–4.
6. `SELECT` list, `DISTINCT`, `ORDER BY`, `TOP`.

Consequences: windows run *after* grouping (you may window over aggregates), cannot appear in `WHERE`/`HAVING`, and need a CTE/subquery to be filtered. A window aggregate without `ORDER BY` covers the whole partition; with `ORDER BY` it defaults to a *running* frame (`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`).

| Question | Tool |
|---|---|
| Total sales per region | `GROUP BY region` |
| Each order and its region's total | `SUM(amount) OVER (PARTITION BY region)` |
| Order's share of region | `amount / SUM(amount) OVER (PARTITION BY region)` |
| Keep latest row per customer | `ROW_NUMBER() OVER (PARTITION BY c ORDER BY ts DESC) = 1` |
| Running total | `SUM(amount) OVER (ORDER BY d ROWS UNBOUNDED PRECEDING)` |
| Count per group, with detail | `COUNT(*) OVER (PARTITION BY g)` |

### 4. Worked example
```sql
-- orders(order_id, customer_id, region, order_date, amount)

-- GROUP BY: one row per region-month
SELECT region, DATEFROMPARTS(YEAR(order_date), MONTH(order_date), 1) AS month_start,
       COUNT(*) AS orders, SUM(amount) AS revenue
FROM   dbo.orders
GROUP  BY region, DATEFROMPARTS(YEAR(order_date), MONTH(order_date), 1);

-- PARTITION BY: every order, plus its region total and share
SELECT order_id, region, amount,
       SUM(amount) OVER (PARTITION BY region)                              AS region_total,
       100.0 * amount / SUM(amount) OVER (PARTITION BY region)             AS pct_of_region,
       COUNT(*)    OVER (PARTITION BY region)                              AS orders_in_region,
       RANK()      OVER (PARTITION BY region ORDER BY amount DESC)         AS rank_in_region
FROM   dbo.orders;

-- Compose: group first, then rank the groups
WITH monthly AS (
    SELECT region, DATEFROMPARTS(YEAR(order_date),MONTH(order_date),1) AS m, SUM(amount) AS rev
    FROM dbo.orders GROUP BY region, DATEFROMPARTS(YEAR(order_date),MONTH(order_date),1)
)
SELECT region, m, rev,
       RANK() OVER (PARTITION BY m ORDER BY rev DESC) AS region_rank,
       rev - LAG(rev) OVER (PARTITION BY region ORDER BY m) AS delta_vs_prev_month
FROM monthly;

-- Top 3 orders per customer
SELECT * FROM (
   SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY amount DESC, order_id) AS rn
   FROM dbo.orders) x
WHERE rn <= 3;
```
Sample rows: a region with orders 100, 100, 50 gives `ROW_NUMBER` 1,2,3, `RANK` 1,1,3, `DENSE_RANK` 1,1,2 — exactly the difference to quote when asked which to use for de-duplication (`ROW_NUMBER`, because you need exactly one survivor).

### 5. Under the hood (optional at L4)
`GROUP BY` is implemented by a **Hash Match (Aggregate)** or a **Stream Aggregate** (needs sorted input). A window with `PARTITION BY` uses **Segment** and **Window Spool** (or batch-mode **Window Aggregate**): the input is sorted by (partition, order) columns, segmented at each partition boundary, and the function evaluated per segment. When an `ORDER BY` frame is `RANGE` (the default) the engine may need an on-disk spool; `ROWS` framing is typically much faster, so write `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` explicitly for running totals. Nulls: `GROUP BY` and `PARTITION BY` both place NULLs in one group; `ORDER BY` sorts NULLs first ascending in SQL Server.

### 6. Performance and design implications
- A POC index (Partition, Order, Covering) removes the Sort under a window; see the actual plan for `Sort` and spills.
- Windows keep all rows, so the result can be large; filter first (`WHERE`) and project only needed columns.
- Prefer `ROWS` to `RANGE` frames; avoid many windows with *different* partitions in one query (each may need its own sort).
- To get one row per group *and* a detail column, `ROW_NUMBER` is usually cheaper than a correlated subquery or self-join.
- `COUNT(*) OVER ()` for a grand total is convenient but repeats on each row — fine for hundreds, wasteful for millions.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GROUP BY ROLLUP / CUBE / GROUPING SETS` | Multi-level subtotals | Reports |
| `OVER (PARTITION BY a)` | Per-group aggregate, rows kept | Share, deviation |
| `ROW_NUMBER / RANK / DENSE_RANK / NTILE` | Ranking | Top-N, dedupe, buckets |
| `LAG / LEAD` | Neighbouring rows | Deltas |
| `ROWS BETWEEN …` | Explicit frame | Running/moving totals |
| `PIVOT` / conditional aggregation | Rows → columns | Wide reports |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `GROUP BY`: you only need the summary | You also need detail columns — you'd have to join back |
| Window: need detail + group context | Result set huge and you only need summaries |
| `ROW_NUMBER`: exactly one per group | Ties should all be returned — `RANK` |
| Conditional aggregation over `PIVOT` | You like fixed syntax and the column list is static |

### 9. Common mistakes
- Selecting a non-grouped column with `GROUP BY` → error / wrong semantics → add to group or aggregate.
- Window in `WHERE` → error → CTE/subquery.
- `GROUP BY` fewer rows than expected → grouping key NULLs merged, or an inner join dropped rows → check NULLs and joins; use `COALESCE` for a labelled "unknown" group.
- Using `RANK` for de-dup → ties survive → `ROW_NUMBER` with tie-breaker.
- `SUM() OVER (ORDER BY d)` with duplicates of `d` and default `RANGE` → tied rows share the same running value → use `ROWS` and a unique tie-breaker.
- Confusing `COUNT(*)` with `COUNT(col)` → NULLs not counted.

### 10. 🎯 Interview kit
**30-second answer:** "`GROUP BY` collapses to one row per group; `PARTITION BY` in a window function computes the same aggregate but keeps every row, so I can show detail next to the group value — percent of total, rank within group, deltas."

**Structure a longer answer:** clarify (grain of output, need detail?) → mechanism (evaluation order, windows after grouping) → trade-offs (cost of sort, frame choice) → scenario (top-3 orders per customer / share of region).

**Likely follow-ups:**
- *`COUNT(*)` with GROUP BY vs window?* → one row vs all rows.
- *ROW_NUMBER vs RANK vs DENSE_RANK?* → unique / gaps / no gaps.
- *How do NULL keys behave?* → one group.
- *Why can't I filter by a window in `WHERE`?* → evaluation order.

**Signals interviewers look for:** evaluation-order awareness, frame defaults, sensible ranking choice.

### 11. Check yourself
- [ ] Rewrite a "region total joined back" query using a window.
- [ ] Predict `ROW_NUMBER/RANK/DENSE_RANK` for 100,100,50.
- [ ] Explain why a window needs a CTE to be filtered.
- [ ] Explain default frame behaviour with `ORDER BY`.
- [ ] List what happens to NULL keys.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between COUNT(DISTINCT column) and COUNT(column), and when do their results differ?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39850_What_is_the_difference_between_COUNT_DISTINCT_column_and_COU.md>)
- [Basic] [What is the difference between the GROUP BY and ORDER BY clauses?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41041_What_is_the_difference_between_the_GROUP_BY_and_ORDER_BY_cla.md>)
- [Basic] [What is the logical order of execution of a SQL query (FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY), and why does it matter when writing transforms?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40065_What_is_the_logical_order_of_execution_of_a_SQL_query_FROM_W.md>)
- [Basic] [What is the difference between HAVING and WHERE when you want to filter groups by an aggregate feature threshold?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40565_What_is_the_difference_between_HAVING_and_WHERE_when_you_wan.md>)
- [Intermediate] [How would you group orders by both region and month at the same time to build a summary report?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39825_How_would_you_group_orders_by_both_region_and_month_at_the_s.md>)
- [Intermediate] [What does PARTITION BY do in a window function, and how is it different from GROUP BY?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40201_What_does_PARTITION_BY_do_in_a_window_function_and_how_is_it.md>)
- [Deep] [Explain GROUPING SETS, ROLLUP, and CUBE for computing subtotals and a grand total in one scan, and how the extra NULL grouping rows can become a downstream correctness trap.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39912_Explain_GROUPING_SETS_ROLLUP_and_CUBE_for_computing_subtotal.md>)
- [Deep] [What is the difference between GROUP BY with COUNT and using COUNT as a window function over PARTITION BY?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40144_What_is_the_difference_between_GROUP_BY_with_COUNT_and_using.md>)

## Related concepts

- [GROUP BY and HAVING](<../Aggregation/GROUP_BY_and_HAVING.md>) *(Aggregation)*
- [WHERE vs HAVING](<../../Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [PostgreSQL Query Basics](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>) *(PostgreSQL Platform)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Reduce-Side Joins and Grouping › GROUP BY
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Batch and Real-Time Processing in Databricks › Grouping data
- *SQL Server Tutorial* (YouTube series) — Part 103: Cube in SQL Server ([watch](https://www.youtube.com/watch?v=9d7fjBgtxos))

