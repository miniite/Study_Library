---
type: "practice"
difficulty: "Deep"
---

# A query does GROUP BY on an unindexed high-cardinality column and hashes in memory, spilling to disk. When does an index avoid the hash aggregate and when is the hash still better?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a database query involves a GROUP BY operation on a high-cardinality (many unique values) column, it can be challenging to optimize effectively. Let's explore when an index can avoid using a hash aggregate and when a hash aggregate remains more efficient.

#### Triaging the Question

The core of this question is about understanding when using indexes versus in-memory hashing for GROUP BY operations on high-cardinality columns. The key aspects are:

High Cardinality: Columns with many unique values.
Hash Aggregate: Using a hash table to aggregate results, potentially spilling to disk if the data exceeds memory capacity.
Index Usage: Whether an index can be leveraged to avoid the hash aggregation.

#### Assumptions
and First Principles
The query is using GROUP BY on a high-cardinality column.
Hash aggregates are performed in memory but may spill to disk if memory constraints are exceeded.
Indexes could potentially help by reducing the number of rows processed, thus alleviating memory pressure.
Step-by-Step Solution
1. Understanding High Cardinality

In high-cardinality columns, there are many unique values, which means that the GROUP BY operation will have to process a large number of distinct keys. This can lead to significant memory usage when using hash aggregates because each key might require its own entry in the hash table.

2. Index Usage for Reducing Memory Pressure

An index on the high-cardinality column could be used to reduce the number of rows that need to be processed by GROUP BY. If the indexed values can directly help in grouping, fewer total rows would need to be hashed and aggregated, potentially reducing memory usage.

3. Memory Considerations
Hash Aggregate Memory Requirements: The amount of memory required for a hash aggregate depends on the number of distinct keys. For high-cardinality columns, this could easily exceed available memory.
Index Efficiency: If an index can significantly reduce the number of rows (by pruning non-relevant entries), it might make in-memory aggregation feasible.
4. When Indexes Avoid Hash Aggregates

An index would avoid hash aggregates if:

The index allows for efficient filtering and direct access to groups, reducing the overall row count.
The filtered result set is small enough to fit into memory without spilling to disk.
5. When Hash Aggregates Remain Better

A hash aggregate might still be more efficient if:

The number of unique keys is too large for any practical index (e.g., millions or billions).
Efficient indexing cannot significantly reduce the row count due to the nature of the data distribution.
Memory constraints are severe, and the data set is massive.

#### Example Scenario

#### Cons
ider a scenario where we have a table orders with a column customer_id, which has high cardinality (millions of unique values). We want to perform:

SQL
SELECT COUNT(*)
FROM orders
GROUP BY customer_id;
Without Index: The query might require a hash aggregate in memory, possibly spilling to disk.
With Index on customer_id: If the index can filter out many irrelevant rows, the number of rows processed could be significantly reduced. This reduces memory pressure and avoids spilling.

#### Closing Summary

In summary, an index can avoid using a hash aggregate when it effectively filters down the row count to a manageable level that fits into memory. However, if the cardinality is too high or filtering cannot drastically reduce the data volume, a hash aggregate might still be more efficient despite potential disk spills.

#### Common Mistakes
and Follow-Ups
Ignoring Memory Constraints: Overlooking the critical role of available memory can lead to suboptimal query plans.
Failing to Consider Index Filtering Effectiveness: Not understanding how well an index filters out irrelevant rows can misjudge the utility of indexing.
Overlooking Disk Spills: Assuming that a hash aggregate must always be avoided without considering the cost of disk I/O operations.
Further Exploration

For more complex scenarios, consider:

Using materialized views or pre-aggregated tables to reduce the load on live queries.
Evaluating the use of distributed databases or data warehousing techniques for very high cardinality datasets.

**Comparison:**

| Column | Type       |
|--------|------------|
| order_id | INT        |
| customer_id | VARCHAR(255)  |
| product_id | INT        |
| quantity | INT        |

**Code:**

```sql
-- Example SQL Query
SELECT COUNT(*)
FROM orders
GROUP BY customer_id;
```

**Example input / output:**

```text
Input:
- Table 'orders' with millions of unique customer_ids.
- Query: SELECT COUNT(*) FROM orders GROUP BY customer_id;

Output:
- Hash aggregate used in memory, potentially spilling to disk if memory exceeds limits.
- Index on `customer_id` might allow for more efficient filtering and reduce row count.
```

**Data:**

```json
{
  "table": {
    "name": "orders",
    "columns": [
      { "name": "order_id", "type": "INT" },
      { "name": "customer_id", "type": "VARCHAR(255)" },
      { "name": "product_id", "type": "INT" },
      { "name": "quantity", "type": "INT" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Query with GROUP BY onhigh-cardinality column
4. Is index available andeffective?
5. Index reduces row count,avoids hash aggregation
6. Hash aggregate used inmemory or spilling to disk

