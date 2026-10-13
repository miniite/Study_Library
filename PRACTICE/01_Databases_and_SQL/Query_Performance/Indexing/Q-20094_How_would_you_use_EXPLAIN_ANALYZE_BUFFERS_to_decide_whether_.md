---
type: "practice"
difficulty: "Deep"
---

# How would you use EXPLAIN (ANALYZE, BUFFERS) to decide whether a slow query is limited by disk reads or by CPU-bound row processing?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN ANALYZE](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose whether a slow query is CPU-bound or I/O-bound, you can use EXPLAIN ANALYZE and EXPLAIN (ANALYZE, BUFFERS) in PostgreSQL to gather detailed execution plans and performance metrics. These tools help identify where the bottleneck lies by providing insights into both the row processing overhead and disk read operations.

The key idea here is that EXPLAIN (ANALYZE, BUFFERS) gives us a comprehensive view of the query's execution plan, showing how many rows are processed and how much time is spent on I/O operations. Let’s break this down step by step:

Understanding the Question: The core question is to determine if a slow query is primarily limited by disk reads (I/O-bound) or CPU usage during row processing.

#### Assumptions
We assume that the database system in question has EXPLAIN and EXPLAIN ANALYZE capabilities, and we are working with PostgreSQL.

Let’s dive into how to use these commands effectively:

Step 1: Prepare the Query

First, identify the slow query you want to analyze. For example:

SQL
SELECT * FROM customers WHERE name LIKE '%Smith%' LIMIT 10;
Step 2: Run EXPLAIN with ANALYZE and BUFFERS

Run the following command in PostgreSQL:

SQL
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM customers WHERE name LIKE '%Smith%' LIMIT 10;

This will provide a detailed execution plan that includes buffer hit statistics and actual query execution time.

Step 3: Analyze the Output

The output from EXPLAIN (ANALYZE, BUFFERS) provides several key pieces of information:

Plan: Shows the execution plan broken down into steps.
Buffers: Details how many rows were read from which buffers (e.g., shared_buffers or disk).
Actual Time: The time taken for each step.
Step 4: Interpretation

To determine if the query is I/O-bound:

High Buffer Hit Rate: If most rows are fetched directly from shared memory, the query is likely CPU-bound.
High Disk Reads/Shared Hit Ratio: If many buffer hits come from disk reads, it indicates that the query might be I/O-bound.

For example:

PLAINTEXT
QUERY PLAN
Seq Scan on customers (cost=0.00..123456.78 rows=10 width=100)
  ->  Seq Scan on customers (cost=0.00..123456.78 rows=10 width=100)
        Filter: (name LIKE '%Smith%')
          Buffers: shared hit=100, read=123, dirtied=5

In this example, if read is significantly higher than hit, it suggests that the query is disk I/O bound.

Step 5: Identify Bottlenecks and Solutions
Disk Bound: If Buffers: shared hit=low, read=high, consider adding indexes or optimizing the schema.
CPU Bound: If Buffers: shared hit=high, read=low, ensure that the query is optimized for CPU usage (e.g., fewer rows processed).

#### Closing Summary

By using EXPLAIN (ANALYZE, BUFFERS), you can accurately diagnose whether a slow query is due to I/O operations or CPU-bound row processing. This helps in making informed decisions about database schema optimization and performance tuning.

#### Common Mistakes/Follow-Ups
Overlooking Buffer Statistics: Always check both hit and read statistics.
Ignoring Plan Details: The plan details can provide insights into where the query is spending most of its time.
Not Considering Indexes: Ensure that appropriate indexes are in place to reduce disk reads.

**Code:**

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM customers WHERE name LIKE '%Smith%' LIMIT 10;
```

