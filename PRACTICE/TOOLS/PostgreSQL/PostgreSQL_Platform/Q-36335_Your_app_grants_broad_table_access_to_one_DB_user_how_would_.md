---
type: "practice"
difficulty: "Intermediate"
---

# Your app grants broad table access to one DB user; how would you move to least-privilege with roles, column-level GRANTs, and a separate migration role, and why does that limit blast radius on a compromise?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of moving from broad table access to least-privilege in PostgreSQL while ensuring minimal blast radius in case of a security breach, we will leverage roles and column-level GRANTs. This approach minimizes the potential damage by restricting permissions to only what is necessary for each user or application.

#### Triaging the Question

The core question here is: How can you implement least-privilege access control using PostgreSQL roles and column-level GRANTS in a way that limits the impact of a security breach?

#### Assumptions

The current app grants broad table access to one DB user.
The goal is to transition this user to more granular permissions while maintaining functionality.
Security best practices are being followed, such as regular audits and monitoring.
Step-by-Step Solution
1. Identify the Existing User and Their Current Permissions

First, we need to understand which tables and columns the current app user has access to:

SQL
\du+ -- List all roles with details (name, password status, etc.)
\z -- List all table permissions for a specific role or all users

-- Example: Find permissions of an existing role
SELECT * FROM information_schema.table_privileges WHERE grantee = 'current_app_user';
2. Create a New Role for Least-Privilege Access

We create a new role that will be used by the application and grant it only the necessary privileges:

SQL
CREATE ROLE least_privileged_role;
GRANT CONNECT ON DATABASE your_database TO least_privileged_role; -- Ensure database connection is allowed

-- Grant specific table access with column-level GRANTS
GRANT SELECT, INSERT ON TABLE public.your_table TO least_privileged_role;

-- Example: Restrict access to specific columns
ALTER DEFAULT PRIVILEGES FOR ROLE least_privileged_role IN SCHEMA public REVOKE ALL ON TABLES FROM public;
GRANT SELECT (column1) ON TABLE public.your_table TO least_privileged_role; -- Grant read-only access to a column

-- Optionally, grant additional permissions as needed
3. Transition Existing User to the New Role

Transfer existing privileges from the current app user to the new role:

SQL
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM current_app_user;
GRANT SELECT, INSERT ON TABLE public.your_table TO least_privileged_role; -- Example grant

-- If needed, you can also transfer ownership of tables or specific columns:
ALTER TABLE your_table OWNER TO least_privileged_role;

-- Ensure that the app is updated to use the new role
4. Implement Role-Based Access Control (RBAC) with a Migration Role

Create a migration role to handle any necessary adjustments during the transition:

SQL
CREATE ROLE migration_role;
GRANT ALL PRIVILEGES ON TABLE public.your_table TO migration_role; -- Allow for changes

-- Use this role temporarily to make any final adjustments before fully transitioning
5. Monitor and Audit Changes

Implement a robust monitoring and auditing mechanism to track who has access to what:

SQL
-- Example: Enable row-level security (RLS)
ALTER TABLE public.your_table ENABLE ROW LEVEL SECURITY;

CREATE POLICY your_policy ON public.your_table FOR SELECT TO least_privileged_role USING (TRUE);

-- Regularly review the audit logs for any suspicious activities

#### Closing Summary
and Common Mistakes

By transitioning from broad table access to role-based, column-level GRANTS, we significantly reduce the blast radius of a potential security breach. This approach ensures that if an attacker gains unauthorized access to one role, they are limited in what they can do within the database.

#### Common Mistakes

Failing to Revoke All Privileges: Not revoking all existing privileges before granting new ones.
Not Using Roles Effectively: Failing to leverage roles for multiple users or applications.
Ignoring Column-Level GRANTS: Granting permissions at the table level instead of column level, which can be overly permissive.
Why This Solution Limits Blast Radius

By restricting access through least-privilege design and using roles with specific permissions, any compromise will limit the impact to only the affected role and its associated privileges. Regular audits and monitoring further enhance security by detecting and responding to unauthorized activities promptly.

**Comparison:**

| Role | Table | Permissions |
|------|-------|-------------|
| least_privileged_role | your_table | SELECT, INSERT |
| migration_role | your_table | ALL PRIVILEGES |

**Code:**

```sql
-- Example SQL code for creating and managing roles and grants
CREATE ROLE least_privileged_role;
GRANT CONNECT ON DATABASE your_database TO least_privileged_role;

ALTER DEFAULT PRIVILEGES FOR ROLE least_privileged_role IN SCHEMA public REVOKE ALL ON TABLES FROM public;
GRANT SELECT (column1) ON TABLE public.your_table TO least_privileged_role;

REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM current_app_user;
GRANT SELECT, INSERT ON TABLE public.your_table TO least_privileged_role;

CREATE ROLE migration_role;
GRANT ALL PRIVILEGES ON TABLE public.your_table TO migration_role;

ALTER TABLE your_table ENABLE ROW LEVEL SECURITY;
CREATE POLICY your_policy ON public.your_table FOR SELECT TO least_privileged_role USING (TRUE);
```

**Example input / output:**

```text
-- Sample input and output for GRANT/REVOKE operations
-- Input:
GRANT SELECT ON TABLE public.your_table TO least_privileged_role;

-- Output:
GRANTED

-- Example of revoking all privileges before granting new ones
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM current_app_user;
```

**Data:**

```sql
-- Sample data schema for a table
CREATE TABLE your_table (
    id SERIAL PRIMARY KEY,
    column1 VARCHAR(255),
    column2 INT,
    -- other columns
);

-- Example insert into the table
INSERT INTO your_table (column1, column2) VALUES ('example', 42);
```

**Flow:**

1. Create least_privileged_role
2. Grant CONNECT ONDATABASE
3. Revoke all privileges fromcurrent_app_user
4. Grant SELECT, INSERT onyour_table toleast_privileged_role
5. Enable ROW LEVEL SECURITY
6. Create migration_role with ALLPRIVILEGES

