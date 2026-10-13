---
type: "practice"
difficulty: "Intermediate"
---

# Does a UNIQUE constraint automatically create an index? Explain.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this interview question, we're exploring the relationship between UNIQUE constraints and indexes in database systems. The key idea is to understand that while a UNIQUE constraint enforces uniqueness within a column or set of columns, it does not automatically create an index. However, under certain conditions, the database management system (DBMS) may choose to create an index for performance reasons.

#### Triage

The question asks whether a UNIQUE constraint creates an index and requires us to explain why or why not. This is probing your understanding of how databases manage constraints and indexes, as well as your ability to articulate the nuances between them.

#### First Principles Solution
Understanding UNIQUE Constraints: A UNIQUE constraint ensures that all values in a specified column (or columns) are distinct and prevents duplicate entries.
Indexes for Performance: Indexes enhance query performance by enabling faster data retrieval, especially when you frequently search or sort on specific columns.
Step-by-Step Solution

UNIQUE Constraint Basics:

When a UNIQUE constraint is applied to a column (or multiple columns), the database ensures that no duplicate values are inserted.
However, this constraint alone does not automatically create an index; it merely enforces uniqueness.

Index Creation for Performance:

The decision to create an index on a UNIQUE column or set of columns is typically driven by performance considerations rather than the presence of the UNIQUE constraint itself.
In some databases, such as MySQL and PostgreSQL, a UNIQUE constraint may implicitly use an index for maintaining uniqueness. However, this behavior can vary between different DBMSs.

Database Behavior:

For example, in PostgreSQL, when you create a UNIQUE index, the database will automatically enforce uniqueness by using the index.
In contrast, in Oracle, creating a UNIQUE constraint does not necessarily create an index; instead, it may use existing indexes or no indexing at all.

#### Closing Summary

In summary, while a UNIQUE constraint ensures that data remains unique, it does not automatically create an index. The decision to create an index is driven by performance considerations and can vary between different DBMSs. Understanding these nuances helps in designing efficient database schemas and optimizing query performance.

#### Common Mistakes/Follow-Ups
Misunderstanding: Believing a UNIQUE constraint always creates an index.
Assumption about Behavior: Assuming all DBMSs behave the same way regarding UNIQUE constraints and indexes.

#### Follow-Up
Question: The interviewer might ask about the trade-offs between using UNIQUE constraints vs. explicitly creating indexes, or how to optimize query performance in different scenarios.

**Code:**

```

-- Example of creating a UNIQUE constraint (no index created by default) CREATE TABLE users ( id SERIAL PRIMARY KEY, email VARCHAR(255) UNIQUE -- No explicit index creation here );

CODE
Copy

```sql
-- Example of creating both UNIQUE and INDEX explicitly
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);

-- Explicitly create an index for the email column
CREATE INDEX idx_email ON users (email);
```

