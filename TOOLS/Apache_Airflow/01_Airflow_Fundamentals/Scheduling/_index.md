---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals/Scheduling"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Scheduling and Execution Dates

**L4 Advanced** · [↑ Airflow Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Airflow runs a DAG *after* each schedule interval ends, labels the run with the interval's start (the logical/execution date), and uses `start_date`, `schedule_interval`, `catchup` and backfills to decide which past intervals to execute.

### 1. The big picture
A pipeline that "runs daily" must answer three questions: *when* does it start, *which slice of data* does that run own, and *what happens to the days it missed* (because it was paused, newly deployed or down)? Airflow's answers are built around the **schedule interval**: time is cut into consecutive, non-overlapping intervals, and one DAG run is created per interval.

The part that surprises everyone: **a run is triggered at the *end* of its interval, not the beginning.** A `@daily` run for 1 January starts just after midnight on 2 January, because only then has the interval `[Jan 1, Jan 2)` finished and all its data can exist. The run carries the interval's *start* as its logical date. Understanding this single rule removes most of the confusion about "off by one day" runs.

### 2. Core concepts and vocabulary
- **start_date** — the first interval's *start* (not when the first run happens). The first run fires one interval after it.
- **schedule_interval** — how often to run: a preset (`@daily`, `@hourly`, `@weekly`, `@monthly`, `@once`, `None`), a cron string (`0 3 * * 1-5`), or a `timedelta` for frequency-based intervals.
- **Logical (execution) date** — the start timestamp of the interval a run is responsible for. Exposed as `{{ ds }}`, `{{ execution_date }}` (older) or `{{ logical_date }}` (newer).
- **Data interval** — `[data_interval_start, data_interval_end)`; in Airflow ≥ 2.2 these names state the idea directly.
- **end_date** — optional last date after which no runs are created.
- **catchup** — whether to create runs for *every* missed interval between `start_date` (or last run) and now. Default `True`.
- **Backfill** — manually (CLI) running a DAG over a chosen historic date range.
- **Cron expression** — five fields: minute, hour, day of month, month, day of week. Cron describes *points in time*, so irregular patterns are possible.
- **Frequency-based schedule** — `timedelta(days=3)`: every 3 days since `start_date`, which cron cannot express (cron's "day of month" resets).
- **Data-aware schedule (Airflow ≥ 2.4)** — a DAG can run when another DAG updates a *dataset*, instead of by clock.

### 3. How it works, step by step
Example: `start_date = 2024-01-01`, `schedule_interval="@daily"`, today is 2024-01-04 08:00.

1. The scheduler computes intervals: Jan 1→Jan 2, Jan 2→Jan 3, Jan 3→Jan 4. Jan 4→Jan 5 is not finished.
2. Jan 1→Jan 2 finished at midnight Jan 2, so its run starts then, with logical date 2024-01-01.
3. With `catchup=True` and a newly unpaused DAG, runs for Jan 1, 2 and 3 are all created immediately and executed in order (subject to `max_active_runs`).
4. With `catchup=False`, only the **latest** finished interval (Jan 3) is run; earlier ones are skipped.
5. Each run's tasks use the logical date to pick their slice: the run for 2024-01-01 reads data from Jan 1 and writes Jan 1's output — even though it physically runs on Jan 2 or later.
6. Re-running that old run later reproduces the same slice, because the task keys on the logical date, not on `now()`.

| Interval | Logical date (`ds`) | Earliest real start |
|---|---|---|
| Jan 1 00:00 → Jan 2 00:00 | 2024-01-01 | 2024-01-02 00:00 |
| Jan 2 00:00 → Jan 3 00:00 | 2024-01-02 | 2024-01-03 00:00 |
| Jan 3 00:00 → Jan 4 00:00 | 2024-01-03 | 2024-01-04 00:00 |

### 4. Worked example
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator

with DAG(
    dag_id="events_daily",
    start_date=datetime(2024, 1, 1),
    end_date=datetime(2024, 12, 31),
    schedule_interval="0 3 * * *",        # 03:00 every day
    catchup=True,
    max_active_runs=2,                    # protect the source system while catching up
) as dag:
    BashOperator(
        task_id="fetch",
        bash_command=(
            "curl -o /data/events/{{ ds }}.json "
            "'https://api.example.com/events?"
            "start={{ data_interval_start }}&end={{ data_interval_end }}'"
        ),
    )
```
Behaviour to predict:
- The first run is for logical date 2024-01-01 but fires on 2024-01-02 at **03:00** (end of the interval + cron offset — for a cron-style schedule the interval is "from this cron tick to the next").
- Unpausing on 2024-03-10 with `catchup=True` queues ~69 runs; `max_active_runs=2` throttles them.
- Re-running a failed day via `airflow tasks clear` or the UI reruns it with the original `ds`, producing the same file name → safe overwrite.

Backfill from the CLI:
```bash
airflow dags backfill events_daily --start-date 2024-02-01 --end-date 2024-02-07
```
Look in the UI's grid/Gantt view: one column per interval, state coloured per task.

### 5. Under the hood   *(optional)*
- **Why end-of-interval triggering?** Airflow was designed for ETL on batch data. Data for a period is only complete after the period ends; firing at the end guarantees the data is there. Hence "execution_date" = start of the period being processed.
- **The scheduler's DAG run creation** — a `DagModel` stores `next_dagrun` and `next_dagrun_data_interval_*`; the scheduler loop (`DagRun` creation in `SchedulerJob._create_dag_runs`) inserts a run when `now >= data_interval_end`, then sets the next pair.
- **Timetables (Airflow ≥ 2.2)** — pluggable classes that compute data intervals. Presets and cron are built-in timetables; custom ones allow "business days only" or "after the market closes".
- **Time zones** — Airflow stores timestamps in UTC. Passing a timezone-aware `start_date` (`pendulum.datetime(..., tz="Europe/Berlin")`) makes cron expressions follow local time, including DST shifts.
- **Frequency-based vs cron** — `timedelta` intervals are relative to the previous run, so there is no calendar alignment; cron aligns to wall-clock points.
- **Dataset-triggered runs** — logical date is the time of the triggering event; there is no regular interval.

### 6. Performance and design implications
- **Catchup floods.** A new DAG with an old `start_date` can create hundreds of runs; set `catchup=False`, or cap with `max_active_runs`, pools and per-task concurrency.
- **Bound dependencies on the logical date,** never `datetime.now()` — otherwise reruns and backfills pull today's data into yesterday's partition.
- **Idempotence is mandatory for backfills:** each run must overwrite its own partition rather than append.
- **Sub-minute precision** — Airflow is not a streaming engine; schedules below one minute are poor practice and add scheduler load.
- **Many DAGs on the same cron tick** (`0 0 * * *`) create thundering herds — stagger minutes or use pools.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `schedule_interval="@daily"` | Midnight-aligned daily intervals | Standard batch |
| `schedule_interval=timedelta(days=3)` | Every 3 days from start | Irregular spacing cron can't do |
| `catchup=False` | Skip past intervals | New DAGs, non-historical jobs |
| `max_active_runs` | Limit parallel DAG runs | Throttle backfills |
| `airflow dags backfill` | Run a date range | Reprocess history |
| `airflow dags pause/unpause` | Stop/start scheduling | Maintenance |
| `{{ ds }}`, `{{ data_interval_start }}` | Template the run's slice | All data-slice tasks |
| `depends_on_past=True` | Task runs only if its previous run succeeded | Ordered, state-carrying loads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `catchup=True` | You need history loaded automatically and tasks are idempotent | The source API cannot take 200 parallel historical pulls |
| `catchup=False` | You only care about latest state (snapshots, alerts) | You later need the skipped dates (use explicit backfill) |
| Cron strings | You need calendar semantics (weekdays, 1st of month) | You need "every N days" exactly |
| `timedelta` schedule | Fixed spacing between runs | Calendar alignment matters |
| `depends_on_past` | A load truly depends on the prior day's result | One failure should not block the whole chain |

### 9. Common mistakes
- **Expecting the run at the start of the day** → data for "today" is not complete, run is one interval late → remember the run fires after the interval ends; read the *previous* interval in your head.
- **Using `datetime.now()` or `date.today()` in tasks** → reruns fetch the wrong period → use `{{ ds }}` / `data_interval_start`.
- **Setting `start_date` to `datetime.now()`** → the first interval never completes → fixed date.
- **Forgetting `catchup` default is True** → surprise historical flood on first deploy → set it explicitly.
- **Changing the schedule of an existing DAG** → old runs no longer match the new intervals → new DAG id.
- **Mixing time zones** → runs shift by hours around DST → use timezone-aware dates and know that internal times are UTC.

### 10. 🎯 Interview kit
**30-second answer:** "Airflow splits time into intervals from `start_date` using the schedule, and triggers one DAG run after each interval ends. The run's logical date is the interval start, so tasks process exactly that slice and can be re-run or backfilled deterministically. `catchup` controls whether missed intervals are executed."

**Structure a longer answer:** clarify (batch interval semantics) → mechanism (start_date, interval end, logical date, catchup) → trade-offs (catchup floods, idempotence, cron vs timedelta) → scenario (daily load that must be re-run for last week).

**Likely follow-ups:**
- *Why did my DAG start a day late?* → It fires at interval end; the logical date is the interval start.
- *What's the difference between catchup and backfill?* → Catchup is automatic scheduling of missed intervals; backfill is a manual CLI/UI request for a chosen range (including already-run ones).
- *How do you avoid duplicate data on rerun?* → Idempotent tasks keyed by logical date (overwrite partition, upsert).
- *Cron vs timedelta?* → Cron aligns to the calendar; timedelta is relative spacing.

**Signals interviewers look for:** you state the end-of-interval rule unprompted; you tie logical date to idempotency; you mention max_active_runs/pools when catching up.

### 11. Check yourself
- [ ] For `start_date=2024-05-01`, `@weekly`, say the logical date and real start of the first run.
- [ ] Explain what `catchup=False` does when a DAG is paused for 10 days and unpaused.
- [ ] Rewrite a task using `datetime.now()` so it is safe to backfill.
- [ ] Choose cron or `timedelta` for "every 3 days" and justify it.
- [ ] Describe how you would reprocess the last 7 days without hammering the source.
<!-- explainer:end -->

## In this section (study in order)

1. [Backfilling and Catchup](<Backfilling_and_Catchup/_index.md>) (L5 Expert)

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Scheduling in Airflow
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Data Model
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Building custom components

