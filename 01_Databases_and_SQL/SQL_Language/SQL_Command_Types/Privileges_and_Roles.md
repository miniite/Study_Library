---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: []
---

# Privileges and Roles

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Privileges say *who may do what to which object*; you hand them out with `GRANT`, take them back with `REVOKE` (or block them with `DENY`), and you manage them at scale by granting to **roles**, not to individual people.

### 1. The big picture
SQL is split into sub-languages by purpose. **DDL** defines structure (`CREATE`, `ALTER`, `DROP`), **DML** reads and changes rows (`SELECT`, `INSERT`, `UPDATE`, `DELETE`), **TCL** controls transactions (`COMMIT`, `ROLLBACK`) and **DCL — Data Control Language** — decides who is *allowed* to run the other three. DCL is small (`GRANT`, `REVOKE`, and in some engines `DENY`), but it is the layer that turns "anyone who can connect can do anything" into a defensible system.

Almost every real access problem is one of two shapes: "this person can't get at the data they need" (a missing grant) or "this person can see something they must not" (an over-broad grant, usually inherited through a role or through `PUBLIC`). Understanding DCL lets you diagnose both without guessing. The same ideas appear in warehouses and lakehouses: Delta Lake / Databricks governance is expressed with the same `GRANT ... ON ... TO ...` grammar, and even Cassandra, which is not relational, has `GRANT`, `REVOKE` and roles in its query language (CQL).

This lesson sits under *SQL command types*. Its neighbours are DDL (which creates the objects privileges attach to), TCL (the grant itself is transactional in some engines, e.g. PostgreSQL and SQL Server, but auto-commits in others, e.g. MySQL and Oracle) and row-level security (finer than object-level grants).

### 2. Core concepts and vocabulary
- **Principal** — whoever is being granted something: a user, a group, a role, or a service account.
- **Securable / object** — the thing being protected: database, schema, table, view, column, function, sequence, procedure.
- **Privilege (permission)** — one allowed action on one securable: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`, `CREATE`, `USAGE`, `REFERENCES`, …
- **Role** — a named bundle of privileges that can be granted to many principals, and can itself be granted to other roles (a *role hierarchy*).
- **Grantor / grantee** — who gives the privilege / who receives it. The engine remembers the grantor, which matters on revoke.
- **`WITH GRANT OPTION`** — lets the grantee pass the same privilege on to others.
- **`PUBLIC`** — a built-in pseudo-role that *everyone* belongs to. Anything granted to `PUBLIC` is visible to every account.
- **`DENY`** (SQL Server, Databricks-style systems) — an explicit negative permission that overrides any grant. Plain `REVOKE` only removes a previously granted privilege; it does not block access arriving by another path.
- **Least privilege** — give each principal only what its job needs, and nothing more.
- **Ownership** — the creator/owner of an object normally has every privilege on it and can grant them; ownership is itself a privilege path to audit.

### 3. How it works, step by step
The ANSI-style grammar, as the Delta Lake guide summarizes it, is:

```sql
GRANT  <privilege> [(column_list)]
  ON   [object_type] <object>
  TO   <user_or_role> [, <user_or_role> ...];
