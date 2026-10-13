---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# FIRST_VALUE and LAST_VALUE

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `FIRST_VALUE` and `LAST_VALUE` return the first or last row's value inside a window *frame* — and because the default frame ends at the current row, `LAST_VALUE` silently returns the current row's value unless you widen the frame.

### 1. The big picture
Aggregates collapse rows; window functions keep every row and attach a computed value to it. `FIRST_VALUE(expr)`, `LAST_VALUE(expr)` and `NTH_VALUE(expr, n)` let each row look at a specific peer in its partition: "this order's amount versus the customer's first order", "the latest status per device on every row". The behaviour is governed by three clauses: `PARTITION BY` (which rows are peers), `ORDER BY` (their sequence), and the **frame** (`ROWS`/`RANGE BETWEEN …`, which peers are visible). Frames also explain running totals and moving averages, so this lesson doubles as the frame primer.

### 2. Core concepts and vocabulary
- **Window** — the set of rows a function sees for the current row.
- **Partition** — rows sharing the `PARTITION BY` value.
- **Frame** — the sub-range of the partition used by frame-sensitive functions; defined by `ROWS`, `RANGE` or `GROUPS` plus start and end bounds (`UNBOUNDED PRECEDING`, `n PRECEDING`, `CURRENT ROW`, `n FOLLOWING`, `UNBOUNDED FOLLOWING`).
- **Default frame** — with `ORDER BY`: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`; without `ORDER BY`: the whole partition.
- **Peer rows** — rows with equal `ORDER BY` values; `RANGE` treats them as one unit, `ROWS` does not.
- **`IGNORE NULLS`** — not implemented in PostgreSQL (as of 16); you emulate it.
- **Frame-insensitive functions** — `ROW_NUMBER`, `RANK`, `LAG`, `LEAD`, `NTILE` ignore the frame; `FIRST_VALUE`, `LAST_VALUE`, `NTH_VALUE`, and aggregates used as windows honour it.

### 3. How it works, step by step
1. Rows are produced by `FROM/WHERE/GROUP BY/HAVING`.
2. Rows are split into partitions and sorted by the window's `ORDER BY`.
3. For each row, the frame is computed relative to it.
4. `FIRST_VALUE` returns the expression from the first row of the frame; `LAST_VALUE` from the last row of the frame; `NTH_VALUE(x, n)` from the n-th.
5. The result is added as a new column; row count is unchanged.

Because step 3's default frame ends at `CURRENT ROW` (plus peers), `LAST_VALUE` returns the current row's own value — the classic surprise.

### 4. Worked example
```sql
-- orders(customer_id, order_id, order_date, amount)
SELECT customer_id, order_id, order_date, amount,
       FIRST_VALUE(amount) OVER w                       AS first_amount,
       LAST_VALUE(amount)  OVER w                       AS wrong_last,   -- = amount!
       LAST_VALUE(amount)  OVER (
         PARTITION BY customer_id ORDER BY order_date, order_id
         ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS right_last,
       SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date, order_id
                         ROWS UNBOUNDED PRECEDING)      AS running_total,
       AVG(amount) OVER (PARTITION BY customer_id ORDER BY order_date, order_id
                         ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3
FROM orders
WINDOW w AS (PARTITION BY customer_id ORDER BY order_date, order_id);
```
Read the result: `wrong_last` equals `amount` on every row; `right_last` is the customer's latest amount on every row. Alternatives that avoid the trap: use the named `WINDOW` clause to reuse definitions, or reverse the order and use `FIRST_VALUE(...) OVER (ORDER BY order_date DESC)`.

Filling forward (last non-null) — emulated since `IGNORE NULLS` is missing; `COUNT(v)` only increments on non-null rows, so each non-null value starts a new group whose first row carries it:
```sql
WITH g AS (SELECT *, COUNT(v) OVER (ORDER BY ts) AS grp FROM readings)
SELECT ts, v, FIRST_VALUE(v) OVER (PARTITION BY grp ORDER BY ts) AS v_filled FROM g;
```

### 5. Under the hood   *(optional at this level)*
- PostgreSQL sorts once per distinct `PARTITION BY/ORDER BY` combination (`WindowAgg` over `Sort`). Windows with identical definitions share one sort, so declare them via a named `WINDOW` clause.
- `RANGE` with an offset (`RANGE BETWEEN 7 PRECEDING AND CURRENT ROW`) requires a single numeric/date/interval `ORDER BY` column and compares *values*; `ROWS` counts *physical positions* in the sort order. `GROUPS` counts peer groups.
- Moving aggregates over sliding frames use an inverse-transition function when available (`SUM`, `COUNT`, `AVG`) so each row is O(1) rather than recomputing the frame.
- An index matching `(partition_col, order_col)` lets the planner skip the sort (an `Index Scan` feeding `WindowAgg`).

### 6. Performance and design implications
- Cost is dominated by the sort per window spec; check `EXPLAIN` for `Sort Method: external merge` (spilling) and raise `work_mem` or pre-filter.
- Filter rows in a CTE/subquery *before* windowing; window functions can't be used in `WHERE`, so top-1-per-group is `WHERE rn = 1` on an outer query.
- `DISTINCT ON (customer_id) … ORDER BY customer_id, order_date DESC` is often faster than `ROW_NUMBER() = 1` for "latest per key".
- Always break `ORDER BY` ties with a unique column, otherwise `FIRST_VALUE`/`ROW_NUMBER` are nondeterministic.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FIRST_VALUE(x) OVER (…)` | first frame value | compare to baseline |
| `LAST_VALUE(x) OVER (… ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)` | last in partition | latest value on each row |
| `NTH_VALUE(x, n)` | n-th frame value | second-highest etc. |
| `LAG(x, n, default)` / `LEAD` | previous / next row | deltas |
| `WINDOW w AS (…)` | named reusable spec | multiple functions, one sort |
| `STRING_AGG(x, ',' ORDER BY y)` / `ARRAY_AGG` | ordered aggregates | lists per group |
| `generate_series(a, b, step)` | row generator | calendar spine, gap fill |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `FIRST_VALUE` with explicit frame | relying on the default frame for `LAST_VALUE` |
| Window function | a simple aggregate + join would do and be cheaper |
| `DISTINCT ON` | portability to other databases is required |
| `ROWS` frame | values can tie and you want peers grouped (`RANGE`) |

### 9. Common mistakes
- `LAST_VALUE` returns the current row → default frame ends at current row → specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.
- Running total with default `RANGE` frame → tied dates share the same total → use `ROWS` with a unique tiebreaker.
- Window function in `WHERE` → error → subquery/CTE.
- Assuming `IGNORE NULLS` exists → syntax error → the count-groups trick above.
- `||` with `NULL` → whole string `NULL` → `CONCAT()` or `COALESCE`; (`CONCAT` ignores `NULL`s).

### 10. 🎯 Interview kit
**30-second answer:** "`FIRST_VALUE`/`LAST_VALUE` return a value from the first/last row of the window frame. The default frame stops at the current row, so for `LAST_VALUE` you must extend it to `UNBOUNDED FOLLOWING`."
**Structure a longer answer:** partition/order/frame → default frame rule → example → cost (sort) → alternatives (`DISTINCT ON`, self-join).
**Likely follow-ups:** *`ROWS` vs `RANGE`?* → positions vs peer values. *Running total of ties?* → use `ROWS`. *Last non-null value?* → grouping trick, no `IGNORE NULLS`. *Why is the window slow?* → sort spill.
**Signals interviewers look for:** you mention the frame immediately, break ties deterministically, and know the planner shares sorts for equal windows.

### 11. Check yourself
- [ ] Explain why `LAST_VALUE(x) OVER (ORDER BY d)` equals `x` on every row.
- [ ] Write a 7-day moving average with a `RANGE` interval frame and with a `ROWS` frame; explain the difference with missing days.
- [ ] Forward-fill a nullable sensor column.
- [ ] Rewrite "latest row per customer" with `DISTINCT ON` and compare plans.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you use the IN operator to match a column against a list of values?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36138_How_do_you_use_the_IN_operator_to_match_a_column_against_a_l.md>)
- [Basic] [How do you remove leading and trailing spaces from a text value using TRIM?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36154_How_do_you_remove_leading_and_trailing_spaces_from_a_text_va.md>)
- [Basic] [How do you concatenate two text columns, and what is the difference between the || operator and CONCAT()?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35835_How_do_you_concatenate_two_text_columns_and_what_is_the_diff.md>)
- [Intermediate] [What does the BETWEEN operator include at its boundaries and how would you filter salaries between 30000 and 50000?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36190_What_does_the_BETWEEN_operator_include_at_its_boundaries_and.md>)
- [Intermediate] [How do you use STRING_AGG to combine grouped rows into a single comma-separated string?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35848_How_do_you_use_STRING_AGG_to_combine_grouped_rows_into_a_sin.md>)
- [Intermediate] [How would you query a value nested inside a JSONB column?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36307_How_would_you_query_a_value_nested_inside_a_JSONB_column.md>)
- [Intermediate] [Your API returns JSON aggregated from many rows and building it in Node is slow; when would you use jsonb_build_object/json_agg to shape the response in Postgres and what are the risks?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36020_Your_API_returns_JSON_aggregated_from_many_rows_and_building.md>)
- [Deep] [What is the NTILE window function and how would you use it to split users into quartiles by spend?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35851_What_is_the_NTILE_window_function_and_how_would_you_use_it_t.md>)

## Related concepts

- [UDF Performance](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [NULL vs Zero in Aggregates](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>) *(SQL Functions)*
- [Window Functions Overview](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>) *(SQL Functions)*

