---
id: "02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Data_Contracts"
tools: []
---

# Data Contract Practice

**L4 Advanced** · [↑ Data Contracts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A data contract is a versioned, enforceable agreement between a data producer and its consumers about structure, meaning, quality and service levels, placed as far upstream as possible so problems are prevented instead of discovered downstream.

### 1. The big picture
In most companies the team that produces data (an application team, a source system owner) is not the team that consumes it (analytics, ML, finance). The producer changes a column, renames a field or alters how a value is computed; the consumer finds out when a dashboard breaks or a model silently degrades. Collaboration around data is usually reactive: someone notices, files a ticket, a pipeline gets patched.

A **data contract** makes the interface explicit. It is the data's API: a formal specification of the schema, semantics, quality rules, freshness and ownership, versioned alongside the code that produces the data, and checked automatically. The central principle in the book material is to place contracts as far upstream (shift-left) as possible: constrain data where it is generated so downstream layers are not poisoned.

Producers tend to dislike contracts (extra work, constraints), consumers love them (certainty). Adoption therefore depends on making contracts cheap for producers and visibly valuable: fewer incidents, faster change reviews.

### 2. Core concepts and vocabulary
- **Data contract** — a formal agreement describing a data asset's schema, semantics, quality expectations, SLAs and terms of use.
- **Producer / consumer** — the team that owns and emits the data / the teams and systems that read it.
- **Contract spec** — the machine-readable document (JSON Schema, Avro/Protobuf IDL, YAML following a data contract standard) that holds the agreement.
- **Data asset** — the physical thing governed: a table, topic, file set or API response.
- **Detection** — mechanisms that notice a violation (schema diff, validation, tests).
- **Prevention** — mechanisms that stop a violating change from shipping or data from landing (CI checks, write-time validation, schema registry).
- **Breaking vs non-breaking change** — a change that can make existing consumers fail or misinterpret data, versus one they can safely ignore.
- **Backward / forward compatibility** — new readers can read old data / old readers can read new data.
- **Hard fail / soft fail** — block the data or deploy outright, versus let it through but alert, quarantine or tag.
- **Schema registry** — a service storing versioned schemas and enforcing compatibility rules on registration.
- **Semantic expectation** — meaning-level promises (`amount` is in minor units, `status` values are from a closed set, `event_time` is UTC).
- **SLA / SLO** — freshness, completeness and availability promises attached to the contract.

### 3. How it works, step by step
A practical contract architecture has four components: data assets, the contract spec, detection, and prevention.

1. **Identify the asset and owners.** Pick one high-value interface (an events topic, an operational table copied into the lake). Name a producer owner and consumer representatives.
2. **Write the spec.** Capture field names, types, nullability, allowed values, keys, PII classification, units, freshness expectation and compatibility policy. Store it in version control next to the producing code.
3. **Wire in detection.** Compare the spec against reality: the database's metadata (information schema) or the registry, and run data tests (null rate, uniqueness, accepted values, row count within range).
4. **Wire in prevention.** In the producer's CI pipeline, a pull request that changes the schema runs the compatibility check; a breaking change fails the build or requires explicit consumer sign-off and a version bump.
5. **Define the violation workflow.** Who is notified, whether it hard-fails or soft-fails, how consumers are given a deprecation window, how the new version rolls out.
6. **Measure.** Track contract coverage, violations caught before production, incident counts and time to resolution.

### 4. Worked example
A producer team owns an `orders` service emitting `order_created` events and a table `orders` that is replicated to the lake.

Contract spec (YAML sketch):

```yaml
name: orders.order_created
version: 2.1.0
owner: team-checkout
schema:
  - {name: order_id,   type: string, required: true, unique: true}
  - {name: customer_id,type: string, required: true, pii: true}
  - {name: amount,     type: integer, unit: "minor currency units", required: true, min: 0}
  - {name: currency,   type: string, enum: [USD, EUR, GBP], required: true}
  - {name: created_at, type: timestamp, tz: UTC, required: true}
  - {name: coupon,     type: string, required: false}
compatibility: backward
freshness: {max_lag: 15m}
quality:
  - null_rate(customer_id) == 0
  - row_count(day) within 30% of 7-day average
```

Scenario: a developer proposes adding a nullable `coupon` column. Is it breaking? Typically **no** for consumers that name the columns they use, and it is backward compatible in Avro/Protobuf when the field has a default. It *can* hurt consumers that use `SELECT *`, positional parsing, strict schema-on-read or column-count checks, so the contract says "additive nullable fields are allowed with a minor version bump and notification". Removing `currency`, changing `amount` from integer to decimal, or changing its unit are breaking and require a major version and a migration window.

In CI the schema diff runs: adding `coupon` passes with a minor version bump; renaming `customer_id` to `buyer_id` fails with a message naming three registered consumers.

### 5. Under the hood
**Compatibility checks in a schema registry.** When a producer registers a new schema version under a subject, the registry compares it to previous versions according to the configured mode:
- *BACKWARD*: consumers using the new schema can read data written with the old one (you may delete fields, or add fields with defaults).
- *FORWARD*: consumers using the old schema can read data written with the new one (you may add fields, or delete fields that had defaults).
- *FULL*: both. The `*_TRANSITIVE` variants check against all prior versions, not just the latest.
Incompatible registrations are rejected, so the producer cannot start writing data that violates the rule.

**Where to enforce.**
| Point | Strength | Weakness |
|---|---|---|
| CI / pull request (design time) | Cheapest, catches before deploy, gives context to the author | Cannot catch bad *data values* produced at runtime |
| Write time (producer or ingestion gate) | Stops bad data before it spreads; error is near the cause | Adds latency and can block the producer; needs quarantine path |
| Read time (consumer or transformation) | Protects the specific consumer; easy to adopt unilaterally | Every consumer repeats it; failures are far from the cause |
Mature setups layer all three; the book's maturity curve moves from reactive read-time checks to proactive upstream prevention.

**Typed libraries.** One approach from the book material is for the producing team to ship compiled, versioned libraries (Avro/Protobuf) with unit tests for backward compatibility, so every stored record is accountable to a contract version and remains readable years later.

**Semantics are the hard part.** Structural checks are easy. The costly incidents come from changed meaning: a status gaining a new value, a timestamp moving from local time to UTC, a field redefined. Contracts need descriptions, units, enumerations and examples, and a human change-review for semantic changes.

### 6. Performance and design implications
- Write-time validation costs CPU per record but saves much more in downstream re-processing and incident time; read-time validation repeats per consumer.
- Overly strict contracts create friction and shadow data paths; start with schema + ownership + a few critical quality rules, then grow.
- Version fan-out: supporting many old versions simultaneously is expensive; define deprecation windows (e.g. two releases or 90 days).
- Metrics to prove value: percentage of critical assets with contracts, violations blocked in CI, production incidents attributed to upstream change (trend down), mean time to detect and resolve, consumer-reported breaking changes.
- Quantify the cost of no contracts: engineer hours per incident times incident count, plus decisions made on wrong data, plus delayed projects due to firefighting.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Schema registry compatibility mode (`BACKWARD`, `FULL`, transitive) | Controls what schema changes are accepted | Event streams and CDC topics |
| JSON Schema / Avro IDL / Protobuf `.proto` | Machine-readable structure | Spec for events or files |
| Information schema queries / catalog API | Read actual table definitions | Detection: compare to contract |
| dbt tests, Great Expectations, Soda checks | Assert quality rules on tables | Detection of value-level violations |
| CI job running schema diff (e.g. `buf breaking` for Protobuf) | Fails PRs with breaking changes | Prevention at design time |
| Open data contract standard (YAML) | Common contract document format | Cross-team, tool-agnostic specs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Hard-fail on structural breaks and critical quality rules (PK null, wrong type) | Low-severity or heuristic rules that would block pipelines needlessly |
| Soft-fail (alert/quarantine/tag) for statistical or advisory rules | Rules whose violation silently corrupts finance or compliance data |
| Contract at the producer boundary (events, CDC source) | You cannot influence the producer; then put a gate at ingestion |
| Central contract registry and tooling | A handful of assets; a Git folder and CI may suffice |
| Build your own spec checker for learning/small scale | Scaling beyond a few contracts; lifecycle management argues for open source or a product |

### 9. Common mistakes
- Treating a schema as a contract → no ownership, semantics or SLAs → add owner, meaning, quality and freshness.
- Writing contracts without producer buy-in → ignored → embed checks in the producer's CI and give them value (impact reports).
- Only enforcing at read time → problems are found late and repeatedly → move checks upstream.
- No versioning/deprecation policy → every change is a fire drill → semantic versioning and notice periods.
- Declaring additive changes always safe → breaks `SELECT *` and strict parsers → define policy and notify.
- Measuring adoption by count of documents instead of incidents prevented → no proof of value → track outcome metrics.

### 10. 🎯 Interview kit
**30-second answer:** A data contract is a versioned, machine-checked agreement between a producer and consumers covering schema, semantics, quality and SLAs. It differs from a plain schema because it has owners, meaning, quality rules, freshness and a change process, and it is enforced in the producer's CI and at ingestion so breaking changes are prevented rather than discovered downstream.

**Structure a longer answer:** problem (reactive breakage) → components (asset, spec, detection, prevention) → enforcement points trade-offs → compatibility and versioning → rollout and metrics → a scenario.

**Likely follow-ups:**
- *Is adding a nullable column breaking?* → usually not for named-column consumers; breaking for positional/`SELECT *`/strict schemas; policy decides.
- *Hard vs soft fail?* → hard for structural and critical rules; soft for advisory or statistical ones with alerting.
- *Where to enforce?* → CI for schema, write-time for values, read-time as defense in depth.
- *How does a registry compatibility check work?* → compares new schema to prior versions under a mode and rejects violations.
- *How do you prove value?* → fewer upstream-caused incidents, faster resolution, coverage of critical assets.

**Signals interviewers look for:** you go beyond schema validation to semantics and ownership, you know where to enforce, and you can quantify impact.

### 11. Check yourself
- [ ] State the four components of a data contract architecture.
- [ ] Classify five schema changes as breaking or non-breaking under BACKWARD compatibility.
- [ ] Argue for CI, write-time and read-time enforcement, and say which you would start with.
- [ ] Draft a short contract for a table you know, including a freshness SLA.
- [ ] Name three metrics you would present to show contracts reduce incidents.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What exactly is a data contract in a modern data platform and how does it differ from a plain schema definition?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11242_What_exactly_is_a_data_contract_in_a_modern_data_platform_an.md>)
- [Intermediate] [A producer proposes adding a nullable column. Is that a breaking change? Why or why not?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11296_A_producer_proposes_adding_a_nullable_column_Is_that_a_break.md>)
- [Deep] [Where would you enforce a data contract: at write time, at read time, or in CI? Argue the trade-offs.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11249_Where_would_you_enforce_a_data_contract_at_write_time_at_rea.md>)
- [Deep] [What is the difference between a hard-fail and a soft-fail contract violation and when do you use each?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11264_What_is_the_difference_between_a_hard-fail_and_a_soft-fail_c.md>)
- [Deep] [What metrics would you track to prove data contracts are actually reducing incidents?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11279_What_metrics_would_you_track_to_prove_data_contracts_are_act.md>)
- [Deep] [What is the cost of NOT having data contracts, and how would you quantify it to a manager?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11294_What_is_the_cost_of_NOT_having_data_contracts_and_how_would_.md>)
- [Deep] [Explain how a compatibility check works in a schema registry when a producer registers a new schema version.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11310_Explain_how_a_compatibility_check_works_in_a_schema_registry.md>)
- [Deep] [A migration will change a contract for 50 downstream jobs. How do you plan and sequence it?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Q-11325_A_migration_will_change_a_contract_for_50_downstream_jobs_Ho.md>)

