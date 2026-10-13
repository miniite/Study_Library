---
type: "practice"
difficulty: "Deep"
---

# A managed Aurora/RDS Postgres shows high IOPS and cost; how do you attribute it to specific queries and reduce it without just scaling the instance up?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Finding Expensive Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Finding_Expensive_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of high IOPS and costs in a managed PostgreSQL (Aurora/RDS) environment, we need to understand how certain queries are driving these resource demands. By analyzing query performance and optimizing specific queries, we can reduce overall cost without simply scaling up the instance. This process involves several steps: profiling, identifying problematic queries, and then applying optimizations.

Firstly, let's reframe the question by understanding what the interviewer is probing:

Profiling: We need to gather detailed information about query performance.
Attribution: Identify which queries are causing high IOPS and why.
Optimization: Apply changes that reduce resource usage without compromising performance.

We'll assume we have access to the PostgreSQL query logs, slow query logs, and possibly a tool like pg_stat_statements for detailed analysis. These tools provide us with metrics such as execution time, lock waiting times, and I/O wait times.

Step-by-Step Solution

Gather Query Performance Data:

Use pg_stat_statements to collect performance data on queries.
SQL
SELECT query, calls, total_time, rows_returned FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
Check the slow query logs in Aurora/RDS for any time-consuming queries.

Identify Problematic Queries:

Analyze the collected data to find queries that are causing high IOPS.
Focus on long-running, frequent, or resource-intensive queries.

Optimize Queries:

Indexing: Ensure that queries have appropriate indexes. Lack of proper indexing can lead to full table scans and high I/O usage.
SQL
CREATE INDEX idx_name ON your_table (column1, column2);
Query Refactoring: Modify the query logic to be more efficient, such as using joins instead of subqueries where possible.
Batch Processing: For large-scale data operations, consider breaking down queries into smaller batches.

Monitor and Validate Changes:

After making changes, monitor the performance again using pg_stat_statements or slow query logs to verify improvements.
Ensure that optimizations do not introduce new bottlenecks.

#### Closing Summary

By systematically profiling, identifying, and optimizing problematic queries, we can significantly reduce IOPS and overall costs without just scaling up the instance. This approach ensures a more sustainable and cost-effective solution while maintaining performance.

#### Common Mistakes/Follow-Ups
Focusing Only on High-Cost Queries: Sometimes, even low-cost but frequently executed queries can cause high IOPS if they are read-intensive.
Over-Optimizing: Avoid overengineering the database schema or queries. Balancing between simplicity and performance is key.
Ignoring Index Maintenance: Ensure that indexes are maintained regularly to keep them efficient.

**Comparison:**

| Query | Calls | Total Time (s) | Rows Returned |
|-------|-------|----------------|---------------|
| A     | 100   | 365            | 125           |
| B     | 80    | 97             | 546           |

**Code:**

```sql
-- Example of creating an index for a high-cost query
CREATE INDEX idx_customer_order ON orders (customer_id, order_date);
```

**Example input / output:**

```text
Input:
SELECT * FROM customer WHERE name LIKE '%John%';

Output:
Time: 12.3 ms, Rows returned: 10

Optimized Query:
CREATE INDEX idx_customer_name ON customer (name);

Re-run Output:
Time: 1.5 ms, Rows returned: 10
```

**Data:**

```json
{
  "queries": [
    {
      "query": "SELECT * FROM customer WHERE name LIKE '%John%'",
      "calls": 100,
      "total_time": 365.24,
      "rows_returned": 125
    },
    {
      "query": "SELECT order_id, SUM(quantity) as total_quantity FROM order_items GROUP BY order_id",
      "calls": 80,
      "total_time": 97.3,
      "rows_returned": 546
    }
  ]
}
```

**Flow:**

1. Profile Queries
2. Gather Data
3. Identify Problematic Queries
4. Optimize Queries (Indexing,Refactoring)
5. Maintain Indexes
6. Monitor Performance

