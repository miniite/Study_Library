---
type: "practice"
difficulty: "Deep"
---

# Is CREATE USER a DDL or DCL command? Explain the classification.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML Statements](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To classify whether CREATE USER is a Data Definition Language (DDL) or Data Control Language (DCL) command, we need to understand the core definitions and use cases of each:

DDL commands are used to create, alter, or drop database objects such as tables, indexes, views, etc.
DCL commands are used for data access control, primarily GRANT and REVOKE.

CREATE USER is a DDL command because it is used to define the structure of a database object—specifically, user accounts or roles within the database.

**Code:**

```sql
-- Example of creating a new user using CREATE USER in SQL
CREATE USER new_user IDENTIFIED BY password;
```

