---
type: "practice"
difficulty: "Deep"
---

# Does defining a primary key automatically create an index? What kind of index is it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Defining a primary key in a database does indeed automatically create an index, but this index is specifically a unique and non-NULL constraint-enforcing B-tree index. Understanding this requires diving into how databases manage constraints and indexing.

#### Triage

The question probes your knowledge of database internals, specifically around the concepts of primary keys (PK) and indexes. You need to explain whether defining a PK inherently creates an index, what type of index it is, and why it matters for constraint enforcement.

#### First-Principles Solution
Primary Key Basics: A primary key uniquely identifies each record in a table. It must be unique and non-NULL.
Indexing Requirement: To enforce these constraints (uniqueness and non-NULL), the database system needs to maintain an index.
B-tree Index: In most relational databases, defining a PK automatically creates a B-tree index. This type of index is chosen because it supports fast lookups, which are critical for enforcing uniqueness.
Detailed Explanation

#### Cons
traint Enforcing Mechanism:

When you define a primary key, the database ensures that no two rows can have identical values in this column (uniqueness) and that every row must have a value (non-NULL).
These constraints are enforced by maintaining an index on the PK.

B-tree Index Characteristics:

A B-tree is a balanced tree data structure where each node contains multiple keys and pointers to child nodes.
In a B-tree index for a primary key, every value in the column has a corresponding leaf node pointer, allowing for efficient lookup operations.
The non-NULL constraint is inherently enforced by the fact that an empty or NULL entry would violate the uniqueness requirement.

Why a B-tree Index:

B-trees are chosen because they provide fast access to data and are well-suited for large datasets, which aligns with their use in primary key management.
The unique nature of PK values means that each node in the index is a leaf, simplifying the structure.

#### Performance Considerations

While B-tree indices ensure efficient query performance (both reads and writes), they come at the cost of additional storage overhead.
However, this trade-off is often acceptable given the critical role primary keys play in data integrity.

#### Closing Summary

Defining a primary key in a database automatically creates a unique and non-NULL constraint-enforcing B-tree index. This design choice ensures efficient enforcement of the PK's uniqueness rule while maintaining optimal query performance. Understanding this relationship between constraints and indexing is crucial for designing robust database schemas.

#### Common Mistakes/Follow-Ups
Assuming Other Index Types: Be wary of assuming that other types of indexes (like hash or bitmap) are used; B-trees are standard.
Ignoring Non-NULL Constraint: Remember, the non-NULL constraint is also enforced by this index.
Understanding Performance Trade-offs: Explore how different index types and their costs impact overall database performance.

**Code:**

```sql
-- Example of creating a table with a primary key
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);
```

**Flow:**

1. Define Primary Key
2. Automatic Index Creation
3. B-tree Index
4. Unique Values Enforcement
5. Non-NULL ConstraintEnforcement

