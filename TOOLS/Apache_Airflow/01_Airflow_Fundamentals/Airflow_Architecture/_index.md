---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals/Airflow_Architecture"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Airflow Architecture

**L4 Advanced** · [↑ Airflow Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Airflow is a small set of cooperating processes — scheduler, executor/workers, webserver and a metadata database — all coordinating through that database and a shared DAGs folder.

### 1. The big picture
Writing a DAG file is only half of Airflow. Something has to *read* that file, decide when a run is due, hand tasks to machines that execute them, record every state change, and let humans watch and intervene. Airflow splits this into components so each can be scaled and restarted independently:

- the **scheduler** decides *what* should run and *when*;
- the **executor** (running inside the scheduler process) decides *how* tasks are dispatched;
- **workers** actually run task code;
- the **webserver** shows state and lets you trigger/clear;
- the **metadata database** is the single source of truth for state;
- the **DAGs folder** (and optionally a triggerer for deferrable tasks) completes the picture.

The important design idea: components do **not** talk directly to each other. The database is the shared blackboard — the scheduler writes task state, workers update it, the webserver reads it. That is why the metadata DB is the most critical dependency.

### 2. Core concepts and vocabulary
- **Scheduler** — long-running process that parses DAG files, creates DAG runs and task instances, and queues eligible tasks.
- **Executor** — pluggable strategy for running queued tasks: `SequentialExecutor`, `LocalExecutor`, `CeleryExecutor`, `KubernetesExecutor`.
- **Worker** — process/container that actually executes a task's callable.
- **Webserver** — Flask app presenting DAGs, runs, logs, and admin pages (connections, variables, pools).
- **Metadata database** — relational DB (SQLite for demos; PostgreSQL or MySQL in production) holding DAG definitions, runs, task instances, XComs, connections, variables.
- **DAGs folder** — directory (or synced repo) of Python files the scheduler and workers must both be able to read.
- **Triggerer** — async process running *deferrable* operators' triggers (waiting without holding a worker slot).
- **Message broker** — Redis/RabbitMQ used by CeleryExecutor to hand tasks to workers.
- **Pool** — named bucket limiting parallel tasks hitting a resource.
- **Task states** — `none → scheduled → queued → running → success | failed | up_for_retry | skipped | upstream_failed`.

### 3. How it works, step by step
1. **Parse.** The scheduler's DAG processor imports every file in the DAGs folder, builds the DAG objects and stores a serialized copy in the metadata DB.
2. **Plan.** In its loop, it finds DAGs whose next interval has finished and creates a *DAG run* plus *task instances* in state `scheduled`/`none`.
3. **Check dependencies.** A task instance is eligible when its upstream tasks are satisfied, pools/concurrency allow it, and `depends_on_past` etc. hold. Eligible tasks become `queued`.
4. **Dispatch.** The executor sends queued tasks to workers: LocalExecutor spawns subprocesses; Celery pushes a message to the broker; Kubernetes launches a pod per task.
5. **Execute.** The worker re-reads the DAG file, runs the operator's `execute()`, streams logs to a log store, and updates the task state in the DB (heartbeating while running).
6. **Retry or finish.** On failure, if retries remain the state becomes `up_for_retry` and the scheduler requeues it after `retry_delay`; otherwise `failed` and downstream tasks become `upstream_failed`.
7. **Observe.** The webserver queries the DB (and log files) to render the grid, graph and Gantt views.

```
          DAGs folder ───────────────┐
              │ parse                │ read at run time
              ▼                      ▼
 Webserver ◄──► Metadata DB ◄──► Scheduler ──► Executor ──► Workers
   (UI/API)       (state)                        (queue)     (run tasks)
```

### 4. Worked example
Minimal local, production-like layout with Docker Compose-style settings (conceptual):
```bash
# environment shared by all components
export AIRFLOW__CORE__EXECUTOR=CeleryExecutor
export AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:***@db/airflow
export AIRFLOW__CELERY__BROKER_URL=redis://redis:6379/0
export AIRFLOW__CORE__DAGS_FOLDER=/opt/airflow/dags

airflow db migrate         # create/upgrade metadata tables (db init in older versions)
airflow scheduler          # process 1
airflow webserver -p 8080  # process 2
airflow celery worker      # process 3..N (scale horizontally)
airflow triggerer          # optional: deferrable operators
```
What to check afterwards:
- `airflow dags list` returns your DAG (scheduler parsed it).
- In the UI, trigger a run and watch a task move `queued → running → success`.
- Kill a worker mid-task: the task heartbeat stops; the scheduler marks it failed/up_for_retry and re-dispatches — evidence that state lives in the DB, not in the worker.

### 5. Under the hood   *(optional)*
- **Scheduler loop** — repeatedly: process finished DAG files → create runs → examine task instances with row-level locks (`SELECT … FOR UPDATE SKIP LOCKED`, which lets several schedulers run in parallel in Airflow ≥ 2.0) → enqueue to executor → process executor events (task finished).
- **DAG processor** — runs file parsing in separate subprocesses with timeouts (`dagbag_import_timeout`); a broken or slow file only affects itself. Re-parse frequency is `min_file_process_interval`.
- **Executors compared** — Sequential: one task at a time, SQLite only, demos. Local: processes on one machine. Celery: fixed pool of long-lived workers, low start latency. Kubernetes: pod per task, isolation and elasticity, slower start. CeleryKubernetes (hybrid) exists.
- **Task execution process** — the worker forks `airflow tasks run`, which loads the DAG, executes under a *task runner*, handles retries/timeouts, and writes to the DB. Heartbeat and a "zombie" check detect dead tasks.
- **Logs** — written locally per task try, optionally shipped to S3/GCS; the webserver fetches them via a log-serving endpoint on the worker or remote storage.
- **XCom** — small key/value messages stored in the metadata DB: inter-task communication, not bulk data transport.
- **High availability** — multiple schedulers (active/active via DB locking), multiple webservers, replicated Postgres; the DB is still the weak point.

### 6. Performance and design implications
- **The metadata DB is the bottleneck.** Every state change is a write; many tasks + frequent heartbeats can saturate it. Use PostgreSQL with enough connections, and a connection pooler for large setups.
- **Parse time = scheduling latency.** Slow top-level code delays new runs for *all* DAGs; monitor `dag_processing.total_parse_time` and per-file durations.
- **Concurrency knobs stack:** `parallelism` (global) → `max_active_tasks_per_dag` → `max_active_runs_per_dag` → pools → per-task `max_active_tis_per_dag`. A task waits at the lowest limit.
- **Workers must see the same code.** Mismatched DAG versions or missing Python packages on a worker cause "works in UI, fails in run".
- **Clean the DB.** Old runs, XComs and logs grow forever; schedule `airflow db clean`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow db migrate` / `db init` | Create or upgrade the metadata schema | Installation, upgrades |
| `airflow scheduler` / `webserver` / `celery worker` | Start each component | Deployment |
| `airflow config get-value core executor` | Show effective config | Troubleshooting |
| `core.executor` | Choose execution backend | Sizing architecture |
| `core.parallelism` | Global task slots | Cluster-wide throttle |
| `scheduler.min_file_process_interval` | Seconds between re-parses of a file | Reduce parse load |
| `core.dagbag_import_timeout` | Max seconds to import a file | Guard against slow DAG files |
| `airflow db clean` | Purge old rows | Housekeeping |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **LocalExecutor** — one reasonably large VM, modest scale | You need horizontal scale or HA |
| **CeleryExecutor** — many short tasks, low latency, static fleet | Workers are idle most of the day (cost), or you need per-task isolation |
| **KubernetesExecutor** — heterogeneous dependencies, bursty load, isolation | Tasks are tiny and pod start-up (seconds) dominates |
| **Managed Airflow** (MWAA, Cloud Composer, Astronomer) — less ops | You need unusual plugins/versions or tight cost control |
| **SQLite** | Only for tutorials — never concurrent or production |

### 9. Common mistakes
- **Running production on SQLite/SequentialExecutor** → one task at a time, locking errors → PostgreSQL + Local/Celery/Kubernetes.
- **DAG code present on the scheduler but not on workers** → tasks fail with import errors → sync DAGs (git-sync, shared volume, baked image).
- **Heavy imports in DAG files** → scheduler lag → lazy-import inside callables.
- **Using XCom for large payloads** → DB bloat/slow UI → store in object storage, pass the URI.
- **Never cleaning metadata** → slow UI and queries → retention policy and `db clean`.
- **Assuming the worker's local disk persists between tasks** → tasks of the same DAG may land on different workers → use shared storage.

### 10. 🎯 Interview kit
**30-second answer:** "Airflow has a scheduler that parses DAGs and creates runs, an executor that dispatches tasks to workers, a webserver for UI/API, and a metadata database that holds all state. They communicate through the database rather than directly, so it's the critical component."

**Structure a longer answer:** clarify (which executor/scale) → walk through the life of a task (parse → schedule → queue → run → state update) → trade-offs (Celery vs Kubernetes, DB load, parse time) → incident scenario (worker died, scheduler lag).

**Likely follow-ups:**
- *Celery vs Kubernetes executor?* → Warm static workers with low latency vs per-task pod isolation and elasticity with startup cost.
- *What if the scheduler dies?* → Running tasks continue; no new tasks are queued until a scheduler returns (or another instance takes over in HA).
- *Where is state stored?* → Metadata DB; workers are stateless.
- *Why is my DAG not showing up?* → Import error, wrong folder, or not yet parsed; check `list-import-errors`.

**Signals interviewers look for:** you describe the database as the coordination point; you know executors' trade-offs; you mention parse time and DB growth as operational concerns.

### 11. Check yourself
- [ ] Draw the five components and the arrows between them.
- [ ] Trace a task from file parse to success, naming each state.
- [ ] Choose an executor for three given scenarios and defend it.
- [ ] Name three config limits that can hold a task in `queued`.
- [ ] Explain what happens to tasks if a worker node is lost mid-run.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Meet Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 33
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow on AWS
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Securing Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context

