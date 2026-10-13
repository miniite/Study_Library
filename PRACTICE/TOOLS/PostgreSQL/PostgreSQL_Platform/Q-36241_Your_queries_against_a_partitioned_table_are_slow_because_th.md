---
type: "practice"
difficulty: "Deep"
---

# Your queries against a partitioned table are slow because the planner locks and inspects every partition at plan time; how does runtime partition pruning differ from plan-time pruning and when does each kick in?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Retention Purging at Scale](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Retention_Purging_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with partitioned tables in PostgreSQL, it's crucial to understand both plan-time and runtime partition pruning because they significantly impact query performance. Plan-time pruning is examined at the query planning stage where the database decides which partitions can be excluded from the execution plan based on predicate conditions. Runtime partition pruning occurs during actual data processing and excludes partitions that do not need to be scanned even if the planner hasn't ruled them out.

To illustrate, consider a time-based partitioned table for order history with daily partitions named orders_YYYYMMDD. If you run a query filtering orders from 2023-10-01 to 2023-10-05, plan-time pruning will check the conditions and exclude partitions outside this range. However, runtime partition pruning ensures that only the necessary partitions are actually read during execution.

Let's break down how each works:

Plan-Time Pruning

Plan-time pruning is controlled by PostgreSQL's query planner. The key configuration parameter here is enable_partition_pruning. If set to on, PostgreSQL will attempt to prune partitions based on the conditions in your WHERE clause. For example, if you have a predicate like order_date >= '2023-10-01' AND order_date <= '2023-10-05', the planner can rule out all other partitions.

Runtime Partition Pruning

Runtime partition pruning is more fine-grained and happens during query execution. It uses the same conditions to exclude partitions that do not contain relevant data, but these checks occur at runtime after the plan has been decided. This ensures that even if the planner makes a mistake or misses some optimizations, the actual execution will still be efficient.

Practical Scenario

#### Cons
ider the following example:

SQL
-- Query with plan-time pruning enabled and predicate conditions
EXPLAIN ANALYZE SELECT * FROM orders WHERE order_date >= '2023-10-01' AND order_date <= '2023-10-05';

In this query, both plan-time and runtime partition pruning can operate. The planner will analyze the predicate conditions to determine which partitions are relevant, and then during execution, it will further verify that no unnecessary partitions are read.

Key Differences

Plan-time pruning happens before any data is fetched, whereas runtime pruning occurs after the query plan has been decided but before actual data processing starts. Plan-time pruning can be more effective in reducing I/O operations by ruling out entire partitions, while runtime pruning ensures that only relevant data is processed even if initial conditions are not fully accurate.

#### Common Mistakes
and Follow-Ups
Misconfiguration of enable_partition_pruning: This can lead to suboptimal plans where unnecessary partitions are included.
Inadequate predicate conditions: Weak or insufficient WHERE clause conditions might prevent effective pruning, causing all partitions to be scanned.
Schema changes not reflected in planner settings: If you add new partitions or remove old ones, ensure that the query planner is updated accordingly.

Understanding both plan-time and runtime partition pruning helps in optimizing queries for large, partitioned tables. It's essential to monitor query performance and adjust configurations as needed to achieve the best balance between planning efficiency and execution speed.

**Code:**

```sql
-- Example query with EXPLAIN ANALYZE
EXPLAIN ANALYZE SELECT * FROM orders WHERE order_date >= '2023-10-01' AND order_date <= '2023-10-05';
```

