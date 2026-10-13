---
type: "practice"
difficulty: "Deep"
---

# A team proposes bitmap indexes to speed up dashboard filters on a table taking 5k writes/sec — how do you evaluate the concurrency risk and steer them to the right structure without blocking their launch?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

Bitmap indexes can be incredibly efficient in certain scenarios, but they come with specific trade-offs, particularly in terms of concurrency. Evaluating whether bitmap indexes are suitable requires a detailed analysis of the use case and the potential risks to concurrent operations. By understanding the underlying mechanics and proposing alternative solutions that align with the team's goals, we can steer them towards a more balanced approach.

#### Triaging the Question

The core issue here is evaluating the concurrency risk associated with implementing bitmap indexes on a high-write-rate table (5k writes/sec) for dashboard filters. The key questions are:

Will the indexes cause contention during write operations?
Are there better index structures that can offer similar performance benefits without increasing write latency?
How do we ensure the solution is scalable and doesn’t block future growth or other workloads?

#### First-Principles Solution
Understanding Bitmap Indexes

Bitmap indexes store a bit vector for each unique value in the indexed column, which allows for very fast lookups but can become problematic during write operations due to their sequential nature. Writes may need to update multiple rows' bitmap entries simultaneously, leading to lock contention.

Concurrency Risks
Write Contention: Bitmaps are updated sequentially and concurrently writing to the same bit vector can lead to significant locking.
Locking Overhead: Increased locking can degrade write performance, especially in a high-write environment like 5k writes/sec.
Index Maintenance: Frequent updates to bitmap indexes can increase overhead during table maintenance operations.
Alternative Index Structures
Covering Indexes: These include all columns needed by the query in the index itself, reducing the need for additional disk seeks and improving overall read performance.
Composite Indexes: Combining multiple columns into a single index can optimize reads based on common filter patterns.
Materialized Views: Precomputing aggregates or filtered data into a separate table can reduce the load on the main table during dashboard queries.
Step-by-Step Solution

Initial Assessment

Understand the specific query patterns and the columns used in filters.
Estimate the number of unique values per indexed column to determine potential bitmap size and write impact.

Concurrent Write Analysis

Simulate or profile the current write operations to understand contention points.
Use tools like pg_stat_activity (for PostgreSQL) to monitor locking during writes.

Write-Optimized Index Evaluation

Propose a covering index that includes all necessary columns for dashboard queries.
Test and measure the performance impact of this index on write operations.

Comprehensive Testing

Set up a testing environment with realistic data and query patterns to simulate production behavior.
Monitor both read and write performance over time to identify any degradation.

Risk Mitigation Strategies

#### Cons
ider using more advanced features like CREATE INDEX CONCURRENTLY if the database supports it, to reduce locking during index creation.
Explore partial indexing or incremental updates for large datasets.

Scaling and Performance Tuning

Ensure that other aspects of the database are optimized (e.g., buffer pool size, I/O throughput).

#### Cons
ider sharding or partitioning strategies if the table is expected to grow significantly.

#### Closing Summary

By thoroughly evaluating the concurrency implications of bitmap indexes and proposing alternative structures like covering or composite indexes, we can ensure that both read and write performance are maintained. This approach aligns with best practices in database design and avoids over-reliance on a single solution.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring concurrent write issues by solely focusing on read performance.

#### Follow-Up
Conduct thorough concurrency testing before deployment.
Mistake: Overoptimizing for indexes without considering overall database architecture.

#### Follow-Up
Balance index usage with other optimizations like query planning and data distribution.

**Comparison:**

```
Index Type	Pros	Cons
Bitmap	Fast reads	High write contention, locking overhead
Covering	Read efficiency	Increased space usage
Composite	Optimized for specific cases	Limited flexibility compared to bitmap
```

**Code:**

```sql
-- Example of creating a covering index in PostgreSQL
CREATE INDEX idx_dashboard_filters ON table_name (column1, column2) INCLUDE (column3, column4);
```

**Example input / output:**

```text
-- Example dry-run output from a test environment

Initial Write Operations (5k/sec):
- Time: 10 seconds, Lock Contention: Low, Overall Performance: Good

Covering Index Creation and Optimization:
- Time: 2 minutes, Lock Contention: High during creation, Overall Performance: Stable after

Final Read Performance Test (Dashboard Filters):
- Time: 5 milliseconds, Lock Contention: Minimal, Overall Performance: Excellent
```

**Data:**

```json
{
  "table": {
    "name": "sales_data",
    "columns": [
      { "name": "product_id", "type": "integer" },
      { "name": "customer_id", "type": "integer" },
      { "name": "sale_date", "type": "timestamp" }
    ],
    "indexes": [
      { "name": "idx_covering", "columns": ["product_id", "customer_id"], "include": ["sale_date"] }
    ]
  }
}
```

**Flow:**

1. Initial Assessment
2. Simulate Write Operations
3. Analyze Concurrency Risks
4. Evaluate Covering Index
5. Test and Optimize
6. Mitigate Risk
7. Implement Solution