## Related concepts

- [Distinguishing Drift from Pipeline Breaks](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>) *(Data Architecture)*
- [Glossary Metadata and Contracts](<../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>) *(Lineage)*
- [Schema Changes Under Long Transactions](<../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>) *(Schema Evolution)*
- [Quality SLAs and Rule Testing](<../../Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [Validation Coverage and Quarantine](<../../Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>) *(Validation)*
- [Governing Real-time Features](<../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>) *(Stewardship)*

## References

- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Why the Industry Now Needs Data Contracts › Chapter 1. Why the Industry Now Needs Data Contracts
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › Chapter 3. The Challenges of Scaling Data Infrastructure
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › Why Data Development Needs a Design Surface
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — The Challenges of Scaling Data Infrastructure › Updating documentation
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Being Compliant with Regulatory Standards › Simplifying the Compliance Journey on Databricks
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Being Compliant with Regulatory Standards › Data Contracts
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Column Names as Contracts › Chapter 14. Column Names as Contracts
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Preventing the Data Lake Abyss How to Ensure That Your Data Remains Valid Over the Years › Establishing Data Contracts
- *Data Pipelines Pocket Reference* — James Densmore — Best Practices for Maintaining Pipelines › Maintain Data Contracts
- *Building Medallion Architectures* — Piethein Strengholt — Case Study Amadeus, a Tech Leader in the Travel Industry › Data Contracts
- *Building Medallion Architectures* — Piethein Strengholt — Medallion Governance and Security › Data Contracts
- *Building Medallion Architectures* — Piethein Strengholt — Medallion Governance and Security › Contracts Within a Catalog
- *Building Medallion Architectures* — Piethein Strengholt — Medallion Governance and Security › Contracts Within a Metastore
- *Data Engineering with Medallion Architecture* — Miki Eto — Understanding Data Mesh, Lakehouse, and Medallion › Expert notes
- *Data Engineering with Medallion Architecture* — Miki Eto — Data Integration Strategy, Business Impact, and ROI › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Glossary

