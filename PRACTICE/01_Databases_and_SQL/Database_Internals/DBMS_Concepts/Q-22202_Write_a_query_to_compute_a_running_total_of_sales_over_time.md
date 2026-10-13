---
type: "practice"
difficulty: "Deep"
---

# Write a query to compute a running total of sales over time.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Given a table of sales with a date/time and an amount, produce a running total — a cumulative sum where each row shows the sum of all sales up to and including that row's date. The interviewer wants a window function solution using SUM(...) OVER (ORDER BY ...).

What the interviewer is probing for:

Do you know that SUM() OVER (ORDER BY date) creates a cumulative aggregate, not a single grand total?
Do you understand the window frame — specifically that ORDER BY implies a default frame of RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW, and the crucial gotcha this default causes with duplicate/tied dates?
Can you extend it to a per-group running total (e.g., per customer/region) with PARTITION BY?

Clarifying questions I'd ask:

Is the data already one row per day, or multiple sales per day? If multiple, do you want the running total at the row grain or at the daily grain (aggregate per day first, then accumulate)?
Running total globally, or restarted per customer / region / month?
How to handle ties on the ordering column — this changes RANGE vs ROWS behavior (the classic trap).

Plan: order by date, use SUM(amount) OVER (ORDER BY date ...), and explicitly control the frame with ROWS to avoid the duplicate-date surprise. Then show the per-group and per-day-aggregated variants.

From first principles to the solution

A running total is a cumulative window aggregate. Unlike GROUP BY SUM (which collapses rows into one total), a windowed SUM ... OVER (ORDER BY date) keeps every row and, for each row, sums all rows from the start of the window up to the current row.

The minimal query:

SQL
SELECT
  sale_date,
  amount,
  SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales
ORDER BY sale_date;

The frame — the part that separates a strong answer from a shallow one. Every ordered window has a frame that defines which rows are included for the current row. When you specify ORDER BY but no explicit frame, most databases default to:

CODE
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

RANGE treats all rows with the same ORDER BY value (peers) as one unit. So if three sales share the same sale_date, the default RANGE frame includes all three in the running total for each of those three rows — they all show the same cumulative value (the total through that date), not an incremental one. That is often what you want at the daily grain, but it surprises people expecting a strict row-by-row accumulation.

To get a strict, row-by-row cumulative sum (each row adds exactly its own amount), specify ROWS:

