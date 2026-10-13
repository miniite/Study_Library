---
type: "practice"
difficulty: "Deep"
---

# How would you index for a query with both an equality filter and a range filter, like status='open' AND created_at > X?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize a query that combines an equality filter (status='open') and a range filter (created_at > X), you need to understand how these filters interact with indexes in databases. The key idea is to design composite indexes that can efficiently cover both parts of the query, ensuring fast retrieval while minimizing storage overhead.

The question probes your understanding of:

Index Selection: Choosing the right type and structure for an index.
Query Optimization Techniques: How to leverage database indexing for complex queries.
Performance Trade-offs: Balancing between storage space and query performance.

Let’s break down the solution step-by-step:

Step 1: Analyze the Query

The query status='open' AND created_at > X requires filtering based on two criteria:

A categorical value (status) that can be indexed.
A timestamp field (created_at) that supports range queries.
Step 2: Index Design

To optimize this query, you should consider creating a composite index. A composite index is an index containing multiple columns in a specific order, which can help the database engine to quickly locate records matching both conditions.

Composite Index Example:
SQL
CREATE INDEX idx_status_created_at ON my_table (status, created_at);

This index would be useful for queries where status is used first and then the range on created_at follows. However, if the query pattern typically starts with created_at, an alternative composite index could be:

SQL
CREATE INDEX idx_created_at_status ON my_table (created_at, status);
Step 3: Query Execution Plan

To understand how the database uses these indexes, you can analyze the execution plan of a sample query. Here’s an example using SQL:

SQL
EXPLAIN SELECT * FROM my_table WHERE status = 'open' AND created_at > X;

The output will show whether the index is being used and how efficiently.

Step 4: Considerations for Different Query Patterns
Most Common Query Pattern: If most queries start with status='open', use the first index.
Random Data Access: For a mix of data access, create an index that considers common usage patterns. You might need to experiment and see which configuration performs better.
Step 5: Index Maintenance

Indexes come at a cost in terms of storage and maintenance (like index rebuilds during inserts/updates). Ensure the chosen index aligns with the expected query patterns and avoid over-indexing, which can degrade write performance.

#### Closing Summary
& Common Mistakes

Optimizing queries involving both equality and range filters requires careful consideration. A composite index on (status, created_at) or (created_at, status) is often effective but depends on the actual usage pattern. Over-indexing can lead to increased storage costs and slower write operations. Always validate with a realistic workload using EXPLAIN output.

#### Common Mistakes
Over-indexing: Creating too many indexes without considering the query distribution.
Ignoring Index Maintenance Costs: Failing to account for the overhead of maintaining indexes during data changes.

**Code:**

```sql
CREATE INDEX idx_status_created_at ON my_table (status, created_at);
CREATE INDEX idx_created_at_status ON my_table (created_at, status);
```

**Example input / output:**

```plaintext
EXPLAIN SELECT * FROM my_table WHERE status = 'open' AND created_at > X;
id	select_type	table	type	possible_keys	key	key_len	ref	rows	Extra
1	SIMPLE	my_table	range	idx_status_created_at	idx_status_	767	NULL	5000	Using where

This indicates the use of the composite index.
```

**Flow:**

1. Yes
2. No
3. Query: status='open' ANDcreated_at > X
4. Is index (status, created_at)used?
5. Index used for range query on`created_at`
6. Consider creating compositeindex (created_at, status)
7. Query executed efficiently
8. Create new index and validatewith EXPLAIN

