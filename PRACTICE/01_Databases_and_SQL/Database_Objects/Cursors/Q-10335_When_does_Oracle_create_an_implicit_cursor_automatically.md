---
type: "practice"
difficulty: "Basic"
---

# When does Oracle create an implicit cursor automatically?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursor Basics and Ref Cursors](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursor_Basics_and_Ref_Cursors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In Oracle, an implicit cursor is created automatically when a SELECT statement or other SQL operations are executed without explicitly declaring a cursor. The core idea is to streamline common operations and minimize boilerplate code.

#### Triaging the Question

The question probes your understanding of how Oracle handles implicit cursors in SQL statements. Specifically, it aims to see if you know when these cursors are automatically generated. Implicit cursors handle simple queries or DML (Data Manipulation Language) operations without requiring explicit cursor declarations by the user.

#### First-Principles Solution

Implicit Cursors in Oracle: In Oracle, an implicit cursor is created automatically for certain SQL statements. This happens whenever a SELECT statement or other data retrieval operations are executed. The key points to remember are:

SELECT Statements: When you run any SELECT query.
PL/SQL Data Manipulation Operations: For INSERT, DELETE, and UPDATE statements, Oracle also creates implicit cursors.

When an Implicit Cursor is Created:

When a SQL statement (e.g., SELECT, INSERT, UPDATE, DELETE) is executed without an explicit cursor.
When you use the EXECUTE IMMEDIATE command with a dynamic SQL string that includes a SELECT query.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, implicit cursors are automatically created by Oracle for simple data retrieval and manipulation operations. Understanding when these cursors are used is crucial for effective SQL programming in Oracle environments.

#### Common Mistakes

Assuming all SQL operations require explicit cursor declarations.
Overlooking the automatic creation of implicit cursors with SELECT, INSERT, UPDATE, or DELETE statements.

#### Follow-Ups

The interviewer might ask:

Difference Between Explicit and Implicit Cursors: How do they differ in terms of performance, control, and use cases?
Dynamic SQL Execution: How does Oracle handle implicit cursors when using EXECUTE IMMEDIATE with dynamic SQL?

**Code:**

```

-- Example of an implicit cursor creation via a SELECT statement SELECT * FROM employees;

CODE
Copy

```sql
-- Example of an implicit cursor creation via an UPDATE statement
UPDATE employees SET salary = 5000 WHERE employee_id = 101;
```

**Flow:**

1. SELECT * FROM employees
2. Implicit Cursor Created
3. UPDATE employees SET salary= 5000 WHERE employee_id =101
4. Implicit Cursor Created

