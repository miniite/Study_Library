---
type: "practice"
difficulty: "Intermediate"
---

# What is a role in Snowflake and how does role-based access control (RBAC) work?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Row Access Policies](<../../../../TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Role management in Snowflake is crucial for ensuring data security and compliance through Role-Based Access Control (RBAC). At its core, a role in Snowflake is a named set of permissions that can be assigned to users or other roles. This allows you to manage access rights centrally and flexibly.

#### Triaging the Question

The question asks about roles in Snowflake, specifically focusing on how RBAC works. The interviewer likely wants to evaluate your understanding of Snowflake's architecture and its security mechanisms. You should assume a basic knowledge of database management systems (DBMS) and permission management concepts.

Role-Based Access Control (RBAC)

In RBAC, permissions are assigned to roles rather than individual users. These roles can then be granted to multiple users or other roles. This hierarchical approach simplifies access management and improves security by allowing centralized control over who has what level of access.

Roles in Snowflake: In Snowflake, a role is a named collection of privileges that you can assign to one or more users or other roles.
Types of Roles:
System Role: Predefined roles provided by Snowflake for specific administrative tasks (e.g., SYSADMIN, ACCOUNTADMIN).
User-Defined Role: Custom roles created by the administrator.
Step-by-Step Explanation
Creating a Role: You can create a role using SQL commands.
SQL
CREATE ROLE my_role;
Granting Privileges to Roles: Assign privileges to this role, such as SELECT, INSERT, or EXECUTE on specific databases and tables.
SQL
GRANT SELECT ON DATABASE sales TO ROLE my_role;
Assigning Roles to Users: Grant the created role to users who need access.
SQL
GRANT ROLE my_role TO USER john_doe;

#### Example Scenario

Imagine you are setting up a data warehouse for an e-commerce company with multiple departments: Sales, Marketing, and Finance.

Sales Department: Needs read-only access to customer data in the sales database.
Marketing Department: Requires read-write access to marketing campaign data in the marketing database.
Finance Department: Needs full access (read/write) to financial records in the finance database.
Create roles for each department:
SQL
CREATE ROLE sales_role;
CREATE ROLE marketing_role;
CREATE ROLE finance_role;
Grant specific privileges to these roles:
SQL
GRANT SELECT ON DATABASE sales TO ROLE sales_role;
GRANT SELECT, INSERT ON SCHEMA marketing TO ROLE marketing_role;
GRANT ALL PRIVILEGES ON DATABASE finance TO ROLE finance_role;
Assign these roles to the respective users:
SQL
GRANT ROLE sales_role TO USER sales_user1, sales_user2;
GRANT ROLE marketing_role TO USER marketing_user1, marketing_user2;
GRANT ROLE finance_role TO USER finance_user1, finance_user2;

#### Closing Summary
and Common Mistakes

Effective role management in Snowflake ensures that access rights are controlled centrally. By using roles, you can easily manage who has access to what data, making it simpler to enforce security policies and comply with regulatory requirements.

#### Common Mistakes

Not understanding the difference between system roles and user-defined roles.
Failing to properly cascade permissions through roles, leading to incomplete or redundant access controls.
Misusing GRANT statements, such as granting privileges without specifying the correct databases or schemas.
Additional Considerations

RBAC is just one aspect of Snowflake’s security framework. Other important components include:

Row-Level Security (RLS): Fine-grained control over data access at a row level.
Policy-Based Access Control: Using policies to enforce complex access controls based on conditions and actions.

**Code:**

```sql
-- Example SQL commands for role management in Snowflake

CREATE ROLE my_role;
GRANT SELECT ON DATABASE sales TO ROLE my_role;
GRANT ROLE my_role TO USER john_doe;
```

