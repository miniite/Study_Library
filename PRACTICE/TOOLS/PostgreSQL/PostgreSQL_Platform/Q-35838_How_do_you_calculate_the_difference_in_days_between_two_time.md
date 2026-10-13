---
type: "practice"
difficulty: "Basic"
---

# How do you calculate the difference in days between two timestamp columns in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the difference in days between two timestamp columns in PostgreSQL, we need to subtract one timestamp from another and then convert the result into a numeric value representing the number of days. This involves leveraging PostgreSQL's built-in functions such as EXTRACT and arithmetic operations.

#### Triage

The question is asking for a method to find the difference in days between two timestamps stored in columns within a PostgreSQL database. The interviewer might be probing your familiarity with date and time handling in SQL, and your ability to manipulate and calculate durations accurately. We will assume that the timestamps are of type TIMESTAMP or TIMESTAMP WITH TIME ZONE.

#### First-Principles Solution
Identify the Columns: Assume we have a table named orders with two timestamp columns: start_time and end_time.
Subtract Timestamps: We can subtract one timestamp from another to get an interval.
Extract Days: Use PostgreSQL's EXTRACT function or convert the resulting interval into days.
Steps:
Subtraction:
SQL
SELECT end_time - start_time AS time_diff FROM orders;
Extract Days:
SQL
SELECT EXTRACT(DAY FROM (end_time - start_time)) +
       EXTRACT(HOUR FROM (end_time - start_time) / 24) +
       EXTRACT(MINUTE FROM (end_time - start_time) / (24 * 60)) /
       1440.0 AS days_diff
FROM orders;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In the first query, we get a time interval which can be directly used in comparisons or further operations. For more precise day calculations, especially when fractional parts are involved, using EXTRACT provides better accuracy.

#### Common mistakes
might include:

Not understanding the difference between interval and timestamp.
Misinterpreting the result of subtracting two timestamps as a simple integer value for days.
Overlooking edge cases where time zones differ or daylight saving times affect the calculation.

If asked to optimize this query, we could consider indexing on the timestamp columns if they are frequently used in such calculations. Additionally, ensure that the performance is acceptable by checking the execution plan with EXPLAIN.

**Code:**

```sql
-- Subtracting timestamps and extracting days
SELECT end_time - start_time AS time_diff FROM orders;

-- Extracting days more precisely
SELECT EXTRACT(DAY FROM (end_time - start_time)) +
       EXTRACT(HOUR FROM (end_time - start_time) / 24) +
       EXTRACT(MINUTE FROM (end_time - start_time) / (24 * 60)) /
       1440.0 AS days_diff
FROM orders;
```

