---
id: "TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture"
level: 4
type: "concept"
topic: "TOOLS/Apache_Airflow/Airflow_Core"
tools: ["Apache_Airflow", "AWS", "Kubernetes"]
---

# Airflow Architecture

**L4 Advanced** · [↑ Airflow Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Airflow is a scheduler plus a metadata database plus a pool of workers: the scheduler decides *what* should run and *when*, the executor decides *where*, workers do the work, and the database holds the state that ties them together.

### 1. The big picture
Apache Airflow (created at Airbnb, now an Apache top-level project) is a **workflow orchestrator**: you describe a pipeline as Python code (a DAG — directed acyclic graph of tasks), and Airflow runs it on a schedule, in dependency order, with retries, logging, alerting and a web UI. The central idea is *workflows as code*: DAG files live in version control, go through code review and CI like any other software, and can be generated dynamically with loops and configuration.

Airflow is an **orchestrator, not a data processing engine**. It tells Spark, a warehouse, a Kubernetes pod or a Python function *when* to run and watches the result; the heavy compute should happen elsewhere. It is also built for **batch** workflows with a clear schedule, not for sub-second event streaming. Understanding the architecture explains most operational behaviour: why a DAG file with slow top-level code slows everything, why the database is a bottleneck, why tasks sit "queued", and why XCom is not a data bus.

### 2. Core concepts and vocabulary
- **DAG** — a Python object grouping tasks and the edges (dependencies) between them, plus scheduling settings. Must have no cycles.
- **Task** — one node of a DAG: a unit of work defined by an operator (or a `@task`-decorated function).
- **Operator** — a template for a kind of task (`BashOperator`, `PythonOperator`, `SparkSubmitOperator`, `KubernetesPodOperator`, A-to-B transfer operators such as `SFTPToS3Operator`).
- **Sensor** — a special operator that waits for a condition (file arrived, partition exists, external task finished).
- **Hook** — a reusable client wrapper for an external system (S3, Postgres, HTTP). Operators use hooks; hooks use **connections**. A hook is *not* an operator: it does no scheduling and is not a node in the graph.
- **Connection** — a named set of credentials/host details stored in the metadata DB, environment variables, or a secrets backend.
- **DAG run** — one execution of a DAG for one schedule slot (or one manual trigger).
- **Task instance** — one execution of one task within one DAG run. It is identified by (dag_id, task_id, run, try number) and carries a state (`queued`, `running`, `success`, `failed`, `up_for_retry`, `skipped`, `upstream_failed` …). The *task* is the definition; the *task instance* is the run of it.
- **Scheduler** — the process that parses DAG files, creates DAG runs and decides which task instances are ready.
- **Executor** — the pluggable mechanism the scheduler uses to actually launch task instances (Local, Celery, Kubernetes …).
- **Worker** — the process/pod that executes the task code.
- **Metadata database** — a relational database (PostgreSQL or MySQL in production; SQLite for toys only) storing DAGs, runs, task instances, connections, variables, XComs, pools.
- **Web server** — the UI/REST API; it reads the metadata DB and lets users trigger, clear, mark and inspect.
- **XCom** — "cross-communication": small values passed between tasks, stored in the metadata DB.
- **Pool** — a named set of slots limiting how many tasks may run concurrently against a shared resource.

### 3. How it works, step by step
1. **Authoring.** You put a `.py` file into the DAGs folder. Every scheduler (and web server / DAG processor) parses it repeatedly — the file is *executed* top to bottom to build the DAG object.
2. **Parsing.** The scheduler reads DAG files in a loop, extracts tasks, dependencies and schedule, and serialises the DAG structure into the metadata DB (so the web server does not need to execute your code).
3. **Creating runs.** For each DAG whose schedule has passed since the last run, the scheduler creates a DAG run and task instances for it.
4. **Checking dependencies.** For each task instance the scheduler checks whether upstream tasks finished in states that satisfy the task's trigger rule, and whether concurrency limits (pool slots, `max_active_tasks`, parallelism) allow it. If yes, the instance becomes *scheduled* → *queued* and is handed to the executor.
5. **Execution.** The executor sends the task to a worker. With **LocalExecutor** it forks a subprocess on the scheduler machine; with **CeleryExecutor** it puts a message on a broker (Redis or RabbitMQ) that long-running Celery workers consume; with **KubernetesExecutor** it launches one pod per task.
6. **Reporting.** The worker runs the operator's `execute()`, streams logs, and writes the final state back to the metadata DB. The scheduler sees the new state and loops back to step 4 for downstream tasks.
7. **Observing.** The web server renders Grid/Graph/Gantt views and logs straight from the DB and log storage.

| Executor | Where tasks run | Strength | Weakness |
|---|---|---|---|
| Sequential | in the scheduler, one at a time | zero setup (SQLite) | no parallelism, dev only |
| Local | subprocesses on one machine | simple, parallel | limited to one host, scheduler shares resources |
| Celery | persistent worker pool via broker | horizontal scaling, low task start latency | broker + workers to operate, shared environment |
| Kubernetes | one pod per task | per-task isolation and images, autoscaling | pod start latency, more moving parts |

### 4. Worked example
```python
from datetime import datetime
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

def transform(**ctx):
    print("run for", ctx["ds"])

with DAG(
    dag_id="daily_sales",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    extract = BashOperator(task_id="extract", bash_command="echo extracting")
    load = PythonOperator(task_id="load", python_callable=transform)
    extract >> load
```
What happens: the scheduler parses the file, sees `@daily`, and at each midnight boundary creates a DAG run. The `extract` task instance is queued first; once it succeeds the scheduler queues `load`. To see this live: Grid view shows one column per run; clicking a square shows state, duration, tries, logs and rendered template fields. From the CLI, `airflow tasks test daily_sales extract 2026-01-01` runs one task instance in the foreground without recording state — ideal for debugging.

### 5. Under the hood
- **The scheduler loop** works on batches: it finds DAG runs that need task instances, then selects "schedulable" task instances using SQL queries with row-level locks (`SELECT … FOR UPDATE SKIP LOCKED`). That is what allows **several schedulers** (HA, Airflow 2.x) to run active-active against one DB without a separate coordinator. The database is therefore the single point of coordination and the most common bottleneck.
- **DAG parsing is separate** from scheduling decisions. Parsing is repeated every `min_file_process_interval` seconds per file. Expensive top-level code (API calls, big imports, DB queries) runs on *every* parse, which is why "keep top-level code light" is the first rule of DAG authoring.
- **Heartbeats.** Running task instances periodically update a heartbeat; if it stops, the scheduler marks the instance failed (zombie detection) and applies retry logic.
- **Celery path:** scheduler → serialised command on the broker queue → worker picks up → worker runs `airflow tasks run` → result/state written to DB. The broker only carries "run this task instance" messages, never data.
- **XCom** values are rows in the metadata DB (pickled/JSON). That is why moving a 200 MB DataFrame through XCom bloats the DB and slows everything. The fix is to write the data to object storage (S3/GCS) and push only the *path* through XCom.
- **Airflow 2.x/3 differences:** HA schedulers and DAG serialisation arrived in 2.0; datasets (data-aware scheduling) in 2.4; Airflow 3 splits out a separate API server and DAG processor and stops tasks from touching the metadata DB directly. Core ideas above remain.

### 6. Performance and design implications
- Keep the **metadata DB healthy**: PostgreSQL on decent hardware, connection pooling (PgBouncer), regular cleanup of old runs/XComs/logs (`airflow db clean`).
- Keep **DAG files cheap to parse**: no network calls or heavy imports at module level; push work into task callables or templated fields.
- Treat Airflow as the **control plane**: run Spark/dbt/SQL elsewhere and let Airflow submit and monitor. A `PythonOperator` crunching a big DataFrame on a shared Celery worker is a design smell; prefer `KubernetesPodOperator` or a job submitted to an external cluster.
- Limit concurrency deliberately with **pools** (e.g. a pool of 5 slots for a fragile source database), `max_active_runs`, `max_active_tasks` and global `parallelism`.
- Prefer sensors in `reschedule` or deferrable mode so waiting does not hold a worker slot.
- Measure with: scheduler heartbeat and DAG parse time metrics, task queued-duration, DB connection count, number of rows in `xcom`/`task_instance`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow db init` / `airflow db migrate` | create or upgrade metadata DB schema | first install, after upgrade |
| `airflow standalone` | start DB, scheduler, webserver for local play | learning/dev |
| `airflow dags list`, `airflow dags list-import-errors` | list DAGs, show parse failures | "my DAG is missing" |
| `airflow tasks test <dag> <task> <date>` | run one task, no state recorded | debugging |
| `airflow dags test <dag> <date>` | run a whole DAG in-process | local verification |
| `executor` | which executor to use | architecture choice |
| `parallelism`, `max_active_tasks_per_dag`, `max_active_runs_per_dag` | concurrency ceilings | tuning throughput/safety |
| `min_file_process_interval`, `dag_dir_list_interval` | how often DAG files are re-parsed/listed | large DAG folders |
| Pools (UI/CLI) | slot-limited resource groups | protect shared systems |
| Connections / secrets backend | credentials storage | never hard-code secrets in DAGs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Airflow for scheduled batch pipelines with dependencies, retries, backfills | you need low-latency event-driven or streaming processing |
| LocalExecutor for a small single-machine deployment | you need more than one machine's capacity or isolation |
| CeleryExecutor for steady high task volume and fast start | you lack the ops capacity for broker + worker fleet |
| KubernetesExecutor / KubernetesPodOperator for per-task images and isolation | tasks are tiny and pod start-up dominates runtime |
| Managed Airflow (MWAA, Cloud Composer, Astronomer) to offload operations | cost sensitivity at large scale or need for exotic customisation |
| Airflow vs cron | cron is fine for one independent script; Airflow earns its keep with dependencies, retries, history, backfills, UI |

### 9. Common mistakes
- Heavy top-level code in DAG files → scheduler spends its time parsing, tasks start late → move work inside callables.
- Passing large data through XCom → metadata DB bloat → store in object storage, pass the URI.
- Running compute on the Airflow worker (Spark driver, big pandas jobs) → worker OOM, noisy neighbours → submit to an external cluster/pod.
- Confusing a hook with an operator → trying to use a hook as a task → wrap it in a `PythonOperator`/`@task` or use the provider operator.
- Using SQLite/SequentialExecutor outside a laptop → no parallelism → PostgreSQL + a real executor.
- Not cleaning old metadata → slow UI and scheduler queries → scheduled `airflow db clean`.

### 10. 🎯 Interview kit
**30-second answer:** Airflow has a scheduler that parses DAG files and decides which task instances are ready, an executor that launches them on workers, a metadata database that stores all state, and a web server for the UI. DAGs are Python code; Airflow orchestrates external systems rather than doing heavy compute itself.

**Structure a longer answer:** clarify (single node or scaled?) → walk the scheduler loop and executor path → pick an executor and justify → mention DB as the coordination point and how you keep DAG parsing and XCom light → close with a real scenario (e.g. Celery + Postgres, Spark jobs submitted to Kubernetes).

**Likely follow-ups:**
- *What does the broker do in CeleryExecutor?* → carries task-run messages from scheduler to workers; no data flows through it.
- *Task vs task instance?* → definition vs one execution per DAG run, with state and try number.
- *What if the scheduler dies?* → running tasks continue on workers; with multiple schedulers another takes over; without, nothing new is scheduled.
- *How to keep Spark off the Airflow worker?* → `SparkSubmitOperator` in cluster mode / `KubernetesPodOperator` / provider operators for Databricks or EMR; Airflow only submits and polls.
- *Why not use XCom for data?* → lives in the metadata DB; use object storage plus a pointer.

**Signals interviewers look for:** you know Airflow is a control plane, you can name where state lives, you know the executor trade-offs, and you have operational instincts (parse cost, DB health, concurrency limits).

### 11. Check yourself
- [ ] Draw the path of one task instance from DAG file to final state under CeleryExecutor.
- [ ] Explain why top-level code in a DAG file is dangerous.
- [ ] Explain task vs task instance vs DAG run.
- [ ] Choose an executor for three scenarios and justify.
- [ ] Describe how you would move a 200 MB intermediate dataset between tasks safely.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../_index.md>)
- [AWS](<../../AWS/_index.md>)
- [Kubernetes](<../../Kubernetes/_index.md>)

## Practice questions

- [Basic] [How do you define dependencies between tasks in Airflow using the bitshift operators?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02018_How_do_you_define_dependencies_between_tasks_in_Airflow_usin.md>)
- [Basic] [What is a task instance in Airflow and how is it different from a task?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02230_What_is_a_task_instance_in_Airflow_and_how_is_it_different_f.md>)
- [Intermediate] [How do you push and pull a value using XCom in a PythonOperator task?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02140_How_do_you_push_and_pull_a_value_using_XCom_in_a_PythonOpera.md>)
- [Intermediate] [What is a provider package in Airflow 2 and why were providers split out of the core?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02194_What_is_a_provider_package_in_Airflow_2_and_why_were_provide.md>)
- [Deep] [What role does a message broker like Redis or RabbitMQ play in the CeleryExecutor setup?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02022_What_role_does_a_message_broker_like_Redis_or_RabbitMQ_play_.md>)
- [Deep] [What is the difference between Airflow orchestrating a job and Airflow actually processing the data itself?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02196_What_is_the_difference_between_Airflow_orchestrating_a_job_a.md>)
- [Deep] [Walk me through how you would set up dynamic task mapping over a variable list of files and cap concurrency so you do not overwhelm a downstream API.](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02052_Walk_me_through_how_you_would_set_up_dynamic_task_mapping_ov.md>)
- [Deep] [With the KubernetesExecutor how do you set per-task pod resource requests limits and node selectors and what happens when a pod is evicted?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02094_With_the_KubernetesExecutor_how_do_you_set_per-task_pod_reso.md>)

## Related concepts

- [Airflow DAGs and Backfills](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Airflow_DAGs_and_Backfills.md>) *(Pipeline Orchestration)*
- [Idempotent Airflow Tasks](<../../Python/Python_Language/Idempotent_Airflow_Tasks.md>) *(Python Language)*
- [Orchestration vs Processing](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>) *(Pipeline Orchestration)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — ETL and ELT › Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Meet Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Anatomy of an Airflow DAG
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Defining dependencies between tasks
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Mastering scheduling tools
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Mastering scheduling tools › Apache Airflow
- *Data Engineering Design Patterns* — Bartosz Konieczny — Sessionization › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Examples
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Apache Airflow Setup and Overview
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › The executor class that airflow should use. Choices include\
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Operators
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — AWS Step Functions › Managed Workflows for Apache Airflow
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — N › O

