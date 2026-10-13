---
type: "practice"
difficulty: "Deep"
---

# How would you write a query to roll up raw 10-second metrics into 1-minute averages for a rollup/downsampling table?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to create an SQL query that aggregates raw 10-second metrics into 1-minute averages. The key steps are defining the time granularity and using appropriate aggregation functions.

#### Triaging the Question

The interviewer is probing your understanding of SQL aggregate functions, date/time handling in SQL, and how to manage time series data effectively for downsampling or rollup operations. We will assume you have a table with timestamps and corresponding metric values, such as timestamp and value. The goal is to reduce the frequency of these metrics from 10 seconds to 1 minute.

#### First-Principles Solution

To aggregate raw 10-second metrics into 1-minute averages, we need to:

Define the time window for aggregation.
Use a rolling window or partitioning approach to group the data.
Calculate the average value within each group.
SQL Query:

Here is how you can write the query:

SQL
SELECT
    DATE_TRUNC('minute', timestamp) AS minute_bucket,
    AVG(value) AS avg_value
FROM
    raw_metrics
GROUP BY
    DATE_TRUNC('minute', timestamp)

This query uses DATE_TRUNC to round down each timestamp to the nearest minute, creating a time bucket for aggregation. The AVG function calculates the average value within each minute.

#### Closing Summary
and Common Mistakes:

In summary, the key is to use appropriate SQL functions like DATE_TRUNC to define the time buckets and aggregate using AVG. A common mistake might be not correctly defining the time granularity or forgetting to group by the truncated timestamp. Another pitfall could be assuming that timestamp is always a DateTime type and not considering other date/time formats.

#### Example
Data:

Let's assume we have the following sample data:

timestamp	value
2023-10-05 14:00:09	10
2023-10-05 14:00:17	12
2023-10-05 14:00:23	8
2023-10-05 14:01:01	15
2023-10-05 14:01:09	11
Worked Input->Output:

For the input data provided, the output would be:

minute_bucket	avg_value
2023-10-05 14:00	10
2023-10-05 14:01	13
Self-check:

This query correctly aggregates the data into 1-minute buckets and calculates the average value.

**Comparison:**

```
timestamp	value
2023-10-05 14:00:09	10
2023-10-05 14:00:17	12
2023-10-05 14:00:23	8
2023-10-05 14:01:01	15
2023-10-05 14:01:09	11
minute_bucket	avg_value
2023-10-05 14:00	10
2023-10-05 14:01	13
```

**Code:**

```sql
SELECT
    DATE_TRUNC('minute', timestamp) AS minute_bucket,
    AVG(value) AS avg_value
FROM
    raw_metrics
GROUP BY
    DATE_TRUNC('minute', timestamp)
```

**Example input / output:**

```text
Input:
timestamp                value
-------------------------- -------
2023-10-05 14:00:09        10
2023-10-05 14:00:17        12
2023-10-05 14:00:23         8
2023-10-05 14:01:01        15
2023-10-05 14:01:09        11

Output:
minute_bucket   avg_value
---------------- -----------
2023-10-05 14:00         10
2023-10-05 14:01         13
```

**Data:**

| timestamp                | value |
|--------------------------|-------|
| 2023-10-05 14:00:09      | 10    |
| 2023-10-05 14:00:17      | 12    |
| 2023-10-05 14:00:23      | 8     |
| 2023-10-05 14:01:01      | 15    |
| 2023-10-05 14:01:09      | 11    |

**Flow:**

1. Yes
2. No
3. Input Data
4. Is Timestamp in Range?
5. Apply DATE_TRUNC()
6. Average Value
7. Skip This Row
8. Output Aggregated Data

