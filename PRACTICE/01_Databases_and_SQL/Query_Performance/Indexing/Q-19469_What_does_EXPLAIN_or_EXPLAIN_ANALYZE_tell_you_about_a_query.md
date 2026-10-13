---
type: "practice"
difficulty: "Intermediate"
---

# What does EXPLAIN (or EXPLAIN ANALYZE) tell you about a query?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN ANALYZE](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

EXPLAIN and EXPLAIN ANALYZE are powerful tools in PostgreSQL that help you understand how a query is being executed by the database engine. By examining the execution plan, you can identify bottlenecks, optimize performance, and make informed decisions to improve query efficiency.

#### Triaging the Question

The question asks what EXPLAIN (or EXPLAIN ANALYZE) tells us about a query. The interviewer is probing your understanding of:

Execution Plan: How PostgreSQL interprets and executes a SQL statement.
Performance Insights: Identifying which parts of the query are slow or inefficient.
Query Optimization Techniques: Strategies to enhance query performance.

#### First-Principles Solution

Let's break down what EXPLAIN (or EXPLAIN ANALYZE) reveals:

Query Plan:

Logical Plan: The order in which PostgreSQL will process the data.
Physical Plan: How the database physically accesses and manipulates the data.

Cost Estimation:

Estimated cost of each operation (in units of CPU time).
Helps you understand where the query spends most of its resources.

Workload Analysis:

Number of rows processed at each step.
Tells us about the overall workload and whether it is balanced or skewed.

Index Utilization:

Whether indexes are being used effectively.
Helps in understanding if your indexes are designed well for the queries they serve.

Parallelism and Concurrency:

Number of parallel workers used by PostgreSQL to execute the query.
Insights into how well the database is utilizing available resources.

Sort Operations:

Whether sorting is being done at each step.
Helps in identifying unnecessary or expensive sorts.

#### Example
Output

Let's consider a simple query and its EXPLAIN output:

SQL
EXPLAIN SELECT * FROM customers WHERE age > 30 AND city = 'New York';

The plan might look something like this:

PLAINTEXT
->  Index Scan using idx_customers_age on customers  (cost=100.00..5000.00 rows=200 width=100)
        Filter: ((age > 30) AND (city = 'New York'::text))

#### Closing Summary

In summary, EXPLAIN and EXPLAIN ANALYZE are invaluable tools for backend developers to gain insights into query execution plans. They help in identifying performance bottlenecks, optimizing indexes, and fine-tuning database queries.

#### Common Mistakes/Follow-Ups
Misinterpreting Costs: Sometimes, the cost numbers can be misleading without context.
Overlooking Indexes: Assuming that every index is being used effectively when EXPLAIN shows otherwise.
Ignoring Filter Conditions: Not realizing why certain conditions are causing performance issues if the filter isn’t being applied efficiently.

#### Follow-ups
could include:

Discussing how to use EXPLAIN to troubleshoot a specific performance issue.
Explaining different types of index structures (B-tree, GiST, SP-GiST) and their impact on query plans.
Describing techniques for rewriting queries to improve performance based on the EXPLAIN output.

**Code:**

```sql
EXPLAIN SELECT * FROM customers WHERE age > 30 AND city = 'New York';
```

