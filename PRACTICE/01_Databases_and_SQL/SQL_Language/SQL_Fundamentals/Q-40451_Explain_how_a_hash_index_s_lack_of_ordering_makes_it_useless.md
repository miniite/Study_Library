---
type: "practice"
difficulty: "Deep"
---

# Explain how a hash index's lack of ordering makes it useless for range scans and ORDER BY, and give one workload where hash beats B-tree despite that. What durability concern did older Postgres hash indexes have?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Leftmost Prefix Rule](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing hash indices in SQL databases like PostgreSQL, we often contrast them with B-tree indices because of their distinct properties and use cases. A key difference between a hash index and a B-tree index lies in how they handle data ordering and scanning operations.

#### Triaging the Question

The question asks us to explain why a hash index's lack of inherent order makes it unsuitable for range scans and ORDER BY. Additionally, we need to provide an example where a hash index outperforms a B-tree index despite this limitation. Finally, the question also delves into an historical durability issue associated with older PostgreSQL hash indexes.

Core Solution
Why Hash Indexes Lack Ordering

A hash index stores data based on the hash value of the indexed column rather than maintaining any sort order. This means that while hash lookups are extremely fast—typically O(1)—they cannot be used for range scans or ORDER BY operations because they do not preserve any logical ordering of the rows.

Why B-Trees Are Better for Range Scans and ORDER BY

B-tree indices, on the other hand, maintain a sorted order of the indexed values. This allows them to efficiently support range queries and ORDER BY clauses by traversing the tree structure in an ordered manner, typically achieving performance that is closer to O(log n) than O(1).

#### Example
Where Hash Beats B-Tree

Despite its lack of ordering, a hash index can outperform a B-tree index in scenarios where we are performing exact match queries. For instance, consider a scenario where you have a database table with millions of records and frequently run queries that look up specific user IDs or product SKUs. In such cases, the speed of O(1) lookups provided by hash indices can be significantly faster than B-tree index scans.

Workload Example

Let's assume we have an e-commerce application where users frequently search for products based on unique identifiers like product_id. The SQL query might look something like:

SQL
SELECT * FROM Products WHERE product_id = '12345';

In this case, a hash index on the product_id column would allow the database to directly compute the hash and find the corresponding row in constant time. In contrast, a B-tree index would need to perform a binary search, which is generally slower for exact match queries.

Historical Durability Concern with Older Postgres Hash Indexes

Older versions of PostgreSQL had a durability concern related to hash indexes. Specifically, if the database crashed or was interrupted during an operation involving a hash index, it could result in data corruption because the hash buckets might not be fully written out before the system failed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while hash indices are excellent for exact match queries due to their O(1) lookup time, they cannot handle range scans or ORDER BY operations effectively. B-trees provide a balanced approach that is better suited for these types of queries. However, in specific scenarios where only exact matches are needed, hash indices can outperform B-trees.

#### Common mistakes
include assuming that hash indexes can be used interchangeably with B-tree indexes for any type of query. Additionally, overlooking the historical durability issues associated with older implementations of hash indexing in PostgreSQL.

For a deeper discussion, follow-ups could explore more complex scenarios involving mixed workloads or performance tuning strategies to balance between different index types.

**Comparison:**

| Column    | Data Type   |
|-----------|------------|
| product_id | UUID       |
| name      | VARCHAR(100) |
| price     | DECIMAL    |
| Rows (Sample Data)           |
|------------------------------|
| 12345678-1234-5678-1234-567812345678, Apple iPhone 13, 999.00 |
| 23456789-2345-6789-2345-678923456789, Samsung Galaxy S21, 899.00 |
| 34567890-3456-7890-3456-789034567890, Google Pixel 6 Pro, 799.00 |

**Code:**

```sql
-- Example SQL queries using hash and B-tree indexes

CREATE TABLE Products (
    product_id UUID PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL
);

-- Using a B-tree index for range scan
EXPLAIN ANALYZE SELECT * FROM Products WHERE product_price BETWEEN 5.99 AND 20.99;

-- Using a hash index for exact match query (assuming Postgres supports UUID as an indexable type)
CREATE INDEX idx_products_hash ON Products USING HASH (product_id);

EXPLAIN ANALYZE SELECT * FROM Products WHERE product_id = '12345678-1234-5678-1234-567812345678';
```

**Example input / output:**

```text
-- Input for range scan query
EXPLAIN ANALYZE SELECT * FROM Products WHERE product_price BETWEEN 5.99 AND 20.99;

-- Output (expected to show a B-tree index scan)
"Bitmap Heap Scan on products  (cost=8.43..161.47 rows=3 width=104)"
"  Recheck Cond: ((product_price >= 5.99) AND (product_price <= 20.99))"
"  ->  Bitmap Index Scan on idx_products_price  (cost=0.00..8.43 rows=3 width=0)"
"        Index Cond: ((product_price >= 5.99) AND (product_price <= 20.99))"

-- Input for exact match query
EXPLAIN ANALYZE SELECT * FROM Products WHERE product_id = '12345678-1234-5678-1234-567812345678';

-- Output (expected to show a hash index scan)
"Index Scan using idx_products_hash on products  (cost=0.29..3.30 rows=1 width=104)"
```

**Data:**

```json
{
  "table": {
    "columns": ["product_id", "name", "price"],
    "rows": [
      {"product_id": "12345678-1234-5678-1234-567812345678", "name": "Apple iPhone 13", "price": 999.00},
      {"product_id": "23456789-2345-6789-2345-678923456789", "name": "Samsung Galaxy S21", "price": 899.00},
      {"product_id": "34567890-3456-7890-3456-789034567890", "name": "Google Pixel 6 Pro", "price": 799.00}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Exact Match Query
6. Use Hash Index?
7. Direct Lookup
8. B-Tree Scan
9. Range Scan Query
10. Use B-Tree Index?
11. B-Tree Scan
12. Not supported, use othermethods

