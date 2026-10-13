---
id: "01_Databases_and_SQL/Database_Operations/Security_and_Access/Federated_Database_Identity"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Security_and_Access"
tools: []
---

# Federated Database Identity

**L5 Expert** · [↑ Security and Access](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Federated database identity lets people and workloads log in with credentials issued by a central identity provider (SSO, short-lived tokens) instead of long-lived passwords stored in each database.

### 1. The big picture
The oldest model is *SQL authentication*: a username and password living inside each database. It is easy but scales badly: passwords are reused across dev/staging/prod, sit in config files, need rotation across fleets, and are impossible to review centrally. Federation moves the question "who is this?" to an **identity provider (IdP)** such as Okta, Azure AD/Entra ID or an LDAP/Kerberos directory. The database *trusts* the IdP's assertion or token and maps it to database roles.

Humans typically get SSO plus MFA; machines get **workload / managed identity** so no secret ever lives in code or a connection string. The direction of travel is "no long-lived passwords anywhere", with a fallback path for legacy tools that only speak passwords.

### 2. Core concepts and vocabulary
- **IdP (identity provider)** — the system that authenticates users and issues proofs of identity.
- **SSO (single sign-on)** — log in once, access many applications.
- **SAML** — XML-based assertion protocol common in enterprise SSO.
- **OpenID Connect (OIDC) / OAuth 2.0** — token-based protocols; OIDC adds identity on top of OAuth.
- **LDAP** — directory protocol for users and groups.
- **Kerberos / GSSAPI** — ticket-based authentication in a trusted domain, no password sent to the database.
- **MFA** — second factor, e.g. TOTP code refreshed every 30 seconds.
- **Managed / workload identity** — a platform-issued identity for a running service (pod, VM, function).
- **Short-lived credential** — token or password valid minutes to hours, generated on demand.
- **Mutual TLS (mTLS)** — both client and server present certificates.
- **Secrets manager / vault** — store that issues, rotates and audits credentials.
- **Break-glass account** — a tightly controlled emergency login that bypasses the IdP.

### 3. How it works, step by step
**Human login with SSO + MFA**
1. The user opens the data tool (console, JDBC client, notebook).
2. The tool redirects to the IdP; the user authenticates (password against LDAP, then MFA code).
3. The IdP returns a SAML assertion or OIDC token with identity and group claims.
4. The database platform validates the signature, expiry and audience.
5. Groups in the token map to database roles; the session gets only those rights.

**Workload login with short-lived credentials (cloud IAM style)**
1. The workload runs with an attached identity (instance role, Kubernetes service account).
2. At connect time it asks the platform's token service for a database auth token (valid ~15 minutes).
3. It connects with that token as the "password" over TLS.
4. The database verifies the token with the IAM service and applies the mapped role.
5. The app library refreshes the token before expiry; no secret is stored.

### 4. Worked example
Comparing the three classic database authentication styles:

| Method | Secret that exists | Strength | Typical use |
|---|---|---|---|
| Password | string in DB and config | weak; reuse, leakage | small apps, legacy tools |
| Client certificate | private key on client | strong; revocable by CA | service-to-service, mTLS |
| Kerberos/GSSAPI | tickets from the KDC | no password on the wire; central policy | domain-joined corporate users |

Rotating a static service password with no downtime (when you cannot avoid one) uses *two valid credentials at once*:

```sql
-- 1. create a second login, same rights
CREATE USER app_svc_b WITH PASSWORD '<new from vault>';
GRANT app_rw TO app_svc_b;
-- 2. roll the application fleet (rolling update) to use app_svc_b
-- 3. verify no sessions remain for app_svc_a
SELECT usename, count(*) FROM pg_stat_activity GROUP BY 1;
-- 4. then retire the old one
DROP USER app_svc_a;
```
The principle is alternating credentials (A/B): the old one stays valid until the fleet is fully moved, so a rolling deploy never meets a rejected password. With IAM auth the same thing becomes unnecessary because the token is fetched fresh.

### 5. Under the hood
- **Token validation** is local: the database caches the IdP's public keys (JWKS) and verifies signatures, issuer, audience and expiry, so an IdP outage does not break *already issued* tokens but blocks new logins. For global systems keep IdP/token services multi-region and cache keys, so login latency does not cross oceans.
- **Kerberos** needs a service principal and a keytab for the database; a wrong keytab or clock skew causes failures that look like network problems.
- **Group-to-role mapping** is the real authorization step; if it is only done at login, group changes at the IdP take effect only at the next token/session.
- **Connection pools** hold sessions authenticated with an *old* token; ensure the pool reconnects before expiry and that max connection lifetime is shorter than token lifetime.
- **mTLS rotation** works by overlapping trust: add the new CA to the trust store, issue new certs, roll, then remove the old CA. A fleet-wide outage happens when trust is swapped in one step.

### 6. Performance and design implications
- Token-based login adds one call per new connection; pooling hides this cost.
- Centralised identity gives one place to disable a leaver; a static password may live on in forgotten config.
- Design for IdP unavailability: break-glass account, cached keys, documented procedure.
- Audit becomes meaningful because database sessions map to real people or named workloads.
- Tools that cannot do token auth need a bridge: a vault dynamic secrets engine that mints a short-lived password, or a proxy that terminates federated auth.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pg_hba.conf` (order matters) | picks auth method per host/db/user | PostgreSQL; first matching line wins |
| `ssl=on`, `sslmode=verify-full` | enforce TLS and certificate verification | all connections |
| `CREATE USER ... FROM EXTERNAL PROVIDER` | Azure AD principal in Azure SQL | SSO for SQL |
| `rds-db:connect` IAM policy / auth token | short-lived DB token on AWS | workload identity |
| Vault database secrets engine | issues expiring DB users | legacy clients needing passwords |
| `ALTER USER ... VALID UNTIL` | expiry on password | enforcing rotation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| SSO + MFA for humans | break-glass is absent and the IdP is a single point of failure |
| Workload identity / IAM tokens | tool or driver cannot refresh tokens |
| Certificates / mTLS | no automated issuance and rotation |
| Kerberos | hosts are not domain-joined or clock sync is unreliable |
| Static password with A/B rotation | you can avoid it; it is the weakest option |

### 9. Common mistakes
- Same password across dev, staging and prod → one leak opens all → separate credentials per environment, preferably federated.
- Rotating by overwriting the secret and restarting everything at once → outage → A/B credentials and rolling update.
- Putting secrets in container images or repos → leakage → secrets manager or workload identity.
- Not validating tokens thoroughly (audience, expiry) → token reuse → use vendor libraries.
- Forgetting to remove old credentials after migration → permanent back door → inventory and disable.
- Migrating without watching legacy dependencies → surprise failures → run both modes in parallel, measure, then cut over.
- One IdP region for a global database → login latency → regional endpoints and key caching.

### 10. 🎯 Interview kit
**30-second answer:** Humans authenticate through the IdP with SSO and MFA; machines use workload identity to fetch short-lived tokens; database roles are mapped from IdP groups, and static passwords are the exception, rotated with A/B credentials.

**Structure a longer answer:** clarify (humans vs machines, cloud/on-prem, legacy tools) → mechanism (IdP → token → role mapping) → trade-offs (IdP dependency, tool compatibility) → migration plan.

**Likely follow-ups:**
- *Multi-year migration from static passwords?* → inventory, shadow/parallel auth, migrate by risk tier, enforce no-new-passwords, decommission and verify via logs.
- *Zero-downtime migration at millions of QPS?* → dual credentials, canary pods, metrics, rollback path.
- *Password vs certificate vs Kerberos?* → table in section 4.
- *What if the IdP is down?* → cached keys, existing tokens, break-glass.

**Signals interviewers look for:** no long-lived secrets, rotation without outage, break-glass thinking, mapping to roles and audit.

### 11. Check yourself
- [ ] Draw the SAML/OIDC login flow to a database.
- [ ] Explain why token lifetime and pool lifetime must be aligned.
- [ ] Describe A/B password rotation.
- [ ] State two failure modes of Kerberos authentication.
- [ ] Outline a staged plan to remove static passwords from hundreds of databases.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Explain the difference between authentication via passwords, certificates, and Kerberos/GSSAPI for a database and when each is appropriate.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21433_Explain_the_difference_between_authentication_via_passwords_.md>)
- [Deep] [How do you rotate a database service account password with zero downtime for a fleet of application pods?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21383_How_do_you_rotate_a_database_service_account_password_with_z.md>)
- [Deep] [Explain how you'd design an application so it uses short-lived IAM-based database credentials instead of static passwords.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21390_Explain_how_you_d_design_an_application_so_it_uses_short-liv.md>)
- [Deep] [Explain how you'd standardize mutual-TLS and certificate lifecycle for database connections across services when cert rotation must never cause a fleet-wide outage.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21490_Explain_how_you_d_standardize_mutual-TLS_and_certificate_lif.md>)
- [Deep] [Describe how you'd evaluate and roll out passwordless / IAM-only database authentication org-wide, including the fallback plan for tools that only speak passwords.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21516_Describe_how_you_d_evaluate_and_roll_out_passwordless_IAM-on.md>)
- [Expert] [Walk me through a multi-year strategy to move an organization from static database passwords to fully federated identity with short-lived credentials across hundreds of databases and thousands of service accounts.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21451_Walk_me_through_a_multi-year_strategy_to_move_an_organizatio.md>)
- [Expert] [Describe how you'd run a live, zero-downtime migration from application-managed credentials to workload-identity-based authentication for a fleet already serving millions of QPS.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21472_Describe_how_you_d_run_a_live_zero-downtime_migration_from_a.md>)
- [Expert] [Describe how you'd design authentication for a globally distributed database when your identity provider is in one region and you cannot accept added login latency worldwide.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21484_Describe_how_you_d_design_authentication_for_a_globally_dist.md>)

## Related concepts

- [Identity and Access Foundations](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/_index.md>) *(Cloud Fundamentals)*
- [Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>) *(Secrets Management)*
- [Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>) *(Azure Platform)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Permission Management › Password-based authentication
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Permission Management › Token-based authentication
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Permission Management › Authentication implementation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Authentication and Identity Management
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Windows Authentication
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › External Authentication Providers
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Kerberos, SPNs, and Delegation (The "Double-Hop")
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Definition: System for Cross-Domain Identity Management
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Credential passthrough
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Identity Management › Single Sign-On
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Manage Database Credentials

