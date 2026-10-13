---
type: "practice"
difficulty: "Expert"
---

# You have predicates on col_a = ? and col_b > ? and an ORDER BY col_b. What composite index column order lets you both filter and sort without a separate sort?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Index Access Paths](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this interview question, we need to understand how database indexes can be leveraged to optimize queries that involve filtering by two columns (col_a and col_b) and sorting primarily by one of these columns (col_b). The key is to use a composite index with the correct column order.

TL;DR

A composite index on (col_a, col_b) will allow both the predicate filter (on col_a and col_b) and the sort (by col_b) to be performed efficiently without requiring an additional sort operation. This is because the database can use the index for filtering and then directly access the sorted order based on col_b.

#### Triaging the Question

The question asks for a composite index that supports both predicate filtering (col_a = ? and col_b > ?) and ordering by col_b. The interviewer is probing your understanding of how indexes work, specifically in the context of combined conditions. We'll need to ensure that the order of columns in the index matches our query requirements.

Step-by-Step Solution
Predicate Filtering: For filtering using a condition like col_a = ? and col_b > ?, the database needs an index where it can quickly locate rows that satisfy both these conditions.
Sorting by col_b: Once filtered, we need to sort the remaining results primarily based on col_b. This requires the index to maintain the order of col_b.

#### First-Principles Solution

A composite index on (col_a, col_b) will meet both requirements:

The first column (col_a) can be used for filtering.
The second column (col_b) will ensure that the results are ordered correctly.
Index Definition
SQL
CREATE INDEX idx_col_a_col_b ON table_name (col_a, col_b);

#### Closing Summary
and Common Mistakes/Follow-Ups

By creating an index on (col_a, col_b), we achieve two important objectives:

The database can use the index to quickly filter rows based on col_a and col_b > ?.
The results are already in the correct order for sorting by col_b.

#### Common Mistakes/Follow-Ups
Incorrect Index Order: If you only create an index on (col_b, col_a), filtering will not be efficient because col_b is not first.
Single-Column Indexes: Creating separate indexes on col_a and col_b might not provide the same performance benefits as a composite index in this scenario.

If asked to extend the solution:

Discuss potential trade-offs, such as the size of the index and its impact on write operations.

#### Cons
ider additional columns or more complex query conditions that might require further optimization.

**Code:**

```sql
CREATE INDEX idx_col_a_col_b ON table_name (col_a, col_b);
```

