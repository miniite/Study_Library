---
type: "practice"
difficulty: "Intermediate"
---

# How can you grant a set of privileges to many users at once using roles?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To grant a set of privileges to multiple users at once, we can use roles in databases like PostgreSQL or Oracle. This approach helps manage permissions more efficiently and simplifies administrative tasks. By assigning specific privileges to a role and then granting that role to various users, you can achieve the desired permission management without manually altering each user's privileges.

#### Triage

The question is asking how to grant multiple privileges to several database users simultaneously using roles. The core idea here involves understanding Data Definition Language (DDL), Data Manipulation Language (DML), and Database Control Language (DCL). Specifically, we need to use the CREATE ROLE, GRANT, and REVOKE commands.

Step-by-Step Solution
Define Roles: First, create roles that encapsulate a set of privileges.
Grant Privileges to Roles: Grant necessary DDL/DML/TCL permissions to these roles.
Assign Roles to Users: Assign the roles to the desired users.

#### Example
in PostgreSQL
SQL
-- Step 1: Create roles with specific privileges
CREATE ROLE role1 WITH LOGIN;
GRANT SELECT, INSERT ON table1 TO role1;

CREATE ROLE role2 WITH LOGIN;
GRANT UPDATE, DELETE ON table2 TO role2;

-- Step 2: Assign roles to users
GRANT role1 TO userA, userB;
GRANT role2 TO userC, userD;

-- Step 3: Revoke if needed
REVOKE role1 FROM userE;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using roles effectively helps manage permissions in a database. By creating roles with specific privileges and assigning them to users, you can simplify the permission management process. Common mistakes include not defining clear roles upfront or assigning too broad privileges.

For follow-ups, one might ask about managing role hierarchies (e.g., granting a super-role) or handling large numbers of users and roles efficiently.

**Code:**

```sql
-- Example SQL for creating roles and granting permissions
CREATE ROLE role1 WITH LOGIN;
GRANT SELECT, INSERT ON table1 TO role1;

CREATE ROLE role2 WITH LOGIN;
GRANT UPDATE, DELETE ON table2 TO role2;

GRANT role1 TO userA, userB;
GRANT role2 TO userC, userD;

REVOKE role1 FROM userE;
```

