---
type: "practice"
difficulty: "Deep"
---

# How do columnar stores like ClickHouse/Redshift make SUM and COUNT dramatically cheaper than a row store, and where does that advantage disappear?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Advanced Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Columnar storage technologies like ClickHouse and Amazon Redshift are designed to optimize queries on large datasets by storing data column-wise rather than row-wise. This design choice significantly improves performance for operations that involve aggregating or summarizing large volumes of data, such as SUM and COUNT. However, the advantage of this approach diminishes in certain scenarios. Let's delve into how these systems achieve their speed-ups and where they might not excel.

Columnar stores enhance query efficiency by allowing direct access to entire columns without reading unrelated rows, which is a common requirement for aggregate functions like SUM and COUNT. This means that when performing an aggregation, only the necessary data needs to be read from disk, significantly reducing I/O operations.

First, let's understand how these systems store data:

Columnar Storage: Data is stored in columns rather than rows.
Compression Techniques: Various compression algorithms are used to reduce storage size and improve read performance.
Execution Engine: Specialized execution engines that can operate on the compressed data efficiently.

Now, let's explore why columnar stores excel with SUM and COUNT:

Direct Column Access: When performing an aggregation like SUM or COUNT, only the relevant column needs to be accessed. This drastically reduces I/O operations compared to a row-oriented store where multiple rows might need to be scanned.
Parallel Processing: Columnar stores can leverage parallel processing by distributing the workload across multiple nodes and threads, further speeding up the computation.
Compression Benefits: Compressed data requires less memory and is read faster, reducing the overhead of I/O operations.

However, there are scenarios where this advantage may diminish:

Small Aggregations or Filters: If a query involves many small aggregations across different columns, the cost of decompressing multiple columns might outweigh the benefits of direct column access.
Selective Scans: When filters (WHERE clauses) significantly reduce the dataset size, row-oriented storage might perform better because fewer rows need to be processed, even if they are stored in a row-wise fashion.

Let's break down an example to illustrate:

#### Example Scenario

Suppose we have a table with millions of records representing user transactions. We want to calculate the total amount and count of transactions for each day.

SQL
SELECT
    date_column,
    SUM(amount) AS total_amount,
    COUNT(*) AS transaction_count
FROM
    transactions
GROUP BY
    date_column;

In a columnar store:

The system can directly access the date_column and amount columns.
Compression algorithms ensure that only relevant data is decompressed.
Parallel processing distributes the workload efficiently.
Performance Analysis

For large datasets, the direct access to columns and parallel execution provide significant performance gains. However, for smaller datasets or complex queries involving many joins and filters, a row-oriented store might be more efficient due to lower overhead in handling small amounts of data.

In summary, columnar stores like ClickHouse and Redshift excel at performing aggregate functions on large datasets by minimizing I/O operations and leveraging parallel processing. However, the advantage diminishes with smaller datasets or complex query patterns that benefit from more detailed row-wise access.

**Comparison:**

| Date       | Total Amount (SUM) | Transaction Count (COUNT) |
|------------|--------------------|--------------------------|
| 2023-10-01 |        125.00      |                          2 |
| 2023-10-02 |       100.00       |                          1 |

**Code:**

```sql
-- Example SQL for a columnar store
SELECT
    date_column,
    SUM(amount) AS total_amount,
    COUNT(*) AS transaction_count
FROM
    transactions
GROUP BY
    date_column;
```

**Example input / output:**

```text
-- Input SQL Query
SELECT
    date_column,
    SUM(amount) AS total_amount,
    COUNT(*) AS transaction_count
FROM
    transactions
GROUP BY
    date_column;

-- Output Results
+-------------+----------------+--------------------+
| date_column |  total_amount   | transaction_count  |
+-------------+----------------+--------------------+
| 2023-10-01  |        125.00   |                2   |
| 2023-10-02  |       100.00    |                1   |
+-------------+----------------+--------------------+
```

**Data:**

```json
{
  "transactions": [
    {"date": "2023-10-01", "amount": 50.0},
    {"date": "2023-10-01", "amount": 75.0},
    {"date": "2023-10-02", "amount": 100.0}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query
6. Is the query complex?
7. Row-oriented Storage
8. Columnar Store
9. Does the query involve manysmall aggregations?
10. Row-oriented Storage may bemore efficient
11. Columnar Store remainsadvantageous

