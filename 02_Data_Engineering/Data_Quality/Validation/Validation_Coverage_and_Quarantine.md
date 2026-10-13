---
id: "02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Validation"
tools: ["Python"]
---

# Validation Coverage and Quarantine

**L4 Advanced** · [↑ Validation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Good validation is not just *having* checks — it is checking the right layers with the right coverage and deciding per rule whether a breach should stop the pipeline, warn, or divert the bad rows into a quarantine table.

### 1. The big picture
Once you have rules, three design questions remain: **where** do they run (coverage across layers), **what happens** when one fails (reaction policy), and **how** do you stop the checks themselves from failing silently. In a medallion layout the bronze layer is the first shield: technical validations (format, schema, completeness) there stop errors propagating into silver and gold. Business rules follow after transformation.

Quality is also relative to purpose: consumers often complain about data that is technically *correct* but not *fit for purpose* ("transaction" meaning purchases vs. any balance-affecting event). Coverage therefore needs business context, not just technical tests.

### 2. Core concepts and vocabulary
- **Validation vs. cleansing** — validation checks data against rules (and may reject it); cleansing fixes the data.
- **Hard failure** — the check stops the pipeline/deployment.
- **Soft warning** — logged and alerted, pipeline continues.
- **Quarantine / dead-letter table** — a side table holding rows that failed validation, with the reason, for later review and replay.
- **Fail-open vs. fail-closed** — a check that errors or finds nothing and *passes* is fail-open (dangerous); fail-closed treats "can't verify" as failure.
- **Reconciliation check** — source vs. target agreement.
- **Single source of truth** — one authoritative definition/location for each fact.
- **Schema registry** — enforces structure of messages (e.g. Avro) but not business meaning.
- **Coverage** — proportion of tables/columns/layers with meaningful tests.

### 3. How it works, step by step
1. **Inventory** critical tables and rank by business impact and incident history.
2. **Layer the checks**:
   - Raw/bronze: schema, types, completeness, row counts, duplicates, PII masking.
   - Silver: business rules, referential integrity, standardisation.
   - Gold/serving: metric ranges, reconciliations to finance, freshness.
3. **Classify each rule** as critical (hard fail), important (warn + quarantine) or informational (log).
4. **Route failures**: valid rows continue; invalid rows → quarantine with columns `batch_id`, `rule_id`, `failure_reason`, `payload`, `detected_at`.
5. **Alert** the owning team through the orchestrator (Slack/pager), including counts and sample rows.
6. **Review and replay**: someone triages quarantine regularly; fixed rows are re-injected; unfixable rows are archived.
7. **Make checks fail-closed**: assert that the check *ran*, that the input was non-empty, and that the result is explicit.
8. **Review coverage** quarterly and after every incident.

### 4. Worked example
Split a batch into good and quarantined rows in one pass (SQL):

```sql
-- stg.orders -> core.orders (good) and dq.orders_quarantine (bad)
WITH checked AS (
  SELECT o.*,
         CASE
           WHEN order_id IS NULL                        THEN 'order_id_null'
           WHEN amount IS NULL OR amount <= 0           THEN 'amount_invalid'
           WHEN NOT EXISTS (SELECT 1 FROM core.customers c
                            WHERE c.customer_id = o.customer_id) THEN 'customer_orphan'
         END AS failure_reason
  FROM stg.orders o
)
INSERT INTO core.orders  SELECT ... FROM checked WHERE failure_reason IS NULL;
INSERT INTO dq.orders_quarantine (batch_id, failure_reason, payload, detected_at)
SELECT :batch_id, failure_reason, TO_JSON(checked), CURRENT_TIMESTAMP
FROM   checked WHERE failure_reason IS NOT NULL;

-- policy gate: fail the batch if too many rows were quarantined
SELECT CASE WHEN 1.0 * SUM(CASE WHEN failure_reason IS NOT NULL THEN 1 END)
                    / NULLIF(COUNT(*),0) > 0.02
            THEN 1/0 END AS gate        -- forces failure if > 2% bad
FROM checked;
```
Reading it: individual bad rows don't stop the load, but a *rate* breach (>2 %) means something systemic — fail the batch. `NULLIF` and an explicit failure expression prevent "no rows ⇒ pass".

In Airflow terms, hard-fail checks raise an exception (task state `failed`), soft checks log and push a metric (or use a `trigger_rule`/`on_failure_callback` and a separate alert task), so they show up on the DAG UI without halting downstream tasks.

### 5. Under the hood   *(optional at L4)*
- **Why checks fail open:** a query that returns zero rows is read as "no violations", but zero rows can also mean the input table was empty, the filter wrong, or the join dropped everything. Fix: add a positive assertion (`row_count > 0`, expected partitions present) and treat missing results as failure.
- **Quarantine mechanics:** batch pipelines write quarantine rows as another output of the same job; streaming pipelines use a dead-letter topic/table so the stream keeps flowing. Quarantine storage grows and complicates reprocessing — apply retention and replay procedures.
- **Registry limits:** a schema registry guarantees field types, not `amount > 0`; business rules need a validation step after deserialisation.
- **Non-invasive DQ on pipelines you don't own:** validate at the boundaries you control — compare landed raw vs. downstream outputs, profile and monitor, and add contracts later.
- **PII-masking verification:** compare raw vs. masked data, scan all paths including nested JSON and free-text with pattern detectors, and assert zero residual matches.

### 6. Performance and design implications
- Early validation is cheaper — a bad batch caught at bronze costs one reload, not a cascade through gold.
- Quarantine keeps SLAs (good data flows) at the cost of eventual consistency and extra operational work.
- Hard-failing everything → frequent outages; warning on everything → ignored alerts. Tier rules deliberately.
- Thresholds should come from historical profiles and be reviewed after incidents.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Quarantine/dead-letter table | Stores rejected rows + reason | Batch & streaming bad-record handling |
| `NULLIF`, explicit assertions | Prevents silent pass | Making checks fail-closed |
| Airflow `on_failure_callback`, `trigger_rule` | Alert/route on failure | Soft vs hard failure |
| dbt `severity: warn \| error` | Per-test reaction level | ELT |
| Delta/Iceberg constraints & expectations | Enforce/drop/quarantine at write | Lakehouse tables |
| Great Expectations checkpoints | Run suites, store results | Orchestrated validation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Hard fail — regulated/financial, downstream irreversible | Non-critical feeds where a single bad row would block everyone |
| Quarantine + alert — high volume, tolerant consumers | Nobody reviews the quarantine → silent data loss |
| Reject whole batch — failure is systemic (schema break) | A few bad rows in an otherwise good batch |
| Skip/drop only bad rows | You must preserve counts for reconciliation (use quarantine, never silent drop) |
| Validate before **and** after load | Resource constrained → at least bronze + gold |

### 9. Common mistakes
- Overusing hard failures → constant outages → separate critical from non-critical.
- Neglecting soft warnings → broader problems unseen → trend warnings over time.
- Not logging why a row failed → can't diagnose → store rule id and reason.
- Ignoring the quarantine → problems persist → assign an owner and review cadence.
- Checks that pass on empty input → fail-open → assert row counts and execution.
- Confusing validation with cleansing → fixing data silently → keep rejects visible.

### 10. 🎯 Interview kit
**30-second answer:** "I put technical checks at ingestion and business checks after transformation, tier rules into hard-fail vs. warn, and send failing rows to a quarantine table with a reason while alerting the owner. I make checks fail-closed by asserting they ran on non-empty input."

**Structure a longer answer:** inventory & prioritise → layers → reaction policy → quarantine design and replay → alerting/ownership → thresholds from history → coverage review.

**Likely follow-ups:**
- *Fail hard vs. quarantine?* → fail hard when downstream is irreversible/regulatory or the issue is systemic; quarantine when volume is high and partial data is acceptable; a rate threshold can switch between them.
- *Check passed because it returned zero rows?* → positive assertions, fail on missing/empty input, test the checker with known-bad data.
- *Inherit pipeline with zero tests?* → week 1 freshness/volume/key tests on top tables; weeks 2–6 schema and referential checks; later business rules, contracts, ownership.
- *Rules the schema registry can't express?* → post-deserialisation validation with rule engine or stream processor.

**Signals interviewers look for:** you reason about consequences of each reaction policy, not just tools; you treat the checker as code that must be tested.

### 11. Check yourself
- [ ] Design a quarantine table schema and the replay process.
- [ ] Explain fail-open vs fail-closed with an example query.
- [ ] Assign each of five rules to hard-fail / warn / quarantine and justify.
- [ ] Describe how a soft warning surfaces in Airflow without failing the DAG.
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is the difference between data validation and data cleansing?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14343_What_is_the_difference_between_data_validation_and_data_clea.md>)
- [Intermediate] [How would you alert the team when a data quality check fails in production?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14427_How_would_you_alert_the_team_when_a_data_quality_check_fails.md>)
- [Deep] [What is the risk of blindly using COALESCE to replace nulls with zeros during validation?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14430_What_is_the_risk_of_blindly_using_COALESCE_to_replace_nulls_.md>)
- [Deep] [Explain the trade-off between failing a pipeline hard on a DQ breach versus loading with a quarantine table and alerting. When have you chosen each in production?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14449_Explain_the_trade-off_between_failing_a_pipeline_hard_on_a_D.md>)
- [Deep] [You inherit a pipeline with zero tests and constant incidents. What's your first-90-days plan to instrument data quality without boiling the ocean?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14476_You_inherit_a_pipeline_with_zero_tests_and_constant_incident.md>)
- [Deep] [How do you validate that a schema-registry-enforced Avro topic still meets business rules that the registry can't express (e.g. amount>0)?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14500_How_do_you_validate_that_a_schema-registry-enforced_Avro_top.md>)
- [Deep] [An SLA breach post-mortem shows a DQ check silently returned zero rows and passed. How do you make checks fail-closed instead of fail-open?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14527_An_SLA_breach_post-mortem_shows_a_DQ_check_silently_returned.md>)
- [Expert] [How would you validate that a PII-masking transform actually masked every path, including nested JSON and free-text fields, before data lands in the lake?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14470_How_would_you_validate_that_a_PII-masking_transform_actually.md>)

## Related concepts

- [Quality SLAs and Rule Testing](<../Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [Data Contract Practice](<../../Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>) *(Data Contracts)*
- [Governing Real-time Features](<../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>) *(Stewardship)*
- [Pipeline Design Patterns](<../../Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Data Quality Monitoring](<../Data_Observability/Data_Quality_Monitoring.md>) *(Data Observability)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Enforcing the source of truth
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Continuous Integration and Deployment › Test
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Quality Assurance › CHAPTER 9Quality Assurance
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Quality Assurance › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Data Quality and Validation in Pipelines
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Testing and Validation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Handling Test Data in CI/CD
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Continuous Integration (CI) for Databases
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Five Best Practices for Stable Data Processing What to Keep in Mind When Setting Up Processes Like › Use Data-Quality Measurement Jobs
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — The Do's and Don'ts of Data Engineering › Don't Rely on Hope
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — There's No Such Thing as Data Quality › Chapter 81. There's No Such Thing as Data Quality
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Your Data Tests Failed! Now What › Chapter 97. Your Data Tests Failed! Now What?
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Technical Validation Checks
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Data Cleansing Considerations
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Filtering
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Unit Tests and AWAP
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Pattern: Constraints Enforcer
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Event-Driven Pipeline Maintenance with EventBridge › Ensuring Data Quality and Reliability: Deequ and DQDL
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Event-Driven Pipeline Maintenance with EventBridge › AWS Glue Data Quality DQDL syntax
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Event-Driven Pipeline Maintenance with EventBridge › Composite rules
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Automated Data Quality Checks and Error Handling

