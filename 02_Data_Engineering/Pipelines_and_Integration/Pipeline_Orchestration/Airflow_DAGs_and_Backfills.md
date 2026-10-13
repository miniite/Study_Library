---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Airflow_DAGs_and_Backfills"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Airflow", "Apache_Spark", "Python"]
---

# Airflow DAGs and Backfills

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An Airflow DAG describes tasks and their dependencies; the scheduler turns it into one run per schedule interval, and a backfill simply asks Airflow to create and execute the runs for past intervals — which only works safely if each task is parameterized by its logical date and is idempotent.

### 1. The big picture
Apache Airflow is a workflow orchestrator: you define pipelines as Python code, and Airflow decides when each step runs, in what order, how often to retry, and what to do when something fails. It does not process big data itself; it coordinates other systems (Spark, warehouses, APIs, containers).

The central object is the **DAG** — a directed acyclic graph of tasks. "Directed" because edges mean "runs after", "acyclic" because loops would never finish. Scheduling in Airflow is *interval-based*: a DAG with a daily schedule produces one **DAG run** per day, each tied to a **data interval** (the slice of time it is responsible for). That design is what makes backfills natural: history is just more intervals that have not run yet.

### 2. Core concepts and vocabulary
- **DAG** — the workflow definition: tasks + dependencies + schedule + defaults.
- **Task** — a node in a DAG; one unit of work.
- **Operator** — the template class that defines what a task does (`BashOperator`, `PythonOperator`, `SQLExecuteQueryOperator`, a Spark submit operator…). A *task* is an instantiated operator in a DAG.
- **DAG run** — one execution of the DAG for a specific interval.
- **Task instance** — one task within one DAG run, with a state (`queued`, `running`, `success`, `failed`, `up_for_retry`, `skipped`…).
- **Scheduler** — the component that parses DAG files, creates runs when intervals complete, and queues ready tasks.
- **Executor** — how queued tasks actually get run (local processes, Celery workers, Kubernetes pods). Scheduler decides *what/when*; executor decides *where/how*.
- **Schedule / `schedule_interval`** — cron expression, preset (`@daily`), timedelta or dataset trigger.
- **`start_date`** — the first logical date the DAG is eligible; the first run happens after that first interval *ends*.
- **Logical (execution) date / data interval** — the start (and end) of the period the run covers; used to parameterize tasks.
- **`catchup`** — whether the scheduler automatically creates runs for all missed intervals since `start_date`.
- **Backfill** — creating and running DAG runs for a historical date range on demand.

### 3. How it works, step by step
1. **Define the DAG file** in the DAGs folder. The scheduler re-parses it regularly, so top-level code must be light.
2. **Scheduler evaluates schedules.** For each active DAG, when `start_date + n × interval` has completed, it creates a DAG run for that data interval. (A daily DAG for 2026-10-11 runs shortly after midnight on 2026-10-12 — the interval must close first.)
3. **Dependency resolution.** Task instances whose upstream tasks succeeded (per trigger rule) are scheduled and sent to the executor.
4. **Execution.** A worker runs the operator's `execute`; logs stream to the UI; on exception the task retries (`retries`, `retry_delay`) or fails.
5. **State and propagation.** Failures mark downstream tasks `upstream_failed` (default trigger rule); the DAG run ends `success` or `failed`.
6. **Backfill / catch-up.** If `catchup=True` and the DAG was paused or new, the scheduler creates all missed runs. For explicit history, you run `airflow dags backfill -s <start> -e <end> dag_id` (or the UI/API equivalent) or *clear* past task instances to re-run them.
7. **Parameterization.** Each task uses templated values such as `{{ ds }}` or `{{ data_interval_start }}` to read/write only its slice.

### 4. Worked example
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="daily_sales",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,                 # don't auto-run 9 months of history
    max_active_runs=2,             # bound parallel backfills
    default_args={"retries": 2, "retry_delay": timedelta(minutes=5)},
) as dag:
    extract = BashOperator(
        task_id="extract",
        bash_command="python extract.py --date {{ ds }}",
    )
    load = BashOperator(
        task_id="load",
        bash_command="python load.py --date {{ ds }} --mode overwrite-partition",
    )
    extract >> load
