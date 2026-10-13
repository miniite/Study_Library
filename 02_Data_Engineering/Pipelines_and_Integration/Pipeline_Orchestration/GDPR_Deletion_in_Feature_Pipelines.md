---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Airflow", "Apache_Spark"]
---

# GDPR Deletion in Feature Pipelines

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A GDPR/DPDP erasure request must reach every copy of a person's data — raw, cleaned, aggregates, feature tables, training snapshots and models — so you design pipelines with a deletion path, lineage to find copies, and an auditable proof it completed within the legal window.

### 1. The big picture
Privacy laws such as GDPR (EU) and India's DPDP Act give people the right to have their personal data erased. In an analytics platform, one person's data is not in one place: it flows from source systems into a raw layer, is cleaned into silver tables, aggregated into gold marts, turned into features, frozen into training snapshots and baked into models. Deleting it from the source database does nothing for these downstream copies.

The engineering problem is hard for structural reasons. Lakes favor immutable, append-only files (so deletion means rewriting files). Aggregates mix many people (so you may need to recompute, not delete). Backups, logs and exports hold copies. And you must be able to *prove* the deletion, not merely perform it. The solution is a designed-in deletion mechanism, not a heroic one-off script per request.

### 2. Core concepts and vocabulary
- **Right to erasure** — the individual's right to have personal data deleted (subject to legal exceptions).
- **PII / personal data** — information relating to an identifiable person.
- **Soft delete** — mark a record deleted (flag/tombstone) so consumers hide it; data still physically exists.
- **Hard delete** — physically remove the data, including from storage files and old versions.
- **Tombstone event** — a deletion message propagated through CDC or a stream.
- **Lineage** — the map of which datasets derive from which; tells you where copies live.
- **Medallion layers** — bronze (raw), silver (cleaned), gold (business-ready).
- **Pseudonymization / tokenization** — replacing identifiers with tokens so a single mapping deletion de-identifies many tables.
- **Retention policy / legal hold** — rules for how long data is kept and exceptions that block deletion.
- **Crypto-shredding** — encrypting per-subject data and deleting the key to render all copies unreadable.

### 3. How it works, step by step
1. **Receive and verify the request**; resolve the person to all identifiers (user id, email hash, device ids) via an identity map.
2. **Create a deletion record** in a central "erasure ledger" (request id, subject keys, received date, deadline).
3. **Propagate to sources**: delete or anonymize in the operational systems; emit tombstones through CDC.
4. **Delete in the lake/warehouse** for each layer using lineage:
   - Raw/bronze: remove records (rewrite affected files) or apply crypto-shredding if immutable storage is required.
   - Silver: delete by key with `DELETE`/`MERGE`, including soft-delete handling for CDC/SCD2 history.
   - Gold/marts: recompute affected aggregates or remove person-level rows; aggregates with k-anonymity thresholds may not need recompute.
5. **Features and training sets**: delete from feature tables, regenerate training snapshots that included the person, and decide whether affected models must be retrained or are considered sufficiently anonymized.
6. **Clean up physical copies**: vacuum old file versions and expire snapshots so time-travel does not retain the data; address caches, extracts, logs and backups per policy.
7. **Verify and attest**: run queries confirming zero matches, record counts and timestamps in the ledger, and produce evidence for auditors.
8. **Monitor SLAs**: alert when requests approach the legal deadline.

### 4. Worked example
```sql
-- 1. Ledger entry (central control table)
INSERT INTO governance.erasure_requests
VALUES ('req-8841', 'user_id', 1042, CURRENT_TIMESTAMP, 'PENDING');

-- 2. Delete from silver (Delta)
DELETE FROM silver.events WHERE user_id = 1042;

-- 3. Delete from feature tables
DELETE FROM features.customer_30d WHERE customer_id = 1042;

-- 4. Physically remove old versions after retention window
VACUUM silver.events RETAIN 168 HOURS;
VACUUM features.customer_30d RETAIN 168 HOURS;

-- 5. Verify
SELECT COUNT(*) FROM silver.events WHERE user_id = 1042;      -- expect 0
```
Reading it: `DELETE` in a table format writes new files without the rows and marks old files removed in the log; the data still exists in old file versions until `VACUUM` (and snapshot expiry in Iceberg). The final count query is the evidence; the ledger row is updated to `COMPLETE` with the timestamp. In a CDC-fed SCD2 dimension you must also remove *history rows* for the person and handle out-of-order or duplicate delete events so you don't resurrect them.

### 5. Under the hood   *(optional at L4)*
- **Append-only vs deletion:** bronze is often kept immutable for replay. Options: (a) rewrite files on request (costly but simple), (b) keep PII only in a small separate table keyed by token so bronze holds no direct identifiers, (c) crypto-shredding by subject key.
- **Why time travel is a risk:** old snapshots contain the "deleted" data. Erasure must run after or together with snapshot expiry; retention windows must fit the legal deadline (typically one month).
- **Aggregates:** if a gold metric is a sum over many users, deleting one user's source rows changes it; either recompute affected partitions or document that aggregated, non-identifiable data is out of scope.
- **ML artifacts:** feature stores and training snapshots hold person-level rows; models may encode information. Policies range from retraining on schedule to demonstrating anonymization; legal guidance applies.
- **Soft vs hard delete in marts:** BI should never show soft-deleted PII, so filters/views must exclude tombstoned rows until physical removal completes.

