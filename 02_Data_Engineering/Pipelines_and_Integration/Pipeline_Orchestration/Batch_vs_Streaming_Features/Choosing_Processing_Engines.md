---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Choosing_Processing_Engines"
level: 5
type: "deep-dive"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "AWS", "Apache_Airflow", "Apache_Flink", "dbt"]
---

# Choosing Processing Engines

**L5 Expert** · [↑ Pipeline Orchestration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Pick the processing engine by workload shape, data size, latency, skills and cost — warehouse SQL for relational modeling, Spark for large or complex distributed processing, stream engines for continuous low-latency work, and serverless functions for small event-driven steps.

### 1. The big picture
There is no single best data processing engine; there are engines that fit particular shapes of work. A modern platform typically runs several: a warehouse or lakehouse SQL engine for modeling and BI, Spark for heavy batch and ML feature jobs, a stream processor (Flink, Kafka Streams, Spark Structured Streaming) for continuous computation, and serverless functions (Lambda, Cloud Functions) for small glue tasks. Choosing well is mostly about avoiding two failure modes: using a heavyweight cluster for a job a SQL statement could do, and forcing a small engine into a workload that needs distribution.

A helpful mental model from the data engineering books is the unification of batch and streaming: batch is just a bounded stream. Engines increasingly support both, so the question shifts from "batch or streaming engine?" to "which execution model, state handling and operational profile do we want?"

### 2. Core concepts and vocabulary
- **Processing engine** — software that executes transformations over data (query engine, distributed compute framework, stream processor).
- **MPP warehouse/SQL engine** — massively parallel SQL execution over columnar storage (Snowflake, BigQuery, Redshift, Databricks SQL).
- **Distributed compute framework** — general engine with APIs beyond SQL (Spark, Dask, Flink).
- **Serverless compute** — platform runs your code on demand, scaling to zero; billed per execution.
- **Cluster vs serverless Spark** — managed/self-provisioned cluster versus on-demand pools you do not size.
- **Push-down** — executing logic where the data lives to avoid movement.
- **Unified batch/stream (Dataflow model)** — one programming model for bounded and unbounded data.
- **Open table format** — Delta, Iceberg, Hudi: lets multiple engines share the same tables.
- **Total cost of ownership** — compute + storage + people + operations.

### 3. How it works, step by step
A repeatable way to choose:

1. **Characterize the workload.** Relational set operations? Custom code or ML? Continuous events? Tiny triggers? Data size per run? Required latency?
2. **Map to candidate engines.**
   - Mostly joins/aggregations on structured data → warehouse SQL (dbt models).
   - Very large scale, complex transformations, DataFrame/ML libraries → Spark.
   - Event-time windows, stateful stream logic, seconds latency → Flink / Structured Streaming.
   - Small, bursty, event-triggered tasks (file arrives, API call) → serverless functions.
3. **Check constraints.** Team skills, existing platform, governance, data residency, vendor lock-in, GPU needs.
4. **Estimate cost.** Per-run compute, idle cost (always-on cluster vs serverless), data movement/egress, maintenance effort.
5. **Prototype with real volumes** and measure runtime and cost.
6. **Decide where each transform lives** and write it down; revisit when volume or requirements change.

### 4. Worked example
Requirement: build features for an ML model from 5 TB of daily events plus a 2 GB reference table, refreshed every 4 hours.

| Step | Engine choice | Why |
|---|---|---|
| Parse and validate raw JSON | Spark | Large volume, custom parsing |
| Join to reference table, aggregate | Spark (broadcast the 2 GB table if executor memory allows) or warehouse SQL | Both work; use warehouse if data already there |
| Business metric marts | Warehouse SQL / dbt | Analysts own the logic |
| Embedding generation for millions of items | Spark with GPU pool or dedicated batch service | Needs specialized compute; avoid driver bottlenecks |
| Trigger on file arrival | Serverless function starting the workflow | Tiny, event-driven |

Reading it: one workflow, four engines, each doing what it is good at. The cost audit question later is "which of these quietly became the biggest bill?" — answered by tagging jobs and examining cluster utilization.

### 5. Under the hood   *(optional at L4)*
- **Spark** executes a DAG of stages with shuffles, offers SQL, DataFrame and ML APIs, and scales horizontally; startup overhead and tuning knowledge are the costs.
- **Warehouses** hide infrastructure and optimize SQL aggressively; limits appear with non-SQL logic, UDF-heavy code, or when data must leave the platform.
- **Stream processors** keep keyed state with checkpoints and event-time semantics; operational complexity is higher than batch.
- **Serverless functions** have memory/time limits and no shared state; fan-out via orchestrators scales them, but heavy data processing per invocation is wrong-fit.
- **Open table formats** reduce lock-in: the same Iceberg/Delta table can be read by several engines, making "which engine" a per-workload choice rather than a platform bet.
- **Serverless Spark vs clusters:** serverless suits bursty, unpredictable workloads (no idle cost, no sizing); fixed clusters can be cheaper for steady, high utilization and give more tuning control.

### 6. Performance and design implications
- Moving data between engines costs time and money; prefer engines that read the shared storage in place.
- Choose the engine closest to the data for set-based work.
- Compute-heavy work (embeddings, ML) needs the right hardware; a general Spark cluster may OOM or crawl.
- Keep an inventory of jobs, owners and costs; unmanaged Spark jobs commonly become the largest line item.
- Prefer fewer engines where possible: each adds skills, monitoring, and security surface.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| dbt / warehouse SQL | Declarative modeling in-place | Relational transformations |
| `spark-submit`, job clusters | Run Spark jobs | Large-scale batch |
| Structured Streaming / Flink job | Continuous processing | Low-latency stateful logic |
| Lambda / Cloud Functions + event triggers | Small event-driven steps | File-arrival hooks, light transforms |
| Cluster policies, autoscaling, tags | Control and attribute cost | Cost governance |
| Open table format (Delta/Iceberg) | Shared storage across engines | Avoid lock-in |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Warehouse SQL — relational logic, analysts own it | Logic needs custom libraries or non-tabular data |
| Spark — big data, complex or ML-adjacent transforms | A single SQL statement would do; cluster overhead is waste |
| Stream engine — event-time, seconds latency | Minutes latency suffices; batch is simpler |
| Serverless — spiky, small tasks, no ops | Long-running or memory-heavy jobs |
| Serverless Spark — bursty ML feature jobs | Constant high utilization where reserved capacity is cheaper |

### 9. Common mistakes
- Using Spark for tiny datasets → startup cost dominates → use SQL or a small engine.
- Dragging data out of the warehouse for trivial work → egress and delay → push down.
- Picking an engine by popularity → skills mismatch → weigh team capability.
- No cost attribution → surprise bills → tag jobs, set budgets and alerts.
- Locking data into one engine's private format → migration pain → open table formats.

### 10. 🎯 Interview kit
**30-second answer:** I choose the engine by workload shape: SQL warehouse for relational modeling, Spark for large or complex distributed processing, a stream engine for stateful low-latency work, serverless for small event-driven tasks. I weigh data size, latency, team skills, cost and lock-in, and use open table formats so engines can share data.

**Structure a longer answer:** clarify requirements → classify workload → candidate engines → constraints and cost → prototype/measure → recommendation and revisit triggers.

**Likely follow-ups:**
- *When push features to the warehouse vs Spark?* → relational aggregations in warehouse; custom/ML logic in Spark.
- *Serverless vs managed clusters?* → serverless for bursty/unpredictable; clusters for steady heavy use.
- *How reduce Spark bill?* → right-size, autoscale, spot, optimize joins/partitions, remove unused jobs.
- *Unified batch and streaming?* → same model; bounded vs unbounded input.

**Signals interviewers look for:** structured reasoning, cost awareness, and willingness to use several engines instead of one favorite.

### 11. Check yourself
- [ ] Place five example workloads on the engine map and justify each.
- [ ] Explain why open table formats change the engine decision.
- [ ] List three cost-control mechanisms for Spark.
- [ ] Describe when serverless functions are the wrong tool.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Apache Airflow](<../../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Flink](<../../../../TOOLS/Apache_Flink/_index.md>)
- [dbt](<../../../../TOOLS/dbt/_index.md>)

