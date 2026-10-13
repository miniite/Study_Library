---
type: "practice"
difficulty: "Deep"
---

# How do you decide when to push filtering into a bitmap/columnar layer at the storage engine versus keeping composite B-tree indexes at the table for a hybrid HTAP workload?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of deciding whether to push filtering into a bitmap or columnar layer in a storage engine or maintain composite B-tree indexes, we need to consider several key aspects such as query patterns, data distribution, performance metrics, and system constraints. This decision involves understanding how different indexing strategies impact query execution, resource utilization, and overall database performance.

The core idea is that composite B-tree indexes are designed for efficient row lookups based on multiple columns, while bitmap indexes or columnar storage are optimized for filtering large datasets and aggregating data efficiently. Pushing filtering into the lower layers (like a bitmap or columnar layer) can reduce memory usage and improve query performance by reducing the amount of data transferred to higher layers.

#### Triaging the Question
Query Patterns: Analyze whether your queries involve simple single-column filters, complex multi-column filters, or large-scale aggregations.
Data Distribution: Understand how data is distributed across columns; this impacts which type of index would be more effective.
Performance Metrics: Monitor and evaluate the performance metrics (e.g., I/O operations, memory usage, CPU time) to understand where bottlenecks occur.
System Constraints: Consider the limitations of your storage engine in terms of memory, disk I/O, and processing power.

#### First-Principles Solution
Simple Single-Column Filters:
B-tree Indexes: These are typically more efficient for single-column filters due to their balanced nature, providing quick row lookups.
Complex Multi-Column Filters or Large-Scale Aggregations:
Bitmap Indexes/Columnar Storage: These offer significant advantages in handling complex filters and aggregations by reducing the amount of data that needs to be processed at higher levels.
Step-by-Step Solution
Data Analysis:
Analyze query logs to identify common filter conditions.
Indexing Strategy Assessment:
Evaluate if composite B-tree indexes can meet performance requirements for complex filters.
Benchmarking:
Set up benchmark tests with various indexing strategies and measure execution times, memory usage, and I/O operations.
Performance Tuning:
Optimize indexing based on the results of the benchmark tests to balance between query speed and resource utilization.

#### Closing Summary

In summary, deciding whether to push filtering into a bitmap or columnar layer involves a careful analysis of the specific use case, data characteristics, and performance requirements. B-tree indexes are generally better for simple single-column filters, while bitmap indexes and columnar storage excel in complex multi-column filters and large-scale aggregations. The key is to balance between query speed and resource utilization.

#### Common Mistakes/Follow-Ups
Over-indexing: Creating too many indexes can lead to increased maintenance overhead and slower write performance.
Ignoring Query Patterns: Failing to analyze real-world query patterns may result in suboptimal indexing strategies.
Not Considering Memory Constraints: Pushing filtering into the storage engine might exacerbate memory bottlenecks if not properly managed.

**Comparison:**

| Query Type | B-tree Index Usage | Columnar Storage Usage |
|------------|-------------------|-----------------------|
| Simple Filter | High performance, low memory | Low performance, high memory |
| Complex Filter | Moderate to low performance | High performance, low memory |

**Code:**

```sql
-- Example of creating a composite B-tree index
CREATE INDEX idx_composite ON table_name(column1, column2);

-- Example of using bitmap indexes (assuming support in the database)
CREATE BITMAP INDEX idx_bitmap ON table_name(column1);
```

**Example input / output:**

```text
Input:
- Query: SELECT * FROM transactions WHERE transaction_date > '2023-01-01' AND amount >= 500;

Output:
- Execution plan using composite B-tree index:
  - I/O operations: 10,000
  - Memory usage: 5 MB
  - CPU time: 0.5 seconds

- Execution plan pushing filtering to columnar storage:
  - I/O operations: 2,000
  - Memory usage: 100 KB
  - CPU time: 0.1 seconds
```

**Data:**

```json
{
  "query": {
    "columns": ["customer_id", "transaction_date", "amount"],
    "filters": [
      {"column": "transaction_date", "value": "2023-01-01"},
      {"column": "amount", "operation": ">=", "value": 500}
    ]
  },
  "index_strategy": {
    "composite_btree": true,
    "bitmap_index": false
  }
}
```

**Flow:**

1. Analyze Query Patterns
2. Create Composite B-tree Index
3. Identify Complex Filters
4. Create Bitmap Index/ColumnarStorage
5. Benchmark Performance
6. Tune and Optimize IndexingStrategy
7. Evaluate Resource Utilization
8. Monitor and Adjust

