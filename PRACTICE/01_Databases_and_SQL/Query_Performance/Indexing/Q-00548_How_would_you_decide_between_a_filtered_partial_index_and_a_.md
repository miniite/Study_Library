---
type: "practice"
difficulty: "Deep"
---

# How would you decide between a filtered/partial index and a bitmap index for a column that's 95% one value and 5% another?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a filtered/partial index and a bitmap index for a column that’s predominantly one value with only occasional variations, we need to consider several factors such as query patterns, data distribution, storage overhead, and performance implications. Let’s break this down step-by-step.

#### Triage

The key question is whether a partially filtered index or a bitmap index would be more efficient given the 95/5 data distribution. The interviewer is probing your understanding of advanced indexing techniques and their trade-offs in specific scenarios. Key assumptions include that we are dealing with a large dataset where storage space and query performance are critical factors.

Step-by-Step Solution
Data Distribution: Given that 95% of the column is one value, any index will be predominantly pointing to this common value.
Filtered Index:
A filtered index includes only rows that meet a specific condition (in this case, where the column equals the common value).
This reduces the size and complexity of the index since it doesn’t store entries for less frequent values.
Bitmap Index:
A bitmap index uses bits to represent whether a row has or does not have a particular value in a column.
For highly skewed data like ours, most bits will be set (i.e., representing the common value).
First Principles
Filtering Efficiency: The filtered index is more efficient because it only stores rows that match the frequent value. This reduces storage and maintenance overhead.
Query Performance:
For queries filtering on the common value, a filtered index will directly point to those rows without needing to scan additional data structures.
Queries for rare values might be less efficient as they still require checking other parts of the index or full table scans.

#### Trade-Offs
and Considerations
Storage Overhead: A filtered index requires less space, which is beneficial if storage costs are a concern. Bitmap indices, while smaller in some cases due to compression, can introduce more complex storage mechanisms.
Query Patterns:
If queries frequently filter on the common value, a filtered index will perform better.
For mixed query patterns where both frequent and infrequent values are queried, consider additional indexes or composite indexes.

#### Closing Summary

Given that 95% of the column’s data is one value, a filtered/partial index would likely be more efficient in terms of storage and performance for queries filtering on the common value. However, if mixed query patterns exist, or if there are significant costs associated with storage space, a bitmap index might still be considered despite its potential inefficiency.

#### Common Mistakes
Overlooking the skewed data distribution.
Not considering the trade-offs between storage overhead and query performance.
Failing to assess additional query patterns beyond those involving the common value.
Final Thoughts

It’s essential to thoroughly analyze the specific use case, including query patterns and cost considerations. In practice, a combination of both techniques or even a hybrid approach might be necessary for optimal performance across various scenarios.

**Code:**

```sql
-- Example SQL for creating a filtered index in PostgreSQL
CREATE INDEX idx_filtered ON table_name (column_name) WHERE column_name = 'common_value';
```

**Flow:**

1. Skewed Data Distribution
2. Filtered Index
3. Bitmap Index
4. Reduced Storage, Efficient forCommon Queries
5. Smaller but More ComplexStorage, Potentially Inefficient
6. Optimized for Common ValueQueries
7. Mixed Query PatternsConsideration

