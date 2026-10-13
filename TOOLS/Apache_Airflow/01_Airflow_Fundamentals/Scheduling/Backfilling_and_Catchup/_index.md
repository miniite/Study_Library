---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals/Scheduling/Backfilling_and_Catchup"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Backfilling and Catchup

**L5 Expert** · [↑ Scheduling and Execution Dates](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Backfilling reprocesses historical intervals with the same DAG code — automatically via `catchup=True` or manually via `airflow dags backfill`/clearing runs — and only works safely when tasks are idempotent and keyed on the logical date.

### 1. The big picture
Pipelines are rarely born on day one of the data. You ship a DAG in March and need January's data; a bug is found and last week must be recomputed; a new column requires reprocessing a year. Airflow handles all three with the same mechanism: **run the DAG for past data intervals**. Because every run is parameterised by its interval (`{{ ds }}`, `data_interval_start/end`), the code that processes today is the code that reprocesses last January.

The two entry points are:
- **Catchup** — automatic: when a DAG is (un)paused or newly deployed, the scheduler creates runs for every missed interval since `start_date`.
- **Backfill / clear** — manual: you pick a range (or clear finished task instances) and Airflow re-executes them.

### 2. Core concepts and vocabulary
- **Catchup** — DAG argument; `True` creates all past runs, `False` only the latest interval.
- **Backfill** — manual run of a past date range (`airflow dags backfill`).
- **Clear** — resets task instances to `none`, so the scheduler re-runs them with the original logical date (UI "Clear" or `airflow tasks clear`).
- **Reprocessing** — recomputing data already produced.
- **Idempotent** — rerun produces the same final state.
- **`max_active_runs`** — limits concurrent runs during a backfill.
- **Logical date** — identifies which slice each run owns.
- **`depends_on_past`** — chain dependency across runs; matters for ordered backfills.
- **Rerun failed only** — `--rerun-failed-tasks`, `--reset-dagruns`.
- **DAG run types** — `scheduled`, `backfill`, `manual`, `dataset_triggered`.

### 3. How it works, step by step
**Catchup**
1. DAG has `start_date=2024-01-01`, daily, `catchup=True`, created on 2024-01-10.
2. Scheduler creates runs for Jan 1…Jan 9 immediately (intervals complete).
3. They run in order (Airflow schedules oldest first), limited by `max_active_runs` and pools.
4. After catching up, it continues normally.

**Backfill command**
1. `airflow dags backfill -s 2024-02-01 -e 2024-02-07 my_dag` creates (or reuses) runs for those intervals.
2. For each run, tasks execute; already-successful tasks are skipped unless `--reset-dagruns`/clear is used.
3. Result recorded as `backfill` run type; the UI shows them with a distinct marker.

**Clear-and-rerun**
1. In the grid view select a task/run → *Clear* (optionally with downstream).
2. State resets; the scheduler picks them up and re-runs with the same logical date.

### 4. Worked example
```python
with DAG(
    dag_id="daily_sales",
    start_date=datetime(2024, 1, 1),
    schedule_interval="@daily",
    catchup=True,
    max_active_runs=3,         # at most 3 intervals processed in parallel
) as dag:
    extract = BashOperator(
        task_id="extract",
        bash_command="python extract.py --day {{ ds }} --out /data/raw/{{ ds }}.parquet",
    )
    load = PostgresOperator(
        task_id="load",
        sql="""
        DELETE FROM fact_sales WHERE sale_date = '{{ ds }}';
        INSERT INTO fact_sales SELECT * FROM staging_sales WHERE sale_date = '{{ ds }}';
        """,
    )
    extract >> load
```
Reprocess a week after a bug fix:
```bash
airflow dags backfill daily_sales \
   --start-date 2024-03-01 --end-date 2024-03-07 \
   --reset-dagruns --rerun-failed-tasks
```
Or clear specific tasks:
```bash
airflow tasks clear daily_sales --start-date 2024-03-01 --end-date 2024-03-07 --task-regex "^load$" --yes
```
What to verify: row counts per `sale_date` do not change on a second run (idempotent), and the UI shows runs 03-01…03-07 with new start times but the same logical dates.

### 5. Under the hood   *(optional)*
- **Scheduler vs backfill job:** classic CLI backfill runs a `BackfillJob` in the CLI process (Airflow 2.x) and uses the DAG's executor; since 2.10/3.0 backfills are scheduled by the scheduler as first-class runs, but the semantic is the same.
- **Catchup logic:** when computing `next_dagrun`, the scheduler starts from the earliest un-run interval if catchup is on; if off, it fast-forwards to the most recent complete interval.
- **Ordering:** `depends_on_past=True` forces intervals to run strictly sequentially; otherwise runs are only throttled by concurrency limits and may complete out of order.
- **Task instance reset:** clearing sets `state=None`, increments `max_tries`, and the scheduler enqueues it again; XComs of cleared tasks are removed.
- **Template stability:** because rendering uses stored run metadata (data interval), reruns see the *original* interval, even weeks later.
- **Code vs data version:** a backfill runs *today's code* on old intervals; if the logic changed in between, outputs reflect new logic — an intentional (fix) or accidental (drift) choice.

### 6. Performance and design implications
- **Throttle:** a naive 365-day catchup can launch hundreds of parallel extractions; set `max_active_runs`, pool slots and source-side rate limits.
- **Partition-aligned writes:** write per-interval partitions so backfills affect only their slice and can run concurrently.
- **Source retention:** APIs may not serve old data; confirm availability before promising a backfill.
- **Compute cost:** historic reprocessing can multiply monthly cost; schedule off-peak and use spot/preemptible capacity.
- **Downstream cascade:** reprocessed partitions should trigger/clear dependent DAGs (marts, reports) — plan lineage.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `catchup=True/False` | Auto-run missed intervals | New DAG decisions |
| `airflow dags backfill -s -e` | Run range manually | History load/repair |
| `--reset-dagruns` | Re-run existing runs | Recompute after fix |
| `--rerun-failed-tasks` | Only failed tasks | Partial repair |
| `airflow tasks clear` | Reset tasks | Targeted rerun |
| UI → Clear (downstream/recursive) | Rerun with dependencies | Interactive repair |
| `max_active_runs`, `pool` | Throttle | Protect systems |
| `depends_on_past` | Sequential intervals | Stateful loads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `catchup=True` | You want history built automatically and sources support it |
| `catchup=False` + explicit backfill | You want control and no surprise floods |
| Delete-insert / partition overwrite | Target supports transactions or partitions |
| Append + dedupe later | Source is immutable and dedupe is cheap — otherwise risk duplicates |
| `depends_on_past=True` | Each load depends on previous state |
| Parallel backfill | Intervals are independent |

### 9. Common mistakes
- **Non-idempotent tasks** → duplicate/rolled-up data on rerun → overwrite partitions.
- **Using `now()`** → wrong slice during backfill → logical date.
- **Unthrottled catchup** → source overload, cluster saturation → `max_active_runs`.
- **Backfilling with changed schedule/start_date on same DAG id** → confusing history → new DAG id.
- **Forgetting downstream refresh** → stale reports → clear/trigger dependents.
- **Assuming old data still exists in source** → empty loads marked success → add row-count checks.

### 10. 🎯 Interview kit
**30-second answer:** "Catchup automatically creates runs for missed intervals; backfill and clearing let me re-run chosen ranges. Both work because tasks key on the logical date and are idempotent, and I throttle with max_active_runs and pools."

**Structure a longer answer:** clarify why reprocessing → mechanism (catchup, backfill, clear) → safeguards (idempotency, throttling, dependents) → scenario (bug fix for last 30 days).

**Likely follow-ups:**
- *How do you make backfills safe?* → Idempotent partition overwrites, logical-date parameters.
- *Backfill vs clear?* → Backfill creates/targets a date range via CLI; clear resets existing task instances.
- *What if old source data is gone?* → Archive raw data (immutable landing zone) for reproducibility.
- *Why is code version relevant?* → Backfill runs current code on old data.

**Signals interviewers look for:** idempotency and lineage thinking; throttling; raw-data retention.

### 11. Check yourself
- [ ] Explain what happens when a catchup DAG is deployed 30 days after its start_date.
- [ ] Write the CLI command to reprocess a week and only failed tasks.
- [ ] Rewrite an append load so backfills are safe.
- [ ] Choose catchup True/False for a daily snapshot DAG and justify.
- [ ] List what must be refreshed downstream after a backfill.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Triggering workflows
- *Designing Data-Intensive Applications* — Martin Kleppmann — Process Pauses
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Building custom components
- *Hadoop: The Definitive Guide* — Tom White — Colophon › link
- *Database Internals* — Alex Petrov — Introduction and Overview › Problem with retransmits

