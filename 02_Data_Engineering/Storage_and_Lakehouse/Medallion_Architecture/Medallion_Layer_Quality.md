---
id: "02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality"
level: 5
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture"
tools: ["AWS"]
---

# Medallion Layer Quality

**L5 Expert** · [↑ Medallion Architecture](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Medallion quality means each layer has a clear job, a contract at its boundary and a measurable bar to pass — raw and immutable in Bronze, validated and conformed in Silver, business-ready in Gold — so quality improves progressively without breaking consumers.

### 1. The big picture
The medallion pattern (Bronze → Silver → Gold) is not a new architecture but a design pattern for organising data in a lakehouse. The names are friendly labels like the layering in warehouses and lakes: progression from Bronze to Gold signals improved quality, structure and validation. The books point out a practical problem: there is no firm consensus about what each layer is *for*, so teams apply the labels differently and quality becomes inconsistent. The solution is to define each layer's purpose, entry and exit criteria explicitly.

Layer separation lets teams isolate stages of processing, improve quality incrementally without disturbing downstream consumers, and introduce new Gold tables safely by testing with a small group first. Quality gates between layers — a data-quality test must pass before data is promoted to Silver — turn governance from a document into an enforceable mechanism.

### 2. Core concepts and vocabulary
- **Bronze** — raw data as received, plus ingestion metadata; append-only and effectively immutable; the replay source and audit record.
- **Silver** — cleaned, typed, deduplicated, conformed data; the first stable layer; often one table per business entity.
- **Gold** — business-level aggregates and models for consumption.
- **Quality gate** — automated checks that must pass to promote data to the next layer.
- **Layer contract** — what the layer guarantees (schema, keys, freshness, completeness) and who owns it.
- **Quarantine** — storage for records that fail validation, kept for repair rather than dropped.
- **Freshness SLA** — maximum acceptable lag between source event and availability in a layer.
- **Schema evolution policy** — explicit rules for adding/renaming/removing columns across layers.
- **Reconciliation** — comparing counts/sums between layers or against the source to prove nothing was lost or duplicated.

### 3. How it works, step by step
1. **Land in Bronze unchanged.** Store the payload, source identifier, extraction timestamp, batch/offset. Do not "fix" data here. Immutability is enforced operationally (write-once policies, no updates) so any error downstream can be reprocessed.
2. **Validate Bronze→Silver.** Apply schema/type checks, null and range checks, referential checks, deduplication on the business key, standardisation (timestamps to UTC, codes to canonical values). Failures go to quarantine with a reason code.
3. **Write Silver with schema control.** Turn automatic schema merging off (e.g., `mergeSchema=false`) so stray source columns do not leak in; evolve explicitly with `ALTER TABLE`.
4. **Reconcile.** Row counts in = rows out + quarantined + intentionally dropped duplicates.
5. **Model Gold.** Build facts/dimensions or aggregates from Silver only (not directly from Bronze), applying business rules and final quality tests.
6. **Publish with SLAs.** Each layer has freshness and completeness targets; monitoring alerts when breached.
7. **Report quality** per layer (pass rates, quarantine volume, freshness) to owners.

### 4. Worked example
```sql
-- Bronze: append-only raw
INSERT INTO bronze.customers_raw
SELECT payload, _source_file, current_timestamp() AS _ingested_at FROM staged;

-- Silver: typed, deduplicated, validated
CREATE OR REPLACE TEMP VIEW silver_candidates AS
SELECT
  CAST(get_json_object(payload,'$.id') AS BIGINT)               AS customer_id,
  lower(trim(get_json_object(payload,'$.email')))                AS email,
  to_timestamp(get_json_object(payload,'$.updated'))             AS updated_at,
  _ingested_at,
  row_number() OVER (PARTITION BY get_json_object(payload,'$.id')
                     ORDER BY to_timestamp(get_json_object(payload,'$.updated')) DESC,
                              _ingested_at DESC) AS rn
FROM bronze.customers_raw;

-- Good rows
MERGE INTO silver.customers t
USING (SELECT * FROM silver_candidates
       WHERE rn = 1 AND customer_id IS NOT NULL AND email LIKE '%@%') s
ON t.customer_id = s.customer_id
WHEN MATCHED AND s.updated_at > t.updated_at THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;

-- Bad rows to quarantine
INSERT INTO quarantine.customers
SELECT *, 'missing_id_or_bad_email' AS reason FROM silver_candidates
WHERE rn = 1 AND (customer_id IS NULL OR email NOT LIKE '%@%');
```
Reading the result: counts of Silver upserts plus quarantine plus rows discarded by `rn > 1` must equal Bronze rows in the batch. A rising quarantine ratio signals source degradation before dashboards show it.

**Defending the Bronze boundary.** "Bronze is immutable, yet you type and dedup — isn't that transformation?" The answer: Bronze keeps the *original payload*; any parsing happens *after* the immutable copy exists (in Silver). If a Bronze table adds typed columns for queryability, the original raw column remains and nothing is overwritten, so the layer stays replayable.

### 5. Under the hood
- **Delta/Iceberg commits** make each layer's write atomic; a failed validation job never leaves a half-written Silver table visible.
- **Exactly-once effect** comes from idempotent MERGE on keys plus checkpointed offsets, not from the transport.
- **CDC vs snapshot sources.** If a source moves from full snapshot to incremental CDC, change only the Bronze ingestion and a small normalising step that produces the same Silver input contract (current-state or change-log), so Silver logic is untouched.
- **Stream plus nightly batch of the same data.** Treat both as Bronze sources with their lineage tagged. Silver reconciles on business key and version/timestamp; the batch acts as a correction/backfill authority, and a reconciliation job reports differences.
- **Freshness propagation.** A six-hour delay in Bronze means Gold's freshness SLA is breached unless consumers can use last-good data; publish "data as of" watermark columns/metadata so consumers can see lag, and define degraded-mode behaviour.
- **Ownership at boundaries.** Each boundary has a producer (layer owner) and a consumer; the contract states schema, keys, freshness, and change notification. In a 40-team platform, Bronze→Silver is usually platform- or source-domain-owned; Silver→Gold belongs to the consuming domain.

### 6. Performance and design implications
- Validation adds compute: push cheap checks (types, nulls) early, expensive cross-table checks selectively and on increments.
- Silver is the right place for dedup and conformance; doing it again in each Gold job multiplies cost.
- Partition Silver on ingestion/event date; cluster on business keys to make MERGE touch fewer files.
- Dimensional Gold vs one-big-table: star schemas serve many analytic consumers consistently; wide denormalised tables optimise a single high-traffic use; choose per consumption pattern, often both from the same Silver.
- Track: freshness lag per layer, quarantine rate, reconciliation delta, test failure rate, reprocessing frequency.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| CHECK / NOT NULL constraints | Reject invalid writes | Hard invariants in Silver |
| Expectations / dbt tests | Declarative quality rules | Promotion gates |
| `mergeSchema=false` | Block automatic column additions | Protect Silver schema |
| `MERGE INTO` | Idempotent upsert | Dedup and CDC apply |
| Quarantine table | Park failing rows | Repair and audit |
| Time travel / `RESTORE` | Roll back a bad write | After a faulty release |
| Freshness monitors | Alert on lag | Per-layer SLAs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Strict gates (fail pipeline) | Source is noisy and business prefers partial data — use quarantine instead |
| Quarantine and continue | Quarantine is never reviewed |
| Three layers | Tiny pipeline where extra hops add only latency |
| Extra "Silver-2" layers | Layer proliferation without clear purpose |

### 9. Common mistakes
- Putting business logic in Bronze → loses replayability → keep Bronze raw.
- Skipping reconciliation → silent data loss → counts-in = counts-out + quarantined.
- Building Gold straight from Bronze → duplicated cleaning logic → always go through Silver.
- No contract at boundaries → ownership disputes → document guarantees and owners.
- Letting schema auto-merge → accidental columns in Silver → explicit evolution.

### 10. 🎯 Interview kit
**30-second answer:** Bronze keeps raw data immutable for replay and audit, Silver validates, deduplicates and conforms it, and Gold models it for consumers; each boundary has a contract and automated quality gate so quality improves progressively and failures are quarantined, not propagated.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *SLAs when Bronze is six hours late?* → Gold freshness degrades; expose watermarks, define degraded mode and escalation.
- *Source changes to CDC?* → change ingestion/normalisation only; keep Silver contract.
- *Stream and batch feeds?* → both Bronze, reconcile in Silver by key/version.
- *Star schema vs OBT?* → by consumption; certified star core plus denormalised derivatives.
- *Is dedup in Silver a violation of Bronze immutability?* → no; raw copy stays untouched.

**Signals interviewers look for:** explicit layer contracts; quarantine and reconciliation; sensible freshness thinking.

### 11. Check yourself
- [ ] Write entry and exit criteria for each of the three layers.
- [ ] Explain how you prove no rows were lost between Bronze and Silver.
- [ ] Describe what happens downstream when a source is delayed by six hours.
- [ ] Explain why Gold should read from Silver, not Bronze.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [What is your position on whether Gold tables should be dimensional (star schema) or wide denormalized OBTs, and how does the consumption pattern change that decision at org scale?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32289_What_is_your_position_on_whether_Gold_tables_should_be_dimen.md>)
- [Deep] [How do you set SLAs and freshness guarantees per layer, and what happens to Gold's SLA when a Bronze source is delayed by six hours?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32309_How_do_you_set_SLAs_and_freshness_guarantees_per_layer_and_w.md>)
- [Deep] [When a source system changes from full-snapshot to incremental CDC delivery, how do you adapt Bronze ingestion without rewriting all of Silver?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32325_When_a_source_system_changes_from_full-snapshot_to_increment.md>)
- [Deep] [How do you handle a source that provides both a stream and a nightly batch of the same data? Which becomes Bronze, and how do you reconcile the two feeds?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32345_How_do_you_handle_a_source_that_provides_both_a_stream_and_a.md>)
- [Deep] [An interviewer probes: 'You said Bronze is immutable — but you also apply dedup and typing. Isn't that already transformation?' How do you defend your layer boundary?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32364_An_interviewer_probes_You_said_Bronze_is_immutable_but_you_a.md>)
- [Expert] [How do you define the contract at each medallion boundary so that Bronze-to-Silver and Silver-to-Gold ownership is unambiguous across a 40-team data platform?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32286_How_do_you_define_the_contract_at_each_medallion_boundary_so.md>)
- [Expert] [When two Silver tables disagree because they applied conflicting business rules, how do you resolve 'whose logic wins' organizationally, not just technically?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32313_When_two_Silver_tables_disagree_because_they_applied_conflic.md>)
- [Expert] [How do you measure whether the medallion architecture is actually delivering ROI vs the old pipelines? What metrics do you put in front of the CFO?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Q-32341_How_do_you_measure_whether_the_medallion_architecture_is_act.md>)

## Related concepts

- [Layered Pipelines Bronze Silver Gold](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>) *(Data Architecture)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [GDPR Deletion in Feature Pipelines](<../../Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>) *(Pipeline Orchestration)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › What Is a Medallion Architecture?
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › The Rise of Lakehouse Architectures
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Medallion Architecture and Its Practical Challenges
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Chapter 2. Laying the Groundwork
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Key considerations
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Concrete example
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Cautions and limits
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › The Medallion Architecture
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Exploring the Bronze Layer
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Exploring the Silver Layer
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Establishes a layer for augmenting data
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Medallion Architecture
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › The layered approach
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Silver layer
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Gold layer
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › The Medallion Architecture
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Putting It All Together › Medallion Architecture
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Putting It All Together › The Bronze Layer (Raw Data)
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Putting It All Together › The Silver Layer
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Batch and Real-Time Processing in Databricks › Batch ETL process demo

