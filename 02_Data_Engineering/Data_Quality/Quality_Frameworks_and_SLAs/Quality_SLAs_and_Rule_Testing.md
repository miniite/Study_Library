---
id: "02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing"
level: 5
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs"
tools: []
---

# Quality SLAs and Rule Testing

**L5 Expert** · [↑ Quality Frameworks and SLAs](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Data quality becomes manageable when quality dimensions are turned into measurable SLIs with SLO/SLA targets per dataset tier, enforced by automated rule tests placed early in the pipeline and tied to business impact and ownership.

### 1. The big picture
"Data quality" means different things to different stakeholders, and a platform nobody trusts is not successful however good the engineering. The first step, as the books stress, is to agree on *baseline expectations* — the organisation's ability to deliver high data availability and health through the whole lifecycle — expressed the same way software teams express reliability: **service-level indicators (SLIs)** measuring quality, **service-level objectives (SLOs)** setting targets, and **service-level agreements (SLAs)** adding consequences or commitments to consumers.

Second, quality must be translated into business value: an SLI like "percentage of records with a valid email address" with a 99.9% SLO links a technical measure to an outcome (campaign deliverability, revenue). Third, validation is shifted left — automated pre-flight checks run in CI and early pipeline stages so schema drift or bad types are caught before they contaminate downstream layers — and rules are treated as code: versioned, reviewed, and orchestrated.

Quality enforcement patterns such as **Audit-Write-Audit-Publish (AWAP)** formalise this: check inputs, write results to a staging area, audit the outputs, and only then publish — like assertions in unit tests for data, with the option to stop the whole run if expectations fail.

### 2. Core concepts and vocabulary
- **Quality dimensions** — accuracy, completeness, consistency, timeliness (freshness), validity, uniqueness (plus integrity, reasonableness).
- **SLI** — a measured quality metric (e.g., % non-null `customer_id`).
- **SLO** — target for an SLI over a window (≥ 99.9% daily).
- **SLA** — external/consumer commitment, often with escalation or credits.
- **Error budget** — allowed amount of failure within the SLO; used to prioritise reliability work.
- **Criticality tier** — classification (tier 1 revenue/regulatory → tier 3 exploratory) that sets strictness.
- **Rule / expectation / test** — an executable assertion about data.
- **Quality gate** — place in the pipeline where failing rules block or divert data.
- **AWAP** — Audit-Write-Audit-Publish.
- **WAP** — Write-Audit-Publish (write to branch/staging, audit, publish).
- **Quarantine / dead-letter** — holding area for failing records.
- **Data contract** — producer-consumer agreement on schema and semantics.
- **Shift left** — validate closer to the source.

### 3. How it works, step by step
1. **Classify datasets** into tiers by business impact and consumer count.
2. **Choose dimensions that matter** for each: finance ledger → accuracy, uniqueness, completeness, timeliness; clickstream → volume, freshness, validity. Do not put an SLA on all six for everything.
3. **Define SLIs precisely**: numerator, denominator, scope (partition/day), measurement query.
4. **Set SLOs/SLAs** with stakeholders using history (what is achievable) and business tolerance (what is needed).
5. **Write rules** (declarative tests): schema, not-null, uniqueness, ranges, referential integrity, reconciliation to source, freshness.
6. **Place gates**: source/ingestion (contract validation), Bronze→Silver (hard rules, quarantine), Silver→Gold (business rules, reconciliation), before publish (AWAP).
7. **Decide failure policy** per rule: block, quarantine and continue with last-known-good, or warn.
8. **Alert and escalate** by severity: page for tier-1 SLA breach, ticket for tier-3.
9. **Report** SLO attainment, error-budget burn, incident count; review monthly.
10. **Test the tests**: unit-test rules with known-bad fixtures; track rule precision and runtime.

### 4. Worked example
```yaml
# dbt-style tests with severity and tiers
models:
  - name: fct_orders
    meta: {tier: 1, owner: sales-data, slo_freshness_hours: 6}
    columns:
      - name: order_id
        tests: [unique, not_null]
      - name: customer_key
        tests:
          - relationships: {to: ref('dim_customer'), field: customer_key}
      - name: amount
        tests:
          - dbt_utils.accepted_range: {min_value: 0, max_value: 1000000}
          - not_null: {config: {severity: warn, warn_if: ">0", error_if: ">100"}}
```
```sql
-- SLI: validity of email in the last day
SELECT 100.0 * count_if(email RLIKE '^[^@]+@[^@]+\\.[^@]+$') / count(*) AS valid_email_pct
FROM silver.customers WHERE updated_date = CURRENT_DATE;       -- SLO target: >= 99.9

-- AWAP skeleton
-- 1. Audit input
SELECT count(*) FROM staging.orders_raw WHERE order_id IS NULL;            -- must be 0
-- 2. Write to a staging table, not the published table
CREATE OR REPLACE TABLE staging.fct_orders_candidate AS SELECT ... ;
-- 3. Audit output: uniqueness, reconciliation, ranges
SELECT count(*) - count(DISTINCT order_id) FROM staging.fct_orders_candidate;   -- 0
-- 4. Publish atomically (swap/exchange/partition replace)
ALTER TABLE gold.fct_orders SWAP WITH staging.fct_orders_candidate;
```
Reading the results: SLO attainment is `days meeting target / days`; error-budget burn shows how close you are to violating the SLO this month. Blocking versus quarantining: if one bad partition should not stop the report, publish last-known-good partition and flag it "stale", rather than failing the entire DAG.

### 5. Under the hood
- **Why not enforce everything at write time (schema-on-write)?** Constraints in the storage layer catch structural errors, but most quality issues are semantic and relational (late data, duplicates across batches, cross-table consistency, distribution shifts) and need pipeline-level rules and monitoring; also at org scale producers cannot be forced to comply, so detection and quarantine are needed.
- **Table-level enforcement:** Delta CHECK/NOT NULL constraints reject an entire violating batch atomically; Iceberg relies on engine/pipeline checks. Good for hard invariants, wrong for soft rules where partial acceptance is wanted.
- **Arbitration of "bad data vs edge case":** encode the decision as a *rule with an owner and documented rationale* in the contract/test repository, so a disputed value becomes a code review rather than a recurring argument; unresolved cases go to a data-owner council with a default of "quarantine, don't drop".
- **Legacy mainframe/batch-window feeds:** you can't change producers, so add controls around them: file-level checks (counts, checksums, control totals), reconciliation to source totals, landing validation in a staging area, quarantine, and SLAs defined in terms of *your* detectable measures and fixed windows.
- **Decentralised tooling (dbt, Spark, Snowflake, Databricks):** standardise on a *rule specification and result schema* (metrics table with dataset, rule, dimension, result, run id), let each engine execute rules in its native tool, and aggregate results centrally for SLOs and dashboards.
- **Chronic SLA misses from upstream breaking changes:** escalate in order: make impact visible (metrics, incidents), add a contract with CI compatibility checks, agree on a change-notice process, add buffering/adaptation layers, then escalate organisationally with the error-budget data.
- **Third-party and SaaS feeds:** you cannot control delivery times; define SLAs relative to *your* observation (freshness since receipt), add wait/retry windows, and publish "data as of" metadata.

### 6. Performance and design implications
- Rule cost: run cheap rules on every batch, expensive full-table scans on schedules or sampled/incremental partitions.
- Scope tests to changed partitions so validation does not become the slowest DAG step.
- Page vs ticket: page when a tier-1 SLO is at risk of breach with consumer impact in the next hours; ticket otherwise; review classification periodically so it does not drift.
- Executive scorecards need honest metrics (SLO attainment, downtime, open incidents by tier, MTTR) — not a wall of green that hides coverage gaps; show *coverage* (% tier-1 tables with tests) next to pass rate.
- Making the business case: translate incidents to cost — stale table → missed campaign revenue, wrong KPI → bad decision, hours of rework; track avoided incidents.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| dbt `tests`, `severity`, `store_failures` | Declarative tests with severity | Warehouse transformations |
| Great Expectations suites / checkpoints | Rule suites with reports | Spark/pandas/SQL |
| Soda checks | YAML SLIs with thresholds | Multi-engine |
| Deequ / Delta CHECK constraints | Engine-level rules | Spark/Delta |
| Orchestrator gates (Airflow sensors/branching) | Block or divert | Failure policy |
| Metrics table of rule results | Central SLO tracking | Platform-wide reporting |
| Data contracts + CI schema compat checks | Producer validation | Shift left |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Block the batch | Consumers prefer partial/last-good data |
| Quarantine and continue | No one owns the quarantine queue |
| Strict SLA with paging | Source is third-party and uncontrollable |
| Broad rule coverage | Runtime and noise exceed value |
| Centralised framework | Teams need native tooling — standardise the spec instead |

### 9. Common mistakes
- Setting SLAs on all dimensions for every dataset → noise and cost → tier and select dimensions.
- SLO without a measurement query → unverifiable → define numerator, denominator, scope.
- Tests only at the end → late detection → shift left with gates at layer boundaries.
- Green dashboards hiding poor coverage → false assurance → report coverage and criticality.
- Dropping bad rows silently → undetected loss → quarantine with reasons.
- No owner per rule → ignored failures → owners and escalation paths in metadata.

### 10. 🎯 Interview kit
**30-second answer:** I tier datasets by business criticality, pick the quality dimensions that matter for each, define SLIs with precise queries and set SLOs/SLAs with stakeholders. Rules run as code at gates — contract validation at ingestion, hard rules and quarantine into Silver, reconciliation and AWAP before publishing Gold — with severity-based alerting and SLO/error-budget reporting.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Which of the six dimensions get SLAs?* → those whose failure causes business harm for that dataset.
- *Block vs quarantine?* → depends on consumer tolerance; default to last-known-good plus flag for non-critical, block for regulatory.
- *Disagreement on bad data?* → encode in a rule with owner and rationale.
- *Upstream keeps breaking?* → visibility → contracts → CI checks → adaptation layer → escalation.
- *Buy-in for quality?* → quantify incident cost and avoided downtime.

**Signals interviewers look for:** SLI/SLO fluency, business translation, pragmatic enforcement, org awareness.

### 11. Check yourself
- [ ] Write an SLI, SLO and SLA for freshness of a tier-1 table.
- [ ] Describe AWAP and where the publish step must be atomic.
- [ ] Choose block vs quarantine for three scenarios and justify.
- [ ] Design a central rule-results table that works across dbt and Spark.
<!-- explainer:end -->

## Practice questions

- [Deep] [How do you decide which of the six data quality dimensions — accuracy, completeness, consistency, timeliness, validity, uniqueness — actually deserve SLAs for a given dataset versus being nice-to-have checks?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14543_How_do_you_decide_which_of_the_six_data_quality_dimensions_a.md>)
- [Deep] [How would you retrofit data quality onto a legacy mainframe-fed warehouse where you can't change the producers and batch windows are fixed and immovable?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14573_How_would_you_retrofit_data_quality_onto_a_legacy_mainframe-.md>)
- [Deep] [When two teams disagree on whether a value is 'bad data' or 'a legitimate edge case', how does your framework arbitrate and where does that decision get encoded?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14596_When_two_teams_disagree_on_whether_a_value_is_bad_data_or_a_.md>)
- [Deep] [What's your framework for classifying datasets into criticality tiers, and how does the tier drive the strictness of SLAs, on-call, and check coverage?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14617_What_s_your_framework_for_classifying_datasets_into_critical.md>)
- [Deep] [How do you build a quality framework that plays well with a decentralized tooling landscape where teams use dbt, Spark, Snowflake, and Databricks all at once?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14641_How_do_you_build_a_quality_framework_that_plays_well_with_a_.md>)
- [Expert] [When an SLA is chronically missed because the upstream team keeps shipping breaking changes, what organizational and technical levers do you pull, and in what order?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14560_When_an_SLA_is_chronically_missed_because_the_upstream_team_.md>)
- [Expert] [How do you build organizational buy-in for data quality investment when leadership sees it as a cost center and wants headcount on features instead?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14586_How_do_you_build_organizational_buy-in_for_data_quality_inve.md>)
- [Expert] [How do you quantify and communicate the customer-facing impact of a data quality breach — connecting a stale table to lost revenue or a bad decision — to justify the framework?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Q-14612_How_do_you_quantify_and_communicate_the_customer-facing_impa.md>)

## Related concepts

- [Governing Real-time Features](<../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>) *(Stewardship)*
- [Data Product SLAs](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>) *(Data Architecture)*
- [Validation Coverage and Quarantine](<../Validation/Validation_Coverage_and_Quarantine.md>) *(Validation)*
- [Data Quality Monitoring](<../Data_Observability/Data_Quality_Monitoring.md>) *(Data Observability)*
- [SLOs and Error Budgets](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/SLOs_and_Error_Budgets.md>) *(Cloud Fundamentals)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — How to Build Your Data Platform Like a Product Go from a (Standard Dev. of) Zero to Bona Fide Data › Sign Off on Baseline Metrics for Your Data and How You Measure It
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Ten Must-Ask Questions for Data-Engineering Projects › Question 9: What Is the SLA?
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Quality Patterns › 10. Data Quality Patterns
- *Data Engineering with Medallion Architecture* — Miki Eto — Data Integration Strategy, Business Impact, and ROI › Structure
- *Data Engineering with Medallion Architecture* — Miki Eto — Data Integration Strategy, Business Impact, and ROI › Key considerations
- *Data Engineering with Medallion Architecture* — Miki Eto — Data Integration Strategy, Business Impact, and ROI › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Data Integration Strategy, Business Impact, and ROI › Cautions and limits
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Chapter 9. Data Quality Design Patterns
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Quality Enforcement
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Quality Observation
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Data Quality Design Patterns
- *Data Engineering Fundamentals* — Zhaolong Liu — Preface
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › Chapter 1. Why Data Quality Deserves Attention---Now
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › Framing the Current Moment
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Chapter 2. Assembling the Building Blocks of a Reliable Data System
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Chapter 3. Collecting, Cleaning, Transforming, and Testing Data

