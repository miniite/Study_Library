---
id: "TOOLS/Snowflake/Snowflake_Platform/Row_Access_Policies"
level: 4
type: "concept"
topic: "TOOLS/Snowflake/Snowflake_Platform"
tools: ["Snowflake"]
---

# Row Access Policies

**L4 Advanced** · [↑ Snowflake Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A row access policy is a schema-level object that Snowflake evaluates on every query against a table or view, so each role or user silently sees only the rows the policy allows.

### 1. The big picture
Snowflake's access control starts with role-based access control (RBAC): privileges go to roles, roles go to users. RBAC alone answers "may this role read this table?" but not "which rows of it?". Row-level security fills that gap. The older answer was a **secure view** per audience, with a `WHERE` clause on `CURRENT_ROLE()`; it works but multiplies objects and leaks if someone grants the base table by mistake. A **row access policy (RAP)** moves the filter onto the table itself, so every path to the data (direct query, views, joins, shares) goes through the same rule.

Row-level security is one half of a pair. Its sibling, **dynamic data masking**, hides *values* in a column; a RAP hides *rows*. Data sharing (cross-account, no copy) turns serving data into exactly this kind of access-control problem, which is why policies matter for sharing as much as for internal teams.

### 2. Core concepts and vocabulary
- **Role / RBAC** — a named bundle of privileges granted to users or other roles; the unit of access in Snowflake.
- **System roles** — `ACCOUNTADMIN` (top; avoid for daily work), `SECURITYADMIN` (manages grants), `USERADMIN` (users and roles), `SYSADMIN` (creates warehouses and databases), `PUBLIC` (granted to everyone automatically).
- **Row access policy** — object with a *signature* (input columns and types) and a body returning `BOOLEAN`; TRUE means the row is visible.
- **Attach** — `ALTER TABLE … ADD ROW ACCESS POLICY p ON (col, …)` binds table columns to the policy arguments.
- **Mapping table** — an ordinary table (role → allowed region) the policy body looks up, so changing access is a data change rather than a DDL change.
- **Secure view** — a view whose definition is hidden and whose optimizer cannot push user predicates in ways that reveal filtered data.
- **Reader account** — an account the provider creates and pays for so a partner without Snowflake can query shared data.

### 3. How it works, step by step
1. A policy admin creates the policy (privilege `CREATE ROW ACCESS POLICY` on the schema).
2. The policy is attached to a table or view (privilege `APPLY ROW ACCESS POLICY`). One RAP per table or view.
3. When any query touches that object, Snowflake injects the policy logic as a filter, binding the listed column values per row.
4. The policy body can call context functions: `CURRENT_ROLE()`, `IS_ROLE_IN_SESSION('X')` (respects role hierarchy), `CURRENT_USER()`, `CURRENT_ACCOUNT()` (use this for shared data, because consumers run in other accounts).
5. Rows for which the body is FALSE are not returned *and* cannot be updated or deleted through that session; aggregates, joins and `COUNT(*)` all see only the visible rows.

### 4. Worked example
```sql
-- mapping table: which role may see which region
CREATE TABLE governance.region_map (role_name STRING, region STRING);
INSERT INTO governance.region_map VALUES
  ('SALES_EMEA','EMEA'), ('SALES_APAC','APAC'), ('ANALYST_GLOBAL','*');

CREATE OR REPLACE ROW ACCESS POLICY governance.region_rap
AS (row_region STRING) RETURNS BOOLEAN ->
  IS_ROLE_IN_SESSION('ACCOUNTADMIN')
  OR EXISTS (
       SELECT 1 FROM governance.region_map m
       WHERE IS_ROLE_IN_SESSION(m.role_name)
         AND (m.region = row_region OR m.region = '*')
     );

ALTER TABLE sales.orders ADD ROW ACCESS POLICY governance.region_rap ON (region);

-- test as each role
USE ROLE SALES_EMEA;  SELECT region, COUNT(*) FROM sales.orders GROUP BY region;  -- only EMEA
USE ROLE SALES_APAC;  SELECT region, COUNT(*) FROM sales.orders GROUP BY region;  -- only APAC
```
A role with no mapping row gets zero rows (default deny), which is the behavior you want.

**Testing for leaks** systematically: (a) per role, compare `COUNT(*)` against the expected slice, and check the per-role counts plus the global role's count reconcile with the unfiltered total; (b) try writes (`UPDATE`/`DELETE`) as a regional role and confirm out-of-scope rows are untouched; (c) query through every dependent view, a `JOIN`, a `CLONE`, and a share; (d) test an unmapped role and `NULL` region values (NULL compares unknown, so decide explicitly what NULL regions mean); (e) read `ACCESS_HISTORY` and `POLICY_REFERENCES` in `ACCOUNT_USAGE` to confirm the policy is attached where you think it is.

### 5. Under the hood
The policy is evaluated at query compile/execution time per row, not materialized, so results always reflect the current mapping table. Because the body can contain a subquery on the mapping table, keep that table small and avoid exotic joins inside the body, as it runs for every query. Policies are objects in a schema, so they are covered by ordinary ownership and grants: whoever owns the policy controls the rule, which is why governance is usually a dedicated schema owned by a security role, separate from table owners. Time Travel and cloning still honor the policy.

### 6. Performance and design implications
- A simple `col = value` style condition prunes micro-partitions; a lookup subquery can still be cheap if the mapping table is tiny. Measure with Query Profile before/after.
- Prefer one policy shared by many tables over per-table copies; changing it once updates all.
- Use role-hierarchy-aware `IS_ROLE_IN_SESSION` instead of equality on `CURRENT_ROLE()` so inherited roles work.
- Keep the filter column (e.g. `region`) clean and `NOT NULL`; otherwise rows silently disappear for everyone.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE ROW ACCESS POLICY` | defines the row filter | once per rule |
| `ALTER TABLE … ADD / DROP ROW ACCESS POLICY` | attach or detach | roll-out; swap to change policy (drop and add in one statement) |
| `CREATE MASKING POLICY` | hides column values | PII columns, alongside RAPs |
| `INFORMATION_SCHEMA.POLICY_REFERENCES` | lists where policies are attached | audit coverage |
| `ACCOUNT_USAGE.ACCESS_HISTORY` | who read which columns | audit trail |
| `CREATE SHARE` / reader account | share without copy / for non-Snowflake partners | external serving |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| RAP: many audiences, one table, rules driven by data | few static audiences where a secure view is simpler |
| Share to an existing Snowflake account (cheap, no extra account to run) | partner has no account (use a reader account: you pay their compute and administer it) |
| Mapping table for rules | rule needs heavy logic (keep the policy body small) |
| Separate dev/stage/prod by databases, roles and warehouses in an account (or separate accounts under an organization) | trusting naming alone; grant no cross-environment roles |

### 9. Common mistakes
- Granting `SELECT` on the base table to a broad role and relying on views → bypass → attach the RAP to the table, not just the view.
- Testing only as the admin → policy looks fine → always test as each real role.
- Forgetting `CURRENT_ACCOUNT()` for shares → consumers see nothing or everything → design the share path explicitly.
- Hard-coding roles in the policy body → redeploy per change → use a mapping table.
- Treating encryption (at rest and in transit via TLS) as access control → it protects against theft of storage, not against authorized-but-over-privileged users.

### 10. 🎯 Interview kit
**30-second answer:** A row access policy is a boolean-returning object attached to a table; Snowflake applies it to every query, using the session's role to decide which rows are visible. I drive it with a mapping table so access is data, and I test it per role including writes and joins.

**Structure a longer answer:** clarify (which audiences, what identifies them) → mechanism (policy + attach + context functions) → trade-offs (vs secure views, performance) → scenario (regional sales teams).

**Likely follow-ups:**
- *RAP vs secure view?* → RAP protects the table itself and every path to it; secure views are per-audience objects.
- *How do you stop partner access leaking via the share?* → policy keyed on `CURRENT_ACCOUNT()`, secure objects only.
- *How would you audit?* → `ACCESS_HISTORY`, `POLICY_REFERENCES`.
- *Reader account vs data share?* → share needs the partner's own account and costs you nothing in compute; reader account works for non-customers but you pay and administer it.

**Signals interviewers look for:** default-deny thinking, testing discipline, knowing roles vs users, separation of policy ownership.

### 11. Check yourself
- [ ] Write a RAP that lets a regional role see its own rows and a global role see all.
- [ ] Explain why an unmapped role sees zero rows, and what a NULL region does.
- [ ] List four places you must test for leakage.
- [ ] Contrast a reader account with a data share.
<!-- explainer:end -->

## Implementations (tools)

- [Snowflake](<../_index.md>)

## Practice questions

- [Basic] [What are the default system-defined roles such as ACCOUNTADMIN, SYSADMIN, and PUBLIC?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39476_What_are_the_default_system-defined_roles_such_as_ACCOUNTADM.md>)
- [Intermediate] [What is a role in Snowflake and how does role-based access control (RBAC) work?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39475_What_is_a_role_in_Snowflake_and_how_does_role-based_access_c.md>)
- [Intermediate] [What is a secure view in Snowflake and why might you use one?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39484_What_is_a_secure_view_in_Snowflake_and_why_might_you_use_one.md>)
- [Intermediate] [What is Snowflake Data Sharing and how does it let you share data without copying it?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39531_What_is_Snowflake_Data_Sharing_and_how_does_it_let_you_share.md>)
- [Intermediate] [Describe how you would implement row-access policies so each regional team sees only its own rows, and how you'd test it doesn't leak.](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39583_Describe_how_you_would_implement_row-access_policies_so_each.md>)
- [Deep] [What is dynamic data masking in Snowflake and how does it protect sensitive columns?](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39536_What_is_dynamic_data_masking_in_Snowflake_and_how_does_it_pr.md>)
- [Deep] [Explain how you'd expose a curated dataset to an external partner via a Reader Account or a Data Share, and the trade-offs of each.](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39581_Explain_how_you_d_expose_a_curated_dataset_to_an_external_pa.md>)
- [Deep] [Explain how you'd implement environment isolation (dev/stage/prod) in a single Snowflake account without cross-environment risk.](<../../../PRACTICE/TOOLS/Snowflake/Snowflake_Platform/Q-39613_Explain_how_you_d_implement_environment_isolation_dev_stage_.md>)

## Related concepts

- [Access Governance for LLM Pipelines](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*
- [Cross-cloud Data Contracts](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>) *(Multi Cloud Data)*
- [Cross-border Access Governance](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>) *(Access Governance)*

## References

- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Reviewing the most asked interview questions
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Loading Data › Configuring a Snowflake Warehouse as a Destination
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Data Sharing

