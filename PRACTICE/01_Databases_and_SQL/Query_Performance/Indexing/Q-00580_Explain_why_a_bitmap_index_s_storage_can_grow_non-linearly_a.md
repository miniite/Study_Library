---
type: "practice"
difficulty: "Deep"
---

# Explain why a bitmap index's storage can grow non-linearly as cardinality creeps up, and how you'd monitor for that.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

A bitmap index's storage size can grow non-linearly with increasing cardinality because each value in a column gets its own bit array. As more distinct values are added, the number of bits required increases exponentially rather than linearly. Monitoring for this growth is crucial to avoid potential performance issues and resource constraints.

#### Triage

The question revolves around understanding why bitmap indices can become very large as cardinality increases, particularly in scenarios with high data diversity. It also probes into monitoring strategies to detect such growth early on. We'll assume a typical relational database system where bitmap indices are used for query optimization.

#### First-Principles Solution
Understanding Bitmap Indices

Bitmap indices store bitmaps (arrays of bits) corresponding to each unique value in the indexed column or combination of columns. Each row is represented by a single bit, with 1 indicating presence and 0 absence of that row for a given value. This approach significantly reduces storage when there are fewer distinct values relative to rows.

Non-Linear Storage Growth

Bitmap indices suffer from non-linear growth because:

Bit Arrays: Each distinct value requires its own bit array.
Cardinality Effect: As the number of unique values (cardinality) increases, the total size of these arrays grows exponentially. For example, if you have 10 distinct values, each with a 32-bit array, that’s 320 bits. With 100 distinct values, it becomes 3200 bits, and so on.

#### Example

#### Cons
ider a table Customers with the column City. If there are only a few cities (e.g., 5), each city might be represented by a small bitmap array. However, if the number of unique cities grows to thousands or millions, the overall storage requirement can become substantial.

Monitoring Strategy

To monitor and manage this growth:

Regular Checks: Implement periodic checks using database management tools to track index size.
Thresholds: Set thresholds for index size. If an index exceeds a certain percentage of the total table size, it may warrant optimization or re-evaluation.
Query Performance Metrics: Monitor query performance and adjust indices based on actual usage patterns.

#### Closing Summary

Monitoring bitmap index growth is essential to maintain database efficiency and avoid resource overallocation. By understanding the non-linear nature of storage increase with cardinality, you can proactively manage these indices to ensure optimal performance.

#### Common Mistakes/Follow-Ups
Ignoring Index Size: Failing to regularly monitor index sizes can lead to significant storage issues.
Overzealous Optimization: Too frequent re-indexing can negatively impact write performance without justifiable gains in read performance.
Insufficient Sampling: Relying on high-cardinality columns for bitmap indices without thorough testing.

If asked, you might be prompted to discuss specific tools or configurations used for monitoring, or to elaborate on the trade-offs between index complexity and query optimization.

**Code:**

```sql
-- Example SQL query to check index size
SELECT
    index_name,
    pg_size_pretty(pg_total_relation_size(indexname)) AS index_size
FROM
    pg_indexes
WHERE
    tablename = 'Customers';
```

