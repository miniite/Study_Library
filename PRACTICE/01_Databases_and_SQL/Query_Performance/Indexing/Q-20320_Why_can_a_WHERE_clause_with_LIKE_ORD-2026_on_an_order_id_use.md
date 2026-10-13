---
type: "practice"
difficulty: "Basic"
---

# Why can a WHERE clause with LIKE 'ORD-2026%' on an order_id use an index while LIKE '%2026' cannot?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Composite Indexes and Column Order](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why a WHERE clause with LIKE 'ORD-2026%' can utilize an index, while LIKE '%2026' cannot, we need to delve into how SQL engines process queries and the role of indexes. In summary, the key difference lies in the pattern used: prefix patterns (like 'ORD-2026%') allow for efficient scanning from a specific point, whereas suffix or middle patterns ('%2026') require full table scans.

#### Triaging the Question

The question is essentially about understanding index usage with different LIKE queries. The interviewer is probing your knowledge of SQL indexing and how query patterns affect performance. We assume the database being used supports B-tree indexes, which are commonly used for range and prefix searches but not as effective for full-text or wildcard matches.

#### First-Principles Solution
1. Index Structure
An index is essentially a data structure that allows for efficient lookup of records based on certain columns.
For ORDER BY queries, the most common type of index used is a B-tree index, which stores entries in sorted order.
2. Prefix Matching (LIKE 'ORD-2026%')
When you have a prefix pattern like 'ORD-2026%', the database can start scanning from the ORD- part and quickly find all records that begin with this sequence.
This is because B-tree indexes store keys in sorted order, allowing for binary search-like operations.
3. Suffix Matching (LIKE '%2026')
For a suffix or middle pattern like '%2026', the database cannot use an index efficiently.
The reason is that the % wildcard can match any sequence of characters, making it impossible for the database to predict where in the index these records might be located.

#### Closing Summary
and Common Mistakes

In summary, prefix patterns (LIKE 'ORD-2026%') can leverage indexes because they allow the database to perform efficient binary searches. In contrast, suffix or middle patterns ('%2026') cannot utilize indexes effectively due to the unpredictability of where such records might be stored.

A common mistake is thinking that any LIKE query with a wildcard in it will not use an index. However, as shown, prefix patterns can still benefit from indexing, while suffix or middle patterns typically require full table scans, which are slower and less efficient.

Common Follow-Ups
Q: Can we optimize the second query ('%2026')?
A: Yes, one approach is to use a function-based index. For example, in PostgreSQL, you could create an index on ORDER_ID with the  POSITION('2026' IN ORDER_ID) function.
Q: What are the performance implications of using an index vs. a full table scan?
A: Indexes generally provide faster access to data because they allow for quick lookups, whereas full table scans require reading every row in the table, which can be much slower.

**Code:**

```sql
-- Example of creating an index on ORDER_ID with a function-based index in PostgreSQL
CREATE INDEX idx_order_id_position ON orders (POSITION('2026' IN order_id));
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Prefix Query
6. Index Used?
7. Efficient Search
8. Full Table Scan
9. Middle or Suffix Query
10. Index Used?
11. Full Table Scan
12. Inefficient Search

