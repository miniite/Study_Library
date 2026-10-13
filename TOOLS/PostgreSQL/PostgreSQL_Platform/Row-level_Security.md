---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Row-level Security

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Row-level security (RLS) lets PostgreSQL attach a predicate to a table so every query automatically sees — and can only change — the rows the current role is allowed to, moving tenant/ownership filtering out of application `WHERE` clauses and into the database where it cannot be forgotten.

### 1. The big picture
Classic access control in SQL is table- and column-level: `GRANT SELECT ON orders TO app_user`. That is all-or-nothing per table. Real applications need *"this user may see only their own orders"* or *"this tenant may see only rows with their tenant_id"*. The usual solution is to append `WHERE tenant_id = ?` in application code — and it fails the moment one query forgets the filter, an ORM bypasses it, or a reporting tool connects directly.

RLS puts the filter in the database. Once enabled on a table, PostgreSQL silently adds the policy predicate to every `SELECT`, `UPDATE` and `DELETE` and checks it against every new or modified row on `INSERT`/`UPDATE`. The same idea exists in other engines: SQL Server calls it Row-Level Security with predicate functions and security policies (the book describes a table-valued predicate function bound with `CREATE SECURITY POLICY … ADD FILTER PREDICATE`), and Redshift offers RLS policies with attached roles. The book's benefits list applies everywhere: centralized security logic at the data layer, less reliance on application code, and consistent enforcement across queries, reports and APIs.

This lesson sits under the security topic with roles/`GRANT`, least privilege and multi-tenancy design.

### 2. Core concepts and vocabulary
- **Role** — PostgreSQL's single concept for users and groups; can own objects, be granted privileges and be members of other roles.
- **`GRANT` / `REVOKE`** — table/column/schema-level privileges; RLS is *additional* to these, not a replacement.
- **Policy** — a named rule on a table: `CREATE POLICY name ON table [FOR cmd] [TO role] USING (…) [WITH CHECK (…)]`.
- **`USING` expression** — decides which *existing* rows are visible/targetable (read, update-from, delete).
- **`WITH CHECK` expression** — validates *new or modified* rows (insert, update-to); if omitted, `USING` is reused.
- **Permissive vs. restrictive policy** — permissive policies are OR-ed together; restrictive ones are AND-ed on top.
- **Table owner / superuser / `BYPASSRLS`** — exempt from RLS by default.
- **`FORCE ROW LEVEL SECURITY`** — makes the table owner subject to policies as well.
- **Session context** — a value your app sets per connection/transaction (e.g. `SET LOCAL app.tenant_id = '42'`) that policies read through `current_setting()`.
- **`LEAKPROOF`** — function marker meaning it cannot reveal its arguments via errors; matters for planner optimization.

### 3. How it works, step by step
1. Enable RLS: `ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;`. With no policy defined, a non-owner sees **no rows** (default deny).
2. Create one or more policies naming the command (`SELECT`, `INSERT`, `UPDATE`, `DELETE` or `ALL`) and the roles.
3. At parse/rewrite time, for each table with RLS, PostgreSQL combines applicable policies into one security qual: `(perm1 OR perm2 …) AND restr1 AND restr2`.
4. That qual is added to the query's `WHERE` as a **security barrier**: it is evaluated before any user-supplied function or operator that might leak data.
5. For writes, the `WITH CHECK` expression is evaluated against the resulting row; failure raises `new row violates row-level security policy`.
6. Rows failing `USING` are invisible: `UPDATE`/`DELETE` just affect zero of them, no error.

| Command | `USING` applies to | `WITH CHECK` applies to |
|---|---|---|
| `SELECT` | rows returned | — |
| `INSERT` | — | new row |
| `UPDATE` | rows that may be updated | new version of row |
| `DELETE` | rows that may be deleted | — |
| `ALL` | read/target side | write side (falls back to `USING`) |

### 4. Worked example
```sql
-- Multi-tenant table
CREATE TABLE invoices (
  id         bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  tenant_id  int  NOT NULL,
  owner_id   int  NOT NULL,
  amount     numeric(12,2) NOT NULL
);
CREATE INDEX ON invoices (tenant_id);       -- policy columns need an index

-- App connects as a low-privilege role, NOT the table owner
CREATE ROLE app_user LOGIN;
GRANT SELECT, INSERT, UPDATE, DELETE ON invoices TO app_user;

ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
ALTER TABLE invoices FORCE  ROW LEVEL SECURITY;   -- owner also bound

CREATE POLICY tenant_isolation ON invoices
  FOR ALL TO app_user
  USING      (tenant_id = current_setting('app.tenant_id')::int)
  WITH CHECK (tenant_id = current_setting('app.tenant_id')::int);

-- Per request (inside a transaction so a pooler cannot leak it):
BEGIN;
SET LOCAL app.tenant_id = '42';
SELECT * FROM invoices;                       -- only tenant 42
INSERT INTO invoices (tenant_id, owner_id, amount) VALUES (7, 1, 10);
-- ERROR: new row violates row-level security policy for table "invoices"
COMMIT;
```
Check what is happening: `EXPLAIN SELECT * FROM invoices;` shows `Filter: (tenant_id = current_setting('app.tenant_id'::text)::integer)` and, with the index, an index scan on `tenant_id`. The user did not write that predicate — the rewriter added it.

