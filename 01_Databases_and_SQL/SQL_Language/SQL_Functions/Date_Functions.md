---
id: "01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Functions"
tools: ["SQL_Server", "MySQL"]
---

# Date Functions

**L4 Advanced** · [↑ SQL Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Date functions let you get "now", build dates from parts, add and subtract intervals, measure differences, extract or truncate parts, and convert to and from text — but names and semantics differ by engine, and time zones and month-ends are where bugs live.

### 1. The big picture
Almost every business question has a time axis: "orders this month", "customers older than 30", "rows loaded in the last hour". Date/time functions are the vocabulary for slicing that axis. They fall into six jobs:

1. **Get the current moment** (`GETDATE`, `NOW`, `CURRENT_TIMESTAMP`).
2. **Construct** a date from parts or text (`DATEFROMPARTS`, `TO_DATE`, `CAST`).
3. **Shift** a date (`DATEADD`, `+ INTERVAL`).
4. **Measure** between two dates (`DATEDIFF`, subtraction, `AGE`).
5. **Extract / truncate** (`DATEPART`, `EXTRACT`, `DATE_TRUNC`, `EOMONTH`, `LAST_DAY`).
6. **Format / convert** (`FORMAT`, `TO_CHAR`, `CONVERT`, time-zone conversion).

The T-SQL books (*Beginning T-SQL*, *T-SQL Fundamentals*) teach the SQL Server names; the question bank mixes MySQL, PostgreSQL and Oracle. The concepts are identical, the spelling differs — so learn the concept and keep a small translation table (section 7).

### 2. Core concepts and vocabulary
- **Date, time, datetime, timestamp** — types that hold just a calendar day, just a clock time, or both. `datetime` in SQL Server has ~3 ms precision; `datetime2(7)` has 100 ns precision (7 fractional digits).
- **Precision** — number of fractional-second digits stored; affects range queries and equality.
- **Time zone / offset** — `datetimeoffset` (SQL Server) and `timestamptz` (PostgreSQL) store an instant plus offset or normalize to UTC; `timestamp without time zone` is just a wall-clock label.
- **UTC** — the reference time standard; store instants in UTC and convert at display.
- **Date part** — the unit argument: `year`, `quarter`, `month`, `day`, `hour`, …
- **Deterministic** — a function that returns the same value for the same inputs. `GETDATE()`/`SYSDATETIME()` are *nondeterministic*; `DATEADD` with fixed inputs is deterministic. This matters for indexed views and computed columns.
- **Boundary counting** — `DATEDIFF` counts how many *boundaries* of the unit are crossed, not elapsed whole units.
- **Epoch** — a reference instant (usually 1970-01-01 UTC) from which Unix timestamps count seconds.

### 3. How it works, step by step
**Current date/time (SQL Server), per T-SQL Fundamentals and Beginning T-SQL:**

| Function | Returns | Type |
|---|---|---|
| `GETDATE()` | local server time | `datetime` (3 decimals) |
| `CURRENT_TIMESTAMP` | same as `GETDATE`, but **ANSI standard** — Itzik recommends it | `datetime` |
| `SYSDATETIME()` | local server time, 7 decimals | `datetime2(7)` |
| `GETUTCDATE()` / `SYSUTCDATETIME()` | UTC | `datetime` / `datetime2` |
| `SYSDATETIMEOFFSET()` | local time with offset | `datetimeoffset` |

None returns *only* a date or time; cast: `CAST(SYSDATETIME() AS DATE)`.

**Building dates:** `DATEFROMPARTS(2022,2,12)`, `DATETIME2FROMPARTS(...)`, `TIMEFROMPARTS(...)`, `EOMONTH(date [, months_to_add])`. Constructing from integer parts avoids culture-dependent string parsing (is `'02/12/2022'` February or December?).

**Shifting and measuring:**
```
DATEADD(datepart, number, date)         -- date + number units
DATEDIFF(datepart, start_date, end_date) -- number of datepart boundaries crossed
```

**Extracting:** `DATEPART(month, d)`, `YEAR(d)`, `MONTH(d)`, `DAY(d)`, `DATENAME(weekday, d)`; standard `EXTRACT(YEAR FROM d)` in PostgreSQL/MySQL/Oracle.

**Parsing and formatting:** `CAST`, `CONVERT(type, value, style)`, `PARSE`/`TRY_PARSE`, `TRY_CONVERT` (return NULL instead of failing); `TO_DATE`/`TO_CHAR` with format masks in Oracle/PostgreSQL; `FORMAT()` in SQL Server (CLR-based, slow on large sets).

### 4. Worked example
```sql
-- SQL Server
SELECT  CAST(SYSDATETIME() AS date)                       AS today,
        DATEADD(day, 7, '2024-01-25')                     AS plus_week,          -- 2024-02-01
        DATEDIFF(day, '2024-01-01', '2024-03-01')         AS days_between,       -- 60 (leap year)
        DATEDIFF(year, '2023-12-31', '2024-01-01')        AS years_crossed,      -- 1 (!), only 1 day apart
        EOMONTH('2024-02-10')                             AS month_end,          -- 2024-02-29
        DATEADD(month, 1, '2024-01-31')                   AS month_plus_1;       -- 2024-02-29
```

Interpretation: `DATEDIFF(year, ...)` returned 1 for a one-day gap because the calendar year boundary was crossed. The last row shows **month-end normalization**: adding a month to Jan 31 gives the last valid day of February, not "Feb 31" or March 2. If you add one month, then subtract one month, you do *not* always get back to Jan 31 — a classic test case for date logic.

**Age in whole years (correct):**
```sql
SELECT DATEDIFF(year, dob, GETDATE())
       - CASE WHEN DATEADD(year, DATEDIFF(year, dob, GETDATE()), dob) > GETDATE() THEN 1 ELSE 0 END AS age
FROM people;
```
`DATEDIFF(year, ...)` alone overstates age for anyone whose birthday has not yet occurred this year; the `CASE` subtracts one in that case. PostgreSQL: `AGE(dob)` or `date_part('year', age(dob))`. MySQL: `TIMESTAMPDIFF(YEAR, dob, CURDATE())`.

**Other engines, same tasks**

```sql
-- PostgreSQL
SELECT now(), CURRENT_DATE, date_trunc('month', now()),
       now() + INTERVAL '7 days',
       (date_trunc('month', now()) + INTERVAL '1 month - 1 day')::date;   -- last day of month
-- MySQL
SELECT NOW(), CURDATE(), DATE_ADD(NOW(), INTERVAL 7 DAY), LAST_DAY(NOW()), DAYNAME(NOW());
-- Oracle
SELECT SYSDATE, TRUNC(SYSDATE), ADD_MONTHS(SYSDATE,1), LAST_DAY(SYSDATE), TO_CHAR(SYSDATE,'DD-Mon-YYYY') FROM dual;
```

### 5. Under the hood
- **Storage.** SQL Server `date` is 3 bytes (days since 0001-01-01), `datetime` is two 4-byte integers (days since 1900-01-01 and 1/300-second ticks, hence the odd 3.33 ms rounding), `datetime2` is 6–8 bytes. PostgreSQL `timestamptz` stores a UTC microsecond count; the zone is applied on input and output only.
- **Statement-level time.** `NOW()` in PostgreSQL is the transaction start time (constant inside a transaction); `clock_timestamp()` is the actual wall clock. SQL Server's `GETDATE()` is evaluated per statement (and a single reference per statement is constant across rows). MySQL has `NOW()` (statement start) vs `SYSDATE()` (actual call time).
- **`DATEDIFF` implementation.** It simply compares calendar-part counters (e.g. year number minus year number), which is why it ignores everything below the unit. `DATEDIFF(hour, '10:59', '11:00')` is 1. For true elapsed units, difference in a finer unit and divide, or use engine intervals.
- **Implicit conversion.** Comparing a `datetime` column with a string forces conversion of one side. If the *column* is converted (e.g. you wrap it in a function), the index becomes unusable (non-SARGable).
- **DST.** Local timestamps are ambiguous or missing on spring-forward/fall-back nights; only UTC or offset-aware types are unambiguous.

### 6. Performance and design implications
- **Never wrap the column.** `WHERE YEAR(order_date) = 2024` scans; write a half-open range: `WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'`. Half-open ranges also fix the "23:59:59.999 missing row" bug on `datetime`.
- **Use `datetime2`/`timestamptz`**, not `datetime`/`timestamp` without zone, for new work.
- **Store UTC**, convert in the presentation layer; keep the original offset if you need local-time analytics.
- **Dimension tables.** A `dim_date` table (one row per day with year, quarter, ISO week, is_holiday) replaces many runtime functions in warehouses and is sargable and joinable.
- **Avoid scalar `FORMAT()`** over millions of rows; `CONVERT` with a style or doing formatting in the BI layer is cheaper.
- **Deterministic functions** are required for computed columns you want to index or persist; `GETDATE()` is not allowed there.

### 7. Commands and settings you should know
| Task | SQL Server | PostgreSQL | MySQL | Oracle |
|---|---|---|---|---|
| Now | `GETDATE()`, `SYSDATETIME()` | `now()`, `CURRENT_TIMESTAMP` | `NOW()` | `SYSDATE`, `SYSTIMESTAMP` |
| Add | `DATEADD(day,7,d)` | `d + INTERVAL '7 day'` | `DATE_ADD(d, INTERVAL 7 DAY)` | `d + 7`, `ADD_MONTHS` |
| Difference | `DATEDIFF(day,a,b)` | `b - a` (int days for date) / `age()` | `DATEDIFF(b,a)`, `TIMESTAMPDIFF` | `b - a` |
| Extract | `DATEPART`, `YEAR()` | `EXTRACT(field FROM d)` | `EXTRACT`, `YEAR()` | `EXTRACT` |
| Truncate | `DATETRUNC` (2022+), `CAST(d AS date)` | `date_trunc('day', d)` | `DATE(d)` | `TRUNC(d)` |
| Month end | `EOMONTH` | `date_trunc` + interval | `LAST_DAY` | `LAST_DAY` |
| Parse / format | `CONVERT`, `FORMAT`, `TRY_CONVERT` | `to_date`, `to_char` | `STR_TO_DATE`, `DATE_FORMAT` | `TO_DATE`, `TO_CHAR` |
| Day name | `DATENAME(weekday,d)` | `to_char(d,'Day')` | `DAYNAME(d)` | `TO_CHAR(d,'Day')` |
| Time zone | `AT TIME ZONE` | `AT TIME ZONE` | `CONVERT_TZ` | `AT TIME ZONE` |
| Epoch | `DATEDIFF_BIG(s,'1970-01-01',d)` | `EXTRACT(EPOCH FROM d)` | `UNIX_TIMESTAMP`, `FROM_UNIXTIME` | arithmetic on `DATE '1970-01-01'` |

Weekday numbering also differs: MySQL `DAYOFWEEK` is 1 = Sunday … 7 = Saturday; MySQL `WEEKDAY` is 0 = Monday … 6 = Sunday; SQL Server's `DATEPART(weekday, …)` depends on `SET DATEFIRST`; PostgreSQL `EXTRACT(DOW …)` is 0 = Sunday and `ISODOW` is 1 = Monday.

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| ANSI forms (`CURRENT_TIMESTAMP`, `EXTRACT`) for portable SQL | You need engine features such as `DATEADD` with `quarter` or `EOMONTH` |
| `datetime2` / `timestamptz` + UTC | Legacy `datetime` — rounding to 1/300 s breaks equality and `23:59:59.999` ranges |
| `dim_date` join for calendar logic | Ad-hoc logic scattered across reports |
| `TRY_CONVERT`/`TRY_PARSE` for dirty input | You need to *know* an input was bad: a silent NULL can hide it |
| Elapsed-time with finer unit then divide | Using `DATEDIFF(year, …)` for age or tenure |

### 9. Common mistakes
- Using `DATEDIFF(month, a, b)` as "months elapsed" → counts calendar boundaries; Jan 31 → Feb 1 is "1 month" → compare dates or adjust.
- Wrapping an indexed date column in `YEAR()`, `CONVERT` or `DATEPART` → table scan → range predicates.
- `BETWEEN '2024-01-01' AND '2024-01-31'` on a datetime → drops Jan 31 after midnight → half-open range.
- Parsing ambiguous strings (`'01/02/03'`) → depends on language/`DATEFORMAT`/`NLS_DATE_FORMAT` → use ISO `'YYYY-MM-DD'` or `'YYYYMMDD'`.
- Mixing local time and UTC in the same table → off-by-N-hours reports and DST bugs → one convention, documented.
- Assuming Feb 30 is representable → `DATEADD(month,1,'01-31')` silently returns Feb 28/29 → test month-ends and leap years.
- Assuming the same weekday numbers across engines → wrong weekend filters on migration.

### 10. 🎯 Interview kit
**30-second answer:** "Date functions get the current time, construct, add, diff, extract/truncate, and format dates. The names differ by engine — `GETDATE/DATEADD/DATEDIFF` in SQL Server, `NOW/INTERVAL/EXTRACT` in PostgreSQL — and the traps are boundary-counting in `DATEDIFF`, month-end normalization, time zones, and wrapping indexed columns in functions."

**Structure a longer answer:** list the six jobs → show one engine's functions → call out two behaviors that surprise → explain how you keep it SARGable and UTC-consistent.

**Likely follow-ups:**
- *Compute age in years?* → birthday-adjusted `DATEDIFF`, or `AGE()` / `TIMESTAMPDIFF`.
- *Truncate a timestamp to the day?* → `CAST(ts AS date)` / `DATE_TRUNC('day', ts)` / `TRUNC(ts)`.
- *Last day of month?* → `EOMONTH` / `LAST_DAY`.
- *Why did adding a month to Jan 31 give Feb 28?* → end-of-month clamping.
- *Epoch to timezone-aware value?* → convert epoch to UTC `timestamp`, then `AT TIME ZONE`; beware DST gaps/overlaps.
- *Why is `WHERE YEAR(col)=2024` slow?* → non-sargable; use a range.

**Signals interviewers look for:** half-open ranges, UTC discipline, cross-engine awareness, and knowing `DATEDIFF` counts boundaries.

### 11. Check yourself
- [ ] Explain why `DATEDIFF(year,'2023-12-31','2024-01-01')` is 1.
- [ ] Rewrite `WHERE MONTH(d)=3 AND YEAR(d)=2024` to use an index.
- [ ] Write "last day of the current month" in two engines.
- [ ] Describe what `GETDATE()` vs `SYSDATETIME()` return and why `CURRENT_TIMESTAMP` is preferred.
- [ ] Compute whole-year age and say why plain `DATEDIFF` is wrong.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How do you get the current date and time using SQL functions like NOW, GETDATE, or SYSDATE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41271_How_do_you_get_the_current_date_and_time_using_SQL_functions.md>)
- [Intermediate] [What does the DATEDIFF function calculate and what arguments does it take?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41272_What_does_the_DATEDIFF_function_calculate_and_what_arguments.md>)
- [Intermediate] [How would you use DATEADD to add 7 days to a given date?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41273_How_would_you_use_DATEADD_to_add_7_days_to_a_given_date.md>)
- [Intermediate] [How do you extract the year, month, or day from a date using EXTRACT or DATEPART?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41274_How_do_you_extract_the_year_month_or_day_from_a_date_using_E.md>)
- [Intermediate] [How do you convert a date into a specific string format using functions like TO_CHAR or FORMAT?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41301_How_do_you_convert_a_date_into_a_specific_string_format_usin.md>)
- [Intermediate] [What does the TO_DATE function do and how do you parse a string into a date with it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41302_What_does_the_TO_DATE_function_do_and_how_do_you_parse_a_str.md>)
- [Intermediate] [How do you get the day name (like Monday) from a date using SQL functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41316_How_do_you_get_the_day_name_like_Monday_from_a_date_using_SQ.md>)
- [Intermediate] [What does the DAYOFWEEK or WEEKDAY function return and how is it numbered?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41317_What_does_the_DAYOFWEEK_or_WEEKDAY_function_return_and_how_i.md>)

