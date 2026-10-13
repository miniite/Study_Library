---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a soft delete and a hard delete, and what are the querying trade-offs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Databases and Schemas](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database management systems (DBMS), both hard deletes and soft deletes serve to remove records from the database but differ significantly in their approach and implications. A hard delete involves permanently removing a record from the database, making it inaccessible even with a SELECT statement or recovery mechanisms. On the other hand, a soft delete marks a record as deleted by updating its status (e.g., setting a deleted_at timestamp) but retains the record in the database. This approach allows for easier data recovery and auditing.

To understand the trade-offs between hard deletes and soft deletes, we need to consider several factors: query performance, data integrity, and operational flexibility. Let's delve into these from first principles.

#### Triaging the Question

The interviewer is probing your understanding of the fundamental concepts behind data deletion strategies in a DBMS context. The key aspects include:

Data Retention: How long do records need to be retained?
Query Performance: What impact does retaining deleted records have on query performance and complexity?
Operational Flexibility: How easy is it to recover or manage soft-deleted data compared to hard-deleted data?

#### Assumptions
We are discussing typical use cases in a relational database context.
The goal of both deletion methods is to eventually remove records from the database, but with different strategies.
Step-by-Step Solution
Hard Deletes

Advantages:

Simplicity: Queries and data retrieval are straightforward since deleted records do not interfere with queries.
Performance: Typically faster due to no need to track or filter out soft-deleted records.
Storage Efficiency: Frees up space more quickly.

Disadvantages:

Irreversibility: Deleted records cannot be easily recovered, which can lead to accidental loss of critical data.
Data Integrity: May cause issues if the application assumes certain records will always exist.
Soft Deletes

Advantages:

Flexibility: Allows for easy recovery and modification of soft-deleted records.
Auditability: Maintains a historical record, enabling better tracking and compliance.
Operational Benefits: Can be useful in scenarios where data needs to be preserved but marked as inactive.

Disadvantages:

Complex Queries: Requires additional logic in queries to exclude or include soft-deleted records.
Performance Overhead: Additional database space is occupied by records that are technically "deleted."
Potential for Data Integrity Issues: Soft deletes can complicate data validation and referential integrity checks.

#### Example Scenario

#### Cons
ider an application managing employee records. If we use a hard delete approach, all employee records will be permanently removed from the database. In contrast, with soft deletes, we might add a deleted_at column to the employees table and update this timestamp when an employee is marked as deleted.

SQL Example
SQL
-- Hard Delete
DELETE FROM employees WHERE id = 1;

-- Soft Delete
UPDATE employees SET deleted_at = NOW() WHERE id = 1;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the choice between hard deletes and soft deletes depends on the specific use case. Hard deletes are simpler but risk data loss, while soft deletes offer flexibility and ease of recovery at the cost of additional complexity in queries.

#### Common mistakes
include:

Ignoring Data Recovery: Assuming that once a record is deleted, it cannot be recovered.
Overusing Soft Deletes: Using them for all deletions without considering the overhead they introduce.

#### Follow-up
questions might explore specific scenarios where one method would be preferable over the other or how to optimize queries in environments using soft deletes.

**Comparison:**

```
Column	Type
id	INT
name	VARCHAR
deleted_at	TIMESTAMP
```

**Code:**

```sql
-- Hard Delete Example
DELETE FROM employees WHERE id = 1;

-- Soft Delete Example
UPDATE employees SET deleted_at = NOW() WHERE id = 1;
```

**Flow:**

1. Yes
2. No
3. User Request
4. Delete Record
5. Is Soft Delete?
6. Update Record with deleted_at= NOW()
7. DELETE FROM employeesWHERE id = [id]

