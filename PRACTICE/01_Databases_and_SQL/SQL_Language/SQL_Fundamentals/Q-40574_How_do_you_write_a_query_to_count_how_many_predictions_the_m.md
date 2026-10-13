---
type: "practice"
difficulty: "Basic"
---

# How do you write a query to count how many predictions the model logged per day from a predictions table for monitoring?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to understand that the predictions table contains records of predictions made by our machine learning model, and each record includes information such as the prediction time or timestamp. Our goal is to count how many predictions were logged per day.

First, let's break down the problem:

We need a way to extract the date from the timestamp.
We should group these dates into days.
Finally, we need to count the number of records for each day.

Now, I will walk you through writing this SQL query step-by-step:

Step 1: Extracting the Date

We assume that the predictions table has a column named prediction_time, which stores the timestamp of when the prediction was logged. We can extract the date part from this timestamp using SQL's DATE function.

SQL
SELECT DATE(prediction_time) AS log_date FROM predictions;
Step 2: Grouping by Day

Next, we need to group these dates into days. This is done using the GROUP BY clause in SQL.

SQL
SELECT DATE(prediction_time) AS log_date FROM predictions GROUP BY log_date;

This gives us a list of distinct dates (days) when predictions were logged.

Step 3: Counting Predictions per Day

Finally, we need to count how many records exist for each day. We do this by adding the COUNT function around our grouped date.

SQL
SELECT DATE(prediction_time) AS log_date, COUNT(*) AS prediction_count FROM predictions GROUP BY log_date;

This query will give us a result set with two columns: log_date and prediction_count. The log_date column shows the day when predictions were logged, and the prediction_count column shows how many predictions were made on that particular day.

#### Closing Summary

In summary, to write an SQL query that counts the number of predictions logged per day from a predictions table, we use the DATE function to extract the date part of the timestamp, group by this date, and then count the records for each day. This is useful for monitoring purposes, helping us track the model's activity over time.

#### Common Mistakes/Follow-Ups
Mistake 1: Forgetting to use DATE function to extract the date part of the timestamp.
Mistake 2: Not using GROUP BY clause to group predictions by day.

#### Follow-Up
Question: What if we want to include days with zero predictions in our result? We would need to use a LEFT JOIN or a FULL OUTER JOIN depending on the SQL dialect.

**Code:**

```sql
SELECT DATE(prediction_time) AS log_date, COUNT(*) AS prediction_count FROM predictions GROUP BY log_date;
```