## Related concepts

- [Timezones and Timestamps](<../SQL_Fundamentals/Timezones_and_Timestamps.md>) *(SQL Fundamentals)*
- [Date and Timestamp Types](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>) *(PostgreSQL Platform)*
- [Date Arithmetic](<../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>) *(MySQL Platform)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Nesting Functions › Using Date and Time Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Nesting Functions › DATEADD
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Nesting Functions › DATEDIFF
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Nesting Functions › DATENAME and DATEPART
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › Date and Time Functions
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › DATEADD
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › DATEDIFF
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › ISDATE
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Density Vector › STATSDATE
- *SQL Server Tutorial* (YouTube series) — Part 26: IsDate, Day, Month, Year and DateName DateTime functions in SQL Server ([watch](https://www.youtube.com/watch?v=kgefKDQ0_1g))
- *SQL Server Tutorial* (YouTube series) — Part 27: DatePart, DateAdd and DateDiff functions in SQL Server ([watch](https://www.youtube.com/watch?v=eYsizQVa_EU))
- *SQL Server Tutorial* (YouTube series) — Part 125: EOMONTH function in SQL Server 2012 ([watch](https://www.youtube.com/watch?v=SScr3TL6hzQ))
- *SQL Server Tutorial* (YouTube series) — Part 126: DATEFROMPARTS function in SQL Server ([watch](https://www.youtube.com/watch?v=XwW5iXzFQ5M))

