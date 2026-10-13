---
type: "practice"
difficulty: "Deep"
---

# Write a query to calculate a running total of sales ordered by date.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Given a table of sales (a date and an amount), produce, for each row, the cumulative sum of sales from the earliest date up to and including that row's date, ordered by date. A "running total" (a.k.a. cumulative sum) is the classic use case for a window function with an OVER (ORDER BY ...) clause.

What the interviewer is probing: Do you reach for a window function (the correct, O(n) modern answer) rather than a correlated subquery or a self-join (O(n²))? Do you understand window frames — specifically the difference between ROWS and RANGE, and what the default frame is? Do you handle ties on the same date correctly? Can you talk about NULLs, determinism, and per-group running totals (PARTITION BY)?

Clarifying questions I'd ask:

What's the grain of the table — one row per order, or already aggregated to one row per day?
If two sales share the same date, should the running total on those rows be equal (both include each other) or reflect a strict row order? This decides RANGE vs ROWS.
Do we need a running total per customer/region (PARTITION BY) or a single global series?
Which engine? Window functions exist in PostgreSQL, MySQL 8+, SQL Server, Oracle, BigQuery, Snowflake — but MySQL ≤5.7 does not, which changes the answer.

I'll assume Postgres/MySQL-8/standard SQL, a table sales(sale_date DATE, amount NUMERIC), possibly multiple rows per day, and a single global running total unless a partition is requested.

From first principles to the solution

A running total at row r is SUM(amount) over all rows whose ordering key is ≤ the ordering key of r. The naive way to express "all rows up to here" is a correlated subquery:

SQL
SELECT s.sale_date, s.amount,
       (SELECT SUM(x.amount) FROM sales x WHERE x.sale_date <= s.sale_date) AS running_total
FROM sales s
ORDER BY s.sale_date;

This is correct for a daily grain but is O(n²): for each of n rows it re-scans up to n rows. On a 10M-row sales table it's unusable. It also silently produces the RANGE semantics (all rows on the same date get the same total).

The right tool is a window function. The clause SUM(amount) OVER (ORDER BY sale_date) tells the engine: sort by sale_date, then for each row compute the sum over a frame of rows. Modern engines evaluate this in a single ordered pass — effectively O(n) after the sort.

SQL
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales
ORDER BY sale_date;
The frame subtlety — the part that separates a good answer from a great one

When you write OVER (ORDER BY sale_date) with no explicit frame, the SQL standard default frame is:

CODE
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

With RANGE, "current row" means all rows with the same ORDER BY value as the current row. So if three sales happen on 2026-03-01, all three rows get the same running total — the total including all three. That is usually the desired behaviour for a daily cumulative sales figure.

If instead you want a strict row-by-row accumulation (each duplicate-date row adds incrementally), specify a ROWS frame:

