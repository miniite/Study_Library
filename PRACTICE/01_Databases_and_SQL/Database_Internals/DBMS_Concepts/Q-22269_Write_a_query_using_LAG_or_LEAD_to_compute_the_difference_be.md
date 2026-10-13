---
type: "practice"
difficulty: "Intermediate"
---

# Write a query using LAG or LEAD to compute the difference between consecutive daily sales.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Classic SQL Query Problems](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Classic_SQL_Query_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, you need to understand how LAG and LEAD window functions work in SQL. These functions allow you to reference rows within the same result set, making it easy to compute differences between consecutive days' sales.

#### Triaging the Question

The question is asking for a SQL query that calculates the difference in daily sales from one day to the next. This involves using either LAG or LEAD. The interviewer might be probing your understanding of window functions and how they can be used effectively for such calculations.

#### First-Principles Solution

Assume we have a table named sales with columns sale_date (in datetime format) and amount representing the sales amount. We'll use the LAG function to fetch the previous day's sale amount and then compute the difference between current and previous days.

Partition by the date to ensure comparisons are done within the same day.
Order by the date in ascending order.
Use LAG to get the sales amount of the previous day.
Compute the difference between the current day's sales and the previous day's sales.
SQL Query
SQL
SELECT
    sale_date,
    amount,
    LAG(amount) OVER (ORDER BY sale_date) AS prev_amount,
    (amount - LAG(amount) OVER (ORDER BY sale_date)) AS daily_difference
FROM
    sales;

This query will return a result set with each row containing the current date, the current day's sales amount, the previous day's sales amount, and the difference between them.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key here is understanding how LAG works. A common mistake might be forgetting to order by the date or not correctly partitioning the data if there are multiple records for a single date. Additionally, ensure that the sales table contains all necessary fields and is properly normalized.

A follow-up could involve discussing performance implications of large datasets or considering additional filters (e.g., only looking at weekdays) based on business requirements.

**Code:**

```sql
SELECT
    sale_date,
    amount,
    LAG(amount) OVER (ORDER BY sale_date) AS prev_amount,
    (amount - LAG(amount) OVER (ORDER BY sale_date)) AS daily_difference
FROM
    sales;
```

