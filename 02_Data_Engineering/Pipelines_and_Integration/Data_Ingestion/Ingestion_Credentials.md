---
id: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Ingestion_Credentials"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion"
tools: ["AWS"]
---

# Ingestion Credentials

**L4 Advanced** · [↑ Data Ingestion](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Ingestion tools hold the keys to every source system, so you authenticate with least-privilege, short-lived and rotated credentials stored in a secrets manager, and you keep sensitive columns from ever reaching the warehouse unprotected.

### 1. The big picture
An ingestion platform connects to databases, SaaS APIs and cloud storage on your behalf. It therefore holds database passwords, API keys and OAuth tokens for many systems, usually with broad read access. Compromise or mishandling of those credentials exposes every source. Alongside access, ingestion decides *what data* enters your platform; personally identifiable information (PII) that lands in a raw zone spreads quickly and is hard to claw back.

This lesson covers how connectors authenticate, how to store and rotate secrets, how to troubleshoot authentication failures, and how to control sensitive data at ingestion (exclusion, masking, hashing, auditing, residency).

### 2. Core concepts and vocabulary
- **API key** — a static secret string identifying the caller; simple, long-lived, easy to leak.
- **OAuth 2.0** — an authorization framework where a user grants an app limited access via a consent flow; the app receives a short-lived **access token** and a **refresh token**.
- **Service account / database user** — a non-human identity with specific grants.
- **Secrets manager** — a managed store (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, GCP Secret Manager) with access control, audit logs and rotation.
- **Least privilege** — grant only the permissions the task needs (read-only on the required schemas).
- **Rotation** — replacing a credential on a schedule or after exposure.
- **PII** — data that identifies a person (name, email, phone, national IDs, precise location).
- **Masking** — replacing a value with an obscured form (`j***@x.com`).
- **Hashing** — one-way transformation (SHA-256, ideally salted/keyed) that supports joins and de-duplication without revealing the value.
- **Tokenization / deterministic encryption** — replace with a token or encrypt so equal inputs give equal outputs; reversible only with a key/vault.
- **Data residency** — the requirement that data stay within a region or jurisdiction.

### 3. How it works, step by step
**Authentication paths**
1. *Databases:* host, port, user, password (or IAM database authentication / certificates), often over a private network or SSH tunnel/VPN. A dedicated read-only user is created with grants only on needed tables; for log-based CDC it also needs replication privileges.
2. *API keys / tokens:* the key is supplied in a header or query; it must be stored securely and rotated.
3. *OAuth:* the user is redirected to the provider, consents to scopes, the platform stores the refresh token and exchanges it for short-lived access tokens automatically. Revocation is possible without changing the user's password.
4. *Cloud storage:* an IAM role assumed by the platform (cross-account role with external ID) instead of static keys.

**Secure operation**
1. Create a dedicated identity per connector/environment.
2. Store secrets in a secrets manager; reference them by ID in configuration; never commit them to Git or put them in plaintext config.
3. Restrict who can read secrets; enable audit logging.
4. Rotate on a schedule using a dual-credential pattern (below) so there is no downtime.
5. Alert on authentication failures and unusual access.

**Protecting sensitive data**
1. Classify columns (PII, financial, secret) using a catalog or scan.
2. Decide per column: exclude, mask, hash/tokenize, or ingest into a restricted zone.
3. Apply the decision at the earliest enforceable point (column selection in the connector, or a transform before the warehouse).
4. Audit: record what was ingested, who accessed it, and prove masking with queries/tests.

### 4. Worked example
Rotating a PostgreSQL password for a CDC connector with no downtime:

```
1. Create a second role  app_ingest_v2  with the same grants as app_ingest_v1.
2. Store the new password in the secrets manager as version v2.
3. Update the connector to read secret version v2; restart/redeploy it.
4. Verify syncs succeed and lag is normal.
5. Revoke / drop app_ingest_v1 after a grace period.
```

Excluding and hashing PII in a Fivetran/Airbyte-style setup plus a safety net in SQL:

```sql
-- Safety net: the landed table must never expose raw email
CREATE VIEW analytics.customers AS
SELECT customer_id,
       SHA2(CONCAT(email, :pepper), 256) AS email_hash,
       country
FROM raw.customers;
-- Raw schema locked down; only the ingestion role and a few admins can read it.
REVOKE ALL ON raw.customers FROM analyst_role;
```

Troubleshooting a connector failing authentication, in order: did the secret change or expire; did the OAuth refresh token get revoked or lose scopes; did IP allow-lists or network paths change; is the user locked or lacking a grant; is the clock/time or certificate expired; does the API return 401 vs 403 (identity vs permission)?

### 5. Under the hood
- **Why OAuth for SaaS.** Passwords are never shared with the integration; tokens are scoped, short-lived and revocable by the user or admin; the provider can audit which app accessed what. Costs: refresh-token handling, token expiry/rotation, and consent flows that need a human to re-authorize when revoked.
- **Hash limitations.** Plain unsalted hashes of low-entropy values (emails, phone numbers) can be reversed by dictionary attack. Use a keyed hash (HMAC with a secret) or tokenization if re-identification risk matters. Deterministic hashing keeps joinability across tables.
- **Where masking happens matters.** If masking runs *after* the raw data lands, the raw copy exists (and may be in logs, backups, staging). Excluding the column in the connector prevents it from leaving the source. Some tools only offer hashing/blocking per column; others require post-load SQL.
- **Compliance concerns.** Regulations such as GDPR/CCPA require lawful basis, minimization, access controls, erasure on request and audit. Ingestion should carry a stable key so erasure requests can propagate (delete across raw, curated and backups with a documented process).
- **Data residency.** Choose destination region and processing region to match requirements; some managed tools process data in their own cloud regions, so check where the connector runs. Use region-specific deployments or self-hosted agents when data may not leave a jurisdiction.
- **Validation of ingested data.** Row counts, checksums and aggregate comparisons verify completeness; masking/hashing changes values, so validate hashed columns by comparing hashes computed on the source side or by comparing keys and counts.

### 6. Performance and design implications
- Secrets lookups add startup latency but not per-row cost; cache within a run.
- Column exclusion reduces volume and cost as well as risk.
- Hashing at load adds CPU; doing it in the warehouse is cheap but leaves raw data present briefly. Pick based on threat model.
- Monitor credential age, failed logins, and which identities read raw schemas.
- Plan rotation for dozens of connectors: automate through the secrets manager and infrastructure-as-code, test rotations in non-production.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE ROLE ... LOGIN; GRANT SELECT ...` | Dedicated least-privilege DB user | Every database connector |
| IAM role with external ID | Cross-account access without static keys | Cloud storage and AWS sources |
| Secrets manager rotation lambda / Vault dynamic secrets | Automated rotation / short-lived creds | Database passwords at scale |
| Connector column selection / hashing option | Exclude or hash columns at source | PII control |
| Column-level security / dynamic masking (warehouse) | Mask on read by role | Defense in depth |
| Private link / VPN / SSH tunnel | Keep traffic off public internet | Databases behind firewalls |
| Audit logs (cloud trail, warehouse access history) | Evidence of access | Compliance audits |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| OAuth with refresh tokens for SaaS | Provider only offers API keys; then restrict scope and rotate |
| Secrets manager with rotation | Secrets in env files, wikis or Git |
| Exclude PII at the connector | Business needs the field; then hash/tokenize or restrict zone |
| Keyed hash (HMAC) for joins | Need to recover original values (use tokenization with vault) |
| Region-pinned deployment | Convenience of default multi-region SaaS processing |

### 9. Common mistakes
- Storing credentials in plaintext config or repos → leak → use a secrets manager and secret scanning.
- Using a superuser/admin account for ingestion → huge blast radius → least privilege per connector.
- Never rotating → long-lived exposure → scheduled rotation with dual credentials.
- Masking only in BI tools → raw data still accessible → mask or restrict at ingestion/storage.
- Plain SHA of emails treated as anonymous → reversible → keyed hash or tokenization.
- Ignoring audit logging → cannot prove compliance → enable and review access logs.

### 10. 🎯 Interview kit
**30-second answer:** Connectors authenticate with database users, API keys or OAuth tokens. I give each connector a least-privilege identity, store secrets in a secrets manager, rotate with a dual-credential pattern, and prefer OAuth where possible. For PII I exclude or hash at ingestion, lock down the raw zone, and keep audit evidence.

**Structure a longer answer:** auth mechanisms → storage and rotation → network path → PII controls → audit and compliance → troubleshooting.

**Likely follow-ups:**
- *Why OAuth?* → scoped, short-lived, revocable, no shared passwords; trade-off is token management.
- *Zero-downtime rotation of dozens of credentials?* → dual credentials, automated via secrets manager, staggered rollout.
- *Prove all PII is masked and access-logged?* → column classification, tests on landed schemas, warehouse access history, role grants report.
- *Residency?* → region-pinned destination and processing, self-hosted agent, verify vendor processing locations.

**Signals interviewers look for:** security-first defaults, awareness that raw landing zones are risk, and ability to operationalize rotation and audit.

### 11. Check yourself
- [ ] Describe the OAuth flow and what is stored by the connector.
- [ ] Write the steps for zero-downtime password rotation.
- [ ] Explain why an unsalted hash of an email is not anonymization.
- [ ] Decide exclude vs hash vs tokenize for five example columns.
- [ ] List the first five things to check for a connector failing with 401/403.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [How do ingestion tools authenticate to sources (API keys, OAuth, database credentials)?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11685_How_do_ingestion_tools_authenticate_to_sources_API_keys_OAut.md>)
- [Basic] [What is OAuth and why do many SaaS connectors use it for authentication?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11687_What_is_OAuth_and_why_do_many_SaaS_connectors_use_it_for_aut.md>)
- [Intermediate] [How would you securely store the credentials that an ingestion tool uses to connect to a source?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11686_How_would_you_securely_store_the_credentials_that_an_ingesti.md>)
- [Intermediate] [How would you validate that the data ingested matches the source (row counts, checksums)?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11699_How_would_you_validate_that_the_data_ingested_matches_the_so.md>)
- [Intermediate] [How would you troubleshoot a connector that keeps failing with an authentication error?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11727_How_would_you_troubleshoot_a_connector_that_keeps_failing_wi.md>)
- [Deep] [How would you exclude sensitive PII columns from being ingested into the warehouse?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11720_How_would_you_exclude_sensitive_PII_columns_from_being_inges.md>)
- [Deep] [What data privacy and compliance concerns arise when ingesting customer data (GDPR, PII)?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11721_What_data_privacy_and_compliance_concerns_arise_when_ingesti.md>)
- [Deep] [How would you mask or hash sensitive fields during or after ingestion?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11722_How_would_you_mask_or_hash_sensitive_fields_during_or_after_.md>)

## Related concepts

- [Access Governance for LLM Pipelines](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Access_Governance_for_LLM_Pipelines.md>) *(Access Governance)*
- [Compliant Access to Encrypted PII](<../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>) *(Encryption and Compliance)*
- [Proving PII Residency](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>) *(Data Architecture)*

## References

- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 2Data Ingestion and Acquisition › CHAPTER 2Data Ingestion and Acquisition(#contents.xhtmlch02a)
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Preparation for Nontechnical Personas › Protecting Sensitive Data

