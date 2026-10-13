---
type: "practice"
difficulty: "Basic"
---

# You wrote a Python script that reads a table row by row using pandas.read_sql with a filter. How does a missing index on the filter column hurt it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you write a Python script to read rows from a database table using pandas.read_sql with a filter, and if there's no index on the filter column, the performance can be significantly impacted. This is because without an index, the database has to perform a full table scan to find matching records, which means it needs to check every row in the table. Here’s how we can break down this issue:

#### Overview
A missing index on the filter column leads to a full table scan, which increases the query time and resource usage.

#### Triage
The question is probing your understanding of database indexing and its impact on query performance. We need to assume that the dataset is large enough for the difference in performance to be noticeable. Additionally, we will consider the nature of the pandas.read_sql function and how it interacts with the database.

#### First-Principles Solution

Understanding Indexes: An index on a column allows the database to quickly find rows that match specific values without having to scan the entire table.
Full Table Scan: Without an index, every row in the table is checked to see if it matches the filter condition. This operation can become very slow for large datasets.

#### Example Scenario
Suppose you have a table with 10 million rows and want to filter by a column age. If there’s no index on age, the database will perform a full scan of all rows, which is inefficient.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Summary: The absence of an index on the filter column leads to a full table scan, making the query very slow and resource-intensive.

#### Common Mistakes
Developers often overlook the importance of indexing in large datasets or assume that indexes are automatically created. They may also think that pandas.read_sql is optimized for such operations, but it relies heavily on the database's optimization.

Closing Thoughts:

Always ensure you have appropriate indexes on columns used frequently in filters to improve performance.
Use tools like EXPLAIN PLAN or SHOW INDEXES to verify and optimize your queries.

