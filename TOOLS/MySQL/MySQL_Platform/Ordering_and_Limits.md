---
id: "TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Ordering and Limits

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A SQL result has *no* order unless you write `ORDER BY`, `LIMIT` just cuts the first N rows of whatever order you gave it, and stable paging and "top N with ties" need a deterministic tie-breaker, keyset seeks, or `DENSE_RANK()` rather than `LIMIT 1`.

### 1. The big picture
Relational tables are sets: the standard guarantees no row order. MySQL often *appears* to return rows in primary-key or insertion order — because that is the order of the index it scans — but the optimizer can switch to a different index, a temporary table, a parallel path or a hash join and the order changes, sometimes only after an upgrade or when the data grows. Every feature that depends on order (top-N lists, pagination, "latest row per user", median) must therefore say so explicitly and make the ordering *total* (no ties left ambiguous).

This lesson ties together `ORDER BY`, `LIMIT/OFFSET`, `GROUP BY` and its ordering, `ONLY_FULL_GROUP_BY`, window ranking functions, and median/percentile tricks in MySQL (which has no `PERCENTILE_CONT`).

### 2. Core concepts and vocabulary
- **`ORDER BY col [ASC|DESC], col2 …`** — sort; default direction is `ASC`; ties are broken by the next expression.
- **`LIMIT n [OFFSET m]`** (`LIMIT m, n`) — return at most n rows after skipping m.
- **Deterministic ordering** — the `ORDER BY` list ends in a unique column so each row has one position.
- **NULL ordering** — in MySQL `NULL` sorts as the *lowest* value: first in `ASC`, last in `DESC`.
- **Keyset (seek) pagination** — paging with `WHERE (sort_col, id) < (?, ?)` instead of `OFFSET`.
- **Logical query order** — `FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT`.
- **`ONLY_FULL_GROUP_BY`** — mode forbidding select-list columns that are neither grouped nor functionally dependent on the group.
- **Ranking functions** — `ROW_NUMBER()` (unique), `RANK()` (ties share rank, gaps), `DENSE_RANK()` (ties share rank, no gaps).
- **Median** — middle value (average of two middle values when the count is even).

### 3. How it works, step by step
1. **`ORDER BY` execution.** If an index delivers rows already in the requested order (and the plan chooses to read it), MySQL streams them. Otherwise it collects rows into a sort buffer and does a **filesort** (in memory, spilling to temp files when larger than `sort_buffer_size`).
2. **`ORDER BY … LIMIT n`** lets MySQL keep only the top n in a priority queue while scanning — much cheaper than a full sort — and, with a matching index, stop after n index entries.
3. **`OFFSET m`** still has to produce and discard m rows; cost grows linearly with page number.
4. **`GROUP BY`** produces groups via an index scan, a sort or a hash/temporary table. Since 8.0 it no longer sorts implicitly: add `ORDER BY` if you need order.
5. **Ties.** `ORDER BY score DESC LIMIT 3` with scores 90, 80, 80, 80, 70 returns one arbitrary choice among the three 80s on each run unless you add a tie-breaker.

### 4. Worked example
```sql
-- Default direction is ASC; multi-key sort; NULLs first in ASC
SELECT name, department, salary
FROM   employee
ORDER  BY department ASC, salary DESC, employee_id ASC;   -- employee_id: tie-breaker

-- Top 5 highest salaries (arbitrary among ties at the 5th place)
SELECT * FROM employee ORDER BY salary DESC LIMIT 5;

-- All employees tied for the top salary (LIMIT 1 would return only one)
SELECT *
FROM (
  SELECT e.*, DENSE_RANK() OVER (ORDER BY salary DESC) AS r
  FROM   employee e
) x
WHERE r = 1;
-- or without windows:
SELECT * FROM employee WHERE salary = (SELECT MAX(salary) FROM employee);

-- Count orders per status, deterministic output
SELECT status, COUNT(*) AS n
FROM   orders
GROUP  BY status
ORDER  BY n DESC, status;

-- Stable OFFSET paging: unique tie-breaker
SELECT id, title, created_at
FROM   post
ORDER  BY created_at DESC, id DESC
LIMIT  20 OFFSET 40;

-- Keyset paging: pass the last row's (created_at, id) from the previous page
SELECT id, title, created_at
FROM   post
WHERE  (created_at, id) < ('2026-10-12 18:30:00', 90210)
ORDER  BY created_at DESC, id DESC
LIMIT  20;
```
**Median** (MySQL 8.0, no `PERCENTILE_CONT`):
```sql
SELECT AVG(val) AS median
FROM (
  SELECT val,
         ROW_NUMBER() OVER (ORDER BY val) AS rn,
         COUNT(*)     OVER ()             AS cnt
  FROM   measurement
  WHERE  val IS NOT NULL
) t
WHERE rn IN (FLOOR((cnt + 1) / 2), CEIL((cnt + 1) / 2));
```
For an odd count both expressions give the middle row; for an even count they give the two middle rows and `AVG` combines them.

