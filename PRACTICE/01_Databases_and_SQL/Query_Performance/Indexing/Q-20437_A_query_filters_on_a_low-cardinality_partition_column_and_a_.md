---
type: "practice"
difficulty: "Deep"
---

# A query filters on a low-cardinality partition column and a high-cardinality lookup column; how do you combine partitioning with clustering/Z-order for both?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitioning and Data Skipping in Lakes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, we need to understand how partitioning and clustering (or Z-order indexing) work together in database indexing to optimize query performance. The key is combining partitioning on a low-cardinality column with clustering or Z-order indexing on a high-cardinality column.

#### Triage

The question asks about optimizing a query that filters on a low-cardinality partition column and a high-cardinality lookup column. The goal is to leverage both partitioning and clustering/Z-order indexing in a way that minimizes the number of partitions scanned during the query execution.

Partitioning: This divides the data into smaller, more manageable chunks based on the partition column. For low-cardinality columns, this can significantly reduce the amount of data read from disk.
Clustering/Clustered Indexing: Organizes rows within a partition by a specific key (e.g., Z-order). This helps in reducing the number of rows to scan within a partition when querying.
Z-Order Indexing: A spatial indexing technique that organizes data points in a two-dimensional or multi-dimensional space, ensuring proximity in the index correlates with physical proximity on disk.

#### First-Principles Solution

Partition by Low-Cardinality Column:

Use the low-cardinality partition column to reduce the number of partitions scanned.
For example, if you have a year column that can take 5 values (2019, 2020, ..., 2023), partitioning by this column will ensure only one or a few partitions are accessed.

Cluster by High-Cardinality Lookup Column:

Use the high-cardinality lookup column to optimize queries that need to filter on specific values.
For example, if you have an order_id column with many unique values, clustering by this column will ensure that rows for a given order are physically close in the partition.

#### Example Scenario

Let's assume we have a table of sales data where we frequently query based on both year and order_id.

SQL
CREATE TABLE sales (
    year INT,
    order_id BIGINT,
    amount DECIMAL(10, 2)
) PARTITIONED BY (year);

-- Cluster by order_id within each partition
CLUSTER sales BY order_id;

This means that all orders from the same year will be grouped together and then further optimized using clustering on order_id.

Combining Partitioning with Clustering/Z-Order Indexing

To combine both effectively:

Partition by Year: Only scan the partitions corresponding to the years we are interested in.
Cluster within Partitions: Within each partition, use Z-order indexing if necessary for additional optimization.
SQL
-- Create a Z-order index on order_id within year
CREATE INDEX idx_sales_order_id ON sales (order_id) USING ZORDER BY (year, order_id);

#### Closing Summary
and Common Mistakes/Follow-Ups

Combining partitioning with clustering or Z-order indexing can significantly improve query performance by reducing the number of partitions to scan and optimizing the rows within those partitions. Key points:

Partition by low-cardinality columns: Reduce the number of partitions scanned.
Cluster by high-cardinality lookup columns: Optimize row access within each partition.

#### Common mistakes
include:

Overlooking the need for clustering/Z-order indexing: Clustering or Z-ordering is crucial for optimizing queries that filter on high-cardinality columns.
Not considering the interaction between partitioning and clustering: Ensure that the combined approach minimizes disk I/O and query time.

#### Follow-ups
might explore scenarios where you have multiple low-cardinality or high-cardinality columns, or how to handle cases with varying cardinalities across different partitions.

**Comparison:**

```
table_sales_partitioned {
    columns: year order_id amount
    partitions: "year=2019" "year=2020"
    clusters: cluster_order_id_within_partitions
}
index_zorder_by_year_order_id {
    name: idx_sales_order_id
    columns: order_id
    using: ZORDER BY (year, order_id)
}
```

**Code:**

```sql
-- Partition by year
CREATE TABLE sales (
    year INT,
    order_id BIGINT,
    amount DECIMAL(10, 2)
) PARTITIONED BY (year);

-- Cluster by order_id within each partition
CLUSTER sales BY order_id;

-- Create a Z-order index on order_id within year
CREATE INDEX idx_sales_order_id ON sales (order_id) USING ZORDER BY (year, order_id);
```

**Example input / output:**

```text
Input:
- Query: SELECT * FROM sales WHERE year = 2019 AND order_id IN (123456, 789012)

Output:
- Step 1: Scan only the partitions for `year` = 2019.
- Step 2: Use clustering/Z-order indexing on `order_id` to find and return rows with `order_id` in {123456, 789012}.
```

**Data:**

```json
{
  "table": {
    "columns": [
      {"name": "year", "type": "INT"},
      {"name": "order_id", "type": "BIGINT"},
      {"name": "amount", "type": "DECIMAL(10,2)"}
    ],
    "partition_by": ["year"],
    "cluster_by": ["order_id"]
  },
  "index": {
    "name": "idx_sales_order_id",
    "columns": ["order_id"],
    "using": "ZORDER BY (year, order_id)"
  }
}
```

**Flow:**

1. Partition by Year
2. Scan Partitions
3. Cluster by Order ID withinPartition
4. Find and Return Rows withSpecific Order IDs
5. Output Results

