---
id: "TOOLS/SQL_Server/SQL_Server_Platform/Date_and_Time_Types"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/SQL_Server_Platform"
tools: ["SQL_Server"]
---

# Date and Time Types

**L4 Advanced** · [↑ SQL Server Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Use `date`, `time`, `datetime2` and `datetimeoffset` for new work; know that legacy `datetime`/`smalldatetime` round and have 3.33 ms ticks, and write date filters as half-open ranges so indexes and boundaries behave.

### 1. The big picture
Date/time handling is a frequent source of silent bugs: rounding at midnight, `BETWEEN` that misses or double-counts rows, implicit conversions that kill index seeks, and time-zone confusion. SQL Server has several types with different range, precision, storage and time-zone awareness; choosing the smallest one that fits is a data-type guideline that applies as much here as to numerics and strings.

### 2. Core concepts and vocabulary
- **`date`** — calendar date only, 3 bytes, 0001-01-01 to 9999-12-31.
- **`time(n)`** — time of day, fractional-second precision 0–7 (default 7), 3–5 bytes.
- **`datetime2(n)`** — date + time, precision 0–7, 6–8 bytes, range 0001–9999.
- **`datetimeoffset(n)`** — `datetime2` plus a UTC offset (+/-hh:mm), 8–10 bytes; stores offset, not a time-zone rule.
- **`datetime`** — legacy, 8 bytes, 1753–9999, accuracy rounded to .000/.003/.007 seconds.
- **`smalldatetime`** — legacy, 4 bytes, minute accuracy, 1900–2079.
- **`SYSDATETIME()` / `GETDATE()`** — current server time as `datetime2(7)` / `datetime`.
- **`SYSUTCDATETIME()`, `SYSDATETIMEOFFSET()`** — UTC and offset-aware variants.
- **Half-open interval** — `>= start AND < end`; the safe way to filter ranges.
- **Language/`DATEFORMAT`** — session settings that can change how ambiguous strings are parsed.

### 3. How it works, step by step
1. Pick the type by need: date only → `date`; need sub-second or full year range → `datetime2(n)`; need offset → `datetimeoffset`.
2. Literal conversion: strings are parsed using session settings, except unambiguous formats. `'20260105'` and ISO `'2026-01-05T10:30:00'` are safe for `datetime2`/`date`; `'2026-01-05'` can be read as yyyy-dd-mm for `datetime` under some `SET LANGUAGE` values.
3. Arithmetic: `DATEADD(unit, n, value)`, `DATEDIFF(unit, start, end)` (counts **boundary crossings**, not elapsed units), `DATEPART`, `EOMONTH`, `DATETRUNC` (2022+), `DATE_BUCKET` (2022+).
4. Comparison: when a column is compared to a value of another type, the lower-precedence type converts; date/time types rank above strings, so a string literal converts to the column type (fine), but a column of `varchar` compared to `datetime` converts the *column* (index scan).
5. Output formatting with `CONVERT(varchar, x, style)` or `FORMAT` (CLR, slow) — do this at the presentation layer.

### 4. Worked example
```sql
CREATE TABLE dbo.Events (
    EventID   INT IDENTITY PRIMARY KEY,
    OccurredAt datetime2(3) NOT NULL DEFAULT SYSUTCDATETIME()
);
CREATE INDEX IX_Events_OccurredAt ON dbo.Events (OccurredAt);

-- Wrong: function on the column (non-sargable) and BETWEEN end-of-day trap
SELECT * FROM dbo.Events
WHERE CAST(OccurredAt AS date) = '2026-03-01';

-- Right: half-open range, sargable
SELECT * FROM dbo.Events
WHERE OccurredAt >= '2026-03-01' AND OccurredAt < '2026-03-02';

SELECT DATEDIFF(year, '2025-12-31', '2026-01-01') AS Years;  -- returns 1!
```
The first query scans because `CAST` wraps the column (although SQL Server special-cases `CAST(col AS date)` into a range seek in modern versions, rely on the range form for any other function). The `DATEDIFF` result is 1 because it counts year boundaries crossed, not full years.

### 5. Under the hood
- `datetime` stores two integers: days since 1900-01-01 and 1/300-second ticks, hence the .003 rounding. A literal `'23:59:59.999'` rounds up to the next day.
- `datetime2` stores time as an integer count of units of 10^-n seconds plus a 3-byte date; precision ≤2 uses 6 bytes, 3–4 uses 7, 5–7 uses 8.
- `datetimeoffset` adds a 2-byte offset; two values representing the same instant compare equal even with different offsets, but display differently.
- Conversion between `datetime` and `datetime2` may produce slightly different values because of rounding: comparing `datetime` column to a `datetime2` literal with extra digits misses matches.
- Plans show `CONVERT_IMPLICIT(...)` on the column side when types mismatch; check the plan XML for `PlanAffectingConvert` warnings.

### 6. Performance and design implications
- Smaller types mean more rows per page: `date` (3 B) vs `datetime` (8 B) on a billion-row fact table is a gigabyte-scale difference.
- Store timestamps in UTC (`datetime2` + `SYSUTCDATETIME()`) and convert on display; use `AT TIME ZONE` when a zone is needed.
- Keep date predicates sargable; avoid `YEAR(col) = 2026`, `DATEDIFF(day, col, GETDATE()) < 7`, `CONVERT(varchar...)`.
- Calendar/date dimension tables beat computed expressions in reporting.
- Verify with `SET STATISTICS IO` and the plan's Predicate vs Seek Predicate.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SYSUTCDATETIME()` | Current UTC `datetime2(7)` | Audit columns, defaults |
| `DATEADD` / `DATEDIFF` | Shift / count boundaries | Range construction, ages (carefully) |
| `EOMONTH` | Month-end date | Reporting periods |
| `AT TIME ZONE` | Convert/attach offset by zone name | Display in local time |
| `TRY_CONVERT(date, x, style)` | Safe parse | ETL of messy text |
| `SET DATEFORMAT` / `SET LANGUAGE` | Changes literal parsing | Know it exists; avoid depending on it |
| `DATETRUNC`, `DATE_BUCKET` | Truncate/bucket (2022+) | Time-series grouping |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `date` for birthdays, invoice dates | Time component is needed |
| `datetime2(0–3)` for event time | You need sub-microsecond precision (use 7) |
| `datetimeoffset` when source gives offsets | You only need UTC (extra bytes) |
| `datetime` | Never for new design; keep only for compatibility |

### 9. Common mistakes
- `BETWEEN '2026-03-01' AND '2026-03-31'` on a datetime column → drops March 31 afternoon → use `< '2026-04-01'`.
- Wrapping the column in a function → index scan → rewrite as range.
- Using `GETDATE()` for UTC auditing → local-time ambiguity at DST changes → `SYSUTCDATETIME()`.
- Treating `DATEDIFF` as elapsed time → off-by-one ages → compute with boundary logic.
- Storing dates as `varchar` → invalid values, conversions → real date types.
- Assuming `datetime` literal `'…23:59:59.999'` is end-of-day → rolls to next day → use half-open.

### 10. 🎯 Interview kit
**30-second answer:** Prefer `date`, `datetime2` and `datetimeoffset` over `datetime`: wider range, controllable precision, less storage and no 3.33 ms rounding. Filter with half-open ranges to stay sargable and boundary-safe.

**Structure a longer answer:** clarify (need time? zone?) → type properties → pitfalls (BETWEEN, DATEDIFF, implicit conversion) → scenario (event table in UTC).

**Likely follow-ups:**
- *`datetime` vs `datetime2`?* → precision, range, storage, rounding.
- *`GETDATE()` vs `SYSDATETIME()`?* → `datetime` vs `datetime2(7)`; both local.
- *How would you get orders for one day efficiently?* → half-open range.
- *How to store a time zone?* → store UTC, plus the zone name separately; use `AT TIME ZONE`.

**Signals interviewers look for:** sargability awareness, UTC storage, knowing the rounding trap.

### 11. Check yourself
- [ ] List the storage and range of `date`, `datetime`, `datetime2(3)`, `datetimeoffset`.
- [ ] Rewrite `WHERE MONTH(OrderDate)=3 AND YEAR(OrderDate)=2026` sargably.
- [ ] Explain why `DATEDIFF(year,'2025-12-31','2026-01-01')` returns 1.
- [ ] Show a plan with `CONVERT_IMPLICIT` and fix it.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between the DELETE and TRUNCATE statements in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41375_What_is_the_difference_between_the_DELETE_and_TRUNCATE_state.md>)
- [Basic] [What are the main editions of SQL Server and when would you use SQL Server Express?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41475_What_are_the_main_editions_of_SQL_Server_and_when_would_you_.md>)
- [Basic] [What is the difference between VARCHAR and NVARCHAR in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41508_What_is_the_difference_between_VARCHAR_and_NVARCHAR_in_SQL_S.md>)
- [Basic] [What is the difference between the GETDATE() and SYSDATETIME() functions in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41520_What_is_the_difference_between_the_GETDATE_and_SYSDATETIME_f.md>)
- [Basic] [What is the difference between the CAST() and CONVERT() functions for data type conversion?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41524_What_is_the_difference_between_the_CAST_and_CONVERT_function.md>)
- [Basic] [What is the difference between the DROP, DELETE, and TRUNCATE commands in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41538_What_is_the_difference_between_the_DROP_DELETE_and_TRUNCATE_.md>)
- [Basic] [What is the BIT data type in SQL Server and how is it typically used?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41543_What_is_the_BIT_data_type_in_SQL_Server_and_how_is_it_typica.md>)
- [Intermediate] [What is the difference between a stored procedure and a user-defined function in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41480_What_is_the_difference_between_a_stored_procedure_and_a_user.md>)

