---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Triggering_and_Sensors"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Triggering Sensors and External Systems

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Besides clock schedules, Airflow DAGs can be started by *events* — sensors that wait for a condition, other DAGs, API/CLI calls and dataset updates — and each choice trades latency, resource use and coupling.

### 1. The big picture
A time schedule assumes data arrives on time. In practice a file lands "sometime after 2 a.m.", a partner API finishes whenever it finishes, or one team's DAG must feed another's. Airflow supports event-driven starts in several ways:

- **Sensors** — tasks that poll for a condition (file exists, partition added, another task finished) and succeed when it becomes true.
- **Triggering from outside** — REST API / CLI / UI (`airflow dags trigger`, "Trigger DAG w/ config").
- **DAG-to-DAG** — `TriggerDagRunOperator` (push) and `ExternalTaskSensor` (pull).
- **Datasets** (Airflow ≥ 2.4) — a producer task declares an `outlet`; consumer DAGs with that dataset in `schedule=[...]` start automatically.
- **Deferrable operators** — wait using the lightweight triggerer process instead of a worker slot.

### 2. Core concepts and vocabulary
- **Sensor** — operator subclass whose `poke()` returns True/False; it re-checks until success or timeout.
- **`poke_interval`** — seconds between checks.
- **`timeout`** — total seconds to wait before failing the sensor.
- **`mode`** — `poke` (hold the worker slot) or `reschedule` (release the slot between checks).
- **`soft_fail`** — on timeout mark *skipped* instead of failed.
- **Deferrable operator / trigger** — operator that suspends itself and lets an async trigger on the triggerer notify it.
- **`TriggerDagRunOperator`** — starts another DAG, optionally with `conf`.
- **`ExternalTaskSensor`** — waits for a task/DAG run in another DAG for the *same logical date* (or a mapped offset).
- **`conf`** — JSON passed when triggering, read as `dag_run.conf` / `{{ dag_run.conf }}`.
- **Dataset** — URI-named logical data object; updated when a task with `outlets=[Dataset(...)]` succeeds.
- **Manual / external trigger** — run with `run_type=manual`; has a logical date of the trigger moment, not an interval.

### 3. How it works, step by step
**Sensor flow (poke mode):**
1. The task starts and occupies a worker slot.
2. `poke()` runs; if false, sleep `poke_interval`, repeat.
3. When true → success; when `timeout` expires → fail (or skip with `soft_fail`).

**Reschedule mode:** after a false poke the task goes to `up_for_reschedule` and frees the slot; the scheduler re-queues it at the next interval.

**Deferrable:** the operator calls `defer(trigger=..., method_name=...)`; the task leaves the worker; the triggerer's asyncio loop watches the condition for thousands of tasks; on event, the task resumes at `method_name`.

**Dataset flow:** producer succeeds → Airflow records a dataset event → scheduler creates a run of every DAG whose dataset schedule is satisfied.

### 4. Worked example
```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.sensors.filesystem import FileSensor
from airflow.operators.python import PythonOperator
from airflow.operators.trigger_dagrun import TriggerDagRunOperator
from airflow.sensors.external_task import ExternalTaskSensor
from airflow import Dataset

orders_ds = Dataset("s3://lake/curated/orders/")

# 1. file-driven ingest, slot-friendly sensor
with DAG("ingest_orders", start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False):
    wait = FileSensor(
        task_id="wait_for_file",
        filepath="/landing/orders_{{ ds_nodash }}.csv",
        poke_interval=300,              # every 5 min
        timeout=6 * 3600,               # give up after 6 h
        mode="reschedule",              # don't hog a slot
        soft_fail=True,
    )
    load = PythonOperator(task_id="load", python_callable=lambda: None, outlets=[orders_ds])
    trigger_report = TriggerDagRunOperator(
        task_id="trigger_report",
        trigger_dag_id="build_report",
        conf={"source_date": "{{ ds }}"},
    )
    wait >> load >> trigger_report

# 2. downstream DAG scheduled by dataset update instead of the clock
with DAG("orders_marts", start_date=datetime(2024, 1, 1), schedule=[orders_ds], catchup=False):
    PythonOperator(task_id="build_mart", python_callable=lambda: None)

# 3. pull-style coupling to another DAG's task for the same logical date
with DAG("sla_check", start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False):
    ExternalTaskSensor(task_id="wait_for_ingest", external_dag_id="ingest_orders",
                       external_task_id="load", mode="reschedule")
```
Manual trigger with configuration from the CLI:
```bash
airflow dags trigger build_report --conf '{"source_date": "2024-05-01"}'
```
Observe: UI shows sensors as *up_for_reschedule* (light turquoise) between pokes; the dataset page lists producer/consumer links.