```

1. **Create the container for permissions.** Create a role (`CREATE ROLE reporting_ro;`), not a bunch of one-off user grants.
2. **Grant privileges to the role.** `GRANT SELECT ON sales.report_daily TO reporting_ro;`
3. **Put people (or services) into the role.** PostgreSQL: `GRANT reporting_ro TO alice;` SQL Server: `ALTER ROLE reporting_ro ADD MEMBER alice;`
4. **The engine evaluates every statement** by collecting the principal's direct grants + grants through all its roles + `PUBLIC`, applying any `DENY`, then allowing or refusing. A statement that fails the check errors out *before* it touches data.
5. **Offboarding or time-boxing** is done with `REVOKE`: *"Sarah got poached by a big tech company"* is the Fundamentals of Data Engineering example — you remove her read access to `data_science_db` the day she leaves.
6. **Verification** reads the system catalog (see §7), not your memory of what you ran.

Note a syntax pitfall: `GRANT ... TO principal` but `REVOKE ... FROM principal`. (Some book listings write `REVOKE ... TO`; that is a typo — engines expect `FROM`.)

What each side of the grant looks like in practice:

| Level | Examples of what you can grant |
|---|---|
| System / server | `CREATE DATABASE`, `CREATE ROLE`, `CONNECT`, `VIEW SERVER STATE` |
| Database / schema | `CREATE TABLE`, `USAGE` on a schema, `CREATE` on a schema |
| Object | `SELECT/INSERT/UPDATE/DELETE` on a table, `EXECUTE` on a procedure |
| Column | `SELECT (name, email) ON customers` |

Cassandra's list is a compact illustration of the same taxonomy: `CREATE/ALTER/DROP`, `SELECT`, `MODIFY` (which *groups* `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`), `AUTHORIZE` (the right to grant/revoke), `DESCRIBE`, `EXECUTE`. Note `MODIFY` is a bundle: the Databricks exam question "what does `GRANT MODIFY ON TABLE inventory TO supply_team` allow?" has the answer *add, delete and modify data* — all three.

### 4. Worked example
Scenario: a reporting service must read three views and nothing else; developers need read/write on staging but read-only on prod, using the same role names in both environments.

```sql
-- PostgreSQL flavour
CREATE ROLE reporting_ro NOLOGIN;
GRANT USAGE ON SCHEMA reporting TO reporting_ro;                 -- needed to even see objects in the schema
GRANT SELECT ON reporting.report_sales,
                reporting.report_inventory,
                reporting.report_employee TO reporting_ro;       -- exactly three views

CREATE ROLE svc_reporting LOGIN PASSWORD '***';
GRANT reporting_ro TO svc_reporting;

-- Same role *names* per environment, different privileges per database
-- in staging:
CREATE ROLE app_dev NOLOGIN;
GRANT USAGE, CREATE ON SCHEMA app TO app_dev;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO app_dev;
-- in prod:
GRANT USAGE ON SCHEMA app TO app_dev;
GRANT SELECT ON ALL TABLES IN SCHEMA app TO app_dev;

-- Make future tables follow the same rule
ALTER DEFAULT PRIVILEGES IN SCHEMA app GRANT SELECT ON TABLES TO app_dev;
```

Check it:

```sql
-- what can svc_reporting do on the tables?
SELECT grantee, table_schema, table_name, privilege_type
FROM information_schema.role_table_grants
WHERE grantee IN ('reporting_ro','svc_reporting');

