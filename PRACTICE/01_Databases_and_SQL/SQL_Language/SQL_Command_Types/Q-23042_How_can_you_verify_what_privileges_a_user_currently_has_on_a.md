---
type: "practice"
difficulty: "Intermediate"
---

# How can you verify what privileges a user currently has on a table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To verify what privileges a user currently has on a table, we need to understand and use SQL commands that interact with system catalogs or information schema views. This process involves DCL (Data Control Language) commands and can be broken down into steps from first principles.

Firstly, let's triage the question:

The interviewer is probing your knowledge of privilege verification mechanisms in a relational database management system.
You need to demonstrate understanding of how permissions are stored and retrieved using SQL queries.

#### Assumptions
include familiarity with DCL commands and an understanding that privilege information is typically stored within metadata structures.

Now, let's walk through the steps:

Identify the User and Table: Determine which user and table you want to check privileges for. For example, if we are checking user alice's permissions on table orders.

Query System Catalog Tables or Information Schema Views:

In PostgreSQL, use information_schema. Specifically, query the INFORMATION_SCHEMA.TABLE_PRIVILEGES view.
SQL
SELECT grantee, privilege_type
FROM information_schema.table_privileges
WHERE table_name = 'orders' AND grantee = 'alice';

Use Built-in Functions:

In Oracle, you can use the DBA_TAB_PRIVS or ALL_TAB_PRIVS views to check privileges.
SQL
SELECT GRANTEE, PRIVILEGE
FROM DBA_TAB_PRIVS
WHERE TABLE_NAME = 'ORDERS' AND GRANTEE = 'ALICE';

Check the User's Direct Privileges:

You might want to see if alice has any direct privileges on table orders.
SQL
SELECT PRIVILEGE
FROM DBA_TAB_PRIVS
WHERE TABLE_NAME = 'ORDERS' AND GRANTEE = 'ALICE';

#### Cons
ider Role-Based Privileges:

Roles can be assigned to users, and these roles might grant additional privileges.
SQL
SELECT ROLE_NAME
FROM DBA_ROLE_PRIVS
WHERE GRANTEE = 'ALICE' AND GRANTED_ROLE IN ('ORDERS_ROLE');

Combining Privileges:

The effective privilege can be the combination of direct and role-based privileges.
SQL
SELECT PRIVILEGE, GRANTED_ROLE
FROM DBA_TAB_PRIVS
WHERE TABLE_NAME = 'ORDERS' AND GRANTEE = 'ALICE';

#### Closing Summary
By leveraging system catalog tables or information schema views specific to the database management system (DBMS) in use, you can verify a user's privileges on a table. This involves querying relevant metadata structures and considering both direct and role-based privileges.

#### Common Mistakes

Forgetting to account for roles that might grant additional permissions.
Not checking information_schema or DBMS-specific views when using generic SQL statements.
Assuming all DBMS have the same structure for privilege storage, which can vary significantly.

