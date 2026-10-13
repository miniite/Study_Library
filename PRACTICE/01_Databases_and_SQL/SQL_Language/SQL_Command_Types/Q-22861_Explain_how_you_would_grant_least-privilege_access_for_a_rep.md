---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would grant least-privilege access for a reporting service that must only SELECT from three specific views.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Privileges and Roles](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the reporting service has "least-privilege" access, we need to restrict its permissions to the bare minimum necessary for it to function correctly. Specifically, this means granting SELECT permissions on only three specific views while revoking any other privileges.

#### Triage
and Assumptions

The key is to:

Identify the required views: Ensure these are clearly defined.
Determine the database management system (DBMS): The solution will vary depending on whether it's MySQL, PostgreSQL, SQL Server, etc.

#### Cons
ider concurrent access: Ensure the reporting service does not impact other operations.
Step-by-Step Solution
Identify the Required Views: Let’s assume the views are report_sales, report_inventory, and report_employee.
Grant SELECT Permissions:
In SQL, we can use the GRANT statement to accomplish this.
Revoke Unnecessary Privileges: To ensure security, revoke any other permissions that might be accidentally granted.

Here’s how you would approach it:

SQL
-- Step 1: Revoke unnecessary privileges (optional but recommended)
REVOKE ALL ON report_sales, report_inventory, report_employee FROM reporting_service;

-- Step 2: Grant SELECT permission on specific views
GRANT SELECT ON report_sales TO reporting_service;
GRANT SELECT ON report_inventory TO reporting_service;
GRANT SELECT ON report_employee TO reporting_service;

#### Closing Summary
and Common Mistakes
Summary: By carefully defining the required views and using GRANT statements, we ensure that the reporting service only has access to select data from three specific views.
Mistakes:
Granting unnecessary privileges (e.g., INSERT, UPDATE, DELETE).
Not explicitly revoking permissions first can lead to unexpected behavior if other users have overly broad permissions.

