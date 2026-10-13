---
type: "practice"
difficulty: "Intermediate"
---

# How would you extract the year and month from a timestamp column for grouping?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Timezones and Timestamps](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract the year and month from a timestamp column in SQL, we typically use functions like DATE_PART or EXTRACT, depending on the SQL dialect you're working with (e.g., PostgreSQL vs. MySQL). This is useful for grouping data by these time dimensions.

First, let's define our problem: suppose we have a table named orders with a column order_time that contains timestamps. We want to group and aggregate data based on both the year and month of this timestamp.

#### Triage

The question is asking how to separate and extract the year and month from a timestamp for grouping purposes.
The interviewer might be probing your understanding of SQL date functions, handling different database systems (e.g., PostgreSQL vs. MySQL), and your ability to write efficient queries.

Step-by-step reasoning:

Understand the Data: Assume we have an orders table with columns order_id, customer_name, order_time.
Extract Year and Month:
For PostgreSQL, use DATE_PART('year', order_time) and DATE_PART('month', order_time).
For MySQL, use YEAR(order_time) and MONTH(order_time).
Group by Year and Month: Combine the extracted year and month to form a composite key for grouping.
Aggregate Data: Perform any necessary aggregation, such as counting orders or summing amounts.

Full Solution:

SQL
-- PostgreSQL Example
SELECT
    DATE_PART('year', order_time) AS year,
    DATE_PART('month', order_time) AS month,
    COUNT(*) AS num_orders
FROM
    orders
GROUP BY
    year, month;

-- MySQL Example
SELECT
    YEAR(order_time) AS year,
    MONTH(order_time) AS month,
    COUNT(*) AS num_orders
FROM
    orders
GROUP BY
    year, month;

#### Closing Summary

We used SQL functions specific to the database system to extract and group by year and month.
Always consider the specific SQL dialect you are working with since different databases may use slightly different syntax.

#### Common Mistakes/Follow-Ups

Failing to account for different SQL dialects can lead to errors. Ensure you know which one is being used or write cross-database-compatible queries if necessary.
Not considering time zones and daylight saving times can cause issues, especially with large datasets spanning multiple regions.

**Code:**

```sql
-- PostgreSQL Example
SELECT
    DATE_PART('year', order_time) AS year,
    DATE_PART('month', order_time) AS month,
    COUNT(*) AS num_orders
FROM
    orders
GROUP BY
    year, month;

-- MySQL Example
SELECT
    YEAR(order_time) AS year,
    MONTH(order_time) AS month,
    COUNT(*) AS num_orders
FROM
    orders
GROUP BY
    year, month;
```

