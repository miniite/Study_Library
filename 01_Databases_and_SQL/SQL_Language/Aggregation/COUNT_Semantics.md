---
id: "01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Aggregation"
tools: []
---

# COUNT Semantics

**L4 Advanced** · [↑ Aggregation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `COUNT(*)` counts rows, `COUNT(col)` counts rows where `col` is not NULL, and `COUNT(DISTINCT col)` counts distinct non-NULL values; every other aggregate ignores NULLs, but `COUNT` is the only one that returns 0 (not NULL) on empty input.

### 1. The big picture
`COUNT` is the most used aggregate and the one with the most subtle variants. Choosing the wrong form silently gives a wrong number (counting NULLs you meant to skip, double-counting after a join) or an unexpectedly slow plan (a distinct count that must sort or hash every value). This lesson separates *what* each form means logically from *what work* it forces the engine to do, and covers the classic traps: NULLs, empty sets, join fan-out, and the cost of exact counts on large tables.

### 2. Core concepts and vocabulary
- **Aggregate function** — collapses a set of rows into a single value (per group, or for the whole result when there is no `GROUP BY`).
- **`COUNT(*)`** — number of rows in the group; takes no column, so NULLs are irrelevant.
- **`COUNT(expr)`** — number of rows where `expr` is **not NULL**.
- **`COUNT(DISTINCT expr)`** — number of different non-NULL values of `expr`.
- **`COUNT(1)` / `COUNT(0)`** — a non-NULL constant, so it counts every row: the same result as `COUNT(*)` and, in mainstream optimizers, the same plan.
- **Scalar aggregate** — aggregate with no `GROUP BY`; always returns exactly one row, even for empty input.
- **Grouped aggregate** — one row per group; groups come from existing rows, so an empty group cannot appear.
- **Fan-out** — a join that multiplies rows (one-to-many), inflating any count or sum computed afterwards.
- **Visibility map / row-count metadata** — engine structures that may let some engines avoid scanning (PostgreSQL index-only scans) but do not make exact counts free.

### 3. How it works, step by step
For `SELECT COUNT(x) FROM t WHERE p`:
1. The engine reads rows satisfying `p` (table scan, index scan or seek).
2. For each row it evaluates `x`; if NULL, the row doesn't increment the counter. With `COUNT(*)` there's nothing to evaluate.
3. For `DISTINCT`, it also has to remember which values it has seen (a hash set, or a sort followed by a streaming comparison).
4. It emits the final counter. For a grouped query, one counter per group key.

**Behaviour table**

| Expression | Counts | On empty input | On all-NULL column |
|---|---|---|---|
| `COUNT(*)` | all rows | `0` | row count |
| `COUNT(col)` | non-NULL `col` | `0` | `0` |
| `COUNT(DISTINCT col)` | distinct non-NULL | `0` | `0` |
| `SUM(col)`, `AVG(col)`, `MIN`, `MAX` | non-NULL only | `NULL` | `NULL` |

**Counting NULLs, conditionally**
- NULLs in a column: `COUNT(*) - COUNT(col)` or `COUNT(*) FILTER (WHERE col IS NULL)` or `SUM(CASE WHEN col IS NULL THEN 1 ELSE 0 END)`.
- Conditional count in one pass: `COUNT(CASE WHEN status = 'active' THEN 1 END)` (the `CASE` yields NULL otherwise, so those rows aren't counted), or `COUNT(*) FILTER (WHERE status='active')` where supported.

### 4. Worked example
```sql
CREATE TABLE customer (customer_id INT PRIMARY KEY, name VARCHAR(50), region VARCHAR(10));
CREATE TABLE orders   (order_id INT PRIMARY KEY, customer_id INT, coupon VARCHAR(10));
INSERT INTO customer VALUES (1,'Ann','EU'),(2,'Bo','EU'),(3,'Cy',NULL);
INSERT INTO orders   VALUES (10,1,'SAVE'),(11,1,NULL),(12,2,'SAVE'),(13,2,'SAVE');

SELECT COUNT(*)                  AS all_orders,       -- 4
       COUNT(coupon)             AS with_coupon,      -- 3 (NULL skipped)
       COUNT(DISTINCT coupon)    AS distinct_coupons, -- 1
       COUNT(DISTINCT customer_id) AS buyers          -- 2
FROM orders;

-- Empty-set behaviour
SELECT COUNT(*), SUM(order_id) FROM orders WHERE 1 = 0;   -- 0, NULL
```

**Customers with zero orders** (count must not be `COUNT(*)` after an outer join):

```sql
SELECT c.customer_id, c.name,
       COUNT(o.order_id) AS order_count     -- not COUNT(*)
FROM customer c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.name;
-- Ann 2, Bo 2, Cy 0
```
`COUNT(*)` here would return 1 for Cy, because the outer join produced one row (with NULLs). Counting a column from the *optional* side gives 0. "Can COUNT return 0 for a group?" Yes: when the group exists (from the preserved side) but its counted column is NULL for every row.

**Fan-out**:

```sql
-- Wrong: counts customers once per order
SELECT COUNT(c.customer_id) FROM customer c JOIN orders o ON o.customer_id = c.customer_id;  -- 4
-- Right
SELECT COUNT(DISTINCT c.customer_id) FROM customer c JOIN orders o ON o.customer_id = c.customer_id;  -- 2
```

**Missing `GROUP BY`**: `SELECT region, COUNT(*) FROM customer;` is an error in standard/strict mode (a non-aggregated column next to an aggregate). Older permissive MySQL modes silently picked an arbitrary `region`.

### 5. Under the hood
- **Plan operators**: `COUNT(*)` becomes a stream aggregate (input sorted/grouped) or a hash aggregate (hash table of group → counter). A scalar `COUNT(*)` with no filter can be answered from the *narrowest* index, since only row existence matters; this is why a nonclustered index is often scanned instead of the clustered one.
- **`COUNT(col)`** must read `col` so a covering index is needed to avoid touching the base table; `COUNT(*)` can use any index. They differ in logic, so the "which is faster" question is really "which index is narrower".
- **`COUNT(DISTINCT col)`** requires de-duplication: sort + stream or hash. Cost grows with the number of distinct values, memory grants and possible spills to disk. Engines that rewrite it (a distinct aggregate over a grouped sub-aggregate) may do two aggregation steps. In distributed engines it forces a shuffle on the column.
- **Exact `COUNT(*)` isn't O(1) in MVCC engines**: InnoDB and PostgreSQL must evaluate row visibility per transaction, so each count scans an index or the table (MyISAM stored a row counter because it had no transactions). PostgreSQL index-only scans help only when the visibility map says pages are all-visible. Workarounds: accept an estimate (`pg_class.reltuples`, `sys.dm_db_partition_stats`, `SHOW TABLE STATUS`), maintain a summary/counter table, or a materialized view, or cache the value.
- **Question-bank correction**: `SUM(col)` does **not** treat NULLs as zeros; it skips them. That is why `AVG(col)` (divides by non-NULL count) differs from `SUM(col)/COUNT(*)` (divides by all rows).

### 6. Performance and design implications
- Prefer `COUNT(*)` for "how many rows"; use `COUNT(col)` only when you want to skip NULLs, and say so.
- `COUNT(DISTINCT …)` over huge sets is expensive: pre-aggregate, use a unique index/PK if distinctness is guaranteed (then plain `COUNT(*)` suffices), or use an approximate algorithm (HyperLogLog, `APPROX_COUNT_DISTINCT`) when ±1–2% error is acceptable. State the error to stakeholders.
- Pagination totals (`COUNT(*)` with the same filter as the page) are costly on large tables; compromise with approximate totals, "next page" only, or capped counts (`COUNT(*)` over `LIMIT 1001`).
- Filtered/partial indexes make counts of a small, hot subset (active users) cheap, at the price of extra write maintenance.
- Compare plans, not folklore: `COUNT(1)` vs `COUNT(*)` are identical in modern optimizers.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `COUNT(*)` | Row count | Default |
| `COUNT(col)` / `COUNT(DISTINCT col)` | Non-NULL / distinct non-NULL | Data quality, cardinality |
| `COUNT(*) FILTER (WHERE …)` (PostgreSQL, others) | Conditional count | Pivots, multi-metric queries |
| `COUNT(CASE WHEN … THEN 1 END)` | Portable conditional count | Any engine |
| `APPROX_COUNT_DISTINCT`, `HLL` functions | Approximate distinct | Very large data |
| `pg_class.reltuples`, `sys.dm_db_partition_stats` | Row-count estimates | Fast "about how many" |
| `ONLY_FULL_GROUP_BY` (MySQL) | Strict grouping errors | Keep enabled |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `COUNT(*)` for row counts | Counting a nullable column by habit |
| `COUNT(DISTINCT x)` for unique entities | Joined fan-out you could avoid by counting on the parent table |
| `GROUP BY` + `COUNT(*)` then count groups | You only need the number of groups: `COUNT(DISTINCT)` is simpler |
| Exact count for finance/audit | Dashboards and pagination where an estimate suffices |
| Approximate distinct for analytics | Reconciliation and billing |

### 9. Common mistakes
- **`COUNT(*)` after `LEFT JOIN` for "zero" groups** → returns 1 not 0 → count a column from the optional side.
- **Counting after a one-to-many join** → inflated totals → `COUNT(DISTINCT parent_key)` or aggregate before joining.
- **`COUNT(col)` expecting to include NULLs** → undercounts → use `COUNT(*)`.
- **Assuming `SUM` of empty/all-NULL returns 0** → NULL propagates → `COALESCE(SUM(x),0)`.
- **`AVG` vs `SUM/COUNT(*)`** → different denominators → be explicit about NULL treatment.
- **Counting inside `WHERE`** → aggregates aren't allowed in `WHERE` → use `HAVING` or a subquery.
- **Expecting O(1) exact counts in InnoDB/PostgreSQL** → surprise full scans → estimates, counters or caching.

### 10. 🎯 Interview kit
**30-second answer:** `COUNT(*)` counts rows, `COUNT(col)` counts non-NULL values, `COUNT(DISTINCT col)` counts distinct non-NULL values. All other aggregates ignore NULLs, and `COUNT` returns 0 on empty input where `SUM`/`AVG` return NULL. Distinct counts cost a sort or hash; exact counts on big MVCC tables aren't free.

**Structure a longer answer:** definitions and NULL rules → empty set and group behaviour → join fan-out and outer-join counting → physical plan differences → performance options (indexes, estimates, summary tables, approximate counts).

**Likely follow-ups:**
- *Count NULLs in a column?* → `COUNT(*) - COUNT(col)` or `COUNT(*) WHERE col IS NULL`.
- *COUNT(DISTINCT) vs. GROUP BY?* → Same logical result for a single column; `GROUP BY` returns the groups, `COUNT(DISTINCT)` just the number; plans are often identical.
- *Why is COUNT(*) slow on InnoDB?* → MVCC visibility means no stored row count; scans smallest index; use estimates/counters.
- *Why does AVG(col) differ from SUM(col)/COUNT(*)?* → NULL rows excluded from AVG's denominator.
- *COUNT(1) vs. COUNT(*)?* → Same.

**Signals interviewers look for:** you handle NULLs and empty sets deliberately, spot fan-out, count from the right side of an outer join, and propose pragmatic alternatives for expensive counts.

### 11. Check yourself
- [ ] Predict the three results of `COUNT(*)`, `COUNT(col)`, `COUNT(DISTINCT col)` on a column containing `1, 1, NULL, 2`.
- [ ] Write a query listing every customer with their order count including zero.
- [ ] Explain why `SELECT region, COUNT(*) FROM customer` fails in strict mode.
- [ ] Explain what `COUNT(*)` returns for an empty table vs. what `SUM` returns.
- [ ] Describe three ways to avoid an exact `COUNT(*)` on a 1-billion-row table.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does the query SELECT COUNT(*) FROM table return if the table is empty?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00802_What_does_the_query_SELECT_COUNT_FROM_table_return_if_the_ta.md>)
- [Basic] [When you write SELECT department, COUNT(*) but forget the GROUP BY, what error or behaviour do you expect in a strict SQL mode?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00912_When_you_write_SELECT_department_COUNT_but_forget_the_GROUP_.md>)
- [Intermediate] [What is the difference between counting rows and counting distinct groups produced by GROUP BY?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00903_What_is_the_difference_between_counting_rows_and_counting_di.md>)
- [Deep] [What does SUM return if the table is empty versus what COUNT returns for the same empty table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00813_What_does_SUM_return_if_the_table_is_empty_versus_what_COUNT.md>)
- [Deep] [Can COUNT return zero for a group? Explain why or why not with GROUP BY.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00840_Can_COUNT_return_zero_for_a_group_Explain_why_or_why_not_wit.md>)
- [Deep] [Does COUNT(1) behave differently from COUNT(*) in terms of result or performance?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00856_Does_COUNT_1_behave_differently_from_COUNT_in_terms_of_resul.md>)
- [Deep] [What is the difference between COUNT of a foreign key column and COUNT(*) in a joined query?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00870_What_is_the_difference_between_COUNT_of_a_foreign_key_column.md>)
- [Deep] [How can COUNT combined with DISTINCT on two columns give unique pairs, and is that always supported?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Aggregation/Q-00886_How_can_COUNT_combined_with_DISTINCT_on_two_columns_give_uni.md>)

## Related concepts

- [NULL vs Zero in Aggregates](<../SQL_Functions/NULL_vs_Zero_in_Aggregates.md>) *(SQL Functions)*
- [NULL Handling](<../SQL_Fundamentals/NULL_Handling.md>) *(SQL Fundamentals)*
- [GROUP BY vs PARTITION BY](<../SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>) *(SQL Fundamentals)*

