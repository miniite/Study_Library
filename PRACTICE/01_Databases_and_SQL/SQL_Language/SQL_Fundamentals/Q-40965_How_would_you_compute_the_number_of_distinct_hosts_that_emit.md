---
type: "practice"
difficulty: "Basic"
---

# How would you compute the number of distinct hosts that emitted an ERROR log in the last hour?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to count the unique hostnames that have logged an "ERROR" message within the past hour. This is a common task in AIOps where we often deal with large volumes of log data and need efficient ways to process it.

The question is probing our understanding of SQL queries, specifically how to use filtering conditions and aggregation functions like COUNT and DISTINCT. We also might consider performance implications if dealing with a very large dataset.

Let's break down the solution step-by-step:

Identify the relevant columns: The query will need two main pieces of information: the hostname and the log message.
Filter by time range: We need to filter logs based on when they were recorded, specifically within the last hour.
Filter by error level: Only count rows where the log message indicates an "ERROR".
Use DISTINCT for uniqueness and COUNT to get the total number of unique hosts.

Here is the SQL query that accomplishes this:

SQL
SELECT COUNT(DISTINCT hostname)
FROM logs
WHERE log_level = 'ERROR'
  AND log_time >= NOW() - INTERVAL 1 HOUR;

Let's go through each part:

COUNT(DISTINCT hostname): This counts the unique hostnames.
logs: This is the table containing our log data.
log_level = 'ERROR': Filters to only include rows where the log level is "ERROR".
log_time >= NOW() - INTERVAL 1 HOUR: Ensures we are considering logs that were recorded in the last hour.

**Code:**

```sql
SELECT COUNT(DISTINCT hostname)
FROM logs
WHERE log_level = 'ERROR'
  AND log_time >= NOW() - INTERVAL 1 HOUR;
```

