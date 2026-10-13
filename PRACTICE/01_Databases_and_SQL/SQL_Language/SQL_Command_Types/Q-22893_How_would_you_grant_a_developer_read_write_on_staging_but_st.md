---
type: "practice"
difficulty: "Intermediate"
---

# How would you grant a developer read/write on staging but strictly read-only on prod using the same role model across environments?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage different levels of access for developers in staging and production databases while maintaining consistency, we can use roles with specific privileges tailored to each environment. Here's a step-by-step breakdown:

#### Overview
We'll create two roles, one for read/write access on the staging database and another for strictly read-only access on the production database.

#### Triage
The question is asking how to enforce different levels of access (read/write vs. read-only) across environments using a role-based model. This involves defining specific privileges for each environment and ensuring they are applied correctly.

#### First-Principles Solution

Define roles with appropriate DDL/DML/DCL/TCL privileges.
Create users and associate them with the relevant roles.
Apply these roles to different databases or schemas.

#### Closing Summary
+ Common Mistakes/Follow-Ups: This approach ensures consistency in role management while providing fine-grained control over access levels across environments. Common pitfalls include misconfiguring roles, not properly managing user associations, and forgetting to apply environment-specific privileges.

Let's go through the detailed steps:

Step 1: Define Roles

First, we need to define two roles: one for read/write access on the staging database and another for strictly read-only access on the production database.

SQL
-- For Staging Database
CREATE ROLE staging_read_write;

GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA staging TO staging_read_write;

-- For Production Database
CREATE ROLE prod_read_only;

GRANT SELECT ON SCHEMA prod TO prod_read_only;
Step 2: Create Users

Next, create users for the developers and assign them to the appropriate roles.

SQL
CREATE USER developer1 IDENTIFIED BY 'password';
CREATE USER developer2 IDENTIFIED BY 'password';

-- Assigning roles to users
GRANT staging_read_write TO developer1;
GRANT prod_read_only TO developer2;
Step 3: Apply Roles Across Environments

Ensure that the appropriate roles are applied to the correct databases or schemas.

SQL
-- Ensure the roles are properly set in the database configuration
Step 4: Monitor and Audit

Regularly monitor access and audit logs to ensure compliance with the defined roles.

SQL
-- Example of an audit table
CREATE TABLE audit_logs (
    user_name VARCHAR(50),
    action VARCHAR(20),
    timestamp TIMESTAMP,
    environment VARCHAR(10)
);

#### Common Mistakes/Follow-Ups
Role Configuration Errors: Ensure that the roles are correctly defined and assigned.
Privilege Conflicts: Double-check for any conflicting privileges that might cause issues.
Environment Consistency: Verify that the roles are applied consistently across all relevant environments.

**Comparison:**

| Column  | Data Type    | Description                            |
|---------|-------------|----------------------------------------|
| user_name | VARCHAR(50) | Name of the user                        |
| action   | VARCHAR(20)  | Action performed (e.g., SELECT, INSERT) |
| timestamp | TIMESTAMP   | Timestamp of the event                  |
| environment | VARCHAR(10) | Environment where the event occurred    |

**Code:**

```sql
-- Creating roles and granting privileges

CREATE ROLE staging_read_write;

GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA staging TO staging_read_write;

CREATE ROLE prod_read_only;

GRANT SELECT ON SCHEMA prod TO prod_read_only;

-- Creating users and assigning roles

CREATE USER developer1 IDENTIFIED BY 'password';
CREATE USER developer2 IDENTIFIED BY 'password';

GRANT staging_read_write TO developer1;
GRANT prod_read_only TO developer2;
```

