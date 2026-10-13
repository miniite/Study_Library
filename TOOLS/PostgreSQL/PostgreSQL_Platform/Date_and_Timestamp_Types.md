---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Date and Timestamp Types

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Use `date` for calendar days, `timestamptz` for moments in time, `interval` for durations; store instants in UTC-semantics (`timestamptz`), filter with half-open ranges, and let `date_trunc`/`EXTRACT` do the bucketing.

### 1. The big picture
Time bugs are the most common silent data-quality bugs: reports off by one day, DST gaps, "last 7 days" that includes today or not. PostgreSQL gives precise types, but you must pick correctly. The key distinction is between a **calendar value** (a birthday, an invoice date — no zone) and an **instant** (when an event happened — one point on the universal timeline).

### 2. Core concepts and vocabulary
- **`date`** — a calendar day, 4 bytes, no time of day.
- **`time`** / **`timetz`** — time of day (avoid `timetz`; it is almost never right).
- **`timestamp`** (`timestamp without time zone`) — a wall-clock reading with no zone; ambiguous as an instant.
- **`timestamptz`** (`timestamp with time zone`) — an absolute instant; stored as UTC microseconds, 8 bytes, displayed in the *session* `TimeZone`.
- **`interval`** — a duration with months, days and microseconds components (`'1 month 2 days'`).
- **Session time zone** — `SHOW timezone;` controls input interpretation and output display of `timestamptz`.
- **`now()` / `CURRENT_TIMESTAMP`** — start time of the current *transaction*.
- **`clock_timestamp()`** — actual wall-clock time at the moment of the call.

### 3. How it works, step by step
1. A `timestamptz` literal such as `'2026-03-08 09:00+05:30'` is converted to UTC and stored as a single number. **The original offset is not kept.**
2. On output the value is converted to the session `TimeZone` and formatted.
3. A `timestamp` (no zone) is stored exactly as written; there is no conversion either way. Casting it to `timestamptz` *assumes* the session zone.
4. Arithmetic: `date - date` → integer days; `timestamp - timestamp` → `interval`; `timestamptz + interval '1 day'` adds a calendar day in the session zone (so it can be 23 or 25 hours across DST).
5. `DEFAULT now()` and `DEFAULT CURRENT_TIMESTAMP` are the same thing — both evaluate once per transaction at the first use.

### 4. Worked example
```sql
CREATE TABLE orders (
  id          bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  created_at  timestamptz NOT NULL DEFAULT now(),
  ship_date   date
);

-- today / now
SELECT CURRENT_DATE, now(), clock_timestamp();

-- last 7 days (half-open, index friendly)
SELECT * FROM orders WHERE created_at >= now() - interval '7 days';

-- a given year, without wrapping the column in a function
SELECT * FROM orders
WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01';

-- bucket by local calendar month
SELECT date_trunc('month', created_at AT TIME ZONE 'Asia/Kolkata') AS month,
       count(*)
FROM orders GROUP BY 1 ORDER BY 1;

-- parts and formatting
SELECT EXTRACT(year FROM created_at) AS y, EXTRACT(month FROM created_at) AS m,
       to_char(created_at, 'DD-Mon-YYYY') AS pretty;

-- differences
SELECT ship_date - created_at::date                       AS days_to_ship,
       age(now(), DATE '1990-05-17')                      AS person_age,
       EXTRACT(epoch FROM (t2 - t1)) / 3600               AS hours
FROM orders, (SELECT now() t2, now() - interval '3 h' t1) x;
```
`AT TIME ZONE 'zone'` flips meaning: applied to `timestamptz` it yields a `timestamp` showing wall-clock time in that zone (what you need for local bucketing); applied to a plain `timestamp` it yields a `timestamptz` interpreting the value as in that zone.

### 5. Under the hood   *(optional at this level)*
- Both `timestamp` and `timestamptz` are 8-byte integers of microseconds since 2000-01-01; `timestamptz` is always UTC internally. "With time zone" does **not** store a zone name.
- Zone rules (DST history) come from the IANA tz database shipped with PostgreSQL/OS; names such as `America/New_York` are correct, fixed abbreviations like `EST` are not DST-aware.
- `interval` keeps months/days/time separately, because "1 month" has no fixed length; this is why `now() + interval '1 month'` on Jan 31 gives Feb 28/29.
- `date_trunc` is `IMMUTABLE` for `timestamp` but only `STABLE` for `timestamptz` (it depends on the session zone), which matters for expression indexes.