**Why repeated calls to a paged API returned different orders:** the query had no `ORDER BY` (or ordered by a non-unique column such as `created_at`). Rows with equal `created_at` can come back in different relative order on each execution, and a row can appear on two pages or none. Fix: total ordering `ORDER BY created_at DESC, id DESC`; for concurrent inserts, use keyset paging (seek from the last seen key) so new rows don't shift positions; for a snapshot view across pages, include a fixed upper bound (`id <= :max_id_at_first_request`).

**`ONLY_FULL_GROUP_BY` error 1055:**
```sql
SELECT customer_id, name, MAX(created_at) FROM orders GROUP BY customer_id;  -- name not grouped
-- Fix options:
SELECT customer_id, ANY_VALUE(name), MAX(created_at) FROM orders GROUP BY customer_id;
SELECT o.customer_id, c.name, o.last_order
FROM (SELECT customer_id, MAX(created_at) AS last_order FROM orders GROUP BY customer_id) o
JOIN customer c USING (customer_id);
```
MySQL accepts a non-grouped column if it is functionally dependent on the grouped columns (e.g., grouping by the primary key).

### 5. Under the hood
- **Filesort variants.** MySQL sorts either (a) full rows ("single-pass") or (b) sort key + row id then re-reads rows ("two-pass"/row-id). `max_length_for_sort_data` and column sizes influence which. Sorting wide `TEXT`/`VARCHAR(4000)` columns inflates the buffer; select fewer columns or sort ids first then fetch.
- **Top-N optimization.** `ORDER BY … LIMIT n` uses a bounded priority queue when n rows fit in `sort_buffer_size`, avoiding a complete sort.
- **Index order.** `INDEX (user_id, created_at)` serves `WHERE user_id = ? ORDER BY created_at DESC` with a backward index scan (descending indexes are natively supported in 8.0). Mixed directions (`a ASC, b DESC`) need a matching index definition (`(a ASC, b DESC)`) in 8.0 to avoid sorting.
- **Grouping and order.** In 5.7 `GROUP BY` implied sort and `GROUP BY a DESC`; both removed in 8.0, so queries that "happened to" return grouped keys sorted now return arbitrary order.
- **Collation.** Text sort order comes from the collation (`'a' < 'B'` under `_ci`, binary collations sort uppercase before lowercase). Same data, different collation → different `ORDER BY`.
- **Ties in windows.** `ROW_NUMBER()` with a non-unique `ORDER BY` assigns numbers to ties arbitrarily; add a unique tie-breaker if the number is exposed.

