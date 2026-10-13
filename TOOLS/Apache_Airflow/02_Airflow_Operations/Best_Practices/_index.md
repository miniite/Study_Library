---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Best_Practices"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Airflow Best Practices

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Good Airflow DAGs are readable, light to parse, built from small idempotent and atomic tasks, parameterised by the logical date, and kept away from heavy computation and large data in the scheduler.

### 1. The big picture
Airflow makes it easy to write a DAG that works once and hurts forever. The practices below come from the "Best practices" chapter of the Harenslak/de Ruiter book and fall into four groups: **writing clean DAGs** (style, config, factories, task groups, versioning), **designing reproducible tasks** (idempotency, determinism, functional style), **handling data efficiently** (limit data volume, incremental loads, cache intermediate data, don't store data on local filesystems), and **managing resources** (pools, concurrency, SLAs). Each one maps to a failure mode you will meet in production: scheduler lag, duplicate rows after a retry, workers running out of memory, a backfill that cannot be reproduced.

### 2. Core concepts and vocabulary
- **Idempotent task** — running it N times with the same inputs leaves the same end state as running it once.
- **Atomic task** — either completes fully or has no visible effect (no half-written outputs).
- **Deterministic task** — same inputs always produce the same outputs (no `now()`, no unordered iteration, no random without seed).
- **Functional paradigm** — tasks as pure functions of their inputs (logical date, upstream data), no hidden state.
- **Top-level code** — anything executed when the DAG file is imported.
- **DAG factory** — function that builds DAGs from configuration to avoid copy/paste.
- **Task group** — UI-level grouping of tasks.
- **Pool** — named slot-limit shared by tasks for scarce resources.
- **SLA** — expected completion time after the interval; a miss triggers a callback/email.
- **Incremental load** — process only the new interval, not the whole history.
- **Cache intermediate data** — write stage outputs to durable storage so downstream retries don't recompute.

### 3. How it works, step by step
A checklist for turning a prototype DAG into a production DAG:

1. **Style** — one convention for imports, task ids (`verb_noun`), consistent dependency notation; format with a linter/formatter in CI.
2. **Config** — keep environment-specific values in Variables/Connections/env or a config file, not in code; read them inside tasks or via templates.
3. **Top level stays light** — no queries, API calls, big imports or heavy computation at module level.
4. **Make every task idempotent & atomic** — write by partition/overwrite or upsert; write to a temp path then rename.
5. **Key everything on the logical date** — `{{ ds }}`/`data_interval_*`, never `now()`.
6. **Incremental processing** — each run handles one interval; full reloads are an explicit manual job.
7. **Persist intermediate outputs** in durable shared storage (object store, warehouse), not worker disks.
8. **Limit data passed through Airflow** — move work to Spark/warehouse; use Airflow to *orchestrate*.
9. **Bound concurrency** — pools, `max_active_runs`, per-task concurrency; set retries, timeouts, SLAs.
10. **Version big changes** by creating a new DAG id instead of editing a live one.

### 4. Worked example
Before (anti-pattern) and after:
```python
# BEFORE: computation + DB hit at parse time, append-only write, uses now()
from datetime import datetime
import pandas as pd
from airflow.hooks.base import BaseHook

api_key = BaseHook.get_connection("my_api").password       # DB query on every parse
baseline = expensive_model_load()                           # runs every ~30 s per scheduler

def _load():
    df = fetch(api_key, since=datetime.now().date())        # not reproducible
    df.to_sql("sales", engine, if_exists="append")          # duplicates on retry

# AFTER: parse-light, idempotent, keyed on logical date, atomic
def _load(conn_id, data_interval_start, data_interval_end, **_):
    key = BaseHook.get_connection(conn_id).password         # fetched at run time
    df = fetch(key, start=data_interval_start, end=data_interval_end)
    with engine.begin() as tx:                              # atomic
        tx.execute("DELETE FROM sales WHERE sale_ts >= %s AND sale_ts < %s",
                   (data_interval_start, data_interval_end))
        df.to_sql("sales", tx, if_exists="append", index=False)

PythonOperator(task_id="load_sales", python_callable=_load,
               op_kwargs={"conn_id": "my_api"}, retries=3, pool="api_pool")
```
The delete-then-insert in a single transaction makes retries and backfills safe. Check the effect by running the same task twice and comparing `COUNT(*)`.

### 5. Under the hood   *(optional)*
- **Why top-level cost matters:** each DAG file is parsed in a separate process on a repeating loop; `min_file_process_interval` and the number of files determine total parser load. Slow files delay *all* scheduling decisions because the parser pool is shared.
- **Why idempotency matters:** Airflow gives *at-least-once* execution — retries, manual clears, zombie recovery and backfills all re-run tasks. Idempotency turns at-least-once into effectively-once.
- **Atomicity techniques:** write to `…/_tmp/run_id/` then move; `INSERT … SELECT` into a staging table + partition swap; transactional sinks; Delta/Iceberg commits.
- **Sensors as slot hogs:** a sensor in `poke` mode occupies a worker slot; `mode="reschedule"` or deferrable operators release it.
- **Task groups vs SubDAGs:** SubDAGs ran as separate DAG runs with their own limits and deadlocks; they are deprecated — use task groups.
- **Dataset/lineage:** keep outputs addressable by interval so lineage and re-runs are explicit.

### 6. Performance and design implications
- **Keep the scheduler fast:** thin DAG files, avoid imports of large libraries at top level (import inside callables), avoid dynamic DAG generation from remote sources at parse time.
- **Don't process big data on the worker:** pandas on 50 GB in a PythonOperator kills the worker; push down to the warehouse or Spark and just trigger it.
- **Parallelism is cheap to declare, expensive to run:** respect the downstream systems with pools.
- **Fewer, larger tasks vs many small ones:** choose granularity to match retry cost and visibility.
- **Metrics to watch:** DAG parse time, scheduler heartbeat, queued-task latency, task duration trends, SLA misses.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pool=`, `pool_slots=` | Limit concurrent tasks per resource | Protect APIs/DBs |
| `retries`, `retry_delay`, `retry_exponential_backoff` | Auto-retry | Flaky sources |
| `execution_timeout` | Kill runaway tasks | Prevent hung jobs |
| `sla=timedelta(...)` / `sla_miss_callback` | Alert on lateness | Reporting deadlines |
| `max_active_runs`, `max_active_tasks` | Throttle DAG | Catch-up, big DAGs |
| `airflow dags test` | Run DAG in-process | CI/dev |
| `TaskGroup` | Visual grouping | Large DAGs |
| `doc_md` | Markdown docs in the UI | Runbooks on the DAG |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Overwrite partition for idempotency | Table is huge and partition boundary isn't available — use upsert/merge |
| DAG factory | The DAGs differ substantially — copy/paste may be clearer |
| Orchestrate external compute | Task is tiny — doing it in the worker is fine |
| Many task groups | Tasks are few; extra nesting adds noise |
| Tight SLAs | Upstream timing is unpredictable — alert noise |

### 9. Common mistakes
- **Appending on retry** → duplicate rows → delete-then-insert / merge.
- **`datetime.now()` in tasks** → irreproducible backfills → logical date.
- **Heavy code at module level** → scheduler lag → move into callables.
- **Local-disk handoff between tasks** → file not found on another worker → shared/object storage.
- **Sensors holding slots** → pool starvation → `reschedule` mode or deferrable.
- **Editing live DAGs destructively** (rename tasks, change schedule) → broken history → new versioned DAG.
- **No timeouts or retries** → a hung task blocks a slot forever → set `execution_timeout`.

### 10. 🎯 Interview kit
**30-second answer:** "Make tasks idempotent, atomic and keyed on the logical date so retries and backfills are safe; keep DAG files light so the scheduler stays fast; push heavy data work to external engines; and bound concurrency with pools and timeouts."

**Structure a longer answer:** clarify the failure scenario → principle (idempotency/determinism/parse-light) → trade-offs → concrete before/after.

**Likely follow-ups:**
- *How do you make an append-only load idempotent?* → Partition overwrite or delete-insert in a transaction; or merge on a natural key.
- *Why avoid top-level code?* → Parsed continuously; slows scheduler and hits external systems.
- *Where should intermediate data live?* → Durable, shared storage, one path per run.
- *How do you keep a sensor from hogging a slot?* → `mode="reschedule"` or deferrable operator.

**Signals interviewers look for:** at-least-once reasoning; concrete idempotency techniques; operational awareness (parse time, pools, SLAs).

### 11. Check yourself
- [ ] Rewrite an append load to be idempotent.
- [ ] List what is wrong with `BaseHook.get_connection()` at DAG top level.
- [ ] Explain atomicity and give two ways to achieve it for file outputs.
- [ ] Choose settings to protect a rate-limited API from a 90-run backfill.
- [ ] State why large DataFrames shouldn't move through XCom.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Best practices
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — preface
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Project Finding the fastest way to get around NYC › Project: Finding the fastest way to get around NYC
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Triggering workflows

