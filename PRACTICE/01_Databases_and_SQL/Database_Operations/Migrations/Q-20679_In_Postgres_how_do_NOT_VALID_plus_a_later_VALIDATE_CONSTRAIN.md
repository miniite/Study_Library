---
type: "practice"
difficulty: "Deep"
---

# In Postgres, how do NOT VALID plus a later VALIDATE CONSTRAINT let you add a check constraint with minimal locking?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Concurrent Index Builds](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of using NOT VALID and VALIDATE CONSTRAINT in Postgres to add a check constraint with minimal locking, we need to understand the underlying mechanics of database constraints and transactions. This approach is particularly useful when dealing with large datasets or complex schema changes.

In essence, the question is probing your understanding of transactional semantics, constraint validation strategies, and how to minimize downtime during critical operations. Let's break this down step by step.

#### Triage

The question asks about adding a check constraint in Postgres using NOT VALID and VALIDATE CONSTRAINT. This involves:

Understanding Transactional Semantics: How constraints are enforced within transactions.

#### Cons
traint Validation Strategies: The difference between immediate validation and deferred validation.
Minimal Locking: Techniques to ensure that schema changes do not cause prolonged blocking of other operations.

#### First Principles Solution
Initial Schema Setup:
Define a table with a column that needs to be updated later without violating an existing constraint.
Adding the Constraint Not Valid:
Use NOT VALID to add the constraint immediately but defer its enforcement until the next transaction.
Updating Data:
Perform bulk updates or insertions that may violate the new constraint.
Validating the Constraint:
Run a VALIDATE CONSTRAINT statement to enforce the constraint, which will fail if there are violations.
Detailed Solution

Initial Schema Setup

SQL
CREATE TABLE example (
    id SERIAL PRIMARY KEY,
    value INT CHECK (value > 0)
);

Adding the Constraint Not Valid

SQL
ALTER TABLE example ADD CONSTRAINT ensure_positive_value CHECK (value > 0) NOT VALID;

Updating Data This step simulates bulk data updates that may violate the constraint.

SQL
-- Simulate a large dataset with some values <= 0
INSERT INTO example (value) SELECT random() * -100 FROM generate_series(1, 100000);

Validating the Constraint Finally, validate the constraint to enforce it across all rows.

SQL
ALTER TABLE example VALIDATE CONSTRAINT ensure_positive_value;

This step will block until all rows are validated and constraints are enforced.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using NOT VALID and VALIDATE CONSTRAINT allows you to add a check constraint with minimal locking by deferring the immediate enforcement of the constraint. The key steps are:

Adding the constraint in a non-blocking manner.
Performing bulk updates or inserts without violating the constraint.
Validating the constraint later, which may block but ensures data integrity.

#### Common mistakes
include not understanding the implications of NOT VALID and expecting constraints to be enforced immediately after definition. Another mistake is failing to validate constraints when they are added, leading to data inconsistencies.

#### Follow-ups
could involve discussing how this approach scales with larger datasets or more complex schemas, or exploring alternative strategies like partitioning data during updates.

**Comparison:**

| id | value |
|----|-------|
| 1  | -56   |
| 2  | -34   |
| 3  | 78    |

**Code:**

```sql
-- Initial Schema Setup
CREATE TABLE example (
    id SERIAL PRIMARY KEY,
    value INT CHECK (value > 0)
);

-- Adding the Constraint Not Valid
ALTER TABLE example ADD CONSTRAINT ensure_positive_value CHECK (value > 0) NOT VALID;

-- Simulate a large dataset with some values <= 0
INSERT INTO example (value) SELECT random() * -100 FROM generate_series(1, 100000);

-- Validate the Constraint
ALTER TABLE example VALIDATE CONSTRAINT ensure_positive_value;
```

**Example input / output:**

```text
-- Input
CREATE TABLE example (
    id SERIAL PRIMARY KEY,
    value INT CHECK (value > 0)
);

INSERT INTO example (value) SELECT random() * -100 FROM generate_series(1, 100000);

ALTER TABLE example ADD CONSTRAINT ensure_positive_value CHECK (value > 0) NOT VALID;

-- Output
-- Data insertion and constraint addition will proceed without immediate validation.
```

**Data:**

```sql
CREATE TABLE example (
    id SERIAL PRIMARY KEY,
    value INT CHECK (value > 0)
);
INSERT INTO example (value) SELECT random() * -100 FROM generate_series(1, 100000);
ALTER TABLE example ADD CONSTRAINT ensure_positive_value CHECK (value > 0) NOT VALID;
```

**Flow:**

1. Initial Schema Setup
2. Add NOT VALID Constraint
3. Insert Data with PotentialViolations
4. Validate Constraint Later
5. Ensure Consistent Data

