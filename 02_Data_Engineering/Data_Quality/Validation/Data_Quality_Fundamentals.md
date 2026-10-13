---
id: "02_Data_Engineering/Data_Quality/Validation/Data_Quality_Fundamentals"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Validation"
tools: []
---

# Data Quality Fundamentals

**L4 Advanced** · [↑ Validation](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Data quality is the gap between the data you have and the data the business *agreed* it should be, measured along a handful of dimensions (accuracy, completeness, timeliness, uniqueness, validity, consistency) and enforced by automated checks at every stage of the pipeline.

### 1. The big picture
Every data pipeline silently makes a promise: "what comes out is trustworthy enough to make decisions with." Data quality is the discipline of turning that promise into something testable. The data engineering lifecycle treats quality as an *undercurrent* — it is not one stage, it runs across ingestion, transformation, storage and serving. Quality can break before ingestion (a source system bug), in production (a bad transform or a retried job), or even during analysis (a join on the wrong key).

The central question is "what do I get compared with what I expect?" That means expectations must be *written down* first — in business metadata, a data contract, a schema, or a rule list. A check without an agreed expectation is just a number. The rest of this topic area (uniqueness, unit changes, load validation, quarantine) are specific families of checks built on this foundation.

A common slogan is "no data is better than bad data": a wrong number that looks right is more dangerous than an obvious gap, because people act on it.

### 2. Core concepts and vocabulary
- **Accuracy** — is the value factually correct (right amount, no duplicate that inflates a count, bot traffic not counted as humans)?
- **Completeness** — are all required records present and all required fields populated with valid values?
- **Timeliness / freshness** — is the data available when it is needed, and how old is the newest record?
- **Uniqueness** — no unintended duplicate entities or events; measured as % of unique keys.
- **Validity** — the value is in the allowed form/range/set (a pincode that has six digits is *valid*; it can still be *wrong*).
- **Consistency / conformity** — the same fact is represented the same way across tables and systems (same units, date format, naming, casing).
- **Integrity** — structural relationships hold (no orphan foreign keys).
- **Expectation / rule** — one testable statement about data ("`order_total >= 0`", "`customer_id` is never null").
- **Schema validation** — checking that incoming data has the agreed columns, types and nullability.
- **Data profiling** — measuring what the data actually looks like (null %, distinct counts, min/max, value patterns) before writing rules.
- **Data contract** — an agreement with the producer on schema, semantics, and freshness, so bad data is stopped at the source.
- **Orphan record** — a child row whose parent key does not exist (an order for a customer that is not in the customer table).
- **GIGO** — garbage in, garbage out: no downstream cleverness fixes flawed input.

### 3. How it works, step by step
A practical quality program has this shape:

1. **Define expectations** with the business: which fields are mandatory, what ranges are plausible, how fresh each dataset must be. Write them down (version control, a catalog, or the test framework itself).
2. **Profile** the data to learn real distributions — rules invented without profiling are either too loose or fire constantly.
3. **Choose where to check.** Cheapest and most valuable: at the boundary (ingest/bronze) for *technical* checks (schema, types, nulls, formats) and after transformation (silver/gold) for *business* checks (reconciliations, cross-field rules, metric ranges).
4. **Express checks as automated tests** that run inside the pipeline (SQL assertions, dbt tests, Great Expectations "expectations", AWS Glue Data Quality rules, Deequ).
5. **Decide the reaction** per check: hard fail (stop the pipeline), soft warning (log + alert), or quarantine (route bad rows aside) — see *Validation Coverage and Quarantine*.
6. **Measure over time.** Record metrics per run (row count, null rate, distinct rate, freshness lag) so you can detect drift and report a scorecard.
7. **Close the loop:** an owner, an alert channel, and a post-mortem habit when a check was missed.

| Dimension | Typical metric | Typical check |
|---|---|---|
| Completeness | % nulls in mandatory column | `COUNT(col)/COUNT(*) >= 0.99` |
| Uniqueness | % duplicate keys | `GROUP BY key HAVING COUNT(*)>1` returns 0 rows |
| Timeliness | now − max(loaded_at) | lag < SLA |
| Validity | % rows matching regex/set/range | `status IN ('active','inactive')` |
| Integrity | orphan FK count | anti-join to dimension returns 0 |
| Consistency | mismatch vs. second system | reconciliation of totals |

### 4. Worked example
A daily `orders` load. Expectations: `order_id` unique and non-null, `amount > 0`, `customer_id` exists in `dim_customer`, newest `created_at` is within 2 hours.

```sql
-- one health row per run; store it in a dq_results table
SELECT
  CURRENT_TIMESTAMP                                              AS checked_at,
  COUNT(*)                                                       AS row_count,
  1.0 * COUNT(order_id)  / NULLIF(COUNT(*),0)                    AS order_id_completeness,
  COUNT(*) - COUNT(DISTINCT order_id)                            AS duplicate_keys,
  SUM(CASE WHEN amount <= 0 THEN 1 ELSE 0 END)                   AS bad_amounts,
  SUM(CASE WHEN c.customer_id IS NULL THEN 1 ELSE 0 END)         AS orphan_orders,
  DATEDIFF(minute, MAX(o.created_at), CURRENT_TIMESTAMP)         AS minutes_since_last_row
FROM stg.orders o
LEFT JOIN dim_customer c ON c.customer_id = o.customer_id;
```
Reading the result: `duplicate_keys > 0` → accuracy/uniqueness breach (probably a replay); `orphan_orders > 0` → the dimension loaded after the fact table or a key is missing; `minutes_since_last_row > 120` → freshness breach; `row_count` far below yesterday → completeness issue even though every individual row looks fine. A passing set of checks lets the batch be promoted to the serving table; a failing one raises an alert with the metric values.

### 5. Under the hood   *(optional at L4)*
Checks are ultimately aggregate queries or per-row predicates. Two execution styles matter:
- **Aggregate/metric checks** (row count, null %, mean) are cheap — one scan — but can miss problems affecting a small fraction of rows, and don't tell you *which* rows. They also miss slow drifts that never cross a threshold.
- **Row-level rules** (`amount > 0`, regex, FK existence) identify the offending records, so they can feed a quarantine table, but cost more (joins, regex).
On columnar formats (Parquet/ORC) per-row-group min/max/null statistics let some checks be answered from metadata without reading data. Declarative tools (Glue Data Quality, Deequ, Great Expectations) compile rules into Spark/SQL jobs and can *recommend* rules by profiling.

### 6. Performance and design implications
- Check cost scales with data size: scan once and compute many metrics in the same pass rather than running one query per rule.
- For very large tables validate the *new partition/batch* (incremental), and sample for expensive checks.
- Too many alerts causes alert fatigue — prioritise columns by business impact and past incident history.
- Surface-area explosion: metrics for every column × segment × statistic is unmanageable by hand, which is why teams add automatic anomaly detection on top of hand-written rules.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK` | Database-enforced rules; reject bad writes | OLTP and warehouses that enforce them |
| dbt `not_null`, `unique`, `accepted_values`, `relationships` | Declarative tests run after models build | ELT pipelines |
| Great Expectations expectation suites | Python/YAML rules + validation results | Spark/pandas/SQL pipelines |
| AWS Glue Data Quality (DQDL rules) | Serverless rule evaluation at rest or in ETL jobs | AWS lakes/warehouses |
| Deequ / PyDeequ | Spark-native constraints and anomaly detection | Large Spark datasets |
| Schema registry / data contract | Producer-side schema enforcement | Streaming and API sources |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Strict hard-fail checks on finance/regulatory data | Non-critical feeds — one bad row shouldn't halt everything |
| Many cheap metric checks for broad coverage | They are the only layer — they miss small, row-level defects |
| Validate at the boundary (bronze) | You only check at the end — errors propagate and are costly to trace |
| Declarative framework | Rules live in someone's head or in ad-hoc notebooks |

### 9. Common mistakes
- No agreed schema/expectations up front → nothing to test against → define the contract first.
- Checking only the main table → orphans and mismatches in related tables go unseen → check relationships and all layers.
- Equating *valid* with *correct* (a well-formed but wrong pincode) → pair format checks with reference-data or reconciliation checks.
- Documenting rules once and never updating → rules go stale; keep them in version control next to the code.
- Treating quality as only the engineer's job → "fit for purpose" definitions come from business users; make ownership shared.

### 10. 🎯 Interview kit
**30-second answer:** "Data quality is how well data matches what the business expects. I measure it along accuracy, completeness, timeliness, uniqueness, validity and consistency, write the expectations down, and automate checks at ingest and after transforms, so that failures either stop the pipeline or are quarantined and alerted."

**Structure a longer answer:** clarify (which dataset, what's the consumer's expectation) → define dimensions and rules → where in the pipeline to check → reaction (fail/warn/quarantine) → monitoring and ownership → close with a real incident you'd have caught.

**Likely follow-ups:**
- *Accuracy vs. precision?* → accuracy = closeness to the true value; precision = level of detail (2.0 vs 2.0000).
- *What is an orphan record?* → a child whose parent key is missing; breaks joins and skews totals; prevented by FKs or caught by anti-join checks.
- *Where do you store DQ rules?* → in version control alongside pipeline code/tests, surfaced in the catalog.
- *Validity vs. accuracy?* → valid = conforms to format/range; accurate = actually true.
- *Data quality vs. reliability (SLA)?* → quality is about content correctness; reliability is about whether it arrived on time and completely — monitor both.

**Signals interviewers look for:** you start from business expectations, not tools; you separate technical from business checks; you know hard-fail vs. quarantine; you mention monitoring over time, not one-off checks.

### 11. Check yourself
- [ ] List six data quality dimensions and give one SQL check for each.
- [ ] Explain why a row-count check alone can pass while the data is wrong.
- [ ] Write an anti-join that finds orphan orders.
- [ ] Describe where you would place technical vs. business checks in a bronze/silver/gold pipeline.
- [ ] Explain "valid but wrong" with an example.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does 'garbage in garbage out' mean in the context of data pipelines?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14350_What_does_garbage_in_garbage_out_mean_in_the_context_of_data.md>)
- [Basic] [What is the difference between accuracy and precision when talking about data quality?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14364_What_is_the_difference_between_accuracy_and_precision_when_t.md>)
- [Basic] [What is a schema validation check and why do you run it when ingesting data?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14375_What_is_a_schema_validation_check_and_why_do_you_run_it_when.md>)
- [Basic] [What does 'data quality' mean and why is it important in a data pipeline?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14380_What_does_data_quality_mean_and_why_is_it_important_in_a_dat.md>)
- [Basic] [What is an orphan record and why is it a data quality problem?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14396_What_is_an_orphan_record_and_why_is_it_a_data_quality_proble.md>)
- [Basic] [What is the value of documenting data quality rules and where would you store that documentation?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14438_What_is_the_value_of_documenting_data_quality_rules_and_wher.md>)
- [Intermediate] [What is a quarantine or dead letter table and why is it useful for bad records?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14357_What_is_a_quarantine_or_dead_letter_table_and_why_is_it_usef.md>)
- [Intermediate] [What is an 'expectation' in a data validation framework and can you give an example?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Validation/Q-14384_What_is_an_expectation_in_a_data_validation_framework_and_ca.md>)

## Related concepts

- [Quality SLAs and Rule Testing](<../Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [Data Contract Practice](<../../Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>) *(Data Contracts)*
- [Data Quality Monitoring](<../Data_Observability/Data_Quality_Monitoring.md>) *(Data Observability)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data accountability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data quality
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Modeling › What Is a Data Model?
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Data Definitions and Logic
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › Chapter 1. Why Data Quality Deserves Attention---Now
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › What Is Data Quality?
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Collecting Data Quality Metrics
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Democratizing Data Quality › Prioritize data quality and reliability
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Event-Driven Pipeline Maintenance with EventBridge › AWS Glue Data Quality
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Data Quality
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Security and Governance › Data Profiling
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Introduction to Data Engineering › 1. Introduction to Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Quality Patterns › 10. Data Quality Patterns
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Understanding data quality monitoring
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Chapter 2. Data Quality Isn't About Pristine Data
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Defining Data Quality
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Measuring Data Quality
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Trustworthiness

