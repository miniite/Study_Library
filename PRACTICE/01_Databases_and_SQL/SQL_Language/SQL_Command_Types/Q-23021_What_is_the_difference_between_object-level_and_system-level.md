---
type: "practice"
difficulty: "Deep"
---

# What is the difference between object-level and system-level privileges in GRANT?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Object-level and system-level privileges in SQL's GRANT command are crucial for understanding how access control works in databases, particularly when managing permissions on specific objects versus the entire database or server level.

#### Triaging the Question

The GRANT statement is used to assign privileges (permissions) to users or roles. The difference between object-level and system-level privileges lies in where these privileges are defined:

System-Level Privileges: These apply to the entire database system, affecting all objects within it.
Object-Level Privileges: These are granted on specific database objects such as tables, views, or columns.
Step-by-Step Explanation
System-Level Privileges
Scope: System-level privileges affect the whole database server and can be granted at the database, schema (or catalog), table, column, or procedure level.
Common Privileges: Examples include CREATE, DROP, SELECT, INSERT, UPDATE, DELETE, REFERENCES, etc.

#### Example
Usage:
SQL
GRANT SELECT ON SCHEMA public TO user1;
This grants the SELECT privilege on all tables within the public schema to user1.
Object-Level Privileges
Scope: Object-level privileges are granted at a more granular level, targeting specific objects like tables or columns.
Common Privileges: These can be similar to system-level privileges but are scoped to specific objects.

#### Example
Usage:
SQL
GRANT SELECT ON TABLE employees TO user1;
This grants the SELECT privilege on the employees table specifically to user1.

#### Closing Summary
and Common Mistakes

In summary, understanding whether you're dealing with system-level or object-level privileges is crucial because it defines the granularity of access control. For instance, granting CREATE TABLE at a system level would allow users to create new tables in any schema, whereas granting the same privilege on an object level restricts this ability only to specific tables.

#### Common mistakes
include:

Granting overly broad permissions (e.g., ALL PRIVILEGES) without considering the specific needs of users.
Not understanding the difference between system and object-level privileges can lead to security risks or misconfigured access controls.

#### Follow-Up
Questions
What are the implications if a user has both object-level and system-level privileges on the same table?
How does REVOKE differ from GRANT in terms of managing permissions?

**Code:**

```sql
-- Example of granting SELECT privilege at the schema level
GRANT SELECT ON SCHEMA public TO user1;

-- Example of granting SELECT privilege on a specific table
GRANT SELECT ON TABLE employees TO user1;
```