For comparison, the SQL Server flow from the book is: (1) create an inline table-valued predicate function with `WITH SCHEMABINDING`, (2) bind it with `CREATE SECURITY POLICY … ADD FILTER PREDICATE … ON dbo.Sales`, (3) test as different users. PostgreSQL's policy folds both steps into one `CREATE POLICY`; SQL Server separates *filter* predicates (hide rows) from *block* predicates (reject writes), which map to `USING` and `WITH CHECK`.

### 5. Under the hood
- **Security barrier quals:** policy predicates are marked so the planner will not push a user function *below* them. Without this, `WHERE leaky_fn(secret_col)` could run on rows the policy should hide and leak values via error messages or `RAISE NOTICE`. The cost: some optimizations (pushing down non-`LEAKPROOF` quals, certain join reorders) are limited, which can change plans. Mark your own pure, non-erroring helper functions `LEAKPROOF` only after you are sure (needs superuser).
- **Evaluation is per row.** A policy containing a subquery (`USING (owner_id IN (SELECT …))`) is re-evaluated like any correlated subquery; planners turn it into a semi-join when they can, but complex policies hurt. Prefer a cheap column comparison, ideally against an indexed column.
- **Foreign keys and unique constraints bypass RLS.** To maintain integrity, referential checks and unique checks run with the privileges of the table owner, so an FK check or a duplicate-key error can reveal that a hidden row exists. This is a known covert-channel; it is the cost of keeping constraints correct.
- **Views run as the view owner by default.** A view over an RLS table evaluates RLS as the *owner* unless created `WITH (security_invoker = true)` (PG 15+). Otherwise the view can bypass the viewer's restrictions.
- **`pg_dump`** sets `row_security = off` by default and errors if a policy would hide rows, so backups are not silently partial; replicas inherit policies since they are catalog data.
- **Cost of `current_setting`:** it is `STABLE`, evaluated once per scan node rather than per row when used as a constant comparison, so it is cheap and index-friendly.

