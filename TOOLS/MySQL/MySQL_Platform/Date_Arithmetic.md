---
id: "TOOLS/MySQL/MySQL_Platform/Date_Arithmetic"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Date Arithmetic

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Pick the right temporal and numeric types (`DATE`/`DATETIME`/`TIMESTAMP`, `DECIMAL`, `INT`/`BIGINT`), do date math with `INTERVAL`, `DATEDIFF` and `TIMESTAMPDIFF`, and store instants in UTC — most "wrong total" and "wrong day" bugs come from type or time-zone choices, not from SQL syntax.

### 1. The big picture
Reports, billing and analytics are dominated by two kinds of arithmetic: arithmetic *on time* (how long between two events, what happened in the last 7 days, revenue per local day) and arithmetic *on numbers that must be exact* (money, quantities, counters). MySQL gives you several types for each, and they behave differently:

- Time: `DATE` (calendar day), `DATETIME` (calendar date + clock time, no zone), `TIMESTAMP` (an instant stored as UTC and converted to the session time zone), `TIME`, `YEAR`.
- Numbers: exact (`DECIMAL`, integer types) vs approximate (`FLOAT`, `DOUBLE`).

Choosing correctly up front is far cheaper than reconciling a report that disagrees with the app by a few cents or by a day. This lesson connects those type choices to the date functions and window-function patterns you use daily (`LAG`/`LEAD`, running totals, date series).

### 2. Core concepts and vocabulary
- **`DATE`** — `YYYY-MM-DD`, 3 bytes. **`DATETIME`** — date and time, 5 bytes + fractional (0–3 bytes for precision 0–6), range 1000-01-01 to 9999-12-31, **no time-zone conversion**.
- **`TIMESTAMP`** — 4 bytes + fractional, range 1970-01-01 00:00:01 UTC to 2038-01-19 03:14:07 UTC; stored as UTC, converted to/from `time_zone` of the session on write/read.
- **Fractional seconds** — `DATETIME(6)` stores microseconds; default precision is 0 (whole seconds).
- **`DECIMAL(M,D)`** — exact fixed-point: M total digits, D digits after the point. `DECIMAL(10,2)` max 99,999,999.99.
- **`FLOAT`/`DOUBLE`** — binary floating point; fast, approximate; `0.1 + 0.2 <> 0.3`.
- **`INTERVAL`** — a duration literal: `INTERVAL 7 DAY`, `INTERVAL 1 MONTH`.
- **UTC** — the zone-free reference; store instants in it, convert for display.
- **Window function** — computes over a set of rows related to the current row without collapsing them: `LAG`, `LEAD`, `SUM() OVER (...)`.
- **Date dimension / series** — a table or CTE with one row per day, used to left-join and fill gaps.

### 3. How it works, step by step
1. **Choose the type by meaning.**
   - A birthday, invoice date, "business day" → `DATE`.
   - An absolute instant in time (when did the payment happen?) → `TIMESTAMP` or `DATETIME` *written in UTC*.
   - A scheduled local time ("9:00 every Monday in Mumbai") → `DATETIME` + a separate zone name, because it must not shift when rules change.
2. **Set the session/time zone deliberately.** `SELECT @@global.time_zone, @@session.time_zone;` — `SYSTEM` means "whatever the OS says", which varies between hosts. Prefer `'+00:00'` at the server and set app sessions explicitly.
3. **Do arithmetic with the right function:**
   - Add/subtract: `date + INTERVAL n unit`, `DATE_ADD`, `DATE_SUB`.
   - Whole-day difference: `DATEDIFF(d1, d2)` (date parts only; result in days).
   - Difference in any unit: `TIMESTAMPDIFF(unit, earlier, later)` — note the argument order.
   - Seconds: `TIMESTAMPDIFF(SECOND, a, b)` or `UNIX_TIMESTAMP(b) - UNIX_TIMESTAMP(a)`.
4. **Write range filters as half-open intervals** on the raw column: `ts >= '2026-10-01' AND ts < '2026-10-08'`.
5. **Use exact types for money** and round only at defined points.
6. **Use window functions** for "previous row" and running totals instead of self-joins.

