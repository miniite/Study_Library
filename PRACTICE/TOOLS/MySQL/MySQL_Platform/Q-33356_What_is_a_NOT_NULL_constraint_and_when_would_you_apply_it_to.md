---
type: "practice"
difficulty: "Basic"
---

# What is a NOT NULL constraint and when would you apply it to a column?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A NOT NULL constraint in MySQL ensures that a column cannot contain any null values. Essentially, it enforces data integrity by forcing each row inserted into or updated within this column to have a non-null value.

#### Triage
The question is asking about the definition and use case of NOT NULL constraints.
The interviewer may be probing your understanding of basic SQL concepts and how they relate to ensuring data quality.

#### First Principles Solution

Defining the Constraint: In SQL, a NOT NULL constraint is specified when creating or altering a table. For example:

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(50),
    hire_date DATE
);

Here, the name column cannot accept null values, whereas other columns like position and hire_date can be nullable.

When to Apply: You would apply a NOT NULL constraint in scenarios where having a non-null value is critical. For example:

Unique Identifiers: IDs or primary keys often need to be unique and not null.
Required Information: Columns representing essential data, such as names, email addresses, etc., should generally have the NOT NULL constraint.

#### Common Mistakes
and Follow-Ups:

Misunderstanding of Null Values:

A common mistake is confusing null with empty strings or zero values. Ensure you understand that null represents an unknown value.

Overuse vs Underuse:

Overusing NOT NULL can lead to data loss if important information might be unknown initially.
Underusing it may result in inconsistent data.

Alternative Constraints:

Discuss the importance of other constraints like UNIQUE, CHECK, and DEFAULT that can complement NOT NULL.

#### Closing Summary

Understanding when to apply a NOT NULL constraint is crucial for maintaining data integrity. It ensures that essential fields always have values, which can prevent logical errors in queries or applications.

**Code:**

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(50),
    hire_date DATE
);
```