```
Backfill a quarter:
```bash
airflow dags backfill -s 2026-07-01 -e 2026-09-30 daily_sales
```
What to look at: the Grid view shows one column per run; each run passes its own `ds`, so `load` overwrites only that day's partition. `max_active_runs=2` prevents the backfill from flooding the cluster. If `load` were an append, rerunning a day would duplicate it — the DAG design, not Airflow, guarantees safety.

### 5. Under the hood   *(optional at L4)*
- **Metadata database** stores DAG runs, task instances and states; the scheduler loop reads and writes it constantly. It is the source of truth and a scaling bottleneck for very large installations.
- **DAG parsing** executes your file repeatedly; slow imports or API calls at top level make the scheduler lag. Keep work inside tasks.
- **Why the lag of one interval:** Airflow treats a run as responsible for data that *has fully arrived* by the end of the interval, which is why runs are stamped with the interval start.
- **Trigger rules** (`all_success`, `all_done`, `one_failed`, …) let cleanup or alert tasks run regardless of upstream outcomes.
- **Pools, priority weights and concurrency settings** (`max_active_tasks`, `parallelism`) throttle load.

### 6. Performance and design implications
- Many tiny tasks or a huge DAG (hundreds of tasks) strain the scheduler; use task groups, dynamic task mapping carefully, and keep per-task overhead in mind.
- Backfills compete with production runs; cap with `max_active_runs`, pools, or run them off-peak.
- Make tasks retry-safe: the same logical date can be executed multiple times.
- Keep heavy transformation in external engines; Airflow tasks should submit and monitor.
- Monitor: scheduler heartbeat, DAG parse time, queued-task age, SLA misses.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow dags backfill -s -e dag_id` | Run historical intervals | Fill gaps or new DAG history |
| `airflow tasks clear` / UI "Clear" | Reset task instances so they re-run | Re-run failed or changed periods |
| `catchup` | Auto-create missed runs | `True` only if history should run automatically |
| `max_active_runs` | Parallel DAG runs cap | Protect systems during backfill |
| `retries`, `retry_delay` | Auto-retry on failure | Transient errors |
| `{{ ds }}`, `data_interval_start/end` | Templated date values | Parameterize tasks |
| `airflow dags trigger` | Manual run | Ad-hoc or out-of-cycle |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `catchup=True` — new DAG must populate history by itself | Long history plus heavy tasks would swamp the cluster |
| Explicit backfill command — controlled range, one time | Teams forget to bound concurrency |
| Clear + re-run — a few bad runs | Many runs; scripted backfill is clearer |
| Schedule-driven — data arrives predictably | Arrival is variable; use sensors or dataset triggers |

### 9. Common mistakes
- Using `datetime.now()` inside tasks → backfills read today's data → use the logical date.
- `start_date` set dynamically or in the future → DAG never starts or behaves unpredictably → use a fixed past date.
- Non-idempotent loads → duplicates on retry/backfill → overwrite partition or merge.
- Heavy code at DAG top level → slow scheduler → move work into tasks.
- Forgetting `catchup` default → hundreds of runs fire unexpectedly → set it explicitly.
- Confusing scheduler and executor → mis-diagnosing "not running" issues → scheduler queues, executor runs.

### 10. 🎯 Interview kit
**30-second answer:** A DAG defines tasks and dependencies; the scheduler creates one DAG run per schedule interval, tied to a logical date, and the executor runs the tasks. A backfill creates runs for past intervals; it is safe because tasks are parameterized by the logical date and write idempotently.

**Structure a longer answer:** clarify → DAG/task/operator → scheduler vs executor → interval semantics and logical date → backfill/catchup/clear → safeguards (concurrency, idempotency).

**Likely follow-ups:**
- *Operator vs task?* → operator is a class/template; task is its instance in a DAG.
- *What happens with a future `start_date`?* → nothing runs until it arrives.
- *How do you backfill without overloading?* → `max_active_runs`, pools, off-peak.
- *DAG with 400 tasks lags — why?* → scheduler parse/queue load; simplify, group, tune scheduler settings.

**Signals interviewers look for:** interval semantics, idempotency, throttling, and knowing Airflow orchestrates rather than processes.

### 11. Check yourself
- [ ] Explain why a daily run for date D executes after D ends.
- [ ] Write a task command that is safe to backfill.
- [ ] Say what `catchup=True` does to a DAG created three months ago.
- [ ] Describe the difference between clearing a task and running a backfill.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is Apache Airflow and what problem does it solve in orchestrating data workflows?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13344_What_is_Apache_Airflow_and_what_problem_does_it_solve_in_orc.md>)
- [Basic] [What is the Airflow web UI used for and what can you see about a DAG there?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13372_What_is_the_Airflow_web_UI_used_for_and_what_can_you_see_abo.md>)
- [Basic] [What is the difference between an Airflow Hook and an Airflow Operator?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13562_What_is_the_difference_between_an_Airflow_Hook_and_an_Airflo.md>)
- [Intermediate] [What is the difference between Airflow's LocalExecutor and SequentialExecutor for a beginner setup?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13428_What_is_the_difference_between_Airflow_s_LocalExecutor_and_S.md>)
- [Intermediate] [Why should credentials for a database live in an Airflow Connection rather than being typed into the DAG code?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13571_Why_should_credentials_for_a_database_live_in_an_Airflow_Con.md>)
- [Intermediate] [What's your strategy for testing an Airflow DAG locally and in CI without spinning up the full data platform?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13170_What_s_your_strategy_for_testing_an_Airflow_DAG_locally_and_.md>)
- [Deep] [How do you prevent a runaway backfill of anomaly baselines from starving the daily alert-enrichment DAG in a shared Airflow and Spark cluster?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13478_How_do_you_prevent_a_runaway_backfill_of_anomaly_baselines_f.md>)
- [Deep] [How do you decide the boundary between what Airflow orchestrates and what logic lives inside the Spark job to keep DAGs thin and reruns clean?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13740_How_do_you_decide_the_boundary_between_what_Airflow_orchestr.md>)

## Related concepts

- [DAG Design and Dependencies](<../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>) *(Airflow Core)*
- [Idempotent Airflow Tasks](<../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>) *(Python Language)*
- [Scheduling Retries and Trigger Rules](<../../../TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules.md>) *(Airflow Core)*
- [Airflow Architecture](<../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>) *(Airflow Core)*

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Meet Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Anatomy of an Airflow DAG
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Scheduling in Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Airflow Database
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Scheduler
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › An ELT Pipeline DAG
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Additional Pipeline Tasks
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Prioritizing critical jobs

