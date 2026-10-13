---
type: "practice"
difficulty: "Intermediate"
---

# How would you estimate the cost of a query in a pay-per-scan warehouse and reduce bytes scanned with partitioning and column pruning?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Partitioning and Partition Pruning](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To estimate the cost of a query in a pay-per-scan warehouse, we need to understand how data is stored, accessed, and filtered. We will use partitioning and column pruning to optimize the query execution and reduce the bytes scanned. Let's break down the problem step-by-step.

#### Triage

The question asks us to estimate the cost of a query in a pay-per-scan warehouse environment while focusing on two optimizations: partitioning and column pruning. The key idea here is understanding how these techniques can reduce the amount of data that needs to be scanned, thereby lowering costs. We will assume the database management system (DBMS) provides statistics about table size, average row length, and distribution patterns.

#### First Principles Solution
Partitioning: This technique divides a large table into smaller, more manageable parts based on some criteria. Partitioning can significantly reduce the amount of data scanned by limiting the scan to only relevant partitions.
Column Pruning: Column pruning involves selecting specific columns from the database rather than scanning all columns. This reduces the overall volume of data that needs to be processed and stored in memory.
Step-by-Step Solution
1. Estimate Initial Bytes Scanned

Assume we have a table orders with the following schema:

SQL
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    order_date DATE,
    quantity INT,
    amount DECIMAL(10, 2)
);
Total number of rows: 10 million
Average row length: 45 bytes

The initial estimate for the total table size is: [ \text{Total Table Size} = 10 \times 10^6 \times 45 \text{ bytes} = 4.5 \text{ GB} ]

2. Apply Partitioning

Let's partition orders by order_date. Assume the data is evenly distributed over a year.

Number of partitions: 365 (one for each day)
Rows per partition: ( \frac{10 \times 10^6}{365} = 274,000 ) rows
Average size of each partition: ( 274,000 \times 45 \text{ bytes} / 1024 / 1024 = 1.23 \text{ MB} )
3. Estimate Bytes Scanned for a Query

#### Cons
ider the following query:

SQL
SELECT * FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-06-30';

With partitioning, only partitions containing data from January 1 to June 30 are scanned. This period covers approximately half a year.

Number of relevant partitions: ( \frac{182}{365} \times 365 = 182 )
Total size of relevant partitions: ( 182 \times 1.23 \text{ MB} = 224 \text{ MB} )

Without partitioning, the entire table (4.5 GB) would be scanned.

4. Apply Column Pruning

Now let’s consider a query that only needs specific columns:

SQL
SELECT order_id, customer_id FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-06-30';

Since we are selecting fewer columns (order_id and customer_id), the average row length for each record reduces to approximately 35 bytes.

Average size of relevant partitions: ( \frac{35}{45} \times 1.23 \text{ MB} = 0.95 \text{ MB} )
Total size of relevant partitions: ( 182 \times 0.95 \text{ MB} = 172 \text{ MB} )

#### Closing Summary

By applying partitioning and column pruning, we can reduce the bytes scanned from the initial 4.5 GB to approximately 172 MB for the given query. This not only reduces cost but also speeds up query execution by reducing the amount of data that needs to be processed.

#### Common Mistakes/Follow-Ups
Assuming uniform distribution: Ensure data is evenly distributed across partitions.
Ignoring table statistics: DBMS may have more accurate statistics on row size and partitioning, which should be leveraged for better estimates.
Neglecting index usage: Indexes can further reduce the number of bytes scanned but were not considered here.

**Comparison:**

| Partition | Rows       | Size (MB) |
|-----------|------------|----------|
| Jan 2023  | 274,000    | 1.23     |
| ...       | ...        | ...      |
| Jun 2023  | 274,000    | 1.23     |

**Code:**

```sql
-- Example SQL to create a partitioned table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    order_date DATE,
    quantity INT,
    amount DECIMAL(10, 2)
) PARTITION BY RANGE (order_date);
```

**Example input / output:**

```text
-- Input: Query with partitioning and column pruning
SELECT order_id, customer_id FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-06-30';

-- Output: Reduced bytes scanned to 172 MB
```

**Data:**

```json
{
  "table": {
    "name": "orders",
    "columns": [
      { "name": "order_id", "type": "INT" },
      { "name": "customer_id", "type": "INT" },
      { "name": "product_id", "type": "INT" },
      { "name": "order_date", "type": "DATE" },
      { "name": "quantity", "type": "INT" },
      { "name": "amount", "type": "DECIMAL(10, 2)" }
    ]
  },
  "partitioning": [
    { "type": "RANGE", "column": "order_date", "granularity": "DAY" }
  ],
  "row_count": 10_000_000,
  "average_row_size": 45
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Query
6. Partitioning?
7. Scan All Data
8. Filter Partitions
9. Column Pruning?
10. Scan Entire Partitions
11. Select Specific Columns
12. Calculate Bytes Scanned

