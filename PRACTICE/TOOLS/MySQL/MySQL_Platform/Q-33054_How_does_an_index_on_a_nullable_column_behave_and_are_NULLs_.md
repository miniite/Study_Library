---
type: "practice"
difficulty: "Deep"
---

# How does an index on a nullable column behave, and are NULLs stored in a B-tree index?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Optimizer Behavior](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with indexes in MySQL, especially on nullable columns, it's crucial to understand how NULL values interact with B-tree indices. This understanding is key for optimizing queries and ensuring that your database performs efficiently.

The core of the question is probing into the storage and behavior of NULL values within a B-tree index. Here’s a step-by-step explanation from first principles:

#### Overview

In MySQL, an index on a nullable column does not store NULLs in the same way as non-nullable columns.
NULLs are handled specially by B-tree indices to maintain their unique properties.

#### Triage

The question is asking about how MySQL treats NULL values when indexing and searching for them.
We need to understand the internal storage mechanism of B-trees in MySQL, particularly how they handle NULL keys.

#### First Principles Solution

A B-tree index is a data structure that stores values in sorted order. Each node contains key-value pairs, where the keys are the indexed columns.
When you create an index on a nullable column, MySQL does not physically store NULLs in the same way as non-nullable values. Instead, it uses a special representation for NULLs to maintain the B-tree's properties.

Detailed Explanation:

In a B-tree, each node can have multiple key-value pairs. For a nullable column index, if there are NULL values present, they are not stored like other non-nullable values.
MySQL uses a special value called 0xFF to represent NULLs in the internal tree structure. This helps maintain the sorted order without actually storing a NULL value as an element.

Impact on Queries:

When you perform a query involving a nullable indexed column, MySQL will handle NULL comparisons according to its internal logic.
For example, SELECT * FROM table WHERE nullable_column IS NULL will still be efficient because the B-tree index helps in quickly locating all rows with NULL values without scanning the entire table.

#### Closing Summary

In summary, an index on a nullable column does not store NULLs as individual elements in the B-tree but uses a special representation to handle them efficiently.
This mechanism ensures that queries involving NULLs can still leverage the benefits of indexing while maintaining performance and correctness.

#### Common Mistakes/Follow-Ups

A common mistake is assuming that NULL values are stored like other non-nullable values, which can lead to confusion when optimizing queries.
Interviewers might follow up by asking how this affects query performance or whether you can create a separate index for nullable columns if needed.

**Code:**

```sql
-- Example of creating an index on a nullable column
CREATE TABLE example_table (
    id INT PRIMARY KEY,
    nullable_column VARCHAR(255) NULL
);

CREATE INDEX idx_nullable ON example_table(nullable_column);
```

