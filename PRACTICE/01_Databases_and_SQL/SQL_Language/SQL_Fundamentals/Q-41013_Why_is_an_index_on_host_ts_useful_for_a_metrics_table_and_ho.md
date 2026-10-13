---
type: "practice"
difficulty: "Intermediate"
---

# Why is an index on (host, ts) useful for a metrics table, and how does it speed up per-host time-range queries?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An index on (host, ts) in a metrics table is particularly useful for optimizing per-host time-range queries because it allows the database to quickly locate data within specific hosts and time ranges without scanning the entire table. This speeds up query performance by reducing the number of disk I/O operations required.

#### Triaging the Question

The question essentially asks two things:

Why an index on (host, ts) is beneficial for metrics tables?
How this index improves the speed of per-host time-range queries?

#### Assumptions
and Context
The metrics table contains data related to various hosts over different timestamps.
Per-host time-range queries are common operations in monitoring systems.

#### First Principles Solution

Understanding the Table Schema:

Assume a basic schema for a metrics table:
SQL
CREATE TABLE metrics (
  id INT PRIMARY KEY,
  host VARCHAR(255),
  ts TIMESTAMP,
  metric_value DECIMAL
);

Query Optimization with Indexes:

An index on (host, ts) creates an ordered structure where the database can quickly locate records based on host and ts.
When querying data for a specific host within a time range, the database can efficiently use this index to pinpoint the relevant rows.

Indexing Logic:

The index on (host, ts) allows the database to first filter by host, then further refine by ts. This minimizes the number of disk reads required.
For example, if you're querying data for host 'h1' between timestamps t1 and t2, the index helps in skipping irrelevant rows quickly.
Step-by-Step Explanation
1. Query Execution Without Index

Without an index, a query like SELECT * FROM metrics WHERE host = 'h1' AND ts BETWEEN 't1' AND 't2' would require scanning the entire table to find matching records, which can be time-consuming for large datasets.

2. Query Execution With Index

With an index on (host, ts), the query execution plan changes:

The database starts by using the index to filter rows based on host = 'h1'.
It then narrows down further to records within the specified timestamp range.
This significantly reduces the number of disk accesses needed.
Practical Example

Let's assume we have 10,000 hosts and each host generates metrics every second. Without an index, scanning the table for a specific host might require examining millions of rows. With the index, the database can quickly narrow down to just the relevant host records.

Sample Data Schema
SQL
CREATE TABLE metrics (
   id INT PRIMARY KEY,
   host VARCHAR(255),
   ts TIMESTAMP,
   metric_value DECIMAL
);

#### Common Mistakes
and Follow-Ups

Incorrect Index Design:

Sometimes, developers might think an index on (ts, host) would be better for time-range queries. However, this can lead to slower performance because the database has to filter by ts first before applying the host condition.

Not Considering Other Columns:

Ensure that additional columns in the query (if any) are also covered by the index or have their own indexes.

Index Size and Maintenance:

Large tables with frequent insertions/updates might require more sophisticated indexing strategies to balance performance and maintenance overhead.

#### Closing Summary

In summary, an index on (host, ts) is highly beneficial for optimizing per-host time-range queries in metrics tables by reducing the number of disk accesses required. This leads to faster query execution times, which is crucial for real-time monitoring systems.

**Code:**

```sql
CREATE TABLE metrics (
   id INT PRIMARY KEY,
   host VARCHAR(255),
   ts TIMESTAMP,
   metric_value DECIMAL
);
```

**Flow:**

1. Yes
2. No
3. Start Query
4. Is (host, ts) Index Present?
5. Use (host, ts) Index to FilterRows
6. Retrieve Specific Data withinTime Range
7. Scan Entire Table for MatchingRecords

