---
type: "practice"
difficulty: "Basic"
---

# How would you explain the difference between a logical and a physical read when profiling a slow pipeline query to a new teammate?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN ANALYZE](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When profiling a slow pipeline query, understanding the difference between logical and physical reads is crucial. A logical read refers to the operation of fetching data from the database based on the logical structure or schema defined by the indexes and tables. Conversely, a physical read involves actually accessing the disk or memory to retrieve the data.

To explain this to a new teammate, I would start with the basics: logical reads are about how SQL queries interact with the logical model, while physical reads are about the actual execution steps that involve reading from storage media. Let's delve into each in detail:

Logical Reads

Logical reads happen when a query accesses data based on the indexes and tables as defined by the database schema. For example, if you have an index on a users table with a column name, a query like SELECT * FROM users WHERE name = 'John' will perform a logical read by using this index to locate the relevant rows.

Physical Reads

Physical reads are the actual disk or memory operations that fetch data. These occur when the database engine accesses the storage layer to retrieve the physical blocks of data needed for the query execution. For instance, if you have an index on users, but your query needs more columns than just those covered by the index, the database will perform a physical read to fetch additional data from the table.

Step-by-Step Reasoning
Understand the Query: Analyze the SQL statement and identify which parts are logically reading indexes versus fetching actual data.
Identify Index Usage: Check if any indexes are being used, and if so, how they are helping or hindering performance.
Check Execution Plan: Use tools like EXPLAIN to understand the query’s execution plan, showing both logical and physical operations.

#### Example

Suppose we have a table orders with an index on customer_id. The following query:

SQL
SELECT * FROM orders WHERE customer_id = 12345;

Would involve a logical read using the index to find the relevant rows. However, if the query needs additional columns not covered by the index, it might also perform a physical read to fetch these extra details from the table.

#### Closing Summary

Understanding logical and physical reads helps in optimizing queries effectively. Logical reads can be optimized by ensuring proper indexing, while physical reads are influenced by factors like data distribution and storage configurations. By monitoring both types of reads, we can pinpoint performance bottlenecks and make informed decisions to improve query efficiency.

#### Common mistakes
include not considering the full execution plan or assuming that index usage always optimizes performance. Always verify with actual data and profiling tools.

**Comparison:**

```
Logical Read	Physical Read
Uses indexes	Fetches data from storage
Faster access	Slower, involves I/O
Based on schema	Actual disk operations
```

**Code:**

```sql
-- Example SQL query
SELECT * FROM orders WHERE customer_id = 12345;
```

**Example input / output:**

```text
Input:
SELECT * FROM orders WHERE customer_id = 12345;

Output (Execution Plan):
Logical Reads:
- Index scan on idx_customer_id

Physical Reads:
- Full table scan to fetch additional columns not covered by index
```

**Data:**

```json
{
  "table": "orders",
  "columns": ["order_id", "customer_id", "product_name", "quantity"],
  "index": {"name": "idx_customer_id", "column": "customer_id"}
}
```

**Flow:**

1. Logical Reads
2. Uses Index
3. Fetches based on schema
4. Physical Reads
5. Fetches data from storage
6. Involves I/O operations