### 4. Worked example
```sql
-- 1. Differences
SELECT DATEDIFF('2026-10-13 23:59:59', '2026-10-12 00:00:01')                  AS days_by_date,   -- 1
       TIMESTAMPDIFF(HOUR,   '2026-10-12 00:00:01', '2026-10-13 23:59:59')     AS hours,          -- 47
       TIMESTAMPDIFF(SECOND, '2026-10-12 00:00:00', '2026-10-12 00:01:30')     AS seconds;        -- 90

-- 2. Last 7 days, index-friendly (no function on the column)
SELECT * FROM orders
WHERE created_at >= NOW() - INTERVAL 7 DAY;

-- 3. Money: exact vs float
CREATE TABLE price_demo (a DECIMAL(10,2), b DOUBLE);
INSERT INTO price_demo VALUES (0.10, 0.10), (0.20, 0.20);
SELECT SUM(a) AS exact_sum, SUM(b) AS float_sum, SUM(b) = 0.30 AS equals_030 FROM price_demo;
-- exact_sum 0.30 | float_sum 0.30000000000000004 | equals_030 0

-- 4. Rounding
SELECT ROUND(2.345, 2) AS r;   -- 2.35 for DECIMAL literals ("round half away from zero")
SELECT ROUND(2.345e0, 2);      -- floating-point input: platform-dependent rounding, avoid for money

-- 5. Day-over-day change with LAG, and a running total
SELECT d, revenue,
       revenue - LAG(revenue) OVER (ORDER BY d)   AS delta,
       SUM(revenue) OVER (ORDER BY d)             AS running_total
FROM daily_sales;

-- 6. Dense date series, filling missing days with zero (8.0 recursive CTE)
WITH RECURSIVE days AS (
  SELECT DATE('2026-10-01') AS d
  UNION ALL
  SELECT d + INTERVAL 1 DAY FROM days WHERE d < '2026-10-31'
)
SELECT days.d, COALESCE(SUM(s.amount), 0) AS revenue
FROM days
LEFT JOIN sales s ON s.sale_ts >= days.d AND s.sale_ts < days.d + INTERVAL 1 DAY
GROUP BY days.d
ORDER BY days.d;
```
Notes: `DATEDIFF` ignores the time part, so rows 1 and 2 disagree on purpose — pick the one matching the business definition ("calendar days apart" vs "elapsed 24-hour periods"). In (6) the `LEFT JOIN` with a half-open range keeps the join sargable and returns a row per day, including zero-sales days.

**Time-zone-correct "per local day" revenue** (data stored in UTC, users in many regions):
```sql
SELECT DATE(CONVERT_TZ(paid_at, '+00:00', u.tz_name)) AS local_day,
       SUM(amount) AS revenue
FROM payments p JOIN users u ON u.id = p.user_id
WHERE p.paid_at >= '2026-10-01' AND p.paid_at < '2026-11-02'   -- widen the UTC range by a day each side
GROUP BY local_day;
```
`CONVERT_TZ` with named zones requires the time-zone tables to be loaded (`mysql_tzinfo_to_sql`). Filter on the raw UTC column first so an index can be used, then convert for bucketing.

### 5. Under the hood
- **`TIMESTAMP` vs `DATETIME`.** `TIMESTAMP` stores seconds since the epoch; on `INSERT` the value is converted from the session zone to UTC, and on `SELECT` converted back. Two servers (or app pools) with different `time_zone` read the same stored instant as different clock strings; writing the same *string* from them stores *different* instants. `DATETIME` stores the literal and never converts, so it is stable, but it carries no instant semantics. Daylight-saving gaps/overlaps can make local `DATETIME` values ambiguous or nonexistent.
- **2038.** `TIMESTAMP` overflows in January 2038; long-lived schemas (contracts, expiry dates) should use `DATETIME`.
- **Auto-init/update.** `DEFAULT CURRENT_TIMESTAMP` and `ON UPDATE CURRENT_TIMESTAMP` can be attached to `TIMESTAMP` and to `DATETIME` (since 5.6.5). In older configurations the first `TIMESTAMP` column got implicit behaviour; `explicit_defaults_for_timestamp=ON` (default in 8.0) removes it.
- **`DECIMAL` storage.** Packed in 4-byte groups of 9 decimal digits; arithmetic is exact in a 65-digit working precision. Division adds scale (`div_precision_increment`, default 4), so `10/3` is `3.3333`. Multiplying two `DECIMAL(10,2)` values yields scale 4; round explicitly before storing.
- **Float rounding.** `FLOAT` has ~7 significant digits, `DOUBLE` ~15–16; `ROUND` on them uses the C library, so the result can differ across platforms. For `DECIMAL`/exact literals MySQL rounds half away from zero.
- **Integer ranges.** `TINYINT` 1 byte (−128…127 / 0…255), `SMALLINT` 2 (±32,767 / 65,535), `MEDIUMINT` 3, `INT` 4 (±2.147 billion / 4.29 billion), `BIGINT` 8 (±9.22e18 / 1.84e19). The `(11)` in `INT(11)` is display width only and is deprecated in 8.0.
- **`SQL_CALC_FOUND_ROWS`.** Deprecated in 8.0.17: it forces the engine to ignore `LIMIT` for counting. Run two statements — the paged query and a separate `COUNT(*)` — or keyset-paginate and avoid the total.