-- role membership
SELECT * FROM pg_auth_members;           -- PostgreSQL
-- SQL Server: sys.database_permissions + sys.database_role_members
```

Walkthrough: `svc_reporting` has no privileges of its own; it inherits `reporting_ro`. A `SELECT` on `reporting.report_sales` passes; a `SELECT` on `app.orders` fails with *permission denied* because nothing grants it. Offboarding is one statement (`REVOKE reporting_ro FROM svc_reporting`), not a hunt through three views.

### 5. Under the hood   *(optional at this level)*
- Privileges are stored as rows in the system catalog (an **ACL** — access control list — per object in PostgreSQL's `pg_class.relacl`; `sys.database_permissions` in SQL Server). The check at execution time is a catalog lookup, cached per session.
- **Role inheritance** can be on or off: PostgreSQL roles have `INHERIT`/`NOINHERIT`; with `NOINHERIT` the member must `SET ROLE` to use the role's privileges. SQL Server database roles always inherit.
- **Ownership chaining** (SQL Server): if a view and the table it reads have the same owner, SQL Server skips permission checks on the underlying table and only checks the view. That is how "grant `SELECT` on the view only" works as a security boundary — and why *different* owners silently break it.
- **Cascade**: if A granted to B `WITH GRANT OPTION` and B re-granted to C, revoking from B requires `CASCADE` (or `REVOKE GRANT OPTION FOR`) to also remove C's grant; without it the engine refuses or leaves C intact depending on the product.

### 6. Performance and design implications
- Permission checks are cheap; the cost is *human*: hundreds of per-user grants are impossible to audit. Design with roles.
- Use three layers of roles: **access roles** (privileges on objects: `sales_read`, `sales_write`), **functional roles** (job functions that bundle access roles: `analyst`, `etl_service`), and **user/service accounts** that are only ever members.
- Prefer granting on **schemas/future objects** (default privileges, schema-level grants in warehouses) so a new table does not need a new grant ticket.
- Security that depends on views (column subsets, filtered rows) is cheap and works with every engine; row-level security policies cost per-query predicate evaluation.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GRANT p ON obj TO r` | Gives privilege to a principal | Normal access provisioning |
| `REVOKE p ON obj FROM r` | Removes a previously granted privilege | Offboarding, time-boxed access |
| `DENY p ON obj TO r` | Explicit block that wins over grants | SQL Server: carve exceptions out of a broad role |
| `WITH GRANT OPTION` | Grantee may re-grant | Rare; delegated admin only |
| `REVOKE GRANT OPTION FOR` | Removes only the re-grant ability | Tidy delegated admins |
| `CREATE ROLE` / `ALTER ROLE ... ADD MEMBER` | Role management | Always |
| `ALTER DEFAULT PRIVILEGES` (PostgreSQL) | Grants applied to future objects | Schemas where devs keep adding tables |
| `information_schema.*_privileges`, `has_table_privilege()`, `HAS_PERMS_BY_NAME()` | Inspect effective rights | Audits, "why can't X read Y?" |
| `SHOW GRANTS` (MySQL, Databricks) / `\dp` (psql) | List grants | Day-to-day checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Roles for everything | Granting directly to users "just this once" — it never gets cleaned up |
| `DENY` for a narrow exception on a big role | Using `DENY` widely — it overrides everything and is hard to reason about |
| Column-level `GRANT SELECT (cols)` for a few hidden columns | Many columns/tables — a view or masking is easier to maintain |
| `WITH GRANT OPTION` for a data owner in a delegated model | Handing it to ordinary users: privilege sprawl and cascade surprises |
| `GRANT ALL` for throwaway sandboxes | Production: `ALL` includes privileges you did not think about (DDL, REFERENCES, TRIGGER) |
| Object grants plus row-level security | Using RLS alone: a missing object grant still blocks everything; use both together |

### 9. Common mistakes
- Revoking from a user and expecting access to disappear → access still comes through a role or `PUBLIC` → inspect effective permissions and revoke at the source (or `DENY`).
- Forgetting `USAGE` on the schema/catalog → "table not found / permission denied" even though `SELECT` is granted → grant the container too.
- Granting `ALL PRIVILEGES` to a service account → it can drop tables → grant only the DML it uses; reserve DDL for migration tooling.
- Leaving default `PUBLIC` privileges (e.g. `CREATE` on the `public` schema in older PostgreSQL) → any account can create objects → `REVOKE CREATE ON SCHEMA public FROM PUBLIC`.
- Revoking a role before checking membership in production → breaks downstream jobs → list `members of role` first, revoke in staging, then in prod.
- Assuming grants are per-session → they are persistent catalog state → script emergency access with a *revoke in a `finally`* path or a time-bound role.

### 10. 🎯 Interview kit
**30-second answer:** "`GRANT` gives a principal a privilege on an object, `REVOKE` removes it, and some engines also have `DENY` which overrides grants. At scale I grant to roles and put users into roles, follow least privilege, and use `PUBLIC` and default privileges deliberately because they are where surprise access comes from."

