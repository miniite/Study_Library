---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Airflow", "Apache_Spark", "AWS", "Kubernetes", "Python"]
---

# Orchestration vs Processing

**L4 Advanced** · [↑ Pipeline Orchestration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An orchestrator decides *what runs, when, and in what order*; a processing engine does the actual work on the data — keep the two roles separate.

### 1. The big picture
A pipeline has two jobs that are easy to mix up. The first is **processing**: reading, transforming, joining, and writing data (Spark, SQL warehouses, Python, dbt). The second is **orchestration**: making sure the right steps run at the right time, in the right order, with retries, alerts, and visibility. Airflow's own documentation describes itself as a spider in the middle of a web: it coordinates work happening in many systems but is not itself a data processing tool.

In its simplest form orchestration is a cron entry that launches a script. In production it becomes a directed graph of dependent steps — some time-scheduled, some triggered by an event, some conditional — spread across systems, with logs and history. A typical example: extract customers from MySQL → clean and enrich in Spark → load to a warehouse → refresh a dashboard. Each step has dependencies, timing constraints, and failure modes; the orchestrator holds that picture together.

### 2. Core concepts and vocabulary
- **Orchestrator / workflow manager** — software that schedules and monitors task graphs (Airflow, Dagster, Prefect, AWS Step Functions, Azure Data Factory, Google Cloud Composer).
- **DAG (directed acyclic graph)** — tasks as nodes, dependencies as arrows, no cycles. Airflow DAGs are written in Python.
- **Task / operator** — a unit of work; an operator is a reusable template (BashOperator, SparkSubmitOperator, a SQL operator).
- **Sensor** — a task that waits for a condition (a file arrives, a partition appears).
- **Scheduler / executor / worker** — scheduler decides what is due; executor decides how tasks are run; workers run them.
- **Data interval / logical date** — the time slice of data a run is responsible for, distinct from the wall-clock time when it runs.
- **XCom** — Airflow's small-message channel between tasks (metadata, not data).
- **Idempotency** — running a task twice for the same interval yields the same result.
- **Backfill** — running a DAG for past intervals.
- **Two DAGs problem** — Airflow DAG (orchestration graph) vs. Spark's internal DAG of stages (processing graph). They are different things.

### 3. How it works, step by step
Taking the Spark example from *Fundamentals of Data Engineering*:
1. Airflow ingests JSON from three APIs and writes each to its own S3 prefix.
2. Airflow triggers a Spark job through an API call.
3. Spark builds *its own* DAG: read three sources into DataFrames, join them (a shuffle), filter with SQL, write Parquet/Delta back to S3. Large inputs spill to disk; the join redistributes data by key across the cluster.
4. Airflow polls Spark (or waits on a sensor) until the job finishes.
5. Airflow marks its task, and then the whole DAG run, as success — or retries / alerts on failure.

The orchestrator never touched a row of data. That is the design: heavy lifting happens where the data and compute are.

What an orchestrator gives you:
- Dependency ordering and conditional branches.
- Schedules and event triggers.
- Retries with delay, timeouts, and alerting.
- Central logs, run history, and a UI showing state per task.
- Parameterization by logical date so reruns are reproducible.

### 4. Worked example
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

default_args = {"retries": 2, "retry_delay": timedelta(minutes=5)}

with DAG("orders_daily", start_date=datetime(2026, 1, 1),
         schedule="@daily", catchup=False, default_args=default_args) as dag:

    wait_for_file = S3KeySensor(
        task_id="wait_for_orders_export",
        bucket_name="lake", bucket_key="raw/orders/{{ ds }}/_SUCCESS",
        timeout=3600, mode="reschedule")

    transform = SparkSubmitOperator(
        task_id="transform_orders_to_silver",
        application="s3://jobs/orders_silver.py",
        application_args=["--date", "{{ ds }}"],      # job overwrites the {{ ds }} partition only
        conn_id="spark_default")

    wait_for_file >> transform
```
What to notice: the DAG file contains only coordination (sensor, submit, date parameter). The Spark script does the transformation and writes only the partition for `{{ ds }}`, so a retry or backfill overwrites rather than appends. `mode="reschedule"` frees the worker slot between sensor checks. Task ids describe the action (`transform_orders_to_silver`) so the UI, logs, and alerts are readable.

### 5. Under the hood
- The **scheduler** parses DAG files repeatedly and creates *DAG runs* and *task instances* in the metadata database; the **executor** (Local, Celery, Kubernetes) dispatches them to workers.
- Heavy Python at DAG top level is re-executed at every parse, slowing the scheduler: keep imports and API calls inside tasks.
- **XCom** is stored in the metadata DB by default, so pushing DataFrames through it overloads the database. Use it for small values (a file path, a row count); pass large state via object storage (S3/GCS/ADLS) and pass the *location*.
- Running the transformation inside an Airflow worker (pandas in a PythonOperator) couples data volume to worker memory and makes the orchestrator your bottleneck.
- Secrets live in Airflow Connections/Variables backed by a secrets manager (Vault, AWS Secrets Manager); jobs receive credentials from the platform, never hard-coded in DAG files or logs.

### 6. Performance and design implications
- Orchestrator load scales with *number of tasks and frequency*, not data volume; thousands of tiny tasks can saturate the scheduler while a single big Spark job barely registers.
- Prefer coarse-grained tasks (one Spark job per logical step) over per-file or per-row tasks; use dynamic task mapping when fan-out is truly needed.
- Parallelism limits: `max_active_runs`, `max_active_tasks`, and **pools** protect downstream systems from stampedes.
- Measure: scheduler delay (queued → running), task duration trends, retry rates, SLA misses.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow dags list` / `dags test` | Verify DAGs import and run once | CI validation |
| `airflow dags backfill` | Run past intervals | Historical reprocessing |
| `schedule`, `catchup` | When runs happen; whether missed intervals are created | Avoid accidental massive catch-up |
| `retries`, `retry_delay`, `execution_timeout` | Failure behavior | Every task |
| `pool`, `max_active_tasks` | Concurrency caps | Protect shared systems |
| XCom / object storage | Task-to-task state | Small values vs large datasets |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Managed Airflow (MWAA, Composer, Astronomer) — you want existing DAGs with low ops | Cost or customization needs outweigh convenience |
| Self-hosted Airflow on Kubernetes — thousands of DAGs, custom isolation | Small team without platform skills |
| Serverless orchestrators (Step Functions, ADF) — cloud-native, simple flows | Complex Python-defined dynamic graphs and a large Airflow codebase |
| Event-driven triggers — data arrival defines start | You need strict time-based SLAs and no dependable arrival signal |

### 9. Common mistakes
- Doing the transform inside the orchestrator → worker OOM, no scaling → submit to Spark/warehouse.
- Pushing big payloads through XCom → metadata DB bloat → store in object storage, pass paths.
- Non-idempotent tasks (blind `INSERT`) → duplicates on retry → overwrite partition or merge by key.
- Using `datetime.now()` in tasks → reruns differ → use the logical date.
- Giant monolith DAG → hard to reason about → split by domain and link with dataset/triggers.

### 10. 🎯 Interview kit
**30-second answer:** Orchestration controls order, timing, retries, and visibility of tasks; processing is the work done on data by engines like Spark or a warehouse. Airflow should trigger and monitor processing, not perform it, and tasks should be idempotent and parameterized by logical date so retries and backfills are safe.

**Structure a longer answer:** clarify (batch vs streaming, scale, who owns it) → core mechanism (DAG, scheduler, executor, operators, sensors) → trade-offs (managed vs self-hosted; XCom vs object storage) → real scenario (nightly Spark transform with sensor and backfill).

**Likely follow-ups:**
- *Why not use cron?* → no dependency graph, retries, history, backfill, or alerting.
- *XCom vs object storage?* → XCom for small metadata; object storage for datasets and for durability across workers.
- *How do you handle credentials?* → secrets backend and connections; never in code.
- *Build vs buy for managed Airflow?* → ops overhead and upgrades versus cost, networking, and customization.

**Signals interviewers look for:** clear boundary between control plane and data plane, idempotency, awareness of scheduler limits.

### 11. Check yourself
- [ ] Explain the two "DAGs" in an Airflow + Spark pipeline.
- [ ] Say why XCom is not for data.
- [ ] Write a DAG with a sensor and a parameterized Spark task.
- [ ] Explain what makes a task safe to retry.
<!-- explainer:end -->

## Go deeper

- [Shared Pipeline Platforms](<Shared_Pipeline_Platforms.md>)

## Implementations (tools)

- [Apache Airflow](<../../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [Python](<../../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is Kafka at a high level and how does it commonly feed data into a Spark or Airflow pipeline?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13413_What_is_Kafka_at_a_high_level_and_how_does_it_commonly_feed_.md>)
- [Intermediate] [What is templating with Jinja in Airflow and how might you inject the execution date into a task?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13397_What_is_templating_with_Jinja_in_Airflow_and_how_might_you_i.md>)
- [Intermediate] [What breaks when you run dbt models directly from a BashOperator versus using Cosmos or the dbt-Airflow integration and how does observability differ?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13644_What_breaks_when_you_run_dbt_models_directly_from_a_BashOper.md>)
- [Intermediate] [How do you manage dependencies and Python environment consistency across Airflow workers and Spark executors?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13200_How_do_you_manage_dependencies_and_Python_environment_consis.md>)
- [Deep] [How do you handle a pipeline that must join telemetry across two clouds for a hybrid deployment while controlling egress cost and latency?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13518_How_do_you_handle_a_pipeline_that_must_join_telemetry_across.md>)
- [Deep] [How do you use Airflow pools priority weights and queues to guarantee the daily critical pipeline preempts an ad-hoc backfill on a shared cluster?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13698_How_do_you_use_Airflow_pools_priority_weights_and_queues_to_.md>)
- [Deep] [How do you make an Airflow-orchestrated training pipeline idempotent so a retry doesn't double-write features or corrupt the model registry?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13165_How_do_you_make_an_Airflow-orchestrated_training_pipeline_id.md>)
- [Deep] [Describe the reference architecture you'd propose to decouple orchestration (Airflow) from execution (Spark on K8s) so you can swap either layer in future without rewriting pipelines.](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13255_Describe_the_reference_architecture_you_d_propose_to_decoupl.md>)

## Related concepts

- [Airflow Architecture](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>) *(Airflow Core)*
- [Pipeline Design Patterns](<../../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [DAG Design and Dependencies](<../../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>) *(Airflow Core)*

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — preface
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Getting started
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Meet Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Beyond the basics
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — AWS Step Functions › AWS Glue Workflows
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Amazon EventBridge › Choosing the Right Orchestration Service
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6 › Chapter 9
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — T › W
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Building End-to-End Data Pipelines › 13. Building End-to-End Data Pipelines
- *Data Pipelines Pocket Reference* — James Densmore — A Modern Data Infrastructure › Workflow Orchestration Platforms
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Chapter 7. Orchestrating Pipelines
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Managed Airflow Options
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Other Orchestration Frameworks
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Fan-Out
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › DataOps summary
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Orchestration
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Upstream Stakeholders › Orchestration

