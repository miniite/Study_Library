---
id: "TOOLS/SQL_Server/08_Operations/Security"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Security

**L4 Advanced** · [↑ Operations and Administration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server security authenticates principals, then authorizes them through a hierarchy of securables and permissions granted via roles (least privilege), adds row-level security and masking for fine-grained exposure, and audits the result.

### 1. The big picture
Security answers three questions in order:
1. **Authentication** — who are you? (Windows/AD identity, SQL login, Azure AD/Entra, contained user.)
2. **Authorization** — what may you do? (`GRANT`, `DENY`, `REVOKE`, roles, schemas.)
3. **Auditing / protection** — what did you do, and how is data protected at rest, in transit and in view? (SQL Server Audit, Extended Events, TDE, Always Encrypted, Row-Level Security, Dynamic Data Masking.)

The design rule behind all of it is the **principle of least privilege**: give each principal only the permissions it needs, through roles, reviewed regularly.

### 2. Core concepts and vocabulary
- **Principal** — an entity that can request access: logins (server level), users (database level), roles, application roles, and certificates/keys that act as principals.
- **Securable** — a resource that can be protected: server, endpoint, database, schema, object (table/view/procedure), column.
- **Permission** — an allowed action on a securable (`SELECT`, `INSERT`, `EXECUTE`, `ALTER`, `CONTROL`…).
- **`GRANT` / `REVOKE` / `DENY`** — give, remove, or explicitly forbid a permission. **`DENY` overrides `GRANT`.**
- **Fixed server roles** — predefined server-wide privilege sets (`sysadmin`, `securityadmin`, `dbcreator`…).
- **Fixed database roles** — predefined database sets (`db_owner`, `db_datareader`, `db_datawriter`, `db_ddladmin`…).
- **User-defined role** — a custom role for an application or job function.
- **Application role** — a database-level identity an application activates with a password; not tied to a login.
- **Contained user** — a database user that authenticates without a server login, easing portability.
- **Ownership chaining** — if objects in a call chain have the same owner, permissions are checked only on the first object.
- **Row-Level Security (RLS)** — a security policy applies a predicate so users see only permitted rows.
- **Dynamic Data Masking (DDM)** — obfuscates column values for users without `UNMASK`.
- **TDE / Always Encrypted** — encryption at rest (files and backups) vs. client-side column encryption where the engine never sees plaintext.
- **SQL Server Audit** — built-in, tamper-resistant logging for compliance.

### 3. How it works, step by step
**Granting access properly**
1. **Authenticate**: prefer Windows/AD (or Entra ID) identities, mapped via AD *groups*.
2. **Create the login** (server) and **user** (database) — or a contained user.
3. **Put users into roles**: AD group → login → database role. Avoid direct grants to individuals.
4. **Grant at the schema level** where possible (`GRANT SELECT ON SCHEMA::Sales TO SalesReaders`), so new objects inherit.
5. **Separate duties**: read-only, read-write and administrative (DDL) roles.
6. **Layer finer controls**: RLS for row visibility, DDM for display masking, encryption for stored/at-rest/in-use data.
7. **Audit**: enable SQL Audit (compliance) or Extended Events (diagnostics) on permission and role changes, review memberships periodically ("privilege creep").
8. **Verify** with `fn_my_permissions` and the permission catalog views.

**How a permission is resolved**
1. Collect all permissions for the user and the roles they belong to.
2. If any `DENY` applies → denied.
3. Else if any `GRANT` applies → allowed.
4. Else → denied (default).
Inheritance flows down the hierarchy (server → database → schema → object → column).

### 4. Worked example
```sql
-- 1. Least-privilege role instead of db_datawriter
CREATE ROLE SalesOrderWriter;
GRANT SELECT                   ON SCHEMA::Sales TO SalesOrderWriter;
GRANT INSERT, UPDATE           ON Sales.[Order] TO SalesOrderWriter;
GRANT EXECUTE                  ON SCHEMA::Sales TO SalesOrderWriter;  -- through procedures

CREATE USER [CONTOSO\SalesApp] FOR LOGIN [CONTOSO\SalesApp];
ALTER ROLE SalesOrderWriter ADD MEMBER [CONTOSO\SalesApp];

-- 2. Check what the user can effectively do
EXECUTE AS USER = 'CONTOSO\SalesApp';
SELECT * FROM fn_my_permissions('Sales.Order', 'OBJECT');
REVERT;

-- 3. Row-level security: users see only their own employee row
CREATE FUNCTION HR.EmployeePredicate(@EmployeeID INT)
RETURNS TABLE WITH SCHEMABINDING
AS RETURN SELECT 1 AS AccessResult WHERE @EmployeeID = USER_ID();
GO
CREATE SECURITY POLICY HR.EmployeeFilter
    ADD FILTER PREDICATE HR.EmployeePredicate(EmployeeID) ON HR.Employees
    WITH (STATE = ON);

-- 4. Audit who has what
SELECT dp1.name AS Grantee, dp2.name AS Grantor, p.permission_name, o.name AS ObjectName
FROM sys.database_permissions p
JOIN sys.database_principals dp1 ON p.grantee_principal_id = dp1.principal_id
JOIN sys.database_principals dp2 ON p.grantor_principal_id = dp2.principal_id
LEFT JOIN sys.objects o ON p.major_id = o.object_id;
```

What happens: the user can read the `Sales` schema and insert/update orders but not alter or drop anything. With the policy on, `SELECT * FROM HR.Employees` silently returns only rows where `EmployeeID` matches the caller; no application `WHERE` is required, and the same applies to reports and APIs. (The predicate above is the book's simple example; real policies often use a mapping table or `SESSION_CONTEXT`.)

### 5. Under the hood
- **Security hierarchy and catalog**: `sys.server_principals`, `sys.database_principals`, `sys.server_permissions`, `sys.database_permissions`; effective-permission functions compute across role membership and hierarchy.
- **Login vs. user**: the login is the server-level authentication identity; the user is its mapping into a database. Orphaned users appear after restoring a database to another server because the SIDs of logins don't match; contained users avoid this.
- **Ownership chaining** lets a procedure grant indirect access: the caller needs `EXECUTE` on the procedure but not `SELECT` on the tables it uses, provided the same owner. This is the foundation of "access only through procedures".
- **RLS** is implemented by the optimizer adding the predicate (an inline table-valued function) to queries on the table as a filter (hides rows) or block (prevents writes that violate it). Because it's applied inside the engine, it works for ad-hoc queries, but side-channel leaks are possible with certain error or timing patterns, and predicates should be simple to avoid hurting plans.
- **DDM** applies the mask at result-presentation time; the data are stored in clear, so it's a convenience control, not encryption, and can be circumvented by inference queries.
- **TDE** encrypts data and log files and backups at the page level (database encryption key protected by a certificate in `master`); it's transparent to applications and costs a little CPU. **Always Encrypted** encrypts columns client-side with keys the server never holds.
- **Application roles** switch the session's security context with `sp_setapprole`; the password has to live in the application, so they are discouraged in favor of AD identities or contained users.
- **Audit** writes to file, Windows event logs; Extended Events has lower overhead but isn't designed for compliance.

### 6. Performance and design implications
- **Role-based design scales**: a few roles and AD groups beat thousands of per-user grants, for administration and for plan/permission-cache efficiency.
- **Schema-level grants** keep security consistent as objects are added.
- **RLS predicates** should be inline, SARGable and index-friendly; avoid multi-statement functions or complex lookups on every row. Test plans.
- **Encryption cost**: TDE adds modest CPU and affects backup compression; Always Encrypted limits what queries can do on encrypted columns (equality only with deterministic, no range/LIKE).
- **Auditing volume**: audit only what compliance requires; log destinations need capacity and retention.
- **Operational hygiene**: reviewing permissions periodically is a process cost but prevents privilege creep.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE LOGIN / USER / ROLE` | Principals | Onboarding |
| `ALTER ROLE … ADD/DROP MEMBER` | Role membership | Grant/revoke access |
| `GRANT / DENY / REVOKE … ON SCHEMA::… TO …` | Permissions | Least-privilege design |
| `fn_my_permissions`, `HAS_PERMS_BY_NAME` | Effective permissions | Verification |
| `sys.database_permissions`, `sys.database_role_members` | Permission/role catalog | Audits |
| `CREATE SECURITY POLICY` | Row-level security | Per-user/tenant row filtering |
| `ALTER COLUMN … ADD MASKED WITH (FUNCTION = …)` | Dynamic data masking | Hide values in UI/reports |
| `CREATE DATABASE ENCRYPTION KEY`, `ALTER DATABASE … SET ENCRYPTION ON` | TDE | Encryption at rest |
| `CREATE SERVER AUDIT / DATABASE AUDIT SPECIFICATION` | Compliance auditing | Regulatory needs |
| `sp_setapprole` | Activate application role | Legacy apps only |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| AD group → role → schema grants | Direct per-user, per-object grants |
| Custom role with only needed permissions | `db_owner`/`db_datawriter` convenience grants |
| `DENY` for explicit exceptions | Overusing `DENY`: it overrides grants and surprises |
| RLS: multi-tenant or per-user data filtering | Heavy predicates; treating RLS as the only defense |
| DDM: reduce casual exposure | Relying on it to stop determined users |
| TDE: compliance for data at rest | Needing protection from DBAs: use Always Encrypted |
| SQL Server Audit: compliance | XE for compliance (not tamper-evident) |
| Contained users/Entra identities | Application roles with shared passwords |

### 9. Common mistakes
- **Using `sa`/`sysadmin` or `db_owner` for applications** → full compromise on a SQL injection → least-privilege roles.
- **Direct grants to users** → unmanageable, hard to audit → roles and AD groups.
- **Granting `db_datawriter` instead of specific INSERT/UPDATE** → write to every table → custom role.
- **Heavy `DENY`** → unexplained access failures → document and use sparingly.
- **Never reviewing membership** → privilege creep, leavers retaining access → periodic review and audit.
- **Treating DDM as encryption** → data still readable by anyone with `UNMASK` or via inference → combine with permissions/encryption.
- **Hard-coded application-role passwords** → leaks → prefer integrated authentication.
- **Not backing up the TDE certificate** → encrypted backups unrecoverable → back it up securely.

### 10. 🎯 Interview kit
**30-second answer:** Security in SQL Server is authentication (logins/users, preferably AD), authorization through roles and permissions at schema/object level with least privilege (`DENY` overrides `GRANT`), extra controls such as row-level security, masking, TDE or Always Encrypted, and auditing with SQL Server Audit. I manage access via AD groups mapped to roles and review memberships regularly.

**Structure a longer answer:** principals and securables → role-based model → least privilege and separation of duties → fine-grained (RLS, DDM) → encryption (TDE, Always Encrypted, TLS) → audit → operations (review, orphaned users, certificates).

**Likely follow-ups:**
- *Login vs. user?* → Server-level authentication vs. database-level mapping; contained users skip logins.
- *What does DENY do?* → Explicit prohibition that overrides any GRANT.
- *How does RLS work?* → Security policy with inline TVF predicates (filter/block) applied automatically by the engine.
- *DDM vs. encryption?* → Masking hides output only; encryption protects stored data.
- *TDE vs. Always Encrypted?* → TDE protects files/backups at rest; Always Encrypted keeps plaintext from the server and DBAs.
- *How do you let a user run a procedure without table access?* → Ownership chaining; grant `EXECUTE` only.

**Signals interviewers look for:** least privilege, roles/groups, awareness of `DENY` precedence, ownership chaining, knowing what each protection feature does *not* do, and audit/compliance separation.

### 11. Check yourself
- [ ] List principals, securables and permissions with an example of each.
- [ ] Explain how `GRANT`, `DENY` and role membership combine into an effective permission.
- [ ] Build a role for an application that can read all tables and insert into one.
- [ ] Write an RLS policy that restricts rows by user and describe its limits.
- [ ] Distinguish DDM, TDE and Always Encrypted by what each protects.
- [ ] Explain why orphaned users occur and two ways to avoid them.
- [ ] Describe SQL Server Audit vs. Extended Events for permission changes.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Introduction to Roles in SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Application Roles
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Auditing Role and Permission Usage
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Best Practices for Role-Based Security
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Principals, Securables, and Permissions Hierarchy
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Role-Based Security and Permissions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Authorization and Principle of Least Privilege
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Introduction to Row-Level Security
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Role Membership Management