## Related concepts

- [T-SQL Basics](<../T-SQL/T-SQL_Basics.md>) *(T-SQL)*
- [Temp Tables vs Table Variables](<../T-SQL/Temp_Tables_vs_Table_Variables.md>) *(T-SQL)*
- [SQL vs NoSQL](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL.md>) *(SQL Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Exact Numeric Data Types
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Character Data Types
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › TEXT and NTEXT: Deprecated
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Data Types
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Data Types › Table Types
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Spatial Data Types
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Thinking About Performance
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extents › Page Types
- *SQL Server Tutorial* (YouTube series) — Part 25: DateTime functions in SQL Server ([watch](https://www.youtube.com/watch?v=6_95vbBrrXI))
- *SQL Server Tutorial* (YouTube series) — Part 28: Cast and Convert functions in SQL Server ([watch](https://www.youtube.com/watch?v=8GHUfb5k-a8))
- *SQL Server Tutorial* (YouTube series) — Part 127: Difference between DateTime and SmallDateTime in SQL Server ([watch](https://www.youtube.com/watch?v=4lXpR6GdHo0))
- *SQL Server Tutorial* (YouTube series) — Part 129: Difference between DateTime and DateTime2 in SQL Server ([watch](https://www.youtube.com/watch?v=IPWdGoDAL20))

