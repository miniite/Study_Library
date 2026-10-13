---
id: "TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules"
level: 4
type: "concept"
topic: "TOOLS/Apache_Airflow/Airflow_Core"
tools: ["Apache_Airflow"]
---

# Scheduling Retries and Trigger Rules

**L4 Advanced** · [↑ Airflow Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Airflow runs a DAG *after* each schedule interval ends, labels every run with the interval it covers, retries failed tasks per configurable policy, and lets each task decide — via its trigger rule — which combinations of upstream outcomes allow it to start.

### 1. The big picture
Three mechanisms control "when and whether does this task run?":
1. **Schedule** — which DAG runs exist (cron/preset/timetable, start date, catchup).
2. **Retries and limits** — what happens when a task instance fails or runs too long (retries, delay, timeout, SLA).
3. **Trigger rules** — given the *final states of upstream tasks* in this run, may this task start?

Most real-world confusion ("why did it run yesterday's date?", "why did my cleanup not run?", "why did it backfill 400 runs?") comes from misunderstanding one of these three, usually the **data interval** semantics.

### 2. Core concepts and vocabulary
- **`start_date`** — the beginning of the first data interval; it is *not* "when the DAG first runs".
- **`schedule` / `schedule_interval`** — a cron string (`"0 0 * * *"`), a preset (`@daily`, `@hourly`), a `timedelta`, a dataset list, or `None` (never scheduled; only manual/API/trigger runs).
- **Data interval** — the time window a run is responsible for: `data_interval_start` → `data_interval_end`.
- **Logical date** (formerly `execution_date`) — in the classic timetables, equal to `data_interval_start`. It is a *label*, not the wall-clock time the run happened.
- **Catchup** — when `True`, the scheduler creates a run for every missed interval since `start_date`; when `False`, only the latest.
- **Retries / `retry_delay` / exponential backoff** — automatic re-attempts of a failed task instance.
- **Task states** — `none`, `scheduled`, `queued`, `running`, `success`, `failed`, `up_for_retry`, `up_for_reschedule`, `skipped`, `upstream_failed`.
- **Trigger rule** — task-level setting that evaluates upstream states.
- **SLA** — a time a task is expected to finish by (relative to the schedule); a miss is reported, not enforced.
- **`execution_timeout`** — hard limit after which the task instance is failed.
- **Sensor modes** — `poke` (holds a worker slot while waiting) vs `reschedule` (releases it between checks).
- **Clear vs Mark** — *clearing* a task instance resets it so the scheduler re-runs it; *mark success/failed* just rewrites the state without running anything.

### 3. How it works, step by step
**Scheduling**
1. The scheduler sees the DAG's schedule and `start_date`; the first interval is `[start_date, start_date + interval)`.
2. A run for an interval is created **only after the interval has ended** — a `@daily` run labelled `2026-03-01` starts shortly after midnight on `2026-03-02`, because only then is the whole day's data complete.
3. The run's task instances are created and gated by dependencies, trigger rules and concurrency limits.
4. On first enabling a DAG with `catchup=True`, runs are created for all past intervals (limited by `max_active_runs`).

**Retries**
1. Task code raises (or the process dies, or the heartbeat stops).
2. If `try_number <= retries`, state becomes `up_for_retry`; after `retry_delay` (growing if `retry_exponential_backoff=True`, capped by `max_retry_delay`) it is re-queued.
3. After the last attempt the state is `failed`; `on_failure_callback` fires; downstream tasks become `upstream_failed` (under the default rule).
4. A *manual re-run* (Clear) starts a fresh attempt regardless of retries and is chosen by a human; automatic retries are for transient problems.

**Trigger rules**
1. When all upstream task instances are in a terminal state, the scheduler evaluates the downstream task's `trigger_rule`.
2. If satisfied the task is scheduled; if it can never be satisfied it is set to `skipped` or `upstream_failed`.

| Rule | Runs when… | Typical use |
|---|---|---|
| `all_success` (default) | every upstream succeeded | normal chains |
| `all_failed` | every upstream failed | failure branch |
| `all_done` | every upstream finished, any outcome | cleanup, notifications |
| `one_success` / `one_failed` | at least one upstream succeeded / failed (fires early) | fastest-wins, fast alerting |
| `none_failed` | no upstream failed or `upstream_failed` (skips OK) | after branches |
| `none_failed_min_one_success` | no failures and at least one success | join after `BranchPythonOperator` |
| `none_skipped` | no upstream skipped | rarely |
| `always` | regardless of state | special cases |

### 4. Worked example
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.empty import EmptyOperator
from airflow.utils.trigger_rule import TriggerRule

with DAG(
    "interval_demo",
    start_date=datetime(2026, 3, 1),
    schedule="0 2 * * *",
    catchup=False,
    max_active_runs=1,
    default_args={
        "retries": 3,
        "retry_delay": timedelta(minutes=2),
        "retry_exponential_backoff": True,
        "execution_timeout": timedelta(hours=1),
        "sla": timedelta(hours=3),
    },
) as dag:
    def load(data_interval_start, data_interval_end, **_):
        print(f"processing [{data_interval_start}, {data_interval_end})")

    extract = PythonOperator(task_id="extract", python_callable=load)
    transform = PythonOperator(task_id="transform", python_callable=load)
    cleanup = EmptyOperator(task_id="cleanup", trigger_rule=TriggerRule.ALL_DONE)
    extract >> transform >> cleanup
```
Reading it: the run that fires at 02:00 on 3 March covers `[2 March 02:00, 3 March 02:00)`, so use `data_interval_start/end` — *not* `now()` — to select data; mixing logical date with "yesterday" arithmetic is the classic off-by-one-day bug. If `transform` fails after 3 retries, `cleanup` still runs (`all_done`), while the DAG run is marked failed because a non-cleanup task failed. (If cleanup is the only leaf task and you want the run to be judged by upstream results, give it care — a successful leaf can mask upstream failures in some designs; the run state is determined by leaf tasks.)

### 5. Under the hood
- **Interval semantics** come from *timetables*. The cron timetable computes `data_interval_end` as the next cron tick after start; the run is created when `now >= data_interval_end`. `logical_date` in the cron case equals `data_interval_start`; with custom timetables it may differ, which is why `data_interval_*` are the reliable variables.
- The scheduler marks `up_for_reschedule` for reschedule-mode sensors: the task instance leaves the worker, and a `task_reschedule` row records the next check time.
- **Zombie / orphaned tasks** (worker died, no heartbeat) are detected by the scheduler and failed so retry logic can apply.
- **Skips propagate**: a `skipped` upstream makes an `all_success` downstream `skipped` too — the reason a join after a branch must use `none_failed_min_one_success` rather than the default.
- **SLA** is evaluated by the scheduler relative to the schedule interval end, and a miss triggers `sla_miss_callback` (signature `dag, task_list, blocking_task_list, slas, blocking_tis`); it does not stop the task or block downstream tasks. Newer Airflow versions replace this with deadline alerts, so check your version.
- **Sensor deadlock**: with `poke` mode, N waiting sensors hold N slots; if the DAG/global limit is reached, the task they are waiting on cannot start, so the system stalls. `reschedule` or deferrable operators fix this.

### 6. Performance and design implications
- **Idempotent tasks** make retries and backfills safe; non-idempotent ones turn retries into duplicates.
- Retry settings should reflect failure type: network flake → few retries with backoff; deterministic data/code error → low or zero retries, fail fast.
- Late data (lands 6 h after the window): don't rerun the day blindly; add a sensor on the actual landing (reschedule/deferrable), partition-aware loads, and a later "reprocess last N days" task or a dataset-triggered run.
- For a "must finish by 06:00" target with source landing 02:00–05:00: schedule early, gate with a reschedule-mode sensor with a timeout, put an SLA/deadline alert before 06:00, and keep downstream steps fast.
- Cap `max_active_runs` for DAGs that backfill to avoid overwhelming sources; use pools for shared systems.
- Monitor: queued duration, retry counts, SLA misses, scheduler heartbeat, task duration trend versus `execution_timeout`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `schedule`, `start_date`, `end_date` | defines runs | every DAG |
| `catchup` | create or skip past runs | set explicitly |
| `retries`, `retry_delay`, `retry_exponential_backoff`, `max_retry_delay` | retry policy | transient failures |
| `execution_timeout` | kill long task | stuck jobs |
| `sla`, `sla_miss_callback` | report lateness | freshness commitments |
| `on_failure_callback` / `on_retry_callback` | alert hooks | Slack/email/pager |
| `trigger_rule` | upstream-state gate | cleanup, joins |
| `depends_on_past`, `wait_for_downstream` | serialise runs | stateful pipelines |
| Sensor `mode="reschedule"` / deferrable | free worker slot while waiting | long waits |
| `airflow dags backfill`, UI Clear | re-run intervals | fixes, late data |
| `airflow dags pause/unpause`, `airflow dags trigger` | control runs | operations |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `catchup=True` when each interval must be processed individually and tasks are idempotent | you only need current state (it floods the system on enable) |
| `all_done` for cleanup that must always run | cleanup depends on upstream outputs being good; upstream stuck states can delay it |
| `none_failed_min_one_success` to join branches | you really require all branches to succeed |
| Automatic retries for transient network/API errors | failures are deterministic; retries only delay the alert |
| `schedule=None` for externally triggered DAGs | you assumed it means "never runs" — it still runs when triggered |
| SLA alerts for freshness visibility | you need enforcement — use `execution_timeout` or deadlines |

### 9. Common mistakes
- Expecting the run labelled today to start today → it starts after the interval ends → plan for the one-interval lag; use `data_interval_*`.
- Using `now()` / logical date arithmetic → off-by-one-day windows, non-reproducible backfills → template with `{{ data_interval_start }}` / `{{ ds }}` consistently.
- Leaving `catchup=True` by accident → hundreds of runs when enabled → set explicitly and cap `max_active_runs`.
- Joining after a branch with default `all_success` → join skipped → use `none_failed_min_one_success`.
- Cleanup with `all_success` → never runs after failure, leaving temp resources → `all_done` (and make it idempotent).
- Confusing "Mark success" with "Clear" → task not actually re-run → clear to re-execute, mark only to override state.
- Poke-mode sensors in bulk → slot exhaustion/deadlock → `reschedule` or deferrable.

### 10. 🎯 Interview kit
**30-second answer:** Airflow creates a DAG run once its schedule interval has ended, labels it with the interval's start (logical date) and exposes `data_interval_start/end`; tasks retry per `retries`/`retry_delay`, and each task's trigger rule decides which upstream outcomes let it start — default `all_success`, `all_done` for cleanup, `none_failed_min_one_success` after branches.

**Structure a longer answer:** clarify the requirement (frequency, lateness, failure tolerance) → explain interval semantics → retry and timeout design → trigger rules for branching/cleanup → close with a concrete scenario such as the 6 a.m. deadline with a variable landing time.

**Likely follow-ups:**
- *logical_date vs data_interval_end?* → logical date is the label (start of window in cron schedules); the window is [start, end); use the interval variables to avoid off-by-one-day.
- *What does `schedule=None` mean?* → no automatic runs; manual/API/trigger only.
- *What happens on SLA miss?* → callback/UI flag; task continues; nothing is blocked.
- *Retry vs manual rerun?* → automatic and policy-driven vs human-initiated Clear; both need idempotent tasks.
- *Cleanup must always run — which rule and gotcha?* → `all_done`; it also runs if upstream was skipped, and the run's final state is derived from leaf tasks.

**Signals interviewers look for:** correct interval mental model, idempotence, fail-fast vs retry judgement, and knowing sensor-slot consequences.

### 11. Check yourself
- [ ] For `@daily` with `start_date=2026-03-01`, state when the first run starts and what its data interval is.
- [ ] Explain the off-by-one-day bug and show the corrected template.
- [ ] Choose trigger rules for a branch-join, a cleanup, and an alert-on-any-failure task.
- [ ] Explain sensor deadlock and two ways to avoid it.
- [ ] Design the schedule/sensor/SLA for a source landing unpredictably between 02:00 and 05:00 with a 06:00 deadline.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between a task retrying automatically and manually re-running it?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02026_What_is_the_difference_between_a_task_retrying_automatically.md>)
- [Basic] [What does setting schedule_interval to None mean for a DAG?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02234_What_does_setting_schedule_interval_to_None_mean_for_a_DAG.md>)
- [Intermediate] [What is a trigger rule in Airflow and what is the default trigger rule?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02179_What_is_a_trigger_rule_in_Airflow_and_what_is_the_default_tr.md>)
- [Intermediate] [A task succeeded but wrote to the wrong partition because the operator built the path from the wrong template var — how do you make templated paths robust and catch this in review?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02109_A_task_succeeded_but_wrote_to_the_wrong_partition_because_th.md>)
- [Deep] [What is the difference between the logical date and data interval start and end in modern Airflow?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02042_What_is_the_difference_between_the_logical_date_and_data_int.md>)
- [Deep] [A daily DAG occasionally double-loads data into the warehouse — how do you make the load task idempotent and what patterns (delete-insert MERGE partition overwrite) would you choose for a partitioned table?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02049_A_daily_DAG_occasionally_double-loads_data_into_the_warehous.md>)
- [Deep] [A stakeholder wants to re-run last month's pipeline with a bug-fixed transform without touching current runs — how do you safely clear and backfill only the affected date range?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02087_A_stakeholder_wants_to_re-run_last_month_s_pipeline_with_a_b.md>)
- [Deep] [Your nightly pipeline cost on MWAA/Composer keeps climbing — how do you attribute cost to DAGs and cut it (right-size workers spot scale-to-zero off-hours)?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02127_Your_nightly_pipeline_cost_on_MWAA_Composer_keeps_climbing_h.md>)

## Related concepts

- [Airflow DAGs and Backfills](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Airflow_DAGs_and_Backfills.md>) *(Pipeline Orchestration)*
- [Idempotent Airflow Tasks](<../../Python/Python_Language/Idempotent_Airflow_Tasks.md>) *(Python Language)*
- [Handling Non-critical Task Failures](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>) *(Pipeline Orchestration)*

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Meet Apache Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Scheduling in Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Triggering workflows
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running tasks in containers
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Scheduler SLAs
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Building Airflow DAGs
- *Data Pipelines Pocket Reference* — James Densmore — Measuring and Monitoring Pipeline Performance › close the cursor and commit the transaction
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Solution
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Introduction to Azure Data Factory › Pipeline Triggers
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Basics of CICD and Pipeline Scheduling › Pipeline Scheduling