### 6. Performance and design implications
- The policy predicate becomes part of every query: put an index on the policy column(s) — usually the leading column of composite indexes (`(tenant_id, created_at)`).
- Avoid joins/subqueries in policies; denormalize `tenant_id` onto child tables instead of joining to a parent to find the owner.
- Connection pooling: session-level `SET` leaks across clients in transaction pooling. Use `SET LOCAL` inside the transaction, or a pooler feature that resets per transaction.
- Policies don't replace column-level access; for hiding columns use column `GRANT`s or views.
- Test as the real application role — superusers and table owners skip RLS and give a false sense of security.
- Cost is visible in `EXPLAIN (ANALYZE, BUFFERS)`: compare plans with `SET row_security = off` (as a privileged role) vs. on to see the overhead.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE … ENABLE ROW LEVEL SECURITY` | Turn RLS on (default deny for non-owners) | Start of RLS rollout |
| `ALTER TABLE … FORCE ROW LEVEL SECURITY` | Apply policies to owner too | Apps connecting as owner |
| `CREATE POLICY … USING … WITH CHECK` | Define visibility and write checks | Per table/command/role |
| `CREATE POLICY … AS RESTRICTIVE` | AND-ed extra rule | Tenant filter that others can't relax |
| `ALTER ROLE … BYPASSRLS` | Exempt a role | Admin/ETL roles — use sparingly |
| `SET LOCAL app.tenant_id = …` / `current_setting()` | Pass identity into policies | Pooled connections |
| `CREATE VIEW … WITH (security_invoker = true)` | Views honour caller's RLS | Any view over RLS tables |
| `SET row_security = off/on` | Error instead of filtering (for dumps) | Backup tooling |
| `pg_policies` | Catalog view of policies | Audit and review |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Shared-schema multi-tenancy where a missed `WHERE` is a breach | Few tenants with huge data — schema- or database-per-tenant gives stronger isolation and easier per-tenant backup |
| Many access paths (BI, APIs, ad-hoc SQL) must be consistent | Policies need complex joins — slow and hard to reason about |
| Compliance requires defense in depth | You rely on a single superuser/owner connection — RLS is bypassed |
| Predicates are simple and indexable | You need to hide the *existence* of data completely (FK/unique errors leak) |

### 9. Common mistakes
- Testing as superuser or owner → "RLS works" illusion, then production leaks → test with the app role and `FORCE` if needed.
- Enabling RLS with no policy → app suddenly sees zero rows → create policies in the same migration.
- Using session-level `SET` with transaction pooling → one tenant's id served to another client → `SET LOCAL` in a transaction.
- Forgetting `WITH CHECK` on `UPDATE` → a user can move a row into another tenant → define it (or rely on `USING` fallback knowingly).
- Policy subquery on an unindexed table → every query slows down → index and simplify.
- Views owned by a privileged role → RLS bypassed → `security_invoker = true`.

### 10. 🎯 Interview kit
**30-second answer:** Row-level security attaches policies to a table so PostgreSQL appends a predicate to every query for the current role: `USING` filters visible rows and `WITH CHECK` validates writes. It enforces tenant or ownership isolation centrally instead of trusting each `WHERE` clause, but table owners and superusers bypass it unless forced, and policies must be indexed and simple.

**Structure a longer answer:** the problem (forgotten filters) → mechanism (enable, policy, security barrier) → how identity reaches the policy (`current_setting`, role) → performance and leak caveats → alternatives (schema-per-tenant, views).

**Likely follow-ups:**
- *What happens with RLS enabled and no policies?* → default deny for non-owners.
- *Difference between `USING` and `WITH CHECK`?* → existing rows vs. new/changed rows.
- *Who bypasses RLS?* → superuser, `BYPASSRLS`, owner unless `FORCE`.
- *How do you pass the tenant to the policy with a pooler?* → `SET LOCAL` inside a transaction.
- *How is it different in SQL Server?* → predicate function + security policy with filter/block predicates.

**Signals interviewers look for:** you know the bypass rules, you mention pooling and indexes, and you can contrast RLS with schema-per-tenant and column-level grants.

### 11. Check yourself
- [ ] Write a tenant-isolation policy with both `USING` and `WITH CHECK`.
- [ ] Explain what a non-owner sees right after `ENABLE ROW LEVEL SECURITY`.
- [ ] List three kinds of roles/objects that bypass or weaken RLS.
- [ ] Explain why policy predicates act as security barriers and what that costs the planner.
- [ ] Describe how you would make RLS safe behind PgBouncer in transaction mode.
- [ ] Map SQL Server's filter/block predicates to PostgreSQL's clauses.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you create a new database and a new user, then grant that user access in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35826_How_do_you_create_a_new_database_and_a_new_user_then_grant_t.md>)
- [Intermediate] [Your app runs on RDS and you can't be superuser; which VACUUM, replication, and extension operations are restricted, and how do you work around not having filesystem or superuser access for tuning?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36268_Your_app_runs_on_RDS_and_you_can_t_be_superuser_which_VACUUM.md>)
- [Intermediate] [Your app grants broad table access to one DB user; how would you move to least-privilege with roles, column-level GRANTs, and a separate migration role, and why does that limit blast radius on a compromise?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36335_Your_app_grants_broad_table_access_to_one_DB_user_how_would_.md>)
- [Intermediate] [How do roles and GRANT/REVOKE work for managing permissions in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36396_How_do_roles_and_GRANT_REVOKE_work_for_managing_permissions_.md>)
- [Deep] [Your multi-tenant app uses Row-Level Security and one tenant sees another's rows after adding a connection pooler; explain the SET vs SET LOCAL session-variable pitfall with pooling and how to set the tenant context per transaction safely.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36331_Your_multi-tenant_app_uses_Row-Level_Security_and_one_tenant.md>)
- [Deep] [You enabled RLS but the table owner and BYPASSRLS roles still see everything; explain FORCE ROW LEVEL SECURITY and how you'd verify the policy actually applies to your app's role.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36332_You_enabled_RLS_but_the_table_owner_and_BYPASSRLS_roles_stil.md>)
- [Deep] [How would you enforce a maximum of N active sessions per user at the database level to prevent runaway concurrency, and what are the pitfalls of enforcing this with a trigger versus in the app?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35806_How_would_you_enforce_a_maximum_of_N_active_sessions_per_use.md>)
- [Deep] [You need per-tenant data isolation and choose schema-per-tenant; at what tenant count does this break down operationally, and what specifically gets painful (migrations, connections, catalog bloat)?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35817_You_need_per-tenant_data_isolation_and_choose_schema-per-ten.md>)

## Related concepts

- [Column-level Access](<../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/_index.md>) *(Security and Access)*
- [Privileges and Roles](<../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Privileges_and_Roles.md>) *(SQL Command Types)*
- [Idempotency Key Tables](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>) *(SQL Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Introduction to Row-Level Security
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Security Predicates in RLS
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Implementing RLS in Practice
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Real-World Scenarios for RLS
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Row-level security
- *Building Data Integration Solutions* — Jay Borthen — Creating a PostgreSQL Database in the Rocky Linux AWS EC2 Instance › Create the postgres user and add to the super users group
- *Building Data Integration Solutions* — Jay Borthen — Creating a PostgreSQL Database in the Rocky Linux AWS EC2 Instance › CREATE USER USERNAME WITH PASSWORD 'PASSWORD' CREATEROLE CREATEDB;
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Portfolio data engineering project › Step 4: Provision a PostgreSQL database server on Azure and upload your extracted data to it.