SQL
SELECT sale_date, amount,
       SUM(amount) OVER (
         ORDER BY sale_date
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM sales;

ROWS counts physical rows, so ties are broken by the row's position in the sort. But ties then make the result non-deterministic unless you add a tiebreaker to ORDER BY (e.g. ORDER BY sale_date, order_id). Always give window functions a deterministic sort key when you use ROWS.

Rule of thumb I state in the interview: default is RANGE; use ROWS + a unique tiebreaker when you want a strict, deterministic per-row cumulative sum. Also note ROWS is generally faster/less memory than RANGE because it doesn't need to look ahead for peer rows.

Per-group running total

If they want a running total per customer (reset for each customer), add PARTITION BY:

SQL
SELECT customer_id, sale_date, amount,
       SUM(amount) OVER (
         PARTITION BY customer_id
         ORDER BY sale_date
       ) AS running_total_per_customer
FROM sales;

The window resets at each partition boundary — customer A's total never bleeds into customer B's.

If the data isn't pre-aggregated and you want one row per day

Combine aggregation with the window in two stages using a CTE:

SQL
WITH daily AS (
  SELECT sale_date, SUM(amount) AS day_total
  FROM sales
  GROUP BY sale_date
)
SELECT sale_date, day_total,
       SUM(day_total) OVER (ORDER BY sale_date) AS running_total
FROM daily
ORDER BY sale_date;

You cannot nest SUM(SUM(amount)) OVER (...) conceptually cleanly, so the CTE (or subquery) that first collapses to daily totals, then applies the window, is the clean pattern. (Some engines do allow SUM(SUM(amount)) OVER (ORDER BY sale_date) with a GROUP BY sale_date in the same query because the window runs after the GROUP BY — worth mentioning, but the CTE is clearer.)

Legacy engine without window functions (MySQL ≤5.7)

If constrained to old MySQL, use a session variable:

SQL
SET @rt := 0;
SELECT sale_date, amount, (@rt := @rt + amount) AS running_total
FROM (SELECT sale_date, amount FROM sales ORDER BY sale_date) t;

This works but is fragile: it relies on evaluation order the standard doesn't guarantee, breaks with duplicate dates' semantics, and can't easily PARTITION. I'd flag it as a workaround and recommend upgrading to MySQL 8.

NULLs and correctness details
SUM ignores NULL amounts, so NULL rows don't corrupt the total but also don't advance it. If NULL should count as 0, wrap with COALESCE(amount, 0).
The outer ORDER BY in the final SELECT is separate from the window's ORDER BY; include it so the displayed result is sorted (window ordering does not guarantee output ordering).
Performance: an index on sale_date (or (partition_col, sale_date)) lets the planner satisfy the window's sort without an extra sort step.
Summary — what a strong answer emphasizes

Reach immediately for SUM(amount) OVER (ORDER BY sale_date). Explicitly discuss the default RANGE ... CURRENT ROW frame and when to switch to ROWS with a deterministic tiebreaker; add PARTITION BY for per-group totals; use a CTE to aggregate-then-window for a daily series; and know the O(n) window vs O(n²) correlated-subquery trade-off. Mention NULL handling and the MySQL-5.7 session-variable fallback as edge cases. Common mistakes: forgetting the frame semantics (surprised by tied dates), omitting a tiebreaker with ROWS (non-deterministic), using a correlated subquery on a large table, and confusing the window ORDER BY with the output ORDER BY.

**Comparison:**

```
Approach	Complexity	Handles duplicate dates	Deterministic	Notes
SUM() OVER (ORDER BY date) (RANGE default)	~O(n) after sort	Tied rows share the running total	Yes	Best for daily cumulative figure
SUM() OVER (... ROWS ... CURRENT ROW)	~O(n) after sort	Strict per-row accumulation	Only with unique tiebreaker in ORDER BY	Faster/less memory than RANGE
PARTITION BY grp ORDER BY date	~O(n) after sort	Per group	Yes	Resets total at each group
Correlated subquery SUM(...) WHERE date <= d	O(n²)	RANGE-like	Yes	Fine on tiny/daily data, unusable at scale
Session variable (@rt := @rt + amount)	O(n)	Fragile with ties	No (relies on eval order)	Only for MySQL ≤5.7
```

**Code:**

```sql
-- Canonical answer: window function, default RANGE frame
SELECT sale_date,
       amount,
       SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales
ORDER BY sale_date;

-- Strict per-row cumulative sum (deterministic): ROWS frame + tiebreaker
SELECT sale_date,
       amount,
       SUM(amount) OVER (
         ORDER BY sale_date, order_id
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM sales;

-- Running total per customer
SELECT customer_id, sale_date, amount,
       SUM(amount) OVER (PARTITION BY customer_id ORDER BY sale_date) AS running_total
FROM sales;

-- One row per day, then running total
WITH daily AS (
  SELECT sale_date, SUM(amount) AS day_total
  FROM sales
  GROUP BY sale_date
)
SELECT sale_date, day_total,
       SUM(day_total) OVER (ORDER BY sale_date) AS running_total
FROM daily
ORDER BY sale_date;
```

**Example input / output:**

```

Input rows (order_id, sale_date, amount):

CODE
Copy
1  2026-03-01  100
2  2026-03-01   50
3  2026-03-02  200
4  2026-03-03   25

Output with RANGE default (SUM(amount) OVER (ORDER BY sale_date)) — tied dates share the total:

CODE
Copy
sale_date   amount  running_total
2026-03-01   100     150   <- both 2026-03-01 rows show 150
2026-03-01    50     150
2026-03-02   200     350
2026-03-03    25     375

Output with ROWS ... CURRENT ROW + tiebreaker ORDER BY sale_date, order_id — strict per-row:

CODE
Copy
sale_date   amount  running_total
2026-03-01   100     100
2026-03-01    50     150
2026-03-02   200     350
2026-03-03    25     375
```

**Data:**

```sql
-- Table + sample rows (note the two sales on 2026-03-01 to expose RANGE vs ROWS)
CREATE TABLE sales (
  order_id  INT PRIMARY KEY,
  sale_date DATE    NOT NULL,
  amount    NUMERIC(10,2) NOT NULL
);

INSERT INTO sales (order_id, sale_date, amount) VALUES
  (1, '2026-03-01', 100),
  (2, '2026-03-01', 50),
  (3, '2026-03-02', 200),
  (4, '2026-03-03', 25);
```

