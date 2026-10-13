---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL"]
---

# Timezones and Timestamps

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Store every instant in UTC (or with its offset) in a proper date-time type, convert to a local zone only when you display or when a *business day* is defined locally, and always express time windows as half-open ranges.

### 1. The big picture
Time looks simple and causes more silent data bugs than almost anything else: a "daily total" that differs between two dashboards, orders that land on the wrong day for users in Mumbai, a job that skips an hour twice a year, a join that matches nothing because one side is a string. The causes are mixing three different ideas:

1. **An instant** — a single point on the universal timeline (the moment a payment was captured).
2. **A local wall-clock time** — what a clock on a wall in some place showed (09:00 in Kolkata).
3. **A calendar date** — a day with no time (a birthday, an invoice date).

Instants should be stored unambiguously. Local times need a zone to be meaningful. Dates are a separate type and shouldn't be given a time just to fit.

### 2. Core concepts and vocabulary
- **UTC** — Coordinated Universal Time; no daylight saving.
- **Offset** — difference from UTC (`+05:30`); a fixed number, not a rule.
- **Time zone** — a region with rules (`Asia/Kolkata`, `America/New_York`) including DST transitions; named by IANA ID (Windows uses its own names in SQL Server).
- **DST (daylight saving time)** — local clock jumps forward (an hour doesn't exist) or back (an hour happens twice).
- **Timestamp without time zone** — `TIMESTAMP`, `DATETIME`, `DATETIME2`: just a date and time, meaning depends on convention.
- **Timestamp with time zone** — PostgreSQL `timestamptz` (stores UTC, displays in session zone); SQL Server `DATETIMEOFFSET` (stores local time + offset).
- **Precision** — fractional seconds: `DATETIME` ~3 ms (rounded to .000/.003/.007), `DATETIME2(7)` 100 ns, PostgreSQL microseconds.
- **Half-open interval** — `ts >= start AND ts < end`.
- **Business day** — a day defined in a particular zone (the store's, the regulator's).

### 3. How it works, step by step
**Capture.** Record events with an instant: `now()`/`CURRENT_TIMESTAMP` (PostgreSQL, standard), `SYSUTCDATETIME()` or `SYSDATETIMEOFFSET()` (SQL Server). Beware of functions that return *server local* time: `GETDATE()`, `SYSDATE` (Oracle), `LOCALTIMESTAMP`. Know the semantics: in PostgreSQL `now()` is fixed at transaction start, `clock_timestamp()` is wall-clock now.

**Store.** UTC in `timestamptz` / `DATETIME2` (convention: column ends with `_utc`) or `DATETIMEOFFSET`. If you must keep the source zone (for analytics like "local hour of day"), store the IANA zone or offset in a second column. Dates that are dates go in `DATE`.

**Convert.**
- PostgreSQL: `ts AT TIME ZONE 'Asia/Kolkata'` converts a `timestamptz` to local `timestamp` (and a plain `timestamp` interpreted *as* that zone to `timestamptz`; the same operator does both depending on input type).
- SQL Server: `ts AT TIME ZONE 'UTC' AT TIME ZONE 'India Standard Time'` (first call attaches UTC to a `datetime2`, second converts). `SWITCHOFFSET` changes the offset of a `DATETIMEOFFSET`.
- BigQuery: `TIMESTAMP` is always UTC; `DATETIME(ts, 'Asia/Kolkata')`.

**Bucket.** Convert to the business zone *then* truncate: `date_trunc('day', ts AT TIME ZONE 'Asia/Kolkata')`. Truncating in UTC and converting afterward puts events between 18:30 and 24:00 UTC on the "wrong" Indian date.

**Compare.** Convert both sides to the same type and zone; use half-open ranges; parse strings with an explicit format.

### 4. Worked example
```sql
-- PostgreSQL
CREATE TABLE orders (
  order_id   bigint PRIMARY KEY,
  placed_at  timestamptz NOT NULL DEFAULT now(),   -- instant, stored as UTC
  store_tz   text        NOT NULL                  -- 'Asia/Kolkata', 'Europe/Berlin'
);

-- Orders per *local* calendar day of the store
SELECT (placed_at AT TIME ZONE store_tz)::date AS local_day,
       count(*)                                AS orders
FROM   orders
WHERE  placed_at >= TIMESTAMPTZ '2026-10-01 00:00+00'     -- half-open range in UTC,
  AND  placed_at <  TIMESTAMPTZ '2026-11-01 00:00+00'     -- sargable on placed_at
GROUP  BY 1
ORDER  BY 1;

-- Hour-of-day and weekday features in the user's zone
SELECT order_id,
       EXTRACT(hour FROM placed_at AT TIME ZONE store_tz) AS local_hour,
       EXTRACT(isodow FROM placed_at AT TIME ZONE store_tz) AS iso_weekday   -- 1=Mon … 7=Sun
FROM   orders;
```

The same in SQL Server:

```sql
SELECT CAST(placed_at_utc AT TIME ZONE 'UTC' AT TIME ZONE 'India Standard Time' AS date) AS local_day,
       COUNT(*) AS orders
FROM   dbo.orders
WHERE  placed_at_utc >= '2026-10-01' AND placed_at_utc < '2026-11-01'
GROUP  BY CAST(placed_at_utc AT TIME ZONE 'UTC' AT TIME ZONE 'India Standard Time' AS date);
```

Normalize incoming string timestamps with mixed offsets during load:

```sql
-- PostgreSQL: input '2026-03-29 02:30:00+05:30' is parsed with its offset, stored as UTC
SELECT '2026-03-29 02:30:00+05:30'::timestamptz;       -- 2026-03-28 21:00:00+00
-- Reject bad rows rather than guessing:
SELECT raw_ts FROM staging WHERE raw_ts !~ '^\d{4}-\d{2}-\d{2}[ T]\d{2}:\d{2}:\d{2}';
```

DST edge cases: on the day clocks go back, `2026-11-01 01:30` in `America/New_York` exists twice; a *local* timestamp without offset cannot say which. On the spring-forward day `02:30` doesn't exist. Storing instants avoids both problems.

### 5. Under the hood
- **PostgreSQL `timestamptz` stores no zone.** It stores microseconds since 2000-01-01 UTC as an 8-byte integer. On output it is rendered in the session `TimeZone`; on input, unless an offset is supplied, the string is interpreted in the session zone. Hence the same query can show different text for different sessions while representing the same instant.
- **`timestamp` (without zone)** ignores any offset in the input string (PostgreSQL silently drops it): a frequent loss-of-information bug.
- **SQL Server `datetime`** is accurate to 1/300 second, so `'23:59:59.999'` rounds up to the next day; `datetime2` and `date` avoid this. Comparing `datetime` columns to `datetime2` parameters causes implicit conversions that can break index seeks.
- **Zone rules are data.** `tzdata` (IANA) is updated several times a year when governments change DST. PostgreSQL ships it with the server; SQL Server reads Windows' registry; a stale database can compute the wrong offset for future dates.
- **Spark/warehouse.** Spark `TIMESTAMP` is an instant rendered in the session zone (`spark.sql.session.timeZone`); `TIMESTAMP_NTZ` is zoneless. Setting the session zone wrongly shifts `to_date()` results.

### 6. Performance and design implications
- Keep predicates on the bare timestamp column with constant bounds; avoid `WHERE CAST(ts AS date) = …` on engines that can't seek through it, and `WHERE date_trunc(…) = …` anywhere.
- Partition and cluster on UTC instants; add a derived `local_date` column when many queries need the business day.
- Index `(store_id, placed_at)` for per-entity time ranges.
- Always include sub-second precision for telemetry; truncate at presentation, not storage.
- Persist `ingested_at` (load time) separately from `event_at` (occurred time); late arrivals make them differ.
- `DEFAULT CURRENT_TIMESTAMP` (or `SYSUTCDATETIME()`) on `created_at` gives every row an audit timestamp without app code.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `now()`, `CURRENT_TIMESTAMP` | Transaction-start instant | Default values |
| `SYSUTCDATETIME()`, `SYSDATETIMEOFFSET()` | UTC / offset-aware time (SQL Server) | Auditing |
| `AT TIME ZONE` | Attach or convert zone | Local-day bucketing |
| `date_trunc`, `DATETRUNC` | Truncate to unit | Daily/weekly buckets |
| `EXTRACT`, `DATEPART` | Pull a field | Hour/weekday features |
| `DATEADD`, `+ INTERVAL` | Date arithmetic | Windows |
| `SET TIME ZONE 'UTC'` / `timezone` setting | Session zone | Make output predictable |
| `TRY_CONVERT(datetime2, s, 126)` | Safe parse with format | Staging loads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| UTC instants for events | You need the *original local time* for legal records — keep zone/offset too |
| `DATETIMEOFFSET` / keep offset | Offsets alone can't predict future DST changes; store a zone ID for future appointments |
| Local wall-clock + zone ID for future events (meetings) | Past events — instants are better |
| Business-day column derived at load | Zone rules might change retroactively — recompute capability |
| `DATE` type for dates | Using midnight timestamps to represent dates across zones |

### 9. Common mistakes
- Using server-local `GETDATE()` everywhere → shifts when servers move region → `SYSUTCDATETIME()`.
- Grouping by `CAST(ts AS date)` in UTC for a local-day KPI → day boundaries off by hours → convert first.
- `BETWEEN '2026-10-01' AND '2026-10-31'` on timestamps → drops most of the last day → half-open range.
- Storing timestamps as strings → wrong sorting, no range pruning, parse errors → native types.
- Assuming every day has 24 hours → DST breaks "add 24 hours" → use calendar arithmetic in the zone.
- Ignoring `datetime` rounding (`.999`) → rows land on the next day → `datetime2`.
- Comparing a `timestamp` and a `timestamptz` implicitly → session-zone dependent result → cast explicitly.

### 10. 🎯 Interview kit
**30-second answer:** Store instants in UTC with a zone-aware type, keep a zone ID when the local context matters, convert to the business zone before bucketing into days, and query with half-open ranges on the raw column so indexes and partitions work.

**Structure a longer answer:** instant vs local vs date → storage choice → conversion functions in the engine → bucketing/time-window rules → DST edge cases → performance and validation checks.

**Likely follow-ups:**
- *Why do "group by day" numbers differ between regions?* → bucketing before converting.
- *`timestamp` vs `timestamptz`?* → no zone vs UTC-normalized instant.
- *How do you handle a local time that occurs twice?* → store an instant or offset.
- *Why not `BETWEEN`?* → inclusive end with time component.

**Signals interviewers look for:** UTC-by-default, zone IDs for future events, half-open intervals, awareness of DST and precision quirks.

### 11. Check yourself
- [ ] Explain why `timestamptz` doesn't store a zone.
- [ ] Write a query counting events per local calendar day for users in two zones.
- [ ] Describe what happens to `02:30` local on a spring-forward day.
- [ ] Rewrite a `CAST(ts AS date) = '2026-10-01'` predicate as a range.
- [ ] Choose types for `created_at`, `birth_date`, `meeting_start` (future, local) and justify.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How would you write a query to return the current date and time from the database server?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39784_How_would_you_write_a_query_to_return_the_current_date_and_t.md>)
- [Basic] [What is the difference between GETDATE, SYSDATE, NOW, and CURRENT_TIMESTAMP across databases?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39787_What_is_the_difference_between_GETDATE_SYSDATE_NOW_and_CURRE.md>)
- [Basic] [How would you write a query to add a computed load_timestamp column so every row in the warehouse records when it was ingested?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40084_How_would_you_write_a_query_to_add_a_computed_load_timestamp.md>)
- [Basic] [How do you extract the hour of day from a metric's timestamp so you can group telemetry by hour?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40956_How_do_you_extract_the_hour_of_day_from_a_metric_s_timestamp.md>)
- [Intermediate] [How would you extract the year and month from a timestamp column for grouping?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39989_How_would_you_extract_the_year_and_month_from_a_timestamp_co.md>)
- [Intermediate] [How do you handle date and time comparisons in SQL, and what pitfalls exist with time zones?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40464_How_do_you_handle_date_and_time_comparisons_in_SQL_and_what_.md>)
- [Intermediate] [How do you store timestamps for LLM calls so that daily aggregations remain correct for users across multiple time zones?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40805_How_do_you_store_timestamps_for_LLM_calls_so_that_daily_aggr.md>)
- [Intermediate] [Explain how you'd normalize event timestamps to UTC when sources send local times with varying offsets, and where 'group by day' off-by-one bugs come from when you bucket before converting.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40197_Explain_how_you_d_normalize_event_timestamps_to_UTC_when_sou.md>)

## Related concepts

- [Date and Timestamp Types](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>) *(PostgreSQL Platform)*
- [Date Functions](<../SQL_Functions/Date_Functions.md>) *(SQL Functions)*
- [Date Arithmetic](<../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>) *(MySQL Platform)*

## References

- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — T-SQL for data analysis
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Temporal tables
- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks › Unreliable Clocks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Timestamps for ordering events
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Whose clock are you using, anyway?
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — integer, but it is typically much more useful to input a timestamp using one of several supported
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Cleaning Data
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Querying Temporal Data

