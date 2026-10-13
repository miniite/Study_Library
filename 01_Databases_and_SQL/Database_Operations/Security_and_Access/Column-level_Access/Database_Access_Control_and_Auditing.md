---
id: "01_Databases_and_SQL/Database_Operations/Security_and_Access/Database_Access_Control_and_Auditing"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Security_and_Access"
tools: []
---

# Database Access Control and Auditing

**L5 Expert** · [↑ Security and Access](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Access control decides who may touch which data; auditing records who actually did, so that you can prove it and detect abuse; both must survive connection pooling, shared service accounts and privileged staff.

### 1. The big picture
Access control and auditing are the two halves of database security operations. Control is *preventive* (RBAC, least privilege, row/column rules, masking, just-in-time access). Auditing is *detective* (a trustworthy record of logins, permission changes, and reads/writes of sensitive data). Without the first you have exposure; without the second you cannot answer an auditor's or an incident responder's question "who saw this row, and when?".

In a modern estate the hard parts are organisational: hundreds of engineers, service accounts, pipelines, CDC streams and ML jobs, all needing different slices of data, under regulations (GDPR, HIPAA, PCI-DSS, SOX). The database reliability engineer cannot own this alone; they partner with application, security and platform teams and provide self-service building blocks.

### 2. Core concepts and vocabulary
- **RBAC (role-based access control)** — permissions attach to roles; people get roles.
- **ABAC (attribute-based access control)** — decisions use attributes (department, region, data tag, time of day).
- **Least privilege** — minimum rights for the minimum time.
- **JIT (just-in-time) access** — elevated rights granted on request, approved, time-boxed, then removed.
- **Standing access** — permanent privileged rights; the goal is to drive it toward zero.
- **Privilege creep** — rights accumulating as people change roles.
- **Row-level security (RLS)** — a policy predicate filters rows by the current user or context.
- **Masking / tokenization** — hiding or replacing sensitive values (masking changes the display; tokenization replaces the value with a surrogate and keeps the real one in a vault).
- **Audit trail** — an append-only, tamper-evident log of security-relevant events.
- **Database-level (native) auditing** vs **trigger-based auditing** — engine-written event records vs your own triggers writing history rows.
- **Service account** — a non-human identity used by an application or pipeline.
- **PUBLIC** — a built-in role every principal belongs to.

### 3. How it works, step by step
**Building the control layer**
1. Classify data (e.g. Tier 1 regulated, Tier 2 internal, Tier 3 general) and decide per tier whether it is access-controlled only, masked, tokenized or encrypted. A written org standard stops per-project improvisation.
2. Design roles per function: read-only, read-write, admin, ETL, support. Applications never connect as the schema owner.
3. Authenticate people through the central identity provider (SSO + MFA) and map groups to database roles; give machines workload identities.
4. Enforce row and column restrictions *in the database* (RLS, views, column grants) rather than only in application code, so security does not silently depend on every app being correct.
5. Replace standing admin rights with JIT, approval-gated elevation.
6. Review access on a schedule; remove what is unused.

**Building the audit layer**
1. Decide what to log: logins and failures, DDL, `GRANT/REVOKE`, role changes, reads/writes on classified objects, use of privileged accounts.
2. Enable native auditing for the broad stream; use triggers or temporal/CDC history only where you need *row-level before/after values*.
3. Ship logs off-box to central, append-only/WORM storage so a compromised DBA cannot erase them.
4. Alert on patterns: off-hours bulk `SELECT`s, new grants to the same principal in a chain, logins from new hosts.
5. Retain and index/partition logs so investigation queries stay fast.

### 4. Worked example
Problem: a pooled application connects with one service user, so the audit log shows only `app_svc` for every action.

```sql
-- Pattern: carry the end user's identity into the session
EXEC sp_set_session_context @key = N'app_user', @value = N'alice@corp.com';

-- Row-level security policy for a multi-tenant table
CREATE FUNCTION sec.fn_tenant(@TenantId int)
RETURNS TABLE WITH SCHEMABINDING AS
RETURN SELECT 1 AS ok
WHERE @TenantId = CAST(SESSION_CONTEXT(N'tenant_id') AS int);

CREATE SECURITY POLICY sec.TenantPolicy
ADD FILTER PREDICATE sec.fn_tenant(TenantId) ON dbo.Orders
WITH (STATE = ON);
```
```sql
-- Server audit (SQL Server) of reads on a sensitive table
CREATE SERVER AUDIT PiiAudit TO FILE (FILEPATH = 'D:\audit\');
ALTER SERVER AUDIT PiiAudit WITH (STATE = ON);
CREATE DATABASE AUDIT SPECIFICATION PiiSpec FOR SERVER AUDIT PiiAudit
ADD (SELECT ON dbo.Customers BY public) WITH (STATE = ON);
```
Reading it: the session context (set by the pool on checkout) lets both the RLS predicate and your own audit rows record *which person*; native auditing writes who ran `SELECT` on `Customers`. Test the negative: connect as a user with a different `tenant_id` and confirm zero rows, and confirm that unprivileged users cannot read the audit files.

### 5. Under the hood
- **RLS** is implemented by the optimizer injecting the predicate into every query on the table, so it participates in plan choice; poorly written inline functions or predicates that can't use indexes slow every query. Attach statistics/indexes on the filter column (e.g. `TenantId`).
- **Native audit** hooks into the engine's event firing and writes asynchronously or synchronously (configurable); synchronous modes add latency, and a full audit target can stop or fail operations depending on the `ON_FAILURE` setting.
- **Trigger-based** auditing runs inside the writing transaction, so it adds write cost and can be bypassed by someone who can disable the trigger; it does not see reads.
- **Pooling** reuses physical connections across users, so the connection's login identity is not the human identity. Options: per-user connections (poor scale), a proxy that maps identities, session context/`SET ROLE`, or application-written audit rows.
- **Privilege chains** (`GRANT ... WITH GRANT OPTION`, role-in-role membership) mean effective rights can be larger than any single grant; detecting escalation means diffing permission snapshots and auditing grant events over time.

### 6. Performance and design implications
- Audit volume grows fast with read-auditing; audit only classified objects and privileged principals, partition and index the audit store by time.
- RLS cost depends on the predicate; test plans with and without the policy.
- Masking and RLS reduce but do not remove the need for network security (TLS) and encryption.
- Separate duties: DBAs administer the engine but should not control the audit store or the key custody.
- Common signal checks: bulk reads at unusual hours, many distinct tables in a short span, new logins/hosts, growth of privileged group membership.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE SECURITY POLICY` / `CREATE POLICY` (PostgreSQL) | row-level filters | multi-tenant tables |
| `CREATE SERVER AUDIT` + specification | native audit | compliance trails |
| `pgaudit`, `log_connections`, `log_statement` | PostgreSQL audit/logging | tracking DDL and access |
| `SET ROLE`, `sp_set_session_context` | carry or lower privilege in session | pooled apps, risky operations |
| `REVOKE ... FROM PUBLIC` | remove default exposure | hardening |
| Temporal tables / CDC | row history | "who changed this ledger row six months ago" |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Native auditing: broad, tamper-resistant, low code | you need old/new column values per row |
| Trigger auditing: row history with business context | write-heavy tables where latency matters |
| Database-enforced RLS: many apps share one DB | single app and simple filters; app logic is enough and tested |
| Proxy with query allow-lists: legacy apps, centralised policy | proxy becomes single point of failure or latency source |
| JIT access: privileged operations are rare | on-call needs instant emergency access (provide break-glass with alerts) |

### 9. Common mistakes
- One shared service user with no context identifier → audit cannot attribute actions → pass user identity in session context or use per-role accounts.
- `SELECT` granted to `PUBLIC` on a PII table → unlimited exposure → revoke, then use audit logs to bound the exposure window.
- Auditing everything → log storage and read times blow up → target objects, partition logs.
- Logs stored on the same server the DBA controls → tampering possible → forward to separate immutable storage.
- No periodic access review → privilege creep → quarterly attestations, automatic expiry.
- Giving developers production PII "by policy" only → technical separation absent → separate environments, masked clones, JIT break-glass.
- Cloning production to lower environments without masking → PII leak and broken referential integrity if masked naively → deterministic masking preserving keys.

### 10. 🎯 Interview kit
**30-second answer:** Preventive control through least-privilege roles, DB-enforced row/column rules and JIT elevation; detective control through off-box, append-only audit of logins, grants and sensitive reads, with user identity propagated through pools.

**Structure a longer answer:** clarify (regulations, threat model: insider, external, accidental) → control mechanism → audit mechanism → trade-offs (performance, operational overhead) → real scenario.

**Likely follow-ups:**
- *How do you detect slow privilege escalation via chained grants?* → audit all GRANT/role events, diff effective-permission snapshots, alert on self-granting and cascades.
- *Database auditing vs trigger auditing?* → native for access/compliance; triggers for row history; know the cost and bypass risks.
- *How do you attribute actions through a pooler?* → session context, proxy identity mapping, per-user roles.
- *How do you handle vendor access?* → time-boxed JIT accounts, session recording, automatic expiry.

**Signals interviewers look for:** separation of duties, tamper-resistant logging, awareness of pooling, negative-case testing, balance of security versus operability.

### 11. Check yourself
- [ ] Explain RBAC vs ABAC and where each fits.
- [ ] Design audit coverage for a PII table without drowning in logs.
- [ ] Show how identity travels through a connection pool.
- [ ] Describe what you would query to find when a PUBLIC grant appeared.
- [ ] Outline a JIT process with approvals, expiry and logging.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Describe a case where you used a security policy or predicate to restrict which rows a batch job could touch and how you tested the negative cases.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21387_Describe_a_case_where_you_used_a_security_policy_or_predicat.md>)
- [Deep] [How would you enforce that developers can run queries in staging but never against production PII, technically not just by policy?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21428_How_would_you_enforce_that_developers_can_run_queries_in_sta.md>)
- [Deep] [How would you design an org standard for when data must be masked, tokenized, encrypted, or simply access-controlled, so teams stop making inconsistent per-project decisions?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21497_How_would_you_design_an_org_standard_for_when_data_must_be_m.md>)
- [Deep] [Explain how you'd standardize and govern the use of application-level row filtering versus database-enforced RLS across teams, so security isn't silently dependent on app correctness.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21546_Explain_how_you_d_standardize_and_govern_the_use_of_applicat.md>)
- [Expert] [How would you detect an inside threat gradually escalating their own privileges through chained grants over months?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21432_How_would_you_detect_an_inside_threat_gradually_escalating_t.md>)
- [Expert] [How do you architect just-in-time, approval-gated privileged database access for thousands of engineers so that standing access trends toward zero across the whole estate?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21463_How_do_you_architect_just-in-time_approval-gated_privileged_.md>)
- [Expert] [How do you architect access control for change-data-capture streams so that Kafka/CDC consumers can only see the columns their downstream is entitled to?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21487_How_do_you_architect_access_control_for_change-data-capture_.md>)
- [Expert] [Explain how you'd secure feature stores and ML training pipelines so that models can't be trained on data the requesting team isn't authorized to see.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21514_Explain_how_you_d_secure_feature_stores_and_ML_training_pipe.md>)

## Related concepts

- [Governing User-owned Transformations](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Governing_User-owned_Transformations.md>) *(Access Governance)*
- [Access Governance for LLM Pipelines](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*
- [Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>) *(Access Governance)*
- [Preventing PII Leakage](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage.md>) *(Stewardship)*
- [Compliant Access to Encrypted PII](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>) *(Encryption and Compliance)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Engineering's Role › Security and privacy
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Permission Management › Objectives
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Permission Management › Regulatory compliance and auditability
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Permission Management › Multifactor authentication
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Chapter 9. Security
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Protecting from Purposeful Damage
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Protecting Data from Exposure
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Database Security as a Function
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Practical Use Cases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Auditing and Monitoring
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Best Practices for Security in Modular Code
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Real-World Case Studies
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Cautions and limits
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Concrete example
- *Data Engineering with Medallion Architecture* — Miki Eto — Medallion Architecture in Multi-cloud › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Medallion Architecture in Multi-cloud › Related topics
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Consensual, Privacy-Aware Data Collection---Brought to You by Data Engineers › Chapter 15. Consensual, Privacy-Aware Data Collection---Brought to You by Data Engineers
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Security for Data Engineers › Chapter 24. Data Security for Data Engineers
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Security for Data Engineers › Monitor, Log, and Test Access
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Security for Data Engineers › Ask for Help
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — The Modern Governance Stack › Benefits of Effective Data Governance
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog and Compute › Implementing Governance: A Two-Part Problem
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog and Compute › Dedicated or Single User Access
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Access Controls and Permissions Model › Chapter 5. Access Controls and Permissions Model

