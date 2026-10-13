---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle date and time comparisons in SQL, and what pitfalls exist with time zones?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Timezones and Timestamps](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Two things. (1) The mechanics of comparing dates/times in SQL correctly — the operators, functions, and the sargability traps that quietly kill index usage or return wrong rows. (2) The deeper systems problem: time zones. For a backend/fullstack SDE at an e-commerce, ride-hailing, food-delivery, or fintech company, this is not academic — an order placed at 11:58 PM IST that shows up "yesterday" in a report, or a scheduled surge-pricing window that fires an hour early, is a real production incident. The interviewer is probing whether I understand that a "timestamp" is meaningless without a reference frame, and whether I write comparisons that stay index-friendly.

#### Assumptions
/ clarifying questions I'd raise: Which engine? (PostgreSQL, MySQL, and SQL Server differ meaningfully.) Do we store UTC or local time? Is there a column indexed on the timestamp? For this answer I'll use PostgreSQL primarily and call out MySQL differences, and I'll assume the correct architecture is store UTC, convert at the edges.

Plan: cover the data types, the sargability rule for range comparisons, the three classic time-zone bugs, and the canonical fixes.

1. The data types decide everything

Before comparing, know what you're comparing:

DATE — calendar date, no time, no zone (2026-07-07).
TIME — wall-clock time, no date.
TIMESTAMP / TIMESTAMP WITHOUT TIME ZONE — a date+time with no zone attached. It's just "wall clock digits." Two of these compare literally, ignoring any real-world offset. This is the #1 source of bugs.
TIMESTAMPTZ / TIMESTAMP WITH TIME ZONE (Postgres) — internally stored as UTC; on input it's converted from the session zone, on output it's rendered in the session zone. Comparisons are always absolute-instant comparisons. This is the type you almost always want for "when did this event happen."

MySQL is different: its DATETIME is zone-naive (like TIMESTAMP WITHOUT TIME ZONE), while its TIMESTAMP type stores UTC internally and converts using @@session.time_zone — but only spans 1970–2038 and silently coerces. SQL Server has datetime2 (naive) and datetimeoffset (carries an explicit offset).

2. Comparing correctly — keep it sargable

The mechanics of comparison are simple (<, <=, BETWEEN, =), but the trap is wrapping the indexed column in a function, which makes the predicate non-sargable (the optimizer can't use the index and does a full scan).

Wrong (common, slow, and subtly incorrect):

SQL
WHERE DATE(created_at) = '2026-07-07'      -- function on the column → full scan

Right (half-open range, index-friendly):

SQL
WHERE created_at >= '2026-07-07'
  AND created_at <  '2026-07-08'

Note I use >= ... < next_day, not BETWEEN '2026-07-07' AND '2026-07-07 23:59:59'. BETWEEN is inclusive on both ends, so it (a) misses rows in the last second (23:59:59.500) and (b) with sub-second precision drops a sliver of the day. The half-open interval [start, next_start) is the canonical correct pattern for "one day," "one month," etc.

Other pitfalls:

Mixing types: comparing a DATE to a TIMESTAMP causes implicit casts; DATE gets promoted to midnight, so created_at = '2026-07-07' matches only the exact instant 2026-07-07 00:00:00, returning almost nothing.
String comparison: comparing a timestamp to a string relies on implicit parsing that varies by locale/format; always pass ISO-8601 (YYYY-MM-DD) literals or bound parameters.
NULL dates: created_at < NOW() silently excludes NULLs (three-valued logic). Decide explicitly whether nulls belong in the result.
3. The three time-zone bugs

Bug A — comparing a naive timestamp to NOW()/CURRENT_TIMESTAMP when they're in different frames. If you store local IST in a naive column but compare against a UTC clock (or vice versa), everything is shifted 5.5 hours. The fix is to never store naive local times for events; store UTC in a TIMESTAMPTZ.

Bug B — "today" is different per user. "Give me today's orders" for a user in San Francisco and one in Bengaluru are different 24-hour windows in absolute time. You must convert to the user's zone to define the day boundary:

SQL
WHERE created_at >= (DATE '2026-07-07' AT TIME ZONE 'Asia/Kolkata')
  AND created_at <  (DATE '2026-07-08' AT TIME ZONE 'Asia/Kolkata')

Here AT TIME ZONE 'Asia/Kolkata' applied to a DATE/naive timestamp interprets that wall-clock midnight as Kolkata time and yields the correct UTC instant, which then compares sargably against the UTC-stored created_at. The column stays untouched, so the index still works.

Bug C — DST and offset drift. Never hard-code offsets like +05:30 or -08:00. Use named zones (America/Los_Angeles), because the offset changes across DST transitions. AT TIME ZONE 'America/Los_Angeles' knows that on a "spring-forward" night 2:30 AM doesn't exist and on "fall-back" 1:30 AM happens twice. India (IST) has no DST, but any global product must handle it. This is why offset-based math (created_at + INTERVAL '5:30') is wrong for anything user-facing.

4. Truncation and bucketing for reports

For grouped reporting (orders per day per city), truncate in the right zone:

SQL
SELECT date_trunc('day', created_at AT TIME ZONE 'Asia/Kolkata') AS ist_day,
       count(*)
FROM orders
GROUP BY 1 ORDER BY 1;

If a report itself filters on a date range, apply the sargable range on the raw created_at in the WHERE, and only convert in the SELECT/GROUP BY.

Summary — what a strong answer emphasizes
Store UTC (TIMESTAMPTZ), convert only at the boundaries (ingest and display/reporting). "UTC in the database, zone in the UI."
Compare with half-open ranges (>= start AND < next), never BETWEEN for day/month windows, and never wrap the indexed column in a function — keep predicates sargable.
Time zone is business logic: define "a day" in the user's/market's named zone via AT TIME ZONE, and use named zones (not fixed offsets) so DST is handled.
Beware naive vs aware type mixing, implicit string/date casts, and NULL under three-valued logic.

#### Common mistakes
to avoid: DATE(col) = ... (kills index), BETWEEN off-by-one-second, storing local time, hard-coded offsets, and assuming the DB session zone matches the app server zone. Follow-up probes an interviewer might add: "How would you index this?" (b-tree on created_at, or a BRIN for append-only event tables); "How do you handle a scheduled job that must fire at 9 AM local every day across DST?" (compute the next UTC instant with the named zone each run, don't add a fixed 24h).

**Comparison:**

```
Type (Postgres / MySQL / SQL Server)	Stores zone?	Compares as	Use for
DATE	no	calendar date	birthdays, pure dates
TIMESTAMP WITHOUT TZ / DATETIME / datetime2	no (naive wall clock)	literal digits	local wall-clock intent only
TIMESTAMPTZ / TIMESTAMP / datetimeoffset	UTC-backed / explicit offset	absolute instant	event timestamps (default)
Pattern	Sargable?	Correct?	Verdict
DATE(created_at) = '2026-07-07'	No	risky (zone)	avoid
created_at BETWEEN d AND d+' 23:59:59'	Yes	off-by-fraction	avoid
created_at >= start AND created_at < next	Yes	yes	use this
```

**Code:**

```

-- Recommended column type (PostgreSQL): stored as UTC internally CREATE TABLE orders ( id bigserial PRIMARY KEY, created_at timestamptz NOT NULL DEFAULT now(), -- absolute instant, UTC city_tz text NOT NULL -- e.g. 'Asia/Kolkata' ); CREATE INDEX idx_orders_created_at ON orders (created_at);

-- SARGABLE "one calendar day in IST" (index used, correct boundaries) SELECT id, created_at FROM orders WHERE created_at >= (DATE '2026-07-07' AT TIME ZONE 'Asia/Kolkata') AND created_at < (DATE '2026-07-08' AT TIME ZONE 'Asia/Kolkata');

-- NON-SARGABLE anti-pattern (full scan) — avoid SELECT id FROM orders WHERE DATE(created_at AT TIME ZONE 'Asia/Kolkata') = DATE '2026-07-07';

-- Reporting: orders per IST day SELECT date_trunc('day', created_at AT TIME ZONE 'Asia/Kolkata') AS ist_day, count(*) AS orders FROM orders GROUP BY 1 ORDER BY 1;

CODE
Copy

```sql
-- MySQL differences: DATETIME is zone-naive; TIMESTAMP is UTC-backed.
-- Convert with CONVERT_TZ (requires loaded tz tables).
SELECT id
FROM orders
WHERE created_at >= CONVERT_TZ('2026-07-07 00:00:00','Asia/Kolkata','UTC')
  AND created_at <  CONVERT_TZ('2026-07-08 00:00:00','Asia/Kolkata','UTC');
```

**Example input / output:**

```

Table orders (created_at shown as stored UTC):

id	created_at (UTC)
1	2026-07-06 18:35:00+00
2	2026-07-07 18:29:00+00
3	2026-07-06 17:00:00+00

Query: "orders on IST day 2026-07-07"

CODE
Copy
WHERE created_at >= (DATE '2026-07-07' AT TIME ZONE 'Asia/Kolkata')  -- = 2026-07-06 18:30:00+00
  AND created_at <  (DATE '2026-07-08' AT TIME ZONE 'Asia/Kolkata')  -- = 2026-07-07 18:30:00+00

Result → id = 1 only.

id 3 (22:30 IST on the 6th) correctly excluded.
id 2 (00:00 IST on the 8th) correctly excluded — shows why BETWEEN ... 23:59:59 would be wrong.

Contrast: naive WHERE created_at BETWEEN '2026-07-07 00:00:00' AND '2026-07-07 23:59:59' (treating stored UTC as if it were IST) would return id 2 and miss id 1 — the classic 5.5-hour shift bug.
```

**Flow:**

1. ingest: convert to UTC
2. query: range on UTCsargable, index used
3. display/report:AT TIME ZONE user_tz
4. User/clientlocal time + zone
5. (DB: TIMESTAMPTZstored as UTC)
6. Result rows
7. User-local output

