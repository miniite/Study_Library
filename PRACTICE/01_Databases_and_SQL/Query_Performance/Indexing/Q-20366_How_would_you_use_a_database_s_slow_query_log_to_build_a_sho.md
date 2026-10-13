---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a database's slow query log to build a shortlist of tables in your pipeline that most need an index review?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Index Tuning in Practice](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To identify which tables in your pipeline most need an index review, we can leverage a database’s slow query log by analyzing the queries causing performance bottlenecks. The goal is to prioritize indexes that will significantly improve query performance. Here's how you would approach this:

TL;DR

You'll first extract relevant information from the slow query log, then analyze query frequency and execution time. By focusing on high-frequency, long-running queries, you can identify tables needing indexing.

#### Triaging the Question

The question is asking for a method to use a database’s slow query log to prioritize tables that need index optimization. The key aspects include:

Understanding Query Performance: Identifying which queries are causing delays.
Analyzing Query Frequency and Execution Time: Determining which tables are involved in frequent, long-running queries.
Prioritizing Tables for Index Review: Focusing on critical tables to improve overall system performance.

#### Assumptions
You have access to the slow query log and can parse it.
The database supports querying and analyzing logs.
There is a reasonable amount of data in the slow query log to draw meaningful insights.
Step-by-Step Solution from First Principles
1. Extracting Relevant Information from Slow Query Log

Start by parsing the slow query log to extract fields such as:

query_id: Unique identifier for each query.
table_name: Name of the table involved in the query.
execution_time: Time taken by the query (in milliseconds).
query_frequency: How often a particular query is run.
2. Analyzing Query Frequency and Execution Time

Use SQL to aggregate and summarize this data:

SQL
SELECT
    table_name,
    COUNT(query_id) AS query_frequency,
    AVG(execution_time) AS avg_execution_time
FROM slow_query_log
GROUP BY table_name;

This query will give you a list of tables with their frequency of execution and average execution time.

3. Identifying Critical Tables

Next, identify the critical tables by filtering out those that are both frequently used and have high execution times:

SQL
SELECT
    table_name,
    COUNT(query_id) AS query_frequency,
    AVG(execution_time) AS avg_execution_time
FROM slow_query_log
GROUP BY table_name
HAVING avg_execution_time > [threshold] AND query_frequency > [threshold];

You can set thresholds based on the average execution time and frequency to filter out less critical queries. For example, if you consider an average execution time of over 100ms and a frequency greater than 50 as critical:

SQL
SELECT
    table_name,
    COUNT(query_id) AS query_frequency,
    AVG(execution_time) AS avg_execution_time
FROM slow_query_log
GROUP BY table_name
HAVING avg_execution_time > 100 AND query_frequency > 50;
4. Prioritizing Tables for Index Review

Based on the results, prioritize tables that have both high execution time and frequent usage:

PLAINTEXT
Table A: Query Frequency = 60, Avg Execution Time = 200ms
Table B: Query Frequency = 80, Avg Execution Time = 150ms
Table C: Query Frequency = 90, Avg Execution Time = 300ms

#### Closing Summary
and Common Mistakes/Follow-Ups

By analyzing the slow query log, you can identify tables that need index optimization. The key steps are:

Parsing and aggregating data from the slow query log.
Filtering out critical queries based on frequency and execution time.
Prioritizing tables for manual review.

#### Common mistakes
include:

Ignoring low-frequency high-time queries or vice versa.
Setting overly strict thresholds which might miss important insights.
Not considering other factors like query complexity, cardinality of the table, etc., when making decisions.

For follow-ups, consider implementing real-time monitoring and automated index management tools to continuously improve performance. Also, ensure that any changes made due to indexing do not negatively impact existing queries or introduce new issues.

**Comparison:**

| table_name | query_frequency | avg_execution_time |
|------------|-----------------|-------------------|
| Table A    | 60              | 200               |
| Table B    | 80              | 150               |
| Table C    | 90              | 300               |

**Code:**

```sql
-- Example SQL Queries for Analysis

-- Extracting Query Frequency and Execution Time
SELECT
    table_name,
    COUNT(query_id) AS query_frequency,
    AVG(execution_time) AS avg_execution_time
FROM slow_query_log
GROUP BY table_name;

-- Identifying Critical Tables
SELECT
    table_name,
    COUNT(query_id) AS query_frequency,
    AVG(execution_time) AS avg_execution_time
FROM slow_query_log
GROUP BY table_name
HAVING avg_execution_time > 100 AND query_frequency > 50;
```

**Example input / output:**

```text
Input:
SELECT
    table_name,
    COUNT(query_id) AS query_frequency,
    AVG(execution_time) AS avg_execution_time
FROM slow_query_log
GROUP BY table_name;

Output:
+------------+-----------------+-------------------+
| table_name | query_frequency | avg_execution_time |
+------------+-----------------+-------------------+
| Table A    | 60              | 200               |
| Table B    | 80              | 150               |
| Table C    | 90              | 300               |
+------------+-----------------+-------------------+
```

**Data:**

```plaintext
table: slow_query_log
fields:
- query_id (INT)
- table_name (TEXT)
- execution_time (INT)  # in milliseconds
- timestamp (DATETIME)
```

**Flow:**

1. Start
2. Extract query log
3. Aggregate data: frequency,execution time
4. Filter critical queries
5. Prioritize tables for review
6. Implement indexing changes
7. Monitor performanceimprovement

