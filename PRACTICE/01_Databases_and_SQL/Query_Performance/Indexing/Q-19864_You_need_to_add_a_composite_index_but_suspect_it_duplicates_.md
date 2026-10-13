---
type: "practice"
difficulty: "Intermediate"
---

# You need to add a composite index but suspect it duplicates an existing one on a prefix. How do you prove redundancy versus a real need for the shorter index on the write path?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address whether adding a composite index is redundant or necessary, we need to understand the nature of our existing indexes and how they interact with read and write paths. Here’s a step-by-step approach from first principles:

Summary: We will analyze the existing indexes on the database table to determine if a proposed shorter prefix-based composite index duplicates any existing one. By doing so, we can decide whether adding this new index is necessary or redundant.

#### Triage

The core question is whether an additional composite index is needed based on its relationship with existing indexes.
We need to consider the database schema, query patterns (both read and write), and potential performance implications.

#### Assumptions
include that we have sufficient knowledge of the current schema and expected future queries.

Step-by-Step Solution:

Understand Existing Indexes: List all existing indexes on the table and their definitions (e.g., single-column, composite).
Identify Query Patterns: Analyze common read and write operations to understand where these indexes are used.
Evaluate New Index: Consider the proposed prefix-based index. Determine its structure and purpose.
Performance Testing:
Dry Run Queries: Simulate queries using both existing and new indexes in a test environment.
Compare Execution Plans: Use database management system (DBMS) tools like EXPLAIN or similar to compare how each index performs for the same query.
Benchmarking: Measure execution times, resource usage, and I/O operations for queries with both existing and new indexes.

#### Conclusion

If the proposed prefix-based composite index does not significantly improve query performance over existing indexes (especially on read paths), it may be redundant.
On the write path, consider the impact of updating multiple indexes; a shorter prefix might reduce some write overhead but could also introduce redundancy if not strictly necessary.

#### Common Mistakes
and Follow-Ups:

Mistake 1: Assuming new indexes will always provide performance benefits without thorough testing.

#### Follow-Up
1: Ensure the database schema is well understood, including any potential data distribution that might affect index efficiency.
Mistake 2: Failing to account for write path overheads when introducing a new prefix-based index.

#### Follow-Up
2: Consider using strategic indexes (covering indexes) or partial indexes if the prefix-based approach does not fully cover necessary use cases.

**Comparison:**

| Existing Indexes | New Index |
|------------------|-----------|
| name             | columns   |
| idx_customer_id  | customer_id          |
| idx_order_date_customer_id | order_date, customer_id       |
| idx_order_date_cust_id_prefix | order_date, customer_id_prefix |

**Code:**

```sql
-- Example SQL to list existing indexes
SHOW INDEXES FROM your_table;
```

**Example input / output:**

```text
Query: SELECT * FROM orders WHERE order_date > '2023-10-01' AND customer_id = 12345;
Existing Index Execution Plan:
  - Using index idx_order_date_customer_id

New Index Execution Plan (Dry Run):
  - Using index idx_order_date_cust_id_prefix
  - Execution time: 1.5s
  - I/O operations: 700ms
```

**Data:**

```json
{
  "existing_indexes": [
    {"name": "idx_customer_id", "columns": ["customer_id"]},
    {"name": "idx_order_date_customer_id", "columns": ["order_date", "customer_id"]}
  ],
  "new_index": {
    "name": "idx_order_date_cust_id_prefix",
    "columns": ["order_date", "customer_id_prefix"]
  }
}
```

**Flow:**

1. Understand Existing Indexes
2. Evaluate New Index
3. Simulate Queries
4. Compare Execution Plans
5. Benchmark Performance
6. Decide on Redundancy

