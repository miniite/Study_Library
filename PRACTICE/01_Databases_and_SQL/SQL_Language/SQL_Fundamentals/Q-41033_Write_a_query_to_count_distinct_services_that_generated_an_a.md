---
type: "practice"
difficulty: "Basic"
---

# Write a query to count distinct services that generated an alert each day over the last week.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that counts the number of unique services (or distinct service names) that triggered alerts each day within the last seven days. The key is to use the DISTINCT keyword combined with a GROUP BY clause to group the data by date and count the unique services for each day.

#### Triaging the Question

The interviewer is likely probing your understanding of SQL, specifically how to work with dates and aggregate functions like COUNT(DISTINCT ...). You'll need to handle time-based filtering and ensure that you correctly identify distinct service names within a given timeframe. Assumptions we can make include:

The table containing alert data has columns for the timestamp and the service name.
The database system supports common SQL functionalities such as date functions and aggregation.

#### First Principles Solution
Filter by Date: We need to restrict our query to only consider records from the last seven days.
Count Distinct Services per Day: For each day, we want to count how many unique services generated alerts.

Here's a step-by-step breakdown:

Extract the date part from the timestamp column using SQL’s date functions (e.g., DATE(timestamp) AS alert_date).
Use a GROUP BY clause on this alert_date.
Apply the COUNT(DISTINCT service_name) function to count unique services per day.
Full Solution
SQL
SELECT
    DATE(timestamp) AS alert_date,
    COUNT(DISTINCT service_name) AS distinct_service_count
FROM
    alerts  -- Assuming the table containing alert data is named 'alerts'
WHERE
    timestamp >= NOW() - INTERVAL 7 DAY
GROUP BY
    alert_date;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This query will return a list of dates from the last seven days and, for each date, the count of unique services that generated alerts. The key steps were:

Filtering records based on the timestamp.
Extracting the date part to group by day.
Counting distinct service names within each grouped date.

#### Common mistakes
could include:

Not correctly using DISTINCT or grouping by the wrong column.
Failing to handle time zones if timestamps are stored with time zone information.
Incorrectly handling the date interval in your SQL dialect.

#### Follow-ups
might include discussing how this query would perform on a large dataset and strategies for optimizing it, such as using indexes on the timestamp column. Additionally, you could explore other types of data aggregations or filtering options depending on the specific use case.

**Comparison:**

```
Column	Description
timestamp	The time of the alert
service_name	Name of the service generating the alert
```

**Code:**

```sql
SELECT
    DATE(timestamp) AS alert_date,
    COUNT(DISTINCT service_name) AS distinct_service_count
FROM
    alerts  -- Assuming the table containing alert data is named 'alerts'
WHERE
    timestamp >= NOW() - INTERVAL 7 DAY
GROUP BY
    alert_date;
```

**Example input / output:**

| alert_date   | distinct_service_count |
|--------------|-----------------------|
| 2023-10-05   | 12                    |
| 2023-10-06   | 15                    |
| ...          | ...                   |

**Flow:**

1. Filter Records
2. Date Extraction
3. Group By Date
4. Count Distinct Services
5. Output Results