SQL
SUM(amount) OVER (
  ORDER BY sale_date, sale_id
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total

ROWS counts physical rows, so ties no longer collapse — but you must add a deterministic tiebreaker (sale_id) to the ORDER BY, otherwise the per-row cumulative value is non-deterministic across executions.

Best practice: if there can be multiple sales per day and you want a clean daily curve, aggregate to one row per day first, then accumulate:

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

This removes the tie ambiguity entirely (one row per date) and is the version most analytics dashboards use.

Per-group running total (e.g., cumulative sales per customer) adds PARTITION BY:

SQL
SUM(amount) OVER (
  PARTITION BY customer_id
  ORDER BY sale_date
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)

PARTITION BY resets the running total at each new customer.

Common variations the interviewer may push on:

Month-to-date reset: PARTITION BY DATE_TRUNC('month', sale_date) ORDER BY sale_date.
Moving/rolling total (last 7 rows): ROWS BETWEEN 6 PRECEDING AND CURRENT ROW.
Without window functions (legacy MySQL <8, older systems): a correlated subquery SELECT SUM(s2.amount) FROM sales s2 WHERE s2.sale_date <= s1.sale_date — correct but O(n²), which is exactly why window functions exist.
Summary / what a strong answer emphasizes
The core is SUM(amount) OVER (ORDER BY sale_date), which turns a plain sum into a cumulative one.
Explain the frame: default is RANGE ... UNBOUNDED PRECEDING TO CURRENT ROW; use ROWS + a tiebreaker for a strict row-by-row running total, or pre-aggregate per day to sidestep ties.
Use PARTITION BY to reset per customer/region/month.
Note performance: window function is a single ordered pass (O(n log n) for the sort); the correlated-subquery alternative is O(n²) and should be avoided at scale. An index on the ORDER BY/PARTITION BY columns helps.

#### Common mistakes
forgetting the frame and being confused when tied dates share a value; omitting a tiebreaker (non-deterministic output); using a correlated subquery on millions of rows; or accidentally writing GROUP BY and destroying the row-level detail. Follow-up probes: "make it reset every month," "turn it into a trailing 30-day total," and "explain why RANGE and ROWS differ on your data."

**Comparison:**

```
Aspect	SUM() OVER (ORDER BY ...) window	Correlated subquery
Keeps row detail	Yes	Yes
Complexity	~O(n log n) (one ordered pass)	O(n²)
Tie handling	Frame-controlled (RANGE vs ROWS)	Depends on <=/< and tiebreaker
Per-group reset	PARTITION BY	Extra correlation predicate
DB support	Standard SQL, MySQL 8+, PG, SQL Server, Oracle	All versions
Frame clause	Meaning for running total
RANGE UNBOUNDED PRECEDING → CURRENT ROW (default with ORDER BY)	Ties (peers) collapse; all show total through that value
ROWS UNBOUNDED PRECEDING → CURRENT ROW	Strict physical row-by-row cumulative (needs tiebreaker)
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW	Trailing 7-row moving total, not a full running total
```

**Code:**

```

-- 1) Minimal running total (default RANGE frame) SELECT sale_date, amount, SUM(amount) OVER (ORDER BY sale_date) AS running_total FROM sales ORDER BY sale_date;

CODE
Copy

```sql
-- 2) Strict row-by-row running total (deterministic, ROWS frame + tiebreaker)
SELECT
  sale_id,
  sale_date,
  amount,
  SUM(amount) OVER (
    ORDER BY sale_date, sale_id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM sales
ORDER BY sale_date, sale_id;
SQL
Copy
-- 3) Clean daily curve: aggregate per day, then accumulate
WITH daily AS (
  SELECT sale_date, SUM(amount) AS day_total
  FROM sales
  GROUP BY sale_date
)
SELECT sale_date, day_total,
       SUM(day_total) OVER (ORDER BY sale_date) AS running_total
FROM daily
ORDER BY sale_date;
SQL
Copy
-- 4) Per-customer running total (partition resets it)
SELECT customer_id, sale_date, amount,
       SUM(amount) OVER (
         PARTITION BY customer_id
         ORDER BY sale_date
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS cust_running_total
FROM sales
ORDER BY customer_id, sale_date;
```

**Example input / output:**

```

Query (1) with default RANGE frame — note the tied date 2026-01-02:

CODE
Copy
sale_date   amount  running_total
2026-01-01   100     100
2026-01-02    50     200    <- both tied rows show the total THROUGH 2026-01-02...
2026-01-02    50     200    <- ...i.e. 100+50+50 = 200 (RANGE treats peers as one unit)
2026-01-03   200     400

Query (2) with ROWS frame + tiebreaker — strict per-row accumulation:

CODE
Copy
sale_id sale_date   amount  running_total
1       2026-01-01   100     100
2       2026-01-02    50     150    <- adds only its own 50
3       2026-01-02    50     200
4       2026-01-03   200     400

Query (4) per-customer:

CODE
Copy
customer_id sale_date   amount  cust_running_total
C1          2026-01-01   100     100
C1          2026-01-02    50     150
C1          2026-01-03   200     350
C2          2026-01-02    50      50   <- resets for new customer
```

**Data:**

```text
sales
+---------+------------+-------------+--------+
| sale_id | sale_date  | customer_id | amount |
+---------+------------+-------------+--------+
| 1       | 2026-01-01 | C1          |   100  |
| 2       | 2026-01-02 | C1          |    50  |
| 3       | 2026-01-02 | C2          |    50  |   <- tie on 2026-01-02
| 4       | 2026-01-03 | C1          |   200  |
+---------+------------+-------------+--------+
```

**Flow:**

1. sales rows
2. ORDER BY sale_dateoptional PARTITION BYcustomer
3. Define frame:UNBOUNDED PRECEDING ->CURRENT ROW
4. SUM amount over frame
5. running_total per row