### 6. Performance and design implications
- **Never wrap the indexed column in a function** (`WHERE DATE(created_at) = '2026-10-13'`, `YEAR(col) = 2026`): the index cannot be used. Use ranges: `created_at >= '2026-10-13' AND created_at < '2026-10-14'`.
- **Functional index (8.0.13+)** if you must: `CREATE INDEX ix ON t ((DATE(created_at)))` — and the query must use the same expression.
- **Store derived buckets** (a `DATE` column or a generated column) when you group by day constantly.
- **`DECIMAL` is slower than integers.** Many systems store money as integer minor units (`BIGINT` cents) and a currency code; either is fine if it is a single, documented source of truth.
- **Window functions** replace self-joins: a "first and last event per session" query with `ROW_NUMBER()`/`FIRST_VALUE` is one pass over a sorted partition, whereas a self-join compares every pair within the group.
- **`AUTO_INCREMENT` overflow** is permanent: at the type's max, the next insert fails with `Duplicate entry`. Monitor `information_schema.tables.AUTO_INCREMENT` against the type max.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `NOW()`, `CURRENT_TIMESTAMP`, `UTC_TIMESTAMP()`, `CURDATE()` | Current instant (session zone), UTC instant, current date | Defaults and filters; use UTC for stored instants |
| `DATE_ADD/DATE_SUB`, `± INTERVAL n unit` | Add/subtract durations | Windows like "last 7 days" |
| `DATEDIFF(a,b)` | Days between dates (time ignored) | Calendar-day gaps |
| `TIMESTAMPDIFF(unit,a,b)` | Difference in any unit, `b − a` | Elapsed time |
| `DATE_FORMAT`, `YEAR()`, `MONTH()`, `EXTRACT()` | Format/extract parts | Display and grouping |
| `CONVERT_TZ(dt, from, to)` | Zone conversion | Local-day rollups |
| `@@session.time_zone`, `SET time_zone='+00:00'` | Session zone | Make behaviour deterministic |
| `ROUND(x,d)`, `TRUNCATE(x,d)`, `CAST(x AS DECIMAL(m,d))` | Round/cut/convert | Define rounding points |
| `LAG`, `LEAD`, `SUM() OVER` | Row-relative and running calculations | Deltas, cumulative totals |
| `explicit_defaults_for_timestamp` | Disable legacy implicit TIMESTAMP defaults | Keep ON |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `TIMESTAMP` — audit columns, "when did it happen" instants, small footprint | Dates past 2038; apps whose sessions use different time zones without discipline |
| `DATETIME` in UTC — long-lived or portable data | You expect the DB to convert zones for you |
| `DATE` — day-only facts (birthday, due date) | You later need the time of day |
| `DECIMAL(19,4)` — multi-step finance, many currencies, FX | `DECIMAL(10,2)` is enough for single-currency prices and totals under ~100M |
| Integer minor units (cents) | Currencies with differing minor units or fractional-cent pricing |
| Money math in SQL — set-based, one source of truth | Rounding rules live in the app (tax per line vs per invoice) — define one place |
| Window functions | Old 5.7 servers (no windows) |

