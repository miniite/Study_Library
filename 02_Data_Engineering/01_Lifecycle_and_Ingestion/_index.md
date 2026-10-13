---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Data Engineering Lifecycle and Ingestion

**L3 Practitioner** · [↑ Data Engineering](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Ingestion is the plumbing stage that moves data from source systems into storage; doing it well means understanding who produces the data upstream, how it is stored, and who will consume it downstream.

### 1. The big picture

This area zooms into the middle of the data engineering lifecycle. Data is generated in source systems you usually do not own, is *ingested* (moved) into storage you do own, and is then transformed and served. Ingestion is where a data engineer first actively designs pipeline behaviour, and it is often the most labour-intensive stage: connecting pipes to other pipes, making sure data flows consistently, securely and on time.

The child topics in this area cover the stage in depth: the lifecycle itself, source systems, ingestion engineering considerations, batch versus streaming ingestion, architecture principles and choices, queries and modelling, and serving. This page is the overview that ties them together and explains how ingestion relates to storage and serving, plus the human side of the work.

### 2. Core concepts and vocabulary

- **Data ingestion** — moving data from one place to another; in the lifecycle, from a source system into storage, as an intermediate step.
- **Data integration** — combining data from disparate sources into a *new* dataset (e.g., CRM + ads + web analytics → user profile). Ingestion is A→B; integration is A+B→C.
- **Internal ingestion** — copying data within one system (table to table, temporary stream caching); counted as part of transformation, not ingestion.
- **Data pipeline** — the whole combination of architecture, systems and processes that moves data through the lifecycle stages; ETL, ELT, reverse ETL and data sharing are patterns within it.
- **Upstream stakeholder** — data producers, typically software engineers who generate the data.
- **Downstream stakeholder** — consumers: analysts, data scientists, ML engineers, systems.
- **Managed connector** — a hosted tool (Fivetran, Matillion, Airbyte style) that handles extraction and loading so you do not write custom plumbing.
- **Data vanity project** — collecting and storing data that is never consumed.

### 3. How it works, step by step

1. **Understand the source.** Know how it generates data and its quirks (frequency, schema, whether it supports change capture).
2. **Choose the movement pattern.** Batch (periodic, bounded sets) or streaming (continuous events); push, pull or poll.
3. **Land the data in storage.** Storage choice shapes everything after: cloud warehouses can store, transform and serve; streaming platforms like Kafka or Pulsar can ingest, store and be queried; object storage often is the transit layer between systems.
4. **Apply the undercurrents on the way.** Encrypt in transit, hold credentials in a key manager, trust-but-verify the source, version-control and test the ingestion code, decouple it from source and destination.
5. **Hand over to transformation and serving**, and close the loop with stakeholders.

Working with people matters as much as the technology:

- **Upstream:** software engineers and data engineers usually sit in separate silos and engineers see data engineering as consumers of "data exhaust". Improving communication is the first step; involve them as stakeholders in data outcomes, make downstream data part of the product (so product managers allocate time), and agree how data changes will be announced to prevent pipeline regressions.
- **Downstream:** know the freshness, access and quality that consumers need; serving feeds back what was missed earlier.

### 4. Worked example

```text
Scenario: marketing wants a single customer profile in the warehouse.

Sources           CRM (API), ad platform (API), web analytics (event stream)
Ingestion (A->B)  managed connectors copy CRM + ads daily to object storage;
                  event stream lands continuously
Integration (A+B->C) a transformation joins the three into customer_profile
Serving           dashboard + reverse ETL: profile flows back into CRM for lead scoring
```

Notice the difference in vocabulary: moving each source's raw data is *ingestion*; joining them into the profile is *integration/transformation*; pushing the profile back into the CRM is *reverse ETL* (serving).

### 5. Under the hood

*(optional)* Storage is "all the way down": data is stored many times as it moves, passing through magnetic disk, SSD and memory, and across distributed services. Streaming frameworks and warehouses blur stage boundaries because one system plays several lifecycle roles. This is why designing ingestion requires knowing what the destination can do (can it deduplicate? support updates? query in place?) rather than treating storage as a passive box.

### 6. Performance and design implications

- Prefer **managed connectors** for commodity sources; invest custom engineering only where it matters.
- **Decouple**: avoid monolithic code tightly bound to one source or destination so either side can change.
- Apply **least privilege** to the ingestion identity; serving has the largest security surface, but ingestion credentials are an attractive target too.
- Track **freshness and failure** per pipeline; ingestion lag is the first thing consumers notice.

### 7. Commands and settings you should know

| Concept | What it does | When to use |
|---|---|---|
| Managed connector sync modes (full refresh, incremental) | Controls how much data is re-copied each run | Choosing cost vs completeness |
| Service account / IAM role per pipeline | Limits blast radius | Every production pipeline |
| Secrets manager / key manager | Keeps source credentials out of code | Any authenticated source |
| Version control + code review for ingestion code | Reproducibility, safe change | Always |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Managed connector | Source is unusual, or per-row pricing gets large |
| Custom ingestion code | A connector already exists and you would be maintaining undifferentiated plumbing |
| Close collaboration with software engineers | Treating them as pure "data exhaust" providers |
| Streaming ingestion | Consumers only look at daily numbers |

### 9. Common mistakes

- Not talking to source owners → schema changes break pipelines → set up change notification and contracts.
- Confusing ingestion with integration → muddled responsibilities → separate raw landing from combined datasets.
- Hard-coding credentials → leak risk → use a secrets/key manager.
- Writing monolithic, tightly coupled loaders → painful migrations → decouple source, transport and sink.

### 10. 🎯 Interview kit

**30-second answer:** Ingestion is moving data from source systems into storage; it differs from integration, which combines sources into a new dataset. Good ingestion starts with understanding the source and the consumers, uses managed tools where possible, and applies security and engineering discipline.

**Structure a longer answer:** define ingestion vs integration → batch vs stream, push vs pull → the destination and what it can do → security and reliability → stakeholders and communication → example.

**Likely follow-ups:** *Ingestion vs integration?* → A→B movement vs combining into a new dataset. *Why talk to software engineers?* → They own the schema changes that break pipelines. *Build vs buy a connector?* → Buy unless it is differentiating.

**Signals interviewers look for:** clear vocabulary, upstream awareness, security habits, pragmatism about managed tools.

### 11. Check yourself

- [ ] Explain ingestion, integration and internal ingestion with one example each.
- [ ] List three things you would agree with an upstream team before building a pipeline.
- [ ] Say which lifecycle stages a cloud data warehouse and Kafka each play.
- [ ] Describe how you would secure credentials for a third-party source.
<!-- explainer:end -->

## In this section (study in order)

1. [The Data Engineering Lifecycle](<Data_Engineering_Lifecycle/_index.md>) (L4 Advanced)
2. [Source Systems and Data Generation](<Source_Systems/_index.md>) (L4 Advanced)
3. [Ingestion Engineering Considerations](<Ingestion_Considerations/_index.md>) (L4 Advanced)
4. [Batch Ingestion](<Batch_Ingestion/_index.md>) (L4 Advanced)
5. [Message and Stream Ingestion](<Message_and_Stream_Ingestion/_index.md>) (L4 Advanced)
6. [Data Architecture Principles](<Data_Architecture_Principles/_index.md>) (L4 Advanced)
7. [Choosing Architectures and Technologies](<Architecture_Choices/_index.md>) (L4 Advanced)
8. [Queries Modeling and Transformation](<Queries_Modeling_and_Transformation/_index.md>) (L4 Advanced)
9. [Serving Data and Reverse ETL](<Serving_and_Reverse_ETL/_index.md>) (L4 Advanced)

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Software Engineering
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Chapter 7. Ingestion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Conclusion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Chapter 6. Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Serving Data
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Undercurrents
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Undercurrents
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Data Ingestion and Exploration
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Undercurrents and Their Impact on Source Systems

