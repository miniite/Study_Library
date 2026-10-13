---
type: "practice"
difficulty: "Deep"
---

# A query planner switched from an index scan to a bitmap heap scan after data grew, and latency doubled. How do you decide whether the switch was correct and what you'd tune?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree and BRIN Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/B-tree_and_BRIN_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why a query planner might switch from an index scan to a bitmap heap scan and how to determine if this decision is optimal, we need to consider several factors including data distribution, query patterns, cost calculations, and performance metrics. The key here is understanding the trade-offs involved in different types of scans.

When data grows significantly, traditional B-tree indexes can become less efficient for certain queries because they require traversing multiple levels of the index to retrieve results. In contrast, a bitmap heap scan involves scanning the entire table (or a portion of it) and using bitmaps to filter out non-matching rows, which can be more efficient when the number of distinct values is small compared to the total dataset.

To decide if the switch was correct, we need to evaluate several aspects:

Query Characteristics: Analyze whether the query is predominantly range-based or point-based.
Data Distribution: Check the cardinality of columns used in the index and how well they can filter rows.
Index Overhead: Compare the cost of maintaining indexes against the benefits gained from using them for specific queries.
Performance Metrics: Monitor system-level metrics such as CPU, I/O, and memory usage to understand where bottlenecks might be.

Let’s break this down step by step:

Step 1: Understanding Index Scans

An index scan uses an index structure to retrieve data efficiently. It is typically faster when the number of rows returned is small relative to the total table size because it can directly access the relevant data without scanning the entire table. However, as the table grows, the overhead of maintaining and traversing multiple levels of the B-tree increases.

Step 2: Understanding Bitmap Heap Scans

A bitmap heap scan involves generating bitmaps for each column used in a WHERE clause to quickly identify rows that match the criteria. This is particularly useful when there are many columns with low cardinality, reducing the need to scan the entire table.

Step 3: Evaluating Query Characteristics
Range-based Queries: If the query involves filtering data over a range (e.g., WHERE age BETWEEN 18 AND 25), an index on age might be more efficient.
Point-based Queries: For queries involving exact matches or small numbers of distinct values, bitmap scans can outperform index scans.
Step 4: Data Distribution Analysis
Check the cardinality of columns involved in WHERE clauses. Low-cardinality columns (e.g., gender with only two values) are good candidates for bitmap scans.
Use statistics like histogram data to understand how well indexes can filter rows.
Step 5: Index Overhead and Maintenance Costs

Evaluate whether maintaining an index is cost-effective compared to the benefits it provides. This includes:

Index Size: Larger indexes take up more storage space.
Update Cost: Frequent updates (inserts, deletes, updates) can degrade performance.
Step 6: Performance Metrics and Monitoring
Monitor I/O operations to understand if disk reads are a bottleneck.
Look at CPU usage during query execution to identify potential bottlenecks in scan processes.

#### Common Mistakes
and Follow-ups
Over-reliance on Indexes: Sometimes, relying too heavily on indexes can lead to suboptimal performance due to increased overhead.
Ignoring Data Distribution: Failing to consider the distribution of data can result in choosing the wrong type of scan.
Not Considering Query Patterns: A one-size-fits-all approach may not be optimal; queries with different characteristics might benefit from different strategies.

To ensure the switch was correct, you would need to conduct a thorough analysis and possibly implement performance tuning steps. This could include:

Re-indexing tables with new statistics.
Using bitmap indexes for low-cardinality columns.
Optimizing query rewrite rules in the database.

#### Cons
idering partitioning or sharding strategies.

By carefully evaluating these aspects, you can make informed decisions about whether a switch to a bitmap heap scan is justified and tune the system accordingly.

**Comparison:**

| Table   | Rows     | Distinct Values |
|---------|----------|----------------|
| users   | 1,000,000| 9              |

**Code:**

```sql
-- Example SQL for creating an index and analyzing its use
CREATE INDEX idx_age ON users (age);
ANALYZE TABLE users COMPUTE STATISTICS;
EXPLAIN PLAN FOR SELECT * FROM users WHERE age BETWEEN 18 AND 25;
```

**Example input / output:**

```text
Input:
- Query: SELECT * FROM users WHERE age BETWEEN 18 AND 25;

Output (Before switch):
- Type: Index Scan
- Cost: 2500
- Rows Examined: 40,000

Output (After switch to Bitmap Heap Scan):
- Type: Bitmap Heap Scan
- Cost: 5000
- Rows Examined: 1,000,000
```

**Data:**

```json
{
  "table_stats": {
    "users": {
      "total_rows": 1000000,
      "columns": [
        {"name": "age", "cardinality": 10, "distinct_values": 9}
      ]
    }
  },
  "query_plan": {
    "type": "index_scan",
    "cost": 2500,
    "rows_examined": 40000
  }
}
```

**Flow:**

1. Range-based Queries
2. Point-based Queries
3. Yes
4. No
5. Initial Index Scan
6. Query Characteristics?
7. Index Scan
8. Bitmap Heap Scan
9. Low-cardinality Columns?
10. Create Bitmap Index andSwitch to Bitmap Heap Scan
11. Keep Index Scan
12. Monitor Performance Metrics
13. Tune Database Settings ifNecessary

