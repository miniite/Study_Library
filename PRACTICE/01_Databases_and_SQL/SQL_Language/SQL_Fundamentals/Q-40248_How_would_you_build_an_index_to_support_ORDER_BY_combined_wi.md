---
type: "practice"
difficulty: "Deep"
---

# How would you build an index to support ORDER BY combined with a WHERE filter efficiently?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Leftmost Prefix Rule](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the challenge of building an efficient SQL index supporting both ORDER BY and WHERE clauses, we need to understand how these operations interact and affect query performance. This problem is common in scenarios where you frequently sort and filter a large dataset based on specific columns.

Firstly, let's triage the question: The core of this problem involves optimizing the retrieval of rows that meet certain criteria (the WHERE condition) while also sorting them in a particular order (ORDER BY). We need to choose an appropriate index type, ensure it covers both conditions efficiently, and consider potential trade-offs.

#### Assumptions
Table Schema: The table has columns such as id, name, value, etc.
Query Example: Consider the following SQL query:
SQL
SELECT * FROM my_table WHERE value > 5 ORDER BY name ASC;
Index Types: We can use B-tree (most common), Hash index, or other specialized indexes.
Step-by-Step Solution:

Understand Indexing for WHERE Clause:

A composite index on (value, name) could be effective because it allows the database to quickly find rows where value > 5.
However, such an index might not be optimal if value is a less selective column.

Understand Indexing for ORDER BY:

An index on name ASC would directly support sorting.
A composite index on (name, value) could potentially improve both filtering and ordering but adds complexity in construction and maintenance.

#### Cons
ider Full-Text Search:

For text-based columns, a full-text index might be beneficial, but this is less common for numeric ORDER BY.

Evaluate Index Types:

B-tree: Suitable for WHERE clauses if value is selective and for ORDER BY.
Hash Index: Useful for exact match queries but not as effective for range-based filtering.

Choose the Most Appropriate Index:

A composite B-tree index on (value, name) can be an optimal choice here. It supports efficient row retrieval based on value > 5 and then sorts by name.

Test and Optimize:

Run performance tests with different data distributions and query patterns to ensure the chosen index is indeed effective.

#### Cons
ider using partial indexes or function-based indexes if specific scenarios require optimization.

#### Closing Summary

In summary, building an efficient SQL index for combined WHERE and ORDER BY clauses involves carefully choosing a composite B-tree index that covers both filtering and sorting conditions. The key is to balance the selectivity of columns used in the WHERE clause with their impact on query performance. Regular testing and profiling are essential to validate the chosen solution.

#### Common Mistakes/Follow-Ups
Using an overly complex or redundant index can degrade write performance without significant benefit.
Failing to consider data distribution and query patterns can lead to suboptimal choices.
Not testing different scenarios can result in poor performance under real-world conditions.

**Code:**

```sql
-- Example of creating a composite B-tree index on (value, name)
CREATE INDEX idx_value_name ON my_table(value, name);
```

