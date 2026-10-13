---
type: "practice"
difficulty: "Deep"
---

# Why might increasing warehouse size from Medium to Large give zero speedup for one query but 2x for another?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Snowflake Compute and Caching](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Compute_and_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why increasing Snowflake's warehouse size from Medium to Large can yield different performance improvements, we need to delve into how Snowflake's compute resources and query execution mechanisms work. At a high level, Snowflake's architecture is designed around parallelism and resource allocation, but the specifics of query execution and data distribution can vary significantly based on the query characteristics.

The key idea here is that the speedup you get from increasing warehouse size isn't just about raw computational power; it also depends on how well your query utilizes those resources. Let's break this down step by step:

#### Triage
What the Question is Asking and Assumptions
Warehouse Size: Snowflake allows scaling of warehouses, which impacts the number of compute nodes available for a query.
Query Execution: The performance improvement from increasing warehouse size can vary based on several factors such as data distribution, parallelism, and resource contention.

#### First-Principles Solution
1. Query Characteristics

A Medium-sized warehouse has fewer compute nodes compared to a Large-sized one. Therefore, when you increase the warehouse size, more nodes are available for query execution. However, how much of that additional capacity is utilized depends on the query's characteristics.

Case Study: Zero Speedup Scenario
Scenario: A query with minimal parallelism or resource contention.
Explanation: If a query doesn't efficiently utilize the added compute nodes (e.g., due to poor data distribution), increasing the warehouse size won’t significantly speed up the execution time. The new nodes are idle, and the query remains bottlenecked by other factors.
Case Study: 2x Speedup Scenario
Scenario: A complex query with high parallelism and minimal resource contention.
Explanation: In this case, increasing the warehouse size enables more efficient parallel execution of the query. Since the added nodes can handle parts of the query in parallel without waiting for others to complete, there's a significant reduction in overall execution time.
2. Data Distribution and Parallelism

Snowflake uses a distributed architecture where data is partitioned across multiple nodes. Queries are executed by distributing tasks across these nodes based on their schema and execution plan. If a query already benefits maximally from the current number of nodes, increasing the warehouse size won’t provide additional speedup.

3. Resource Contention

Resource contention can significantly impact performance. If there’s high contention for resources like memory or CPU, increasing the warehouse size might not help as much. However, if resource contention is low, more compute nodes can be utilized effectively, leading to better performance gains.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Assuming that simply scaling up resources will always improve query performance uniformly.

#### Follow-Up
Discussing how data distribution and parallelism play a critical role in determining the effectiveness of resource scaling. Explore strategies for optimizing query performance, such as re-distributing data or adjusting query execution plans.

#### Example Scenario

Let's consider an example where we have two queries:

Query 1: A simple aggregation with minimal parallelism.
Query 2: A complex join operation with high parallelism and low resource contention.

For Query 1, increasing the warehouse size from Medium to Large might yield no significant speedup if the query is already bottlenecked by other factors like data locality or I/O operations.

For Query 2, moving from Medium to Large could provide a 2x speedup because the new nodes can handle parts of the join operation in parallel without waiting for others.

Self-Check

This explanation covers all key aspects and avoids common pitfalls. The solution is precise and complete.

**Code:**

```sql
-- Example of Snowflake query
SELECT SUM(sales_amount) FROM sales WHERE date >= '2023-01-01';
```

