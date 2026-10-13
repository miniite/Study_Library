---
type: "practice"
difficulty: "Basic"
---

# What is the difference between GRANT and REVOKE?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a relational database management system (RDBMS), managing permissions to access and manipulate data is crucial for maintaining security. GRANT and REVOKE are two fundamental commands used in Database Control Language (DCL) to manage these permissions.

#### Triaging the Question

The question asks about the difference between GRANT and REVOKE. These commands are part of DCL, which stands for Data Control Language. The interviewer is probing your understanding of how permissions are managed within a database environment. You should explain that both GRANT and REVOKE deal with permission management but do so in opposite ways.

Detailed Explanation from First Principles
GRANT: This command allows you to give specific privileges or roles to users, groups, or other entities. For example, you can grant SELECT, INSERT, UPDATE, DELETE permissions on a table to a user.
REVOKE: Conversely, this command removes the granted privileges that were previously given by GRANT. It effectively nullifies the permission changes made by previous GRANT statements.

#### Example Scenario
s

Let's say you have a table named employees.

If you run:

SQL
GRANT SELECT ON employees TO user1;

This grants the SELECT privilege to user1, allowing them to query the employees table.

To remove this permission, you would use:

SQL
REVOKE SELECT ON employees FROM user1;
Why These Commands are Important

These commands help maintain data security and access control. For instance, if an employee leaves a company, their database access can be revoked to prevent unauthorized data exposure.

#### Common Mistakes
and Follow-Ups

Common Mistake: Not understanding the implications of GRANT vs REVOKE. Both are crucial for different use cases.

#### Follow-Up
"Can you explain how these commands interact with roles, and what happens if a user has both GRANT and REVOKE permissions?"

#### Closing Summary

In summary, GRANT is used to give specific privileges to users or groups, while REVOKE is used to remove those granted privileges. Both are essential for managing database access and ensuring data security.

**Code:**

```sql
-- GRANT example
GRANT SELECT ON employees TO user1;

-- REVOKE example
REVOKE SELECT ON employees FROM user1;
```

