---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Airflow Fundamentals

**L3 Practitioner** · [↑ Apache Airflow](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Airflow fundamentals are one idea seen from five angles — a DAG of tasks (anatomy), wired by dependencies and branching, run on a time-interval schedule, parameterised by templating, and executed by a scheduler/executor/worker/metadata-DB architecture.

### 1. The big picture
Apache Airflow is a workflow *orchestrator*: it does not move or transform data itself, it decides **what runs, in what order, when, and what to do on failure**, and records everything. You describe a workflow as Python code (a DAG); Airflow turns each schedule interval into a run, each task into a tracked task instance, and shows the state in a UI.

The five sibling lessons form a ladder:

| Lesson | Question it answers |
|---|---|
| **DAG Anatomy** | What is a DAG, an operator, a task? How do I write one? |
| **Dependencies and Branching** | How do tasks depend on each other, and how do I run paths conditionally? |
| **Scheduling** (+ Backfilling and Catchup) | When does a run happen and which data interval does it own? |
| **Templating** | How does the same code process a different slice each run? |
| **Airflow Architecture** | Which processes make this happen and where is state kept? |

### 2. Core concepts and vocabulary
- **DAG / task / operator / task instance / DAG run** — structure and execution units.
- **Logical (execution) date & data interval** — the slice a run is responsible for.
- **Catchup / backfill** — executing missed or historical intervals.
- **Template / context** — Jinja placeholders rendered per run.
- **Trigger rule** — policy deciding when a task may start.
- **Scheduler / executor / worker / webserver / metadata DB** — runtime components.
- **XCom** — small messages between tasks.
- **Idempotency** — the property that makes retries and backfills safe.

### 3. How it works, step by step
1. You write `dags/my_dag.py` declaring tasks and edges.
2. The scheduler parses it and stores a serialized DAG.
3. After each interval completes, the scheduler creates a DAG run with the interval as its logical date.
4. Task instances become eligible according to dependencies and trigger rules.
5. The executor dispatches them to workers; each task renders its templates with the run context and executes.
6. States are written to the metadata DB; the UI reads them; failures retry, alert or skip downstream.

### 4. Worked example
```python
with DAG("daily_orders", start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False) as dag:
    extract   = BashOperator(task_id="extract",
                             bash_command="python extract.py --day {{ ds }}")      # templating
    validate  = ShortCircuitOperator(task_id="validate", python_callable=rows_ok)  # branching/conditional
    load      = PythonOperator(task_id="load", python_callable=load_partition)     # idempotent partition write
    notify    = EmptyOperator(task_id="notify", trigger_rule="all_done")
    extract >> validate >> load >> notify                                          # dependencies
```
Each concept appears once: schedule + logical date (`@daily`, `{{ ds }}`), dependencies (`>>`), a conditional gate, an idempotent load and a trigger rule for the final step. The architecture is what executes it.

### 5. Under the hood   *(optional)*
- The **database** is the coordination point: scheduler writes intents, workers write results, webserver reads.
- **Parse time vs run time:** DAG files are imported continuously (parse) and task callables executed on workers (run). Many bugs are parse/run confusion.
- **At-least-once execution:** retries, clears and backfills re-run tasks, so *idempotency* is the contract between you and Airflow.
- **Interval semantics:** runs fire at interval end and are labelled with interval start.

### 6. Performance and design implications
- Keep DAG files light; keep tasks small, idempotent and keyed on the logical date.
- Use pools/concurrency to protect downstream systems.
- Push heavy compute to engines built for it (Spark, warehouse); Airflow orchestrates.
- Budget for the metadata DB — it is the shared bottleneck.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow dags list` / `list-import-errors` | Show parsed DAGs / failures | Missing DAG |
| `airflow tasks test` / `dags test` | Run without state | Development |
| `airflow dags backfill` | Reprocess a range | History/repair |
| `schedule_interval`, `start_date`, `catchup` | Define cadence | Every DAG |
| `trigger_rule`, `retries`, `pool` | Control execution | Robustness |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Airflow for batch orchestration with dependencies, retries, history | Sub-second streaming or event-per-record processing |
| Many small tasks | Overhead of thousands of tiny tasks per run |
| Python-defined DAGs | Non-technical users need drag-and-drop |
| Time-based schedules | Data arrival is event-driven (sensors/datasets) |

### 9. Common mistakes
- Top-level work in DAG files → scheduler lag → move into callables.
- `now()` instead of the logical date → irreproducible → templating.
- Non-idempotent loads → duplicates → partition overwrite/merge.
- Misreading interval semantics → off-by-one data → remember "run after interval end".
- Default `all_success` after a branch → skipped joins → `none_failed*`.

### 10. 🎯 Interview kit
**30-second answer:** "Airflow is a Python-defined orchestrator: DAGs of tasks with dependencies, scheduled by intervals with a logical date, parameterised via templating, and executed by scheduler, executor and workers with state in a metadata database."

**Structure a longer answer:** clarify batch orchestration vs processing → DAG model → scheduling semantics → templating/idempotency → architecture → trade-offs.

**Likely follow-ups:** *Why does my run start after the interval?* → interval-end triggering. *How are retries safe?* → idempotency. *What is stored in the metadata DB?* → runs, task instances, XComs, connections, variables.

**Signals interviewers look for:** mental model of parse vs run, logical date, idempotency, architecture components.

### 11. Check yourself
- [ ] Draw the DAG-run lifecycle from file to success.
- [ ] Explain logical date and why backfills depend on it.
- [ ] Name the five architecture components and their responsibilities.
- [ ] Identify two anti-patterns in a sample DAG file.
- [ ] Choose a trigger rule for cleanup and for a join after a branch.
<!-- explainer:end -->

## In this section (study in order)

1. [Airflow Architecture](<Airflow_Architecture/_index.md>) (L4 Advanced)
2. [Anatomy of a DAG](<DAG_Anatomy/_index.md>) (L4 Advanced)
3. [Scheduling and Execution Dates](<Scheduling/_index.md>) (L4 Advanced)
4. [Templating and the Task Context](<Templating/_index.md>) (L4 Advanced)
5. [Dependencies Branching and Triggers](<Dependencies_and_Branching/_index.md>) (L4 Advanced)

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Meet Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Operating Airflow in production
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Scheduling in Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Anatomy of an Airflow DAG
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Beyond the basics
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — preface
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Orchestration
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Communicating with external systems
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Testing

