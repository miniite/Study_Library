---
type: "practice"
difficulty: "Basic"
---

# How would you compute the difference in days between two date columns?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the difference in days between two date columns, we need to understand how SQL handles dates and arithmetic operations on them. The basic approach involves using a date subtraction operation followed by extracting the number of days.

The core idea is straightforward: subtract one date from another to get an interval, then use a function like EXTRACT or DATE_PART (depending on your database system) to extract the number of days from that interval.

Let's go through this step-by-step:

Understand the Question: The interviewer is probing how you handle SQL operations and understand date arithmetic in databases.

#### Assumptions
We assume a typical SQL setup, such as PostgreSQL or MySQL, which provide built-in functions to work with dates.

#### First Principles Solution
First, subtract one date from the other to get an interval.
Then, use a function to extract the number of days.

For example, if we have two columns start_date and end_date, we want to compute the difference in days.

Here’s how you would write the SQL query:

SQL
SELECT
    (end_date::date - start_date::date) AS day_difference;

In this case, both start_date and end_date are assumed to be of type timestamp, datetime, or similar. The ::date cast ensures that only the date part is considered for subtraction.

For databases like PostgreSQL, you can also use:

SQL
SELECT
    (EXTRACT(EPOCH FROM end_date - start_date) / 86400)::integer AS day_difference;

Here, EPOCH gives the time in seconds since the Unix epoch, and dividing by 86400 (number of seconds in a day) converts it to days. The cast ensures we get an integer result.

#### Closing Summary

The solution involves subtracting one date from another to get an interval, then using a function to extract the number of days. Different SQL databases have different functions for this; common ones include PostgreSQL’s EXTRACT and MySQL’s subtraction directly followed by a cast.

#### Common Mistakes/Follow-ups
Mistake: Forgetting to convert timestamps to dates before subtracting.

#### Follow-up
Discuss edge cases, such as negative intervals or very large intervals that might overflow integer limits. Also, consider time zones if your application deals with them.

**Code:**

```sql
-- PostgreSQL Example
SELECT
    (end_date::date - start_date::date) AS day_difference;

-- MySQL Example
SELECT
    DATEDIFF(end_date, start_date) AS day_difference;
```

**Example input / output:**

```

-- Sample input and output WITH dates AS ( SELECT '2023-10-01 14:00:00'::timestamp as start_date, '2023-10-05 10:00:00'::timestamp as end_date ) SELECT (end_date - start_date) AS day_difference FROM dates;

CODE
Copy
Result:
```sql
day_difference
-----------------
        4 days
SQL
Copy
-- PostgreSQL Example with EXTRACT
WITH dates AS (
    SELECT '2023-10-01 14:00:00'::timestamp as start_date, '2023-10-05 10:00:00'::timestamp as end_date
)
SELECT
    (EXTRACT(EPOCH FROM end_date - start_date) / 86400)::integer AS day_difference
FROM dates;

Result:

SQL
Copy
day_difference
-----------------
        4
```

