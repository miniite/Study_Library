---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to detect a broken pipeline where a table has zero rows for today but had data yesterday?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect a broken pipeline where a table has zero rows on the current day but had data yesterday, we can use SQL to compare the number of rows in the table between consecutive days. The core idea is to check if there are any changes in the row count from one day to the next.

#### Triaging the Question

The question is asking for an SQL query that identifies a specific scenario: a pipeline where today's data is missing, but there was data yesterday. This scenario can indicate issues like scheduled jobs failing or data ingestion problems. We need to compare the number of rows in the table for two consecutive days (yesterday and today).

#### Assumptions
The table has a column created_at that stores the timestamp when each row was inserted.
The database supports window functions, which are essential for comparing the row counts between two dates.
Step-by-Step Solution
Identify Yesterday's Date: We need to calculate yesterday's date using SQL's date functions.
Count Rows by Date: Use a GROUP BY clause to count the number of rows per day.
Compare Row Counts: Compare the row counts for today and yesterday.
SQL Query
SQL
WITH daily_counts AS (
    SELECT
        DATE_TRUNC('day', created_at) AS date,
        COUNT(*) AS row_count
    FROM
        your_table_name
    GROUP BY
        DATE_TRUNC('day', created_at)
),
yesterday AS (
    SELECT
        row_count
    FROM
        daily_counts
    WHERE
        date = CURRENT_DATE - INTERVAL '1 day'
),
today AS (
    SELECT
        row_count
    FROM
        daily_counts
    WHERE
        date = CURRENT_DATE
)

SELECT
    COALESCE(yesterday.row_count, 0) AS yesterday_row_count,
    COALESCE(today.row_count, 0) AS today_row_count,
    CASE
        WHEN today.row_count = 0 AND yesterday.row_count > 0 THEN 'Broken Pipeline Detected'
        ELSE 'Pipeline is Fine'
    END AS pipeline_status
FROM
    yesterday
FULL OUTER JOIN
    today ON 1=1;
Explanation
daily_counts CTE: This common table expression (CTE) calculates the row count for each day.
yesterday and today CTEs: These extract the row counts specifically for yesterday and today.
Final SELECT Statement: This compares the row counts between the two days and provides a status indicating whether the pipeline is broken or not.

#### Closing Summary

This SQL query efficiently identifies if a pipeline has failed by comparing the number of rows in a table for consecutive days. If today's count is zero but yesterday's was non-zero, it indicates a potential issue that needs investigation.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to account for time zones or daylight saving changes.

#### Follow-Up
Exploring additional checks like verifying job logs or checking data sources to pinpoint the exact cause of the failure.

**Comparison:**

| Column Name | Type           |
|-------------|----------------|
| id          | INT            |
| created_at  | TIMESTAMP      |
| id | created_at       |
|----|------------------|
| 1  | 2023-10-05T09:30:00Z |
| 2  | 2023-10-06T10:00:00Z |

**Code:**

```sql
WITH daily_counts AS (
    SELECT
        DATE_TRUNC('day', created_at) AS date,
        COUNT(*) AS row_count
    FROM
        your_table_name
    GROUP BY
        DATE_TRUNC('day', created_at)
),
yesterday AS (
    SELECT
        row_count
    FROM
        daily_counts
    WHERE
        date = CURRENT_DATE - INTERVAL '1 day'
),
today AS (
    SELECT
        row_count
    FROM
        daily_counts
    WHERE
        date = CURRENT_DATE
)

SELECT
    COALESCE(yesterday.row_count, 0) AS yesterday_row_count,
    COALESCE(today.row_count, 0) AS today_row_count,
    CASE
        WHEN today.row_count = 0 AND yesterday.row_count > 0 THEN 'Broken Pipeline Detected'
        ELSE 'Pipeline is Fine'
    END AS pipeline_status
FROM
    yesterday
FULL OUTER JOIN
    today ON 1=1;
```

**Example input / output:**

```text
Input:
- Sample data for `your_table_name`:

| id | created_at       |
|----|------------------|
| 1  | 2023-10-05T09:30:00Z |
| 2  | 2023-10-06T10:00:00Z |

Output:
| yesterday_row_count | today_row_count | pipeline_status        |
|---------------------|-----------------|------------------------|
| 2                   | 0               | Broken Pipeline Detected |
```

**Data:**

```json
{
  "your_table_name": {
    "schema": [
      {"name": "id", "type": "INT"},
      {"name": "created_at", "type": "TIMESTAMP"}
    ],
    "sample_data": [
      {"id": 1, "created_at": "2023-10-05T09:30:00Z"},
      {"id": 2, "created_at": "2023-10-06T10:00:00Z"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Yesterday's Date
6. Is Today's Row Count Zero?
7. Detect Broken Pipeline
8. Was There Data Yesterday?
9. Pipeline is Fine

