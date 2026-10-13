---
id: "01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Security_and_Access"
tools: []
---

# Column-level Access

**L4 Advanced** · [↑ Security and Access](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Column-level access means a user can see or change only the columns they need, and you get there by giving permissions to roles (not people) and granting the least that works.

### 1. The big picture
A database stores many kinds of data in one table: names next to salaries, order totals next to card numbers. Table-level permission is too coarse: "can read `Employees`" would expose everything. Column-level access narrows that to "can read `Employees.Name` and `Employees.Department`, but not `Salary`".

This sits inside the wider topic of authorization (what an authenticated user may do). It builds on the **principle of least privilege (POLP)**: give every user, job and application only the minimum rights needed, for only as long as needed. The neighbouring topics are database access control and auditing (who did what), encryption (making stolen data unreadable) and federated identity (who the user really is).

### 2. Core concepts and vocabulary
- **Authentication** — proving who you are (password, certificate, token).
- **Authorization** — deciding what an authenticated principal may do.
- **Principal** — anything that can be granted rights: a user, a role, an application login.
- **Securable** — the thing being protected: database, schema, table, column, procedure.
- **Role** — a named bundle of permissions; users are made members of roles.
- **GRANT / DENY / REVOKE** — give a permission, explicitly forbid it (wins over GRANT in SQL Server), or remove a previous GRANT/DENY.
- **Column-level GRANT** — a permission list attached to named columns of a table.
- **View** — a stored query; you can give access to the view but not the base table.
- **Dynamic data masking / masking policy** — the data is returned, but obscured (`aXXX@XXXX.com`) for unprivileged users.
- **Row-level security (RLS)** — a predicate filters which rows a user sees (the row-wise sibling of column control).
- **Ownership chaining** — if a view/procedure and the table it reads have the same owner, SQL Server skips the permission check on the table.

### 3. How it works, step by step
There are four common ways to hide columns. Which to use depends on how strict the guarantee must be.

1. **Column-level GRANT.** `GRANT SELECT ON dbo.Employees (Name, Department) TO HRReader;` The engine checks, at compile time, that every column referenced (including in `WHERE`, `JOIN`, `SELECT *`) is allowed. `SELECT *` fails for such a user, which surprises people.
2. **View.** Create a view with only the safe columns, grant `SELECT` on the view, grant nothing on the table. Thanks to ownership chaining the user never needs rights on the base table.
3. **Stored procedure.** Expose only `EXECUTE` on a procedure that returns the right columns and accepts validated parameters. The user can't run arbitrary queries at all.
4. **Masking.** Everyone with `SELECT` can run the query, but a masking function rewrites the output unless the user holds an `UNMASK`-type permission. This is a presentation control, not a hard barrier.

Around all four sits the role design:

1. Create roles by job function (`ReportReader`, `AppWriter`, `Admin`).
2. Grant permissions to the roles only.
3. Add users/groups to roles.
4. Review membership periodically and remove unused permissions.

### 4. Worked example
```sql
-- Setup: a table with a sensitive column
CREATE TABLE dbo.Employees (
    EmployeeId  int PRIMARY KEY,
    Name        nvarchar(100) NOT NULL,
    Department  nvarchar(50)  NOT NULL,
    Salary      decimal(10,2) NOT NULL
);

CREATE ROLE HRReport;
GRANT SELECT ON dbo.Employees (EmployeeId, Name, Department) TO HRReport;

CREATE USER analyst WITHOUT LOGIN;
ALTER ROLE HRReport ADD MEMBER analyst;

-- Test as that user
EXECUTE AS USER = 'analyst';
SELECT Name, Department FROM dbo.Employees;   -- works
SELECT Salary FROM dbo.Employees;             -- error: SELECT permission denied on column 'Salary'
SELECT * FROM dbo.Employees;                  -- error too (includes Salary)
REVERT;
```
What to notice: the permission failure is raised on the *column*, so error text itself names the object. Always test with `EXECUTE AS` (and test the negative cases), because a missing denial is silent.

The view variant is more future-proof: when someone adds a `Bonus` column later, a column GRANT list does not include it, and neither does the view, so new columns are hidden by default.

### 5. Under the hood   *(optional at this level)*
Permission checks run in the engine's permission-resolution step, after name binding and before execution. SQL Server resolves a user's effective permission as the union of grants across the user and all roles, with any DENY overriding. Column-level permissions are stored in the permissions catalog (`sys.database_permissions` with `minor_id` = column id). Because a table-level `GRANT SELECT` implies all columns, a stray table-level grant to `PUBLIC` or a broad built-in role (such as `db_datareader`) silently makes the column restriction pointless. Always check effective permissions with `sys.fn_my_permissions` or `HAS_PERMS_BY_NAME`.

### 6. Performance and design implications
- Column permissions are metadata checks; the runtime cost is negligible compared with I/O.
- Views and procedures add no real overhead, but too many layers of views make plans hard to read.
- Masking happens at result time and adds slight CPU; it does not stop inference attacks (filtering on a masked column like `WHERE Salary > 100000` still leaks information).
- Design for **default deny**: new columns and new tables start invisible.
- Put a reusable `ExecSP`-style role in the `model` database so each new database begins with a role that has just `EXECUTE`; avoid overloading `model`, as everything there is copied to every new database.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GRANT SELECT ON t (c1,c2) TO role` | column-level read grant | strict column allow-list |
| `DENY SELECT ON t (c) TO role` | explicit block, overrides grants | exceptions to a broad role |
| `REVOKE ...` | removes a prior grant/deny | cleanup |
| `CREATE ROLE` / `ALTER ROLE .. ADD MEMBER` | role management | always, instead of per-user grants |
| `EXECUTE AS USER` / `REVERT` | impersonate to test | verifying permissions |
| `sys.fn_my_permissions`, `sys.database_permissions` | inspect effective/explicit rights | audits, troubleshooting |
| `ADD MASKED WITH (FUNCTION=...)` | dynamic data masking | reduce casual exposure |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Column GRANT: few users, simple allow-list | many tables or frequent column changes (hard to maintain; `SELECT *` breaks) |
| View: stable shared projection for a team | users can reach the base table by another path |
| Procedure: you need parameter validation and no ad-hoc SQL | reporting tools that need to browse tables |
| Masking: support staff should see partial values | you need a hard guarantee against a determined insider |
| RLS: restriction depends on *who* and *which row* | you only need to hide a column |

### 9. Common mistakes
- Granting `db_datawriter`/`db_owner` because it is quick → users can alter far more than required → create a custom role with only the needed `INSERT`/`UPDATE`.
- Granting rights to individual users → impossible to review → grant to roles, add people to roles.
- Leaving `SELECT` on `PUBLIC` → every principal inherits it → revoke it and find the exposure window in audit logs.
- Application connects as the schema owner or `sa` → one SQL injection owns everything → connect as a low-privilege role.
- Treating masking as security → inference through filters/joins → pair with column grants or encryption for truly sensitive data.
- Never reviewing → privilege creep as people change teams → schedule access reviews and revoke unused rights.

### 10. 🎯 Interview kit
**30-second answer:** I apply least privilege through roles. For column restriction I prefer a view or a column-level GRANT on a role, test it with impersonation, and add masking only as a convenience layer, not a boundary.

**Structure a longer answer:** clarify (who needs which columns, how strict) → core mechanism (roles + column grants/views/procedures) → trade-offs (maintenance, masking leakage) → close with a scenario (analyst can see 40 of 60 columns).

**Likely follow-ups:**
- *View vs column GRANT vs masking?* → view/grant are enforcement, masking is obfuscation and can be inferred.
- *How do roles interact with object ownership if the owner is dropped?* → objects become orphaned or the drop fails; transfer ownership first.
- *How do you stop a connection string leak from being catastrophic?* → separate low-privilege logins per service, short-lived credentials.
- *WITH GRANT OPTION risk?* → recipients can re-grant, so rights spread beyond review.

**Signals interviewers look for:** default-deny thinking, roles over users, tests for negative cases, awareness that `SELECT *` and `PUBLIC` break column rules.

### 11. Check yourself
- [ ] Explain why a table-level grant defeats a column-level restriction.
- [ ] Write a role that lets an analyst read three of five columns, and test it with `EXECUTE AS`.
- [ ] Compare view, column GRANT, procedure and masking in one table.
- [ ] Describe what happens to a new column added later under each approach.
- [ ] Explain how you would find who currently has access to a sensitive column.
<!-- explainer:end -->

## Go deeper

- [TDE vs Application Encryption](<TDE_vs_Application_Encryption.md>)
- [Federated Database Identity](<Federated_Database_Identity.md>)
- [Database Access Control and Auditing](<Database_Access_Control_and_Auditing.md>)

## Practice questions

- [Intermediate] [What is the difference between authentication and authorization at the database layer and where does a connection pooler complicate that boundary?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21378_What_is_the_difference_between_authentication_and_authorizat.md>)
- [Intermediate] [Explain the difference between GRANT on a schema and GRANT on all tables in a schema, and how default privileges close that gap.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21400_Explain_the_difference_between_GRANT_on_a_schema_and_GRANT_o.md>)
- [Intermediate] [Explain how you'd grant an application the ability to read a sequence and insert rows but never truncate or alter the table.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21441_Explain_how_you_d_grant_an_application_the_ability_to_read_a.md>)
- [Deep] [How do database roles interact with object ownership when the owning role is later dropped, and how do you avoid orphaned objects?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21392_How_do_database_roles_interact_with_object_ownership_when_th.md>)
- [Deep] [What are the security implications of allowing users to create their own indexes or functions in a shared schema?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21412_What_are_the_security_implications_of_allowing_users_to_crea.md>)
- [Deep] [Walk me through hardening a fresh PostgreSQL install: which default privileges, roles, and settings do you change first and why?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21427_Walk_me_through_hardening_a_fresh_PostgreSQL_install_which_d.md>)
- [Expert] [How would you implement row-level security in PostgreSQL for a multi-tenant table and what happens to your query plans once RLS policies are attached?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21371_How_would_you_implement_row-level_security_in_PostgreSQL_for.md>)
- [Expert] [Explain how you'd secure cross-database or federated queries (database links / foreign data wrappers) so credentials aren't exposed to callers.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21410_Explain_how_you_d_secure_cross-database_or_federated_queries.md>)

## Related concepts

- [Privileges and Roles](<../../../SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>) *(SQL Command Types)*
- [Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>) *(PostgreSQL Platform)*
- [Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Security Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Principle of Least Privilege
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Ownership Chaining
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Schema Binding for Security
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Security
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Security › 14 Security
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Security › 14.1 #94 Not implementing the principle of least privilege
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Security › 14.2 #95 Not disabling the sa account
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Security › 14.7 #100 Exposing the business to SQL injection attacks
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Consequences
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *Building Medallion Architectures* — Piethein Strengholt — Medallion Governance and Security › Data Security and Access Management

