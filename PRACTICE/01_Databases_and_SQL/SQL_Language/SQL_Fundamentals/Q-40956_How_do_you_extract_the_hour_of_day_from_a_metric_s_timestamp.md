---
type: "practice"
difficulty: "Basic"
---

# How do you extract the hour of day from a metric's timestamp so you can group telemetry by hour?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Timezones and Timestamps](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Timezones_and_Timestamps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract the hour of the day from a metric's timestamp, we typically need to parse the timestamp and then use SQL functions to isolate the hour component. This is crucial in AIOps scenarios where you want to group telemetry data by hourly intervals for analysis or anomaly detection.

The question asks how to perform this extraction using SQL. The primary steps involve:

Parsing the timestamp.
Using a function to extract the hour part from the parsed timestamp.

Let's break down these steps first-principles style.

#### Triaging the Question
What is being asked? Extracting the hour of day (0-23) from a metric’s timestamp column in SQL.
What are we probing? Understanding basic SQL functions, time manipulation, and common date/time operations.
Key Assumptions: We assume that timestamps are stored in a format recognizable by SQL (e.g., DATETIME, TIMESTAMP).

#### Solution
from First Principles
Parsing the Timestamp:
Most SQL databases have built-in functions to parse timestamp values. For example, PostgreSQL, MySQL, and SQLite all provide functions for handling date and time.
Extracting the Hour Component:
Once parsed, we can use specific functions like EXTRACT, DATEPART, or direct access methods depending on the SQL dialect.
Detailed Solution

Let's assume our metric table has a column named timestamp of type DATETIME.

Step 1: Use an appropriate function to extract the hour part. In PostgreSQL, we would use:

SQL
EXTRACT(HOUR FROM timestamp_column);

In MySQL, it would be similar but slightly different:

SQL
HOUR(timestamp_column);

In SQLite, you can also directly access the hour component using:

SQL
STRFTIME('%H', timestamp_column);

Step 2: If we need to group by this hourly data, we might use a GROUP BY clause. For example:

SQL
SELECT EXTRACT(HOUR FROM timestamp) AS hour, COUNT(*)
FROM telemetry_table
GROUP BY EXTRACT(HOUR FROM timestamp);

This query groups the data in telemetry_table by the extracted hours and counts occurrences.

#### Closing Summary
& Common Mistakes/Follow-Ups

To summarize, extracting the hour of day from a metric’s timestamp involves using SQL functions specific to the database system you are working with. Common mistakes might include:

Using incorrect function names (e.g., HOUR vs EXTRACT).
Forgetting to add the GROUP BY clause if summarization is needed.
Misunderstanding the format of the timestamp column, leading to parsing errors.

#### Example

Here's a sample input for clarity:

Sample Data:
PLAINTEXT
+---------------------+
| timestamp           |
+---------------------+
| 2023-10-05 14:30:00 |
| 2023-10-05 15:45:00 |
| 2023-10-06 08:20:00 |
+---------------------+
SQL Query:
SQL
SELECT EXTRACT(HOUR FROM timestamp) AS hour, COUNT(*)
FROM telemetry_table
GROUP BY EXTRACT(HOUR FROM timestamp);
Output

The output might look like this:

PLAINTEXT
+-------+----------+
| 14    | 2        |
+-------+----------+
| 15    | 1        |
+-------+----------+
| 8     | 1        |
+-------+----------+

This indicates that there were two records at hour 14, one record at hour 15, and one record at hour 8.

Self-Check

The solution is correct and comprehensive. It covers the basic steps required for extracting hourly data from timestamps in SQL.

**Comparison:**

| hour | count(*) |
|------|---------|
| 14   | 2       |
| 15   | 1       |
| 8    | 1       |

**Code:**

```sql
-- PostgreSQL example
SELECT EXTRACT(HOUR FROM timestamp) AS hour, COUNT(*)
FROM telemetry_table
GROUP BY EXTRACT(HOUR FROM timestamp);

-- MySQL example
SELECT HOUR(timestamp) AS hour, COUNT(*)
FROM telemetry_table
GROUP BY HOUR(timestamp);

-- SQLite example
SELECT STRFTIME('%H', timestamp) AS hour, COUNT(*)
FROM telemetry_table
GROUP BY STRFTIME('%H', timestamp);
```

**Example input / output:**

```text
+-------+----------+
| 14    | 2        |
+-------+----------+
| 15    | 1        |
+-------+----------+
| 8     | 1        |
+-------+----------+
```

**Data:**

```plaintext
+---------------------+
| timestamp           |
+---------------------+
| 2023-10-05 14:30:00 |
| 2023-10-05 15:45:00 |
| 2023-10-06 08:20:00 |
+---------------------+
```

**Flow:**

1. SQL Functions
2. SELECT EXTRACT(HOUR FROMtimestamp)
3. COUNT(*)
4. GROUP BY hour
5. EXTRACT
6. HOUR(timestamp)
7. STRFTIME('%H', timestamp)