### 9. Common mistakes
- **`FLOAT`/`DOUBLE` for money** → sums drift by cents → `DECIMAL` or integer cents.
- **`WHERE DATE(col) = …`** → full scan → half-open range on the raw column.
- **Using `BETWEEN` for timestamps** with an end date `'2026-10-31'` → misses rows after midnight → `< next day`.
- **`TIMESTAMPDIFF` argument order** (earlier first) → negative or sign-flipped results.
- **`DATEDIFF` on datetimes** expecting elapsed days → ignores time → use `TIMESTAMPDIFF`.
- **Mixed server time zones** → the same `TIMESTAMP` displays differently → standardise on UTC.
- **Rounding at each step** vs at the end → different totals → define rounding once per business rule.
- **Signed `INT` auto-increment** → overflow → `BIGINT UNSIGNED`.
- **`SQL_CALC_FOUND_ROWS`** → deprecated, often slower → separate `COUNT(*)`.

### 10. 🎯 Interview kit
**30-second answer:** Use `DECIMAL` (or integer cents) for money and `DATETIME`/`TIMESTAMP` in UTC for instants. `TIMESTAMP` converts to the session zone and ends in 2038; `DATETIME` stores literally. Do date math with `INTERVAL`, `DATEDIFF`, `TIMESTAMPDIFF`, and write range predicates on the raw column so indexes work.

**Structure a longer answer:** state the semantic need (instant vs local time vs day) → type choice and range/precision → how arithmetic functions behave → index-friendly predicates → time-zone and rounding policy → monitoring (overflow, drift).

**Likely follow-ups:**
- *How do you get "per local day" revenue from UTC data?* → `CONVERT_TZ` (or per-user offset) in the bucket expression; keep the UTC range filter sargable.
- *Why did the DB total differ from the app's by paise?* → float vs decimal or different rounding point; make SQL `DECIMAL` the source of truth and define round-half-up once.
- *What do the numbers in `DECIMAL(10,2)` mean?* → total digits and digits after the point.
- *Compute day-over-day change?* → `value − LAG(value) OVER (ORDER BY day)`.
- *When does `AUTO_INCREMENT` overflow?* → at 2,147,483,647 for signed `INT`; then inserts fail.

**Signals interviewers look for:** you reason from meaning to type; you avoid functions on indexed columns; you know zone conversion mechanics; you treat rounding as a policy.

### 11. Check yourself
- [ ] Explain precisely what happens to a `TIMESTAMP` value when two sessions with different `time_zone` read it.
- [ ] Write "orders in the last 7 days" so that an index on `created_at` is used.
- [ ] Show why `SUM` of a `DOUBLE` column fails an equality test and what to use instead.
- [ ] Generate a row-per-day series and left-join sales to it with zeros filled.
- [ ] Explain `DATEDIFF` vs `TIMESTAMPDIFF`.
- [ ] State the maximum values of `INT`, `INT UNSIGNED` and `BIGINT`.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between the INT, SMALLINT, and BIGINT integer types in terms of storage?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33352_What_is_the_difference_between_the_INT_SMALLINT_and_BIGINT_i.md>)
- [Basic] [How do you get the current date and current timestamp in a MySQL query?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33370_How_do_you_get_the_current_date_and_current_timestamp_in_a_M.md>)
- [Intermediate] [How would you use DATE_ADD or INTERVAL to find records created in the last 7 days?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33371_How_would_you_use_DATE_ADD_or_INTERVAL_to_find_records_creat.md>)
- [Intermediate] [When would you choose DECIMAL(10,2) over DECIMAL(19,4) for a currency column and what does the second number control?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33009_When_would_you_choose_DECIMAL_10_2_over_DECIMAL_19_4_for_a_c.md>)
- [Intermediate] [Which data type would you use to store a monetary amount and why not FLOAT or DOUBLE?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33224_Which_data_type_would_you_use_to_store_a_monetary_amount_and.md>)
- [Deep] [What are the LAG and LEAD window functions and how would you compute a day-over-day change with them?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33031_What_are_the_LAG_and_LEAD_window_functions_and_how_would_you.md>)
- [Deep] [How would you generate a dense date series (a row per day) to left-join sales onto, filling missing days with zero?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33073_How_would_you_generate_a_dense_date_series_a_row_per_day_to_.md>)
- [Deep] [Explain how you'd choose between DECIMAL arithmetic in MySQL versus doing money math in the application, considering rounding, currency, and multi-step calculations.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33168_Explain_how_you_d_choose_between_DECIMAL_arithmetic_in_MySQL.md>)

## Related concepts

- [Timezones and Timestamps](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>) *(SQL Fundamentals)*
- [Classic SQL Query Problems](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>) *(DBMS Concepts)*
- [Advanced Aggregation](<../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>) *(Aggregation)*

