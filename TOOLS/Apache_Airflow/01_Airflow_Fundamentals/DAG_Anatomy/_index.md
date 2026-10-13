---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals/DAG_Anatomy"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Anatomy of a DAG

**L4 Advanced** · [↑ Airflow Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An Airflow DAG is a Python file that declares a set of tasks (instances of operators) and the dependency arrows between them; Airflow parses the file, builds a graph, and runs each task once its upstream tasks have succeeded.

### 1. The big picture
A pipeline is a series of steps that must run in a certain order: fetch data, transform it, load it, notify someone. Airflow models this as a **directed acyclic graph (DAG)**: *directed* because each edge has a direction (A must finish before B), *acyclic* because there are no loops (B can never depend, directly or indirectly, on itself). Because a graph has no cycles, the scheduler can always find a task whose upstream work is complete and run it, and the workflow is guaranteed to end.

In Airflow the graph is not drawn in a UI or written in YAML; it is **ordinary Python code**. That means you can generate tasks in loops, share helper functions, import libraries and keep the DAG in version control. The cost is that the file is *executed* by Airflow over and over, which creates a few rules (section 9) about what may and may not run at the top level of the file.

This lesson covers the building blocks (DAG object, operators, tasks, dependencies). Scheduling, templating and architecture are covered in the sibling lessons.

### 2. Core concepts and vocabulary
- **DAG** — the container object: an id, a start date, a schedule, and a collection of tasks. Defined with `DAG(...)` or the `with DAG(...) as dag:` context manager.
- **Operator** — a class describing *one unit of work* (`BashOperator`, `PythonOperator`, `EmailOperator`, `SimpleHttpOperator`, database operators…). All inherit from `BaseOperator`. An operator has a single responsibility.
- **Task** — an operator *instantiated inside a DAG*. Operators supply the logic; the task is the manager/wrapper that Airflow uses to track state (queued, running, success…) and to show it in the UI. In everyday speech "task" and "operator" are used interchangeably.
- **Task instance** — one execution of a task for one particular DAG run (task + logical date). This is what carries a state and a log.
- **DAG run** — one execution of the whole DAG for one schedule interval.
- **Dependency** — "this task runs only after that one succeeded", written with `>>` / `<<`.
- **Task id** — name unique within the DAG; used in the UI, logs and for cross-task data exchange.
- **default_args** — dictionary of operator arguments applied to every task in the DAG (owner, retries, retry_delay…).
- **Task group** — purely visual grouping of tasks in the UI; does not change execution.

### 3. How it works, step by step
1. The scheduler periodically scans the DAGs folder and *imports every Python file* that looks like it contains DAGs.
2. Each module-level `DAG` object found is registered. Operator instantiations inside it become tasks; `>>` statements become edges.
3. The scheduler checks the schedule: for each interval that has finished, it creates a **DAG run** and one **task instance** per task.
4. A task instance whose upstream tasks have all succeeded becomes *scheduled → queued*, is handed to the executor, and a worker runs the operator's `execute()` method.
5. When the task ends, its state is written to the metadata database (success, failed, up_for_retry, skipped…), which makes downstream tasks eligible.
6. When all tasks have a final state, the DAG run itself is marked success or failed.

The same Python file is therefore used in two very different moments: *parse time* (the scheduler importing the file to learn the graph — happens every few seconds) and *run time* (a worker executing one task's callable — happens once per attempt).

### 4. Worked example
```python
import pathlib, json, requests
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

default_args = {"owner": "data-eng", "retries": 2, "retry_delay": timedelta(minutes=5)}

def _get_pictures():
    pathlib.Path("/tmp/images").mkdir(parents=True, exist_ok=True)   # idempotent: safe on retry
    with open("/tmp/launches.json") as f:
        launches = json.load(f)
        urls = [l["image"] for l in launches["results"]]
    for url in urls:
        r = requests.get(url)
        with open(f"/tmp/images/{url.split('/')[-1]}", "wb") as out:
            out.write(r.content)

with DAG(
    dag_id="download_rocket_launches",
    start_date=datetime(2024, 1, 1),
    schedule_interval="@daily",
    catchup=False,
    default_args=default_args,
) as dag:

    download_launches = BashOperator(
        task_id="download_launches",
        bash_command="curl -o /tmp/launches.json -L 'https://ll.thespacedevs.com/2.0.0/launch/upcoming'",
    )
    get_pictures = PythonOperator(task_id="get_pictures", python_callable=_get_pictures)
    notify = BashOperator(task_id="notify",
                          bash_command='echo "There are now $(ls /tmp/images | wc -l) images."')

    download_launches >> get_pictures >> notify
```
Reading it:
- Three tasks, two arrows. `download_launches` has no upstream; `notify` has no downstream.
- `>>` is Python's right-shift operator overloaded by Airflow purely for readability — there is no bit shifting involved. `a >> b` means "a before b"; `b << a` is the same thing.
- `python_callable=_get_pictures` passes the *function itself*, not its result; Airflow calls it on the worker at run time.
- `BashOperator` is the quickest way to call a CLI tool; use `PythonOperator` when parsing or branching logic would be unreadable in shell.

### 5. Under the hood   *(optional at this level)*
- **Operator vs task** — `BaseOperator.__init__` registers the new object with the "current DAG" (that is why the `with DAG` context manager works: it sets a context variable). The resulting object *is* the task; Airflow's `TaskInstance` model wraps it per run.
- **Dependency storage** — upstream/downstream relations are stored as sets of task ids on each task (`task.upstream_task_ids`). The scheduler serializes the whole DAG to JSON in the metadata DB (`serialized_dag`) so the webserver does not have to parse your files.
- **Fan-out/fan-in** — `a >> [b, c] >> d` creates two parallel branches that rejoin; `d` waits for *both* (trigger rule `all_success` by default).
- **Trigger rules** — change when a task becomes eligible: `all_done`, `one_failed`, `none_failed`, `all_failed`… essential for cleanup tasks and after branching.
- **Dynamic structure** — because it is Python, `for table in tables:` can create one task per table. Task ids must still be unique and stable between parses.

### 6. Performance and design implications
- **Parse cost is paid constantly.** Every file in the DAGs folder is imported on a loop (default roughly every 30 s per file). Heavy imports, API calls or DB queries at module top level slow the scheduler and hammer external systems.
- **Keep tasks small and single-purpose** so a failure only repeats a small piece of work and the UI tells you exactly which step broke.
- **Avoid giant monolithic Python callables** that do five things; split them so retries and parallelism work for you.
- **One DAG per logical pipeline.** Hundreds of tasks in one DAG slow UI rendering; use task groups or split DAGs.
- **Never pass big data between tasks** via return values (XCom lives in the metadata DB). Write to storage and pass the path.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow dags list` | Lists DAGs the scheduler has parsed | Check that your file was picked up |
| `airflow dags list-import-errors` | Shows files that failed to import | DAG missing from UI |
| `airflow tasks test <dag> <task> <date>` | Runs one task locally, no state recorded | Debugging a task body |
| `airflow dags test <dag> <date>` | Runs the whole DAG in-process | End-to-end dry run |
| `airflow dags trigger <dag>` | Creates a manual run | Ad-hoc execution |
| `default_args` | Shared retries, owner, retry_delay… | Avoid repeating arguments |
| `retries`, `retry_delay` | Automatic re-attempts of a failed task | Flaky external dependencies |
| `trigger_rule` | When a task may start given upstream states | Cleanup/notification tasks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `BashOperator` for one-line CLI calls | Shell quoting becomes complicated, or you need error handling |
| `PythonOperator` for transformation/logic | The function pulls heavy dependencies the worker lacks |
| A dedicated operator (S3, BigQuery, Postgres…) | A plain Python call to the SDK would be clearer than learning the operator's quirks |
| Many small tasks | Task count explodes into thousands — scheduler/UI overhead grows |
| One bigger task | Need independent retry, parallelism or observability of sub-steps |

### 9. Common mistakes
- **Doing work at module level** (querying a DB, calling an API, computing a value in the DAG body) → runs on *every parse*, not when the task runs → move it inside the callable.
- **Creating a cycle** (`a >> b >> a`) → Airflow refuses to load the DAG with a cycle error → redraw the graph.
- **Duplicate `task_id`** in one DAG → import error → use unique ids (include loop variable in the id).
- **Dynamic `start_date` such as `datetime.now()`** → the start date moves every parse, so runs never get scheduled predictably → use a fixed date.
- **Renaming tasks or changing start date/schedule of a live DAG** → history in the UI no longer lines up and the scheduler gets confused → create a new versioned DAG (`dag_v2`) for large changes.
- **Non-idempotent tasks** (append without de-duplication) → retries produce duplicates → write tasks that overwrite their partition or upsert.

### 10. 🎯 Interview kit
**30-second answer:** "A DAG is a Python-defined graph of tasks with directed, acyclic dependencies. Operators define the work, tasks are operators placed in a DAG, and the scheduler turns each schedule interval into a DAG run made of task instances that execute when their upstream tasks succeed."

**Structure a longer answer:** clarify (task vs operator vs task instance) → mechanism (parse → DAG run → task instances → executor → state in metadata DB) → trade-offs (granularity, parse cost, idempotency) → a real scenario (daily ingestion with retries and a failure alert).

**Likely follow-ups:**
- *Why must it be acyclic?* → A cycle means no valid execution order and potentially infinite runs; acyclicity guarantees a topological ordering exists.
- *What's the difference between an operator and a task?* → Operator = reusable class with the logic; task = its instance in a DAG, managed by Airflow; task instance = that task for one run.
- *Why is top-level code in a DAG file a problem?* → The file is parsed continuously by the scheduler, so top-level work runs constantly.
- *How do you run tasks in parallel?* → Put them in a list: `start >> [a, b, c] >> end`; actual parallelism depends on executor and pool/concurrency limits.

**Signals interviewers look for:** you separate parse time from run time; you mention idempotency and small single-purpose tasks; you know `>>` is just syntax; you can name trigger rules.

### 11. Check yourself
- [ ] Explain operator vs task vs task instance without notes.
- [ ] Write a three-task DAG with a fan-out and fan-in using `>>`.
- [ ] Say what happens when a DAG file does an HTTP call at module level.
- [ ] Explain why Airflow has no cycles and what error appears if you create one.
- [ ] List what you change when a DAG outgrows a single file (task groups, split DAGs, factories).
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Anatomy of an Airflow DAG
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Best practices

