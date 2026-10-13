---
type: "practice"
difficulty: "Deep"
---

# You need a functional index on lower(email) to dedup case-insensitive emails during a load. Why must the query expression match the index exactly?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a functional index on lower(email) effectively deduplicates case-insensitive emails, it's crucial that the query expression matches the index exactly. This is because the database relies on indexes to perform efficient operations such as sorting and filtering. Let’s break down why this matters from first principles.

#### Triage

The core of the question revolves around understanding how functional indexing works in a database context. Specifically, we need to explain why exact matching between the query expression and the index is necessary for optimal performance and correctness.

#### First-Principles Solution
Understanding Functional Indexes

A functional index is an index built on a computed column or function of another column. In this case, lower(email) creates a new virtual column in the database that maps each email to its lowercase version. The index can then be used to quickly look up emails based on their lowercased values.

Query Expression Matching

When you write a query like:

SQL
SELECT * FROM users WHERE lower(email) = 'example@example.com';

The database uses the lower(email) functional index to perform this lookup efficiently. However, if your query does not exactly match the index (e.g., using a different function or column), it may bypass the index and force a full table scan, leading to slower performance.

Why Exact Matching is Necessary
Index Utilization: The database can only use an index if the exact same expression is used in the query as was indexed. This ensures that the index keys are aligned with the query’s key.

#### Cons
istency and Correctness: If the query uses a different method to lower the email (e.g., LOWER(email) instead of lower(email)), it may produce slightly different results, leading to incorrect deduplication.
Performance Optimization: When the index is used correctly, the database can leverage precomputed values to perform the operation faster.

#### Closing Summary
and Common Mistakes

To summarize, using an exact matching query expression ensures that the database can utilize the lower(email) functional index for optimal performance and correctness in deduplication. Failure to match the index may result in a full table scan or incorrect results due to differences in how the email is processed.

#### Common Mistakes
Incorrect Case Handling: Using different case functions (e.g., LOWER() vs lower()) can lead to misaligned indexes.
Partial Matches: Queries that do not use the exact function (e.g., using a simple equality without applying the same transformation) will bypass the index.

**Code:**

```sql
-- Example query with functional index
SELECT * FROM users WHERE lower(email) = 'example@example.com';
```