### 6. Performance and design implications
- Never write `WHERE EXTRACT(year FROM created_at) = 2026` or `WHERE created_at::date = …` — the function on the column defeats a plain B-tree index. Use a range, or an expression index on the exact expression.
- For large append-only time series, a **BRIN** index on `created_at` is tiny and effective when rows arrive in time order; partition by range for retention.
- Store `timestamptz` in `UTC`-semantic form, convert only at presentation/bucketing.
- Keep the zone separately (a `text` column with the IANA name) if you need the user's original zone.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TIME ZONE 'UTC'` | session zone | make jobs deterministic |
| `now()`, `CURRENT_DATE` | transaction start | audit columns, defaults |
| `clock_timestamp()` | real current time | timing inside a function |
| `date_trunc('day', ts)` | round down | bucketing for GROUP BY |
| `EXTRACT(field FROM ts)` | numeric part | seasonality, dow, epoch |
| `ts AT TIME ZONE 'x'` | convert | local-day reporting |
| `to_char`, `to_timestamp`, `make_date` | format / build | I/O |
| `generate_series(start, end, '1 day')` | calendar spine | fill gaps in reports |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `timestamptz` — events, logs, audit | storing local "wall-clock appointments in the future" (zone rules may change; store local time + zone name) |
| `date` — birthdays, invoice dates | needing time of day |
| `timestamp` — a value truly zone-less | mixing with `timestamptz` implicitly |
| `date_trunc` for buckets | grouping by `EXTRACT(month)` alone (merges different years) |

### 9. Common mistakes
- Using `timestamp` for events across regions → ambiguous instants → use `timestamptz`.
- `BETWEEN '2026-01-01' AND '2026-01-31'` on timestamps → drops most of Jan 31 → half-open range.
- Assuming `now()` advances inside a transaction → it doesn't → use `clock_timestamp()`.
- Grouping by `date_trunc('day', created_at)` in UTC for a local report → rows land on the wrong day → convert with `AT TIME ZONE` first.
- Forgetting the session `TimeZone` differs between app servers and psql → different literal interpretation → set it explicitly or send offsets.

### 10. 🎯 Interview kit
**30-second answer:** "`timestamptz` stores a UTC instant and displays in the session zone; `timestamp` stores a wall-clock with no zone; `date` is a day. Use `timestamptz` for events, filter with half-open ranges, bucket with `date_trunc` after `AT TIME ZONE`."
**Structure a longer answer:** instant vs calendar value → storage → conversion rules → DST/bucketing → indexing consequences.
**Likely follow-ups:** *Does `timestamptz` store the zone?* → no. *`now()` vs `clock_timestamp()`?* → transaction start vs real. *`date_trunc` vs `EXTRACT`?* → rounded timestamp vs numeric part. *How to model "state as of date X"?* → history table with `tstzrange`, or `valid_from/valid_to`.
**Signals interviewers look for:** DST awareness, sargable filters, and the "instant vs wall-clock" vocabulary.

### 11. Check yourself
- [ ] Explain why `'2026-03-08 02:30'` may not exist in `America/New_York`.
- [ ] Rewrite a `WHERE EXTRACT(year …)` filter so it uses an index.
- [ ] Produce daily counts for last 30 days including zero-count days with `generate_series`.
- [ ] State what `SET TIME ZONE` changes and what it never changes.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you get the current date and current timestamp in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36157_How_do_you_get_the_current_date_and_current_timestamp_in_Pos.md>)
- [Basic] [What is the difference between the DATE type and the TIMESTAMP type in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36158_What_is_the_difference_between_the_DATE_type_and_the_TIMESTA.md>)
- [Basic] [How do you filter rows where a date column falls in a specific year?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36159_How_do_you_filter_rows_where_a_date_column_falls_in_a_specif.md>)
- [Basic] [How do you calculate the difference in days between two timestamp columns in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35838_How_do_you_calculate_the_difference_in_days_between_two_time.md>)
- [Intermediate] [How would you format a timestamp into a readable date string like 'DD-Mon-YYYY' using to_char?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36215_How_would_you_format_a_timestamp_into_a_readable_date_string.md>)
- [Intermediate] [How would you calculate a person's age in years from a date_of_birth column?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36216_How_would_you_calculate_a_person_s_age_in_years_from_a_date_.md>)
- [Intermediate] [What is the difference between DEFAULT now() and DEFAULT CURRENT_TIMESTAMP for a column?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35831_What_is_the_difference_between_DEFAULT_now_and_DEFAULT_CURRE.md>)
- [Intermediate] [How would you extract the year and month from a timestamp column using EXTRACT or DATE_TRUNC?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35836_How_would_you_extract_the_year_and_month_from_a_timestamp_co.md>)

## Related concepts

- [Timezones and Timestamps](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>) *(SQL Fundamentals)*
- [Date Functions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>) *(SQL Functions)*
- [Date Arithmetic](<../../MySQL/MySQL_Platform/Date_Arithmetic.md>) *(MySQL Platform)*