### 6. Performance and design implications
- **Match the index to the sort:** equality columns, then the `ORDER BY` columns (see Optimizer Behavior). `EXPLAIN` shows `Using filesort` when it must sort.
- **Deep `OFFSET` is O(offset):** `LIMIT 20 OFFSET 1000000` reads a million rows. Use keyset pagination, or a "deferred join": `SELECT p.* FROM post p JOIN (SELECT id FROM post ORDER BY created_at DESC, id DESC LIMIT 20 OFFSET 1000000) x USING (id)` which scans only index entries for the skipped part.
- **Total counts:** `COUNT(*)` over the filter is a second query (or `COUNT(*) OVER ()` in 8.0); on huge tables use approximate or cached counts.
- **Sorting in the app vs DB:** sort in SQL when you also `LIMIT`; sorting a full large result in the app wastes network and memory.
- **Don't sort inside views/CTEs** and expect the outer query to preserve it — only the outermost `ORDER BY` is guaranteed.
- **Window ranking** requires a sort per partition; an index on `(partition_cols, order_cols)` can eliminate it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ORDER BY a, b DESC` | Sort with tie-breakers | Any ordered output |
| `LIMIT n OFFSET m` | Cut rows | Small pages, top-N |
| `ROW_NUMBER() / RANK() / DENSE_RANK() OVER (…)` | Rank rows | Top-N per group, ties |
| `COUNT(*) OVER ()` | Total in same pass | Paged APIs |
| `ANY_VALUE(col)` | Pick any value in group | Satisfy `ONLY_FULL_GROUP_BY` knowingly |
| `sort_buffer_size`, `max_length_for_sort_data` | Sort memory/strategy | Large sorts (session-level only) |
| `EXPLAIN` / `EXPLAIN ANALYZE` | See `Using filesort`, row counts | Verify index ordering |
| `ORDER BY FIELD(col,'a','b')` | Custom order list | Small enumerations (can't use index) |
| `sql_mode='ONLY_FULL_GROUP_BY'` | Strict grouping | Keep ON |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ORDER BY … LIMIT n` + matching index | Sort key isn't indexable (expression, mixed directions pre-8.0) |
| `OFFSET` paging | Deep pages or concurrent inserts (duplicates/skips) |
| Keyset paging | Users need "jump to page 37" |
| `DENSE_RANK() = 1` | Old 5.7 servers (no windows) — use `= (SELECT MAX…)` |
| `ANY_VALUE` | The column really does differ within the group (you'll get an arbitrary value) |
| Window `ROW_NUMBER` for median | Very large tables — consider sampling or pre-aggregated stats |

### 9. Common mistakes
- **No `ORDER BY` on paginated queries** → unstable pages → total ordering.
- **Ordering by a non-unique column only** → duplicates/skips across pages → add `id`.
- **Relying on `GROUP BY` to sort (5.7 habit)** → random order in 8.0 → explicit `ORDER BY`.
- **`LIMIT 1` to get "the max row" when ties matter** → hides ties → `DENSE_RANK` or subquery.
- **Large `OFFSET`** → slow and growing latency → keyset.
- **Selecting non-grouped columns** → error 1055 or arbitrary values → group properly/`ANY_VALUE` consciously.
- **Assuming `NULL`s sort last** → they sort first in `ASC` → `ORDER BY col IS NULL, col` to control.
- **Sorting by a function of a column** → filesort → generated/indexed column.
- **Using `SQL_CALC_FOUND_ROWS`** → deprecated → separate count.

### 10. 🎯 Interview kit
**30-second answer:** Without `ORDER BY` row order is undefined, so always specify it, ending with a unique column. `LIMIT` takes the first N after sorting; `OFFSET` gets slower with depth, so use keyset pagination for large or live data. For top values with ties use `DENSE_RANK()`, and for median use `ROW_NUMBER()`/`COUNT()` windows.

**Structure a longer answer:** statement semantics → determinism and ties → index/filesort interaction → pagination strategy → edge cases (NULLs, collations, 8.0 `GROUP BY` change).

**Likely follow-ups:**
- *Logical order of clauses?* → FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY, LIMIT.
- *Why do aliases work in `ORDER BY` but not `WHERE`?* → `SELECT` is evaluated before `ORDER BY`, after `WHERE`.
- *What does `ONLY_FULL_GROUP_BY` guard against?* → nondeterministic values for non-grouped columns.
- *How do you page consistently while rows are inserted?* → keyset + upper-bound snapshot.
- *`RANK` vs `DENSE_RANK`?* → gaps after ties vs none.
- *Why is `ORDER BY RAND() LIMIT 1` bad?* → sorts the whole table; pick random id ranges instead.

**Signals interviewers look for:** determinism, awareness of pagination cost, correct use of windows, and understanding of 8.0 behavioural changes.

### 11. Check yourself
- [ ] Explain why a query without `ORDER BY` can return different orders on different days.
- [ ] Write the keyset query for "next page" ordered by `(created_at DESC, id DESC)`.
- [ ] Return all rows tied for first place with and without window functions.
- [ ] Compute a median in MySQL 8.0.
- [ ] Fix an `ONLY_FULL_GROUP_BY` error in two ways.
- [ ] Explain what `Using filesort` means and how an index removes it.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [How do you sort query results and what is the default sort direction of ORDER BY?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33342_How_do_you_sort_query_results_and_what_is_the_default_sort_d.md>)
- [Basic] [How do you return only the first N rows of a result and what does LIMIT do?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33343_How_do_you_return_only_the_first_N_rows_of_a_result_and_what.md>)
- [Basic] [How do you sort a result first by one column ascending and then by another descending?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33377_How_do_you_sort_a_result_first_by_one_column_ascending_and_t.md>)
- [Basic] [How would you count the number of orders grouped by status in an orders table?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33398_How_would_you_count_the_number_of_orders_grouped_by_status_i.md>)
- [Basic] [What does the ORDER BY ... LIMIT combination return and how does adding DESC change it?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33013_What_does_the_ORDER_BY_LIMIT_combination_return_and_how_does.md>)
- [Intermediate] [A query returns different row orders on repeated runs because there's no ORDER BY on a GROUP BY — how do you explain and guarantee deterministic ordering?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33532_A_query_returns_different_row_orders_on_repeated_runs_becaus.md>)
- [Deep] [How do you use DENSE_RANK to fetch all rows tied for the top value, unlike LIMIT 1?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33033_How_do_you_use_DENSE_RANK_to_fetch_all_rows_tied_for_the_top.md>)
- [Deep] [How would you find the median value of a column in MySQL?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33555_How_would_you_find_the_median_value_of_a_column_in_MySQL.md>)

## Related concepts

- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [GROUP BY and HAVING](<../../../01_Databases_and_SQL/SQL_Language/Aggregation/GROUP_BY_and_HAVING.md>) *(Aggregation)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Set operators
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation (cont.)
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Development standards › 2.6 #5 Using ordinal column position
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Sorting Data

