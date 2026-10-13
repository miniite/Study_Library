---
type: "practice"
difficulty: "Deep"
---

# You add ORDER BY id at the end of a paginated query 'for stability' and the plan changes to a sort. Why, and how do you keep both stable order and index usage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keyset Pagination](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address why adding ORDER BY id at the end of a paginated query causes a sort operation and how to maintain both stability in order and efficient use of indexes, let's first understand the core concepts involved.

TL;DR:

Adding ORDER BY id forces the database to perform a sorting operation, even if an index on id exists. To keep both stable order and effective index usage, you need to leverage indexing properly or consider alternative query strategies.

#### Triaging the Question

The question is essentially asking why a specific query transformation occurs and how to handle it. The key points are:

Why does adding ORDER BY id cause a sort operation?
How can we maintain stable order while using indexes effectively?

#### Assumptions
We have a paginated query.
An index on the column id is available.
The database engine uses cost-based optimization.

#### First Principles Solution
Why Sorting Occurs with ORDER BY

When you add ORDER BY id, the database optimizer must ensure that the results are sorted based on this column. Even if an index exists, the query planner might choose to perform a sort operation due to various factors such as the cost of maintaining partial ordering in intermediate steps or other optimization considerations.

Understanding Index Usage

Indexes can be used efficiently for SELECT operations where the condition matches the leading columns of the index (e.g., WHERE id = x). However, when an ORDER BY clause is added, the database might choose to sort the result set even if an index on id exists. This happens because:

The query planner might consider a full sort cheaper than maintaining partial ordering.
The cost of sorting might be lower in terms of overall execution time.
Maintaining Stable Order and Index Usage

To maintain both stable order and efficient index usage, you can use a combination of techniques:

Primary Key as the ORDER BY Column: Ensure that id is part of the primary key or a unique index.
Covering Indexes: Create an index on (id, other_columns) if possible. This helps in reducing the number of disk accesses during sorting.
LEAD/LAG Functions: For more complex scenarios, consider using window functions like ROW_NUMBER() to simulate pagination without full sorts.

#### Example Scenario

#### Cons
ider a table users with an index on id:

SQL
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);

A paginated query might look like this:

SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

When you add ORDER BY id, the database engine will likely perform a full sort, even if an index on id exists.

Alternative Strategies

To avoid unnecessary sorts and maintain stable order:

Primary Key as ORDER BY:

SQL
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

Covering Index with LEAD Function:

SQL
WITH numbered_users AS (
    SELECT *, ROW_NUMBER() OVER (ORDER BY id) AS rn FROM users
)
SELECT * FROM numbered_users WHERE rn BETWEEN 21 AND 30;

#### Closing Summary
and Common Mistakes
Adding ORDER BY can force a sort operation, even with an existing index.
Use primary key as the order by column for simplicity.

#### Cons
ider covering indexes to reduce disk access during sorting.
Leverage window functions like ROW_NUMBER() to simulate pagination without full sorts.

#### Common Mistakes
and Follow-Ups
Ignoring Index Usage: Failing to consider how an index can be used effectively with ORDER BY.
Over-Simplification: Thinking that adding a covering index always solves the issue.
Performance Overhead: Not considering the potential performance overhead of using window functions or sorting.

By understanding these principles, you can optimize queries and maintain both stable order and efficient use of indexes in your database operations.