### 6. Performance and design implications
- Per-request deletes rewrite many files; batch requests (daily/weekly) and use partitioning/clustering by subject token to limit rewrite scope.
- Keep an inventory of PII columns and datasets (catalog tags) so propagation is automatic.
- Design identifiers so tokens, not raw PII, flow to downstream layers.
- Make delete jobs idempotent and observable: they will be re-run.
- Guardrails: pipelines that filter or drop records must be explicit and auditable, so deletion is distinguishable from accidental data loss that biases features.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DELETE FROM … WHERE` / `MERGE … WHEN MATCHED THEN DELETE` | Remove rows by key | Silver/gold/feature tables |
| `VACUUM` (Delta) / `expire_snapshots`, `remove_orphan_files` (Iceberg) | Physically purge old versions | After deletion, per retention |
| Catalog PII tags / column masking | Find and protect personal data | Inventory and access control |
| Tombstone events in CDC | Propagate deletes | Streaming/CDC pipelines |
| Erasure ledger table | Track requests and proof | Audit and SLAs |
| Per-subject encryption keys | Crypto-shredding | Immutable stores, backups |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Hard delete + vacuum — strict compliance, modest volume | Rewrite costs are extreme and tokenization could avoid PII spread |
| Tokenization — many tables reference the person | Mapping table itself becomes unprotected |
| Crypto-shredding — immutable logs/backups | Key management complexity; keys must be per-subject |
| Soft delete + filtered views — interim hiding | Treated as final deletion; data persists physically |
| Recompute aggregates — small identifiable groups | Large k-anonymous aggregates where recompute adds no privacy value |

### 9. Common mistakes
- Deleting from the source only → copies persist downstream → use lineage and a propagation workflow.
- Forgetting time travel/snapshots → data recoverable → vacuum/expire within the deadline.
- Missing feature stores, extracts and training sets → hidden copies → inventory all derived assets.
- Resurrecting data via replay or late CDC events → deleted rows reappear → apply tombstones idempotently and check ordering.
- No evidence → cannot demonstrate compliance → ledger with verification queries.

### 10. 🎯 Interview kit
**30-second answer:** Erasure has to reach every derived copy: raw, silver, gold, features, training snapshots, and models. I keep a ledger of requests, use lineage and PII tags to find tables, delete by key or recompute aggregates, vacuum old versions so time travel doesn't retain data, and record verification as proof — and I design with tokenization or crypto-shredding to make this cheap.

**Structure a longer answer:** clarify legal scope → inventory/lineage → deletion mechanics per layer → physical purge → verification/audit → architectural choices to reduce cost.

**Likely follow-ups:**
- *How keep bronze immutable?* → separate PII with tokens, rewrite on request, or crypto-shred.
- *Soft vs hard delete?* → soft hides; hard removes physically; compliance needs hard.
- *What about trained models?* → retrain on schedule or justify anonymization; policy decision.
- *How handle out-of-order CDC deletes?* → versioned tombstones and deterministic merge logic.

**Signals interviewers look for:** lineage thinking, awareness of snapshots and backups, and a design that is automated and auditable.

### 11. Check yourself
- [ ] List every place a user's data may live in a medallion lakehouse with ML features.
- [ ] Explain why `DELETE` alone does not erase data in Delta/Iceberg.
- [ ] Describe crypto-shredding and its main requirement.
- [ ] Say what evidence you would store to prove a deletion.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What is anomaly detection at a basic level and how does a clean data pipeline make it more reliable?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13422_What_is_anomaly_detection_at_a_basic_level_and_how_does_a_cl.md>)
- [Intermediate] [Why do many pipelines land raw data as-is first before cleaning it, instead of cleaning during extraction?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13609_Why_do_many_pipelines_land_raw_data_as-is_first_before_clean.md>)
- [Intermediate] [How would you keep secrets like API keys out of your DAG code and git repository?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13637_How_would_you_keep_secrets_like_API_keys_out_of_your_DAG_cod.md>)
- [Deep] [What is the difference between raw, cleaned, and aggregated layers (bronze silver gold) in a telemetry pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13442_What_is_the_difference_between_raw_cleaned_and_aggregated_la.md>)
- [Deep] [Why can using overwrite mode accidentally delete more data than you expect?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13587_Why_can_using_overwrite_mode_accidentally_delete_more_data_t.md>)
- [Deep] [How do you handle a GDPR-style deletion request that must propagate into already-computed enriched-alert tables and correlation training snapshots?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13514_How_do_you_handle_a_GDPR-style_deletion_request_that_must_pr.md>)
- [Deep] [How do you design a bronze-silver-gold medallion architecture and what specific responsibilities and quality guarantees does each layer carry?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13687_How_do_you_design_a_bronze-silver-gold_medallion_architectur.md>)
- [Expert] [How would you keep the bronze layer append-only and immutable while still supporting GDPR deletions that must reach the gold aggregates?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13688_How_would_you_keep_the_bronze_layer_append-only_and_immutabl.md>)

## Related concepts

- [Idempotent Incremental Loads](<../Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>) *(Idempotent Incremental Loads)*
- [GDPR Deletion in Immutable Lakes](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>) *(Data Architecture)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Transformation and Enrichment Patterns › 8. Data Transformation and Enrichment Patterns
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Silver Layer
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Implementation of a Metadata-Driven Approach
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Implementation of Data Cleansing Tasks
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Final Recommendations
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › PII data
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Overwriting
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Immutable Dataset
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Problem
- *Data Engineering with Medallion Architecture* — Miki Eto — Evolution of Data Architecture › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — MLOps for AI Model Deployment and Monitoring › Process or steps
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Transitioning to the silver layer
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Integrating AWS Glue sensitive data detection
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6 › Chapter 9

