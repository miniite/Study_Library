---
type: "practice"
difficulty: "Intermediate"
---

# Does TRUNCATE reset the AUTO_INCREMENT / identity counter? Explain.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DELETE vs TRUNCATE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database, TRUNCATE and DROP are often used to remove data from tables, but they differ significantly in how they operate and affect the table's structure. Specifically, regarding the AUTO_INCREMENT or identity counter, it is crucial to understand what TRUNCATE does.

TL;DR: TRUNCATE does not reset the AUTO_INCREMENT value; this can be done using specific SQL commands.

When you use TRUNCATE, the operation removes all rows from a table but does not reinitialize the AUTO_INCREMENT counter. Instead, it simply sets the last used value to 1 and allows new inserts to resume from there.

#### Triaging the Question

The question is asking about the behavior of the AUTO_INCREMENT or identity column after using the TRUNCATE command. The interviewer might be probing your understanding of database operations and their effects on table structure, particularly how data manipulation commands work with auto-incrementing fields.

#### Assumptions
We are considering a relational database management system (RDBMS) that supports auto-increment.
The question specifically refers to the TRUNCATE command.
Detailed Solution

To understand this fully, let's break down what happens when you use TRUNCATE.

What TRUNCATE Does:

TRUNCATE is a DDL (Data Definition Language) command that removes all rows from a table but does not reinitialize the AUTO_INCREMENT counter.
It is faster and uses fewer resources compared to using DELETE because it doesn't log individual row deletions.

Impact on AUTO_INCREMENT:

After executing TRUNCATE, the last used value of the auto-increment column remains, but subsequent inserts will start from 1 again.
For example, if you had an id column with a current maximum value of 100 before truncation, after TRUNCATE, the next insert would set id = 1.

Why AUTO_INCREMENT is Not Reset by TRUNCATE:

The auto-increment counter is stored in the system catalog or metadata of the database.
TRUNCATE only clears data and marks the table as empty, but it does not update this metadata.

Reseting AUTO_INCREMENT Explicitly:

If you need to reset the auto-increment value explicitly, you would use a DDL command like:
SQL
ALTER TABLE table_name AUTO_INCREMENT = new_value;
For example, if you want to start with id = 101, you could run:
SQL
ALTER TABLE my_table AUTO_INCREMENT = 101;

#### Cons
iderations:

If the sequence of IDs is crucial for your application or audit trails, using TRUNCATE might not be ideal.
Using DELETE FROM table_name; would leave the auto-increment value intact but would use more resources.

#### Closing Summary
and Common Mistakes

In summary, TRUNCATE does not reset the AUTO_INCREMENT counter. To reset it explicitly, you should use an ALTER TABLE command with a specific new value.

#### Common mistakes
include confusing TRUNCATE with DELETE, assuming that TRUNCATE resets the auto-increment, or failing to recognize when explicit resetting of the sequence is necessary for your application logic.

#### Follow-Ups
How does this behavior differ across different RDBMS like MySQL, PostgreSQL, and SQL Server?
Can you give an example where explicitly resetting the auto-increment might be important?

**Code:**

```sql
-- Example TRUNCATE command
TRUNCATE TABLE my_table;

-- Resetting AUTO_INCREMENT
ALTER TABLE my_table AUTO_INCREMENT = 101;
```

