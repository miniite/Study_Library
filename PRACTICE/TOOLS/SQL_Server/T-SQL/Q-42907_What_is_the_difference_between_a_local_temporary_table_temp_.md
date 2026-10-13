---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a local temporary table (#temp) and a global temporary table (##temp)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Local temporary tables (#temp) and global temporary tables (##temp) are both used to store data temporarily in SQL Server, but they have distinct differences that affect their visibility and lifespan.

#### Triaging the Question

The core of this question is understanding the scope and lifetime characteristics of these two types of temporary tables. The interviewer is probing your knowledge of how T-SQL manages temporary storage and the implications for database design and performance.

#### First-Principles Solution
Local Temporary Tables (#temp):
Visibility: A local temporary table created with a # prefix is only visible within the current session that created it.
Lifetime: It is automatically dropped when the user disconnects from SQL Server or when the last reference to the table goes out of scope, which typically occurs at the end of the batch or transaction in which it was created.
Global Temporary Tables (##temp):
Visibility: A global temporary table created with a ## prefix is visible across all sessions and transactions within the same database.
Lifetime: It remains available until the last session that created it disconnects, or if explicitly dropped using the DROP TABLE statement.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Understanding these differences can help in optimizing query performance and avoiding conflicts between different sessions. For example, using a global temporary table might be necessary when multiple users need to reference the same data temporarily across their respective sessions. However, misuse could lead to unintentional sharing of session-specific data.

#### Example Scenario
s

Local Temporary Table: Ideal for storing data that is specific to and only needed by one user or transaction.

SQL
-- Creating a local temporary table in SQL Server
CREATE TABLE #temp (
    id INT,
    name VARCHAR(100)
);

Global Temporary Table: Suitable for scenarios where multiple sessions need to reference the same data temporarily and persistently across transactions.

SQL
-- Creating a global temporary table in SQL Server
CREATE TABLE ##temp (
    id INT,
    name VARCHAR(100)
);
Closing Thoughts

Misunderstanding the scope and lifecycle of these tables can lead to errors such as accidental sharing of session-specific data or premature deletion of shared data. Always consider the implications carefully based on your use case.

**Code:**

```sql
-- Example of creating a local temporary table
CREATE TABLE #temp (
    id INT,
    name VARCHAR(100)
);

-- Example of creating a global temporary table
CREATE TABLE ##temp (
    id INT,
    name VARCHAR(100)
);
```