**Structure a longer answer:** clarify the engine → privilege model (grant/role/inheritance) → how I'd implement it (roles, schemas, views, RLS) → trade-offs (flexibility vs auditability) → a real scenario such as the three-view reporting service above.

**Likely follow-ups:**
- *`REVOKE` didn't take effect — why?* → access arrives via another role, `PUBLIC`, ownership, or a cascaded grant; check effective permissions.
- *`REVOKE` vs `DENY`?* → revoke removes a grant; deny is a standing block that wins.
- *Why roles over per-user grants?* → one change, many users; auditability; cleaner offboarding.
- *Column-level security?* → column grants, or a view that omits columns, or dynamic data masking.
- *Grant versus row-level security?* → grants decide *which objects*; RLS decides *which rows*; you usually need both.
- *Lock down DDL to migration tooling only?* → revoke `CREATE/ALTER/DROP` from everyone else, give it to one deploy role, audit with DDL triggers/event logs.
- *Temporary elevated access?* → grant a time-limited role, log it, revoke automatically; never leave it to memory.

**Signals interviewers look for:** you think in roles and defaults, you know `PUBLIC` and cascades are the usual traps, you can describe how to *audit* effective rights, and you separate object-level from row-level controls.

### 11. Check yourself
- [ ] Write the statements to give a reporting service read access to three views and nothing else.
- [ ] Explain why a revoked user can still read a table, and list three places the access could come from.
- [ ] Contrast `REVOKE` and `DENY` with an example where only `DENY` works.
- [ ] Describe what `MODIFY` (Databricks/Cassandra) bundles, and what `ALTER DEFAULT PRIVILEGES` is for.
- [ ] Query the catalog to list all privileges held by one account on one table.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between GRANT and REVOKE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22881_What_is_the_difference_between_GRANT_and_REVOKE.md>)
- [Intermediate] [What is the WITH GRANT OPTION clause used for in a GRANT statement?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22979_What_is_the_WITH_GRANT_OPTION_clause_used_for_in_a_GRANT_sta.md>)
- [Intermediate] [How can you grant a set of privileges to many users at once using roles?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22982_How_can_you_grant_a_set_of_privileges_to_many_users_at_once_.md>)
- [Intermediate] [How can you verify what privileges a user currently has on a table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23042_How_can_you_verify_what_privileges_a_user_currently_has_on_a.md>)
- [Intermediate] [Explain how you would grant least-privilege access for a reporting service that must only SELECT from three specific views.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22861_Explain_how_you_would_grant_least-privilege_access_for_a_rep.md>)
- [Intermediate] [How would you grant a developer read/write on staging but strictly read-only on prod using the same role model across environments?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22893_How_would_you_grant_a_developer_read_write_on_staging_but_st.md>)
- [Deep] [What is the difference between object-level and system-level privileges in GRANT?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23021_What_is_the_difference_between_object-level_and_system-level.md>)
- [Deep] [When you REVOKE a privilege that was granted WITH GRANT OPTION and re-granted by the grantee, what happens to the downstream grants?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22877_When_you_REVOKE_a_privilege_that_was_granted_WITH_GRANT_OPTI.md>)

## Related concepts

- [Column-level Access](<../../Database_Operations/Security_and_Access/Column-level_Access/_index.md>) *(Security and Access)*
- [Row-level Security](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>) *(PostgreSQL Platform)*
- [Database Access Control and Auditing](<../../Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>) *(Security and Access)*

## References

- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Foundations of Lakehouse Governance and Security › Governing Data Access with SQL Grants
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Data control language
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — cassandra@cqlsh GRANT MODIFY ON KEYSPACE reservation TO reservationservice; This allows your
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › Granting Permissions
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › Object privileges
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › SELECT privilege
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › Granting privileges by role
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Schema-Based Permissions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Fine-Grained Permissions
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Access Controls and Permissions Model › Privileges Versus Permissions
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Access Controls and Permissions Model › Permissions Model

