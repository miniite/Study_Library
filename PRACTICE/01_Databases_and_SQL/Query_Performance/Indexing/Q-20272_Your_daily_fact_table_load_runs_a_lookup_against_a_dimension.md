---
type: "practice"
difficulty: "Basic"
---

# Your daily fact table load runs a lookup against a dimension table on business_key. Which column would you index and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize your daily fact table load, which performs lookups against a dimension table using business_key, you should index the business_key column in the dimension table. This choice is driven by the high likelihood of frequent lookups on this specific key, thereby reducing query execution time and improving overall performance.

#### Triaging the Question

The core of this question is to understand which column to index for optimizing a lookup operation between two tables—specifically, a fact table and a dimension table. The interviewer is probing your understanding of how indexes work, their impact on query performance, and your ability to make informed decisions based on data characteristics.

#### First-Principles Solution
Identify the Lookup Column: In this case, it's business_key.
Understand Indexes: An index improves read operations by reducing the number of required disk accesses.
Cost-Benefit Analysis: Weigh the benefits (faster query execution) against potential costs (index maintenance overhead).
Detailed Explanation

Indexes are data structures that improve the speed of data retrieval operations on a database table at the cost of additional writes, space, and maintenance during updates. For our scenario:

Column to Index: The business_key column is chosen because it's likely to be used frequently as a lookup key in your fact table.
Index Type: A B-tree index would be suitable for most use cases involving exact match lookups like this one.

Here’s why the business_key should be indexed:

Frequent Lookups: The fact table is loaded daily and performs frequent lookups, making the business_key a prime candidate for indexing.
Exact Matches: Indexes on columns used in equality conditions (like =) are highly effective because they allow for direct access to matching rows without scanning the entire table.
Why Not Other Columns?
Other Dimensions: If there are no other columns that get frequent lookups, focusing on business_key makes sense.
Redundancy: Ensure that business_key is unique or almost unique in the dimension table. Indexes work best when they help distinguish rows effectively.

#### Closing Summary
and Common Mistakes

By indexing business_key, you ensure faster query execution during daily fact table loads, which can significantly reduce load times and improve overall system performance. Common mistakes include not considering the frequency of lookups, choosing an inappropriate index type (e.g., using a full-table scan when an index could be used), or overlooking maintenance costs associated with indexes.

**Code:**

```sql
-- Example SQL to create B-tree index on business_key in dimension table
CREATE INDEX idx_business_key ON dimension_table(business_key);
```

