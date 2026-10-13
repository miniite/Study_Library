---
type: "practice"
difficulty: "Intermediate"
---

# How do roles and GRANT/REVOKE work for managing permissions in PostgreSQL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage user permissions effectively in PostgreSQL, understanding how roles and GRANT/REVOKE work is crucial. This overview covers the basics of roles and how you can use GRANT and REVOKE to control access to database objects like tables, functions, and schemas.

#### Triaging the Question

The question revolves around two key concepts: roles in PostgreSQL and the commands used for managing permissions—GRANT and REVOKE. The goal is to understand how these components can be utilized to enforce a robust permission model. Specifically, we will explore:

What are roles?
How do you create and manage roles?
What are the functions of GRANT and REVOKE in the context of roles?

#### First-Principles Solution

In PostgreSQL, a role is an entity that can be granted privileges on database objects or other roles. Roles can be either users (who have a password) or groups (which do not require passwords). The primary commands for managing permissions are GRANT and REVOKE.

1. Creating and Managing Roles

To create a new role, you use the SQL command:

SQL
CREATE ROLE <role_name> [WITH PASSWORD | NOLOGIN] [INHERIT | NOINHERIT];
WITH PASSWORD: The role can log in to the database.
NOLOGIN: The role cannot log in but can be granted privileges on objects and other roles.
INHERIT: Inherits the permissions of a superuser.
NOINHERIT: Does not inherit the permissions of a superuser.

To assign a password, you use:

SQL
ALTER ROLE <role_name> PASSWORD 'your_password';
2. Granting and Revoking Permissions

GRANT is used to give privileges to roles or users on database objects. Common GRANT options include:

SELECT: Read data from a table.
INSERT: Insert new rows into a table.
UPDATE: Update existing rows in a table.
DELETE: Delete rows from a table.

#### Example

SQL
GRANT SELECT, INSERT ON TABLE customers TO sales;

To revoke permissions, use REVOKE followed by the same options as GRANT. For example:

SQL
REVOKE UPDATE, DELETE ON TABLE products FROM public;

#### Closing Summary
and Common Mistakes

Roles in PostgreSQL provide a flexible way to manage user access. By using GRANT and REVOKE, you can finely control permissions on various database objects.

#### Common Mistakes

Over-granting Permissions: Granting more privileges than necessary, which can lead to security vulnerabilities.
Inconsistent Role Management: Failing to update roles when the application's requirements change.
Password Expiry: Not regularly updating passwords or ensuring they are strong.

#### Example
Workflow

Suppose you have a sales role and want to grant it read and write access to the customers table:

SQL
CREATE ROLE sales;
GRANT SELECT, INSERT ON TABLE customers TO sales;

Then, if you later need to revoke the INSERT privilege, you can do so with:

SQL
REVOKE INSERT ON TABLE customers FROM sales;
Final Thoughts

Understanding roles and permissions in PostgreSQL is critical for maintaining a secure and efficient database environment. By carefully managing roles and using GRANT/REVOKE, you can ensure that only authorized users have the necessary access to your data.