### 5. Under the hood   *(optional)*
- **Poke mode cost:** one worker slot per waiting sensor; 100 sensors can starve the pool and deadlock the DAG that would produce the file they wait for.
- **Reschedule mode** stores the next retry time on the task instance (`up_for_reschedule`); each poke is a full task start (process spin-up, DB writes) — avoid very short intervals.
- **Triggerer** runs a single asyncio event loop with many triggers (`TimeDeltaTrigger`, `FileTrigger`, cloud-specific triggers); when a trigger fires, the task instance resumes on a normal worker. Requires running `airflow triggerer`.
- **ExternalTaskSensor alignment:** by default compares the *same logical date*; if schedules differ, use `execution_delta` or `execution_date_fn`. Mismatched dates are the #1 reason it waits forever.
- **TriggerDagRunOperator** creates a `manual`-type run, with `logical_date` defaulting to now; set `execution_date` or `trigger_run_id` for deterministic ids and `wait_for_completion=True` to block until the child finishes.
- **Datasets** track events in a table; consumers run once per *batch* of producer updates and have no interval — their `logical_date` is when the event was processed.

### 6. Performance and design implications
- Use `reschedule` or deferrable for waits longer than a few minutes; use `poke` only for short waits.
- Choose `poke_interval` matching the arrival uncertainty (minutes, not seconds).
- Set realistic `timeout`s so stuck sensors don't pile up across daily runs.
- Prefer datasets or push-triggering over cross-DAG sensors to reduce coupling and polling load.
- Put sensors in a **dedicated pool** to cap how many may wait concurrently.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FileSensor`, `S3KeySensor`, `SqlSensor`, `HttpSensor` | Wait on a resource | Late-arriving inputs |
| `mode="reschedule"` | Free the slot between pokes | Long waits |
| `deferrable=True` (on supporting operators) | Use triggerer | Many/long waits |
| `TriggerDagRunOperator(wait_for_completion=True)` | Start and optionally await child | Orchestrator DAGs |
| `ExternalTaskSensor(execution_delta=...)` | Align different schedules | Cross-DAG deps |
| `outlets=[Dataset(...)]`, `schedule=[...]` | Data-aware scheduling | Producer/consumer DAGs |
| `airflow dags trigger --conf` | Manual run with config | Ad-hoc, API integration |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Sensor in same DAG | The wait is for an event owned by the same pipeline |
| `TriggerDagRunOperator` | You want the parent to control children; avoid circular triggers |
| `ExternalTaskSensor` | Two DAGs have the same cadence and you need strict ordering |
| Datasets | Producer/consumer relationship is clean and Airflow ≥ 2.4 |
| Deferrable operators | The provider ships a deferrable variant; triggerer is running |
| Plain schedule | Arrival time is predictable — simplest option |

### 9. Common mistakes
- **Sensors in `poke` mode with long timeouts** → worker slots exhausted → reschedule/deferrable + pool.
- **ExternalTaskSensor with misaligned schedules** → waits for non-existent run → set `execution_delta`.
- **No timeout** → sensors wait indefinitely, runs pile up → explicit timeout and `soft_fail` or alert.
- **Trigger loops** (A triggers B triggers A) → runaway runs → one-directional flow.
- **Using the clock to guess arrival** (`TimeSensor`, sleep) → brittle → wait on the actual artefact.
- **Forgetting `conf` is only for manual/triggered runs** → `KeyError` on scheduled runs → use `.get()` with defaults.

### 10. 🎯 Interview kit
**30-second answer:** "When data arrives irregularly, I use sensors in reschedule or deferrable mode to wait for the actual artefact, trigger other DAGs with `TriggerDagRunOperator` or datasets for loose coupling, and manage waiting load with pools and timeouts."

**Structure a longer answer:** clarify arrival pattern → choose trigger mechanism → resource and coupling trade-offs → failure handling (timeout, soft fail, alert).

**Likely follow-ups:**
- *Poke vs reschedule vs deferrable?* → Slot held / slot freed with full restart / slot freed with async triggerer.
- *ExternalTaskSensor never succeeds — why?* → Logical dates do not align between DAGs.
- *How do datasets differ from sensors?* → Event-driven scheduling with no polling and no worker slot.
- *How to pass parameters to a triggered DAG?* → `conf`.

**Signals interviewers look for:** slot-awareness, handling late data, preference for decoupling, knowledge of triggerer.

### 11. Check yourself
- [ ] Configure a sensor that waits up to 4 h without hogging a slot.
- [ ] Explain why 200 poke-mode sensors can deadlock a pool.
- [ ] Fix an ExternalTaskSensor between a daily DAG and one at 06:00.
- [ ] Describe dataset-triggered scheduling and its logical date.
- [ ] Compare push (trigger) vs pull (sensor) DAG coupling.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Communicating with external systems
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Beyond the basics
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Triggering workflows