## Practice questions

- [Deep] [How would you compute embeddings for millions of log templates in a Spark batch job for a log-clustering model without OOM or a driver bottleneck?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13513_How_would_you_compute_embeddings_for_millions_of_log_templat.md>)
- [Deep] [How do you decide between running feature transforms in Spark SQL versus the DataFrame API versus UDFs and what's the performance cost of each?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13167_How_do_you_decide_between_running_feature_transforms_in_Spar.md>)
- [Deep] [How do you decide whether a slow feature pipeline should be optimized in code re-partitioned or just scaled with more hardware?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13208_How_do_you_decide_whether_a_slow_feature_pipeline_should_be_.md>)
- [Deep] [How do you evaluate whether to standardize the org on Spark at all versus letting teams pick Spark Flink Ray or warehouse-native compute per workload?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13254_How_do_you_evaluate_whether_to_standardize_the_org_on_Spark_.md>)
- [Deep] [What's your framework for deciding whether a given transform belongs in the warehouse (dbt/SQL) the lakehouse (Spark) or streaming (Flink) as an org-level standard rather than per-team preference?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13267_What_s_your_framework_for_deciding_whether_a_given_transform.md>)
- [Deep] [How would you decide the org's stance on Python versus Scala for Spark pipelines given talent availability performance and long-term maintainability?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13322_How_would_you_decide_the_org_s_stance_on_Python_versus_Scala.md>)
- [Expert] [How would you migrate a Lambda architecture with separate batch and speed layers to a single Kappa streaming pipeline and what do you give up?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13713_How_would_you_migrate_a_Lambda_architecture_with_separate_ba.md>)
- [Expert] [What is your multi-year strategy to migrate the org off a legacy Hadoop/YARN Spark estate onto Spark-on-Kubernetes or serverless Spark while everything keeps running?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13271_What_is_your_multi-year_strategy_to_migrate_the_org_off_a_le.md>)

## Related concepts

- [Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>) *(Data Platform Strategy)*
- [Unified Batch and Streaming Platforms](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Platforms.md>) *(Batch vs Stream)*
- [Log-first vs Processor-first Platforms](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Log-first_vs_Processor-first_Platforms.md>) *(Streaming Architecture)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Lambda Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › The Dataflow Model and Unified Batch and Streaming
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Hybrid Cloud
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › SQL and general-purpose code-transformation tools
- *Building Data Integration Solutions* — Jay Borthen — Data Ingestion and Streaming Tools › Conclusion
- *Building Data Integration Solutions* — Jay Borthen — Index › S
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Introducing Data Lakes
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Amazon Managed Service for Apache Flink › AWS Lambda for Transformation
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Amazon Managed Service for Apache Flink › Choosing the Right Batch Transformation Service
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Chapter 5: Big Data Analytics
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Three Distributed Programming Concepts to Be Aware of When Choosing an Open Source Framework › Chapter 1. Three Distributed Programming Concepts to Be Aware of When Choosing an Open Source Framework
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Six Dimensions for Picking an Analytical Data Warehouse › Querying and Transformation Features

