---
type: "practice"
difficulty: "Basic"
---

# How do you count the number of ERROR-level log lines per service in the last 15 minutes?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that counts the number of log entries with an ERROR level for each unique service within the last 15 minutes. The key steps are identifying the relevant columns, filtering by time and error level, and grouping by service.

#### Triage

The interviewer is probing your understanding of basic SQL operations such as WHERE, GROUP BY, and aggregate functions like COUNT. You should assume that the log data table has columns for service_name (indicating which service the log belongs to), log_level (the severity level of the log entry), and a timestamp column. The assumption here is that the logs are stored in a database with timestamps, and we can use SQL's date functions to filter recent entries.

#### First-Principles Solution
Identify Columns: We need columns for service_name and possibly a log_level or similar field.
Filter by Time: Use a WHERE clause to select logs that occurred in the last 15 minutes.
Filter by Log Level: Ensure only ERROR-level logs are included.
Group by Service: Aggregate the count of logs for each service.
SQL Query
SQL
SELECT
    service_name,
    COUNT(*) AS error_count
FROM
    log_table
WHERE
    log_level = 'ERROR'
    AND timestamp >= NOW() - INTERVAL 15 MINUTE
GROUP BY
    service_name;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query will return a list of services along with the number of ERROR-level logs each has in the last 15 minutes. Common mistakes include forgetting to use COUNT(*) or using an incorrect column name for filtering. Follow-up questions might involve discussing performance optimizations, handling large datasets, or integrating this result into a monitoring dashboard.

**Code:**

```sql
SELECT
    service_name,
    COUNT(*) AS error_count
FROM
    log_table
WHERE
    log_level = 'ERROR'
    AND timestamp >= NOW() - INTERVAL 15 MINUTE
GROUP BY
    service_name;
```

