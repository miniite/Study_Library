---
type: "practice"
difficulty: "Deep"
---

# What is the difference between UPDATE and MERGE for changing existing rows?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database context, UPDATE and MERGE are both used to modify existing records in a table, but they operate under different principles and have distinct use cases.

#### Triage
of the Question

The interviewer is probing your understanding of SQL commands for modifying data. You need to explain the differences between UPDATE and MERGE, their syntax, and when each should be used. Key assumptions include familiarity with basic SQL operations (DDL/DML).

#### First-Principles Solution

1. UPDATE:

Syntax: The UPDATE statement is straightforward. It changes one or more columns in existing rows of a table.
SQL
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
Use Case: When you want to change the values of specific columns for certain rows that meet a WHERE clause.

2. MERGE:

Syntax: The MERGE statement is more complex and combines operations from both INSERT and UPDATE. It allows you to specify which rows should be inserted, updated, or deleted based on the existence of corresponding rows in another table.
SQL
MERGE INTO target_table t
USING source_table s ON (t.matching_column = s.matching_column)
WHEN MATCHED THEN
    UPDATE SET column1 = s.column1, column2 = s.column2, ...
WHEN NOT MATCHED THEN
    INSERT (column1, column2, ...) VALUES (s.column1, s.column2, ...);
Use Case: When you need to handle both UPDATE and INSERT operations in a single statement, possibly based on the existence of matching rows.

#### Closing Summary

In summary, while both UPDATE and MERGE modify existing records, UPDATE is simpler and more straightforward for direct row updates. On the other hand, MERGE provides a more powerful and flexible approach that can handle both updating and inserting new records based on matching criteria.

#### Common Mistakes/Follow-Ups
Misunderstanding Conditions: Failing to correctly use the WHERE clause in UPDATE or not understanding the conditions in MERGE.

#### Performance Considerations
Not considering the performance implications of using MERGE, especially when dealing with large datasets.
Data Consistency: Ignoring potential data inconsistencies that might arise from incorrect logic within MERGE.

