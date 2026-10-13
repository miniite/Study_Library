---
id: "TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies"
level: 4
type: "concept"
topic: "TOOLS/Apache_Airflow/Airflow_Core"
tools: ["Apache_Airflow", "Python"]
---

# DAG Design and Dependencies

**L4 Advanced** · [↑ Airflow Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A good Airflow DAG is a small, readable graph of idempotent tasks whose dependencies state exactly what must finish before what, written with light top-level code and a deliberate split between "one big DAG" and "several DAGs connected by datasets".

### 1. The big picture
A pipeline is a set of steps with ordering constraints. Airflow models this as a **directed acyclic graph**: *directed* because every edge says "this before that" (so a task never runs before its prerequisites finish), and *acyclic* because a task can never point back to an already-completed task — a cycle would make the run unable to finish or to be ordered at all. Airflow refuses to load a DAG containing a cycle.

DAG design is about answering: *how many tasks, what is each one's boundary, how do they depend on each other, and how do separate DAGs hand over to one another?* There is no single right answer; the guiding ideas are small tasks, idempotence, clear failure boundaries and a limited blast radius.

### 2. Core concepts and vocabulary
- **DAG** — container: id, schedule, start date, default arguments, tasks and edges.
- **Task / operator** — a node. The *operator* is the class (the recipe); the *task* is an instance of it placed in a DAG with a `task_id`.
- **Dependency (edge)** — "downstream runs after upstream". Written with `>>` / `<<` or `set_downstream` / `set_upstream`.
- **`default_args`** — dictionary of settings applied to every task (owner, retries, retry_delay, email, SLA …) so you do not repeat them.
- **TaskFlow API** — `@dag` / `@task` decorators: plain Python functions become tasks, return values become XComs, and dependencies are inferred from data flow.
- **TaskGroup** — a visual and logical grouping of tasks within one DAG (replaces SubDAGs, which ran as separate DAG runs and caused deadlocks and scheduling oddities).
- **Dynamic task generation** — creating tasks in a loop or with *dynamic task mapping* (`.expand()`).
- **Dataset / data-aware scheduling** — a DAG declares which dataset it *produces* and another DAG declares it is scheduled when that dataset is *updated*.
- **Backfill** — running a DAG for past schedule slots.
- **Idempotent task** — running it twice for the same slot leaves the same end state.

### 3. How it works, step by step
**Defining dependencies**
1. Create tasks: `a`, `b`, `c`, `d`.
2. Chain with the bitshift operators: `a >> [b, c] >> d` means *a* first, then *b* and *c* in parallel, then *d* when both finish. `a >> b` is identical to `b << a` and to `a.set_downstream(b)`.
3. Use `chain(a, b, c)` or `cross_downstream([x, y], [p, q])` for longer or fully-crossed wiring (a list `>>` a list is *not* allowed directly).
4. The scheduler then starts a task only when its upstreams satisfy its trigger rule (default: all succeeded).

**Choosing task boundaries**
1. Ask "what do I want to be able to **retry independently**?" — each retry unit is a task.
2. Ask "what do I want to **see and alert on** separately?" — that is another boundary.
3. Do not split so finely that task start-up overhead and metadata rows dominate, nor so coarsely that one failure re-does an hour of work.
4. Make each task **atomic and idempotent**: write to a temporary location then swap/merge, use `MERGE`/overwrite-partition rather than blind `INSERT`, and parameterise by the data interval.

**Connecting DAGs**
- *Same DAG* when steps share a schedule and belong to one failure domain.
- *Dataset scheduling* (`schedule=[Dataset("s3://bucket/orders")]`) when a producer DAG should trigger a consumer automatically on publish; the consumer sees only that the dataset updated.
- `TriggerDagRunOperator` when the producer explicitly starts another DAG; `ExternalTaskSensor` when a consumer waits for a specific task in another DAG with an aligned schedule.

### 4. Worked example
```python
from datetime import datetime, timedelta
from airflow.decorators import dag, task
from airflow.datasets import Dataset

orders_ds = Dataset("s3://lake/curated/orders/")

@dag(
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    default_args={"owner": "data-eng", "retries": 2,
                  "retry_delay": timedelta(minutes=5)},
    tags=["orders"],
)
def orders_pipeline():
    @task
    def extract(ds=None) -> str:
        # writes raw file for the day and returns its PATH, not the data
        return f"s3://lake/raw/orders/{ds}.parquet"

    @task
    def validate(path: str) -> str:
        return path

    @task(outlets=[orders_ds])
    def load(path: str, ds=None):
        # idempotent: overwrite the partition for {ds}
        ...

    load(validate(extract()))

orders_pipeline()
```
Walk-through: `extract` returns a *path*, so XCom stays tiny. Dependencies are inferred from the call chain. `load` declares an *outlet*; any DAG with `schedule=[orders_ds]` runs after a successful `load`. Check the Graph view, run `airflow dags test orders_pipeline 2026-02-01` locally, and `airflow dags list-import-errors` in CI to catch broken DAGs.

### 5. Under the hood
- The DAG file is imported as a Python module; the `DAG` object (usually created at module level or via decorator) is discovered in the module namespace. After parsing, the structure is **serialised to JSON** in the metadata DB; the scheduler and web server work from that, not from your Python.
- The cycle check is a graph traversal at DAG-bag load time; a cycle raises `AirflowDagCycleException` and the DAG shows as an import error.
- A dependency is stored as upstream/downstream task-id sets on each task. At run time, a task instance's *dependency context* evaluates: upstream states vs trigger rule, pool slots, `depends_on_past`, concurrency caps.
- `depends_on_past=True` makes a task instance wait for the *same task's* previous run to succeed; it creates a serial chain across runs and is a frequent cause of "everything is stuck" after one failure.
- Dataset-triggered runs are created by the scheduler when it observes a producer task instance with matching outlets succeed; no polling task is occupying a slot.
- SubDAGs used a separate DAG run and a sensor-like operator, hence deadlock risk; TaskGroups only change presentation and ID prefixes (`group_id.task_id`), so cross-group dependencies must reference the prefixed IDs.

### 6. Performance and design implications
- One 300-task DAG: single place to see lineage, but a bigger blast radius, slower UI, slower parse, and harder reasoning about failures. Several smaller DAGs linked by datasets: isolated failures and ownership, independent schedules and deploys, at the cost of cross-DAG visibility (lineage must be inspected via the Datasets view or a catalog).
- Dynamic generation from config is powerful but re-runs generation every parse; keep generators cheap and deterministic.
- Dynamic task mapping creates map indexes at run time — useful for per-file/partition fan-out, but cap the width (e.g. `max_active_tis_per_dag`).
- Test cheaply: `DagBag` import tests in CI (no import errors, no cycles, every task has an owner/retries), unit tests for custom operators with mocked hooks, and an end-to-end run against a sandbox.
- Zero-downtime deploys: deploy DAG files atomically (git-sync to a new directory or an image rollout) so the scheduler never parses a half-copied file.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `a >> b`, `chain()`, `cross_downstream()` | wire dependencies | always |
| `default_args` | shared task settings | retries, owner, SLA |
| `@dag`, `@task` | TaskFlow API | clearer Python-first DAGs |
| `TaskGroup` | group tasks in the UI | large DAGs |
| `schedule=[Dataset(...)]`, `outlets=` | data-aware triggering | producer/consumer DAGs |
| `TriggerDagRunOperator`, `ExternalTaskSensor` | explicit cross-DAG control | aligned schedules, manual handoff |
| `airflow dags backfill` / UI "Clear" | re-run past slots | after fixing logic or late data |
| `airflow dags test`, `airflow dags list-import-errors` | local run / validation | dev and CI |
| `params`, `dag_run.conf` | runtime configuration | table name, path without code edits |
| `max_active_runs`, `tags` | concurrency cap, UI filtering | operations |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fine-grained tasks for independent retry and monitoring | each task is only seconds long and overhead dominates |
| One DAG for a tightly-coupled flow with one owner and schedule | multiple teams and schedules are bundled together |
| Datasets for decoupled producer/consumer | you need to wait on one specific task state in a fixed-schedule peer (use sensor) |
| `TaskGroup` for organisation | you need isolation — grouping gives none |
| `depends_on_past` for strictly sequential state (running totals) | independent partitions — it serialises the backfill and propagates failure |
| Dynamic generation from config for many similar pipelines | generation needs network/DB calls at parse time |

### 9. Common mistakes
- Calling a task and an operator the same thing → muddled design discussions → operator is the class, task is its placement in a DAG.
- Storing state in local files on one worker → fails when the next task lands on another worker → use shared object storage.
- Non-idempotent loads (`INSERT` append) → duplicates after retry/backfill → overwrite partition or `MERGE`.
- Using `datetime.now()` in tasks → results depend on wall clock → use the data interval / `ds` template.
- Putting heavy logic in the DAG file → slow parsing → move it into callables or packages.
- Cycles or cross-group IDs wrong after SubDAG→TaskGroup migration → import error / wrong wiring → use prefixed task IDs, check with `list-import-errors`.
- Forgetting `catchup` semantics → surprise mass backfill on first enable → set `catchup=False` unless intended.

### 10. 🎯 Interview kit
**30-second answer:** A DAG is a directed acyclic graph of tasks: directed so every task waits for its upstreams, acyclic so the run can always finish. In Airflow you wire tasks with `>>`, set shared settings in `default_args`, keep tasks idempotent and small, and connect separate DAGs through datasets or triggers.

**Structure a longer answer:** clarify (scale, ownership, schedule) → how dependencies and trigger rules decide execution → task boundary and idempotence rules → single DAG vs multiple linked DAGs trade-off → close with a concrete example and how you test and deploy.

**Likely follow-ups:**
- *Why acyclic?* → a cycle has no valid execution order and could run forever.
- *One 300-task DAG or many small ones?* → many small linked by datasets for isolation/ownership; one large only when truly one failure domain.
- *How do you make DAG B run when DAG A produces a table?* → dataset outlet/schedule, or `TriggerDagRunOperator`; avoid polling sensors that hold slots.
- *What is a backfill and when?* → re-running historical intervals after logic changes or late data; tasks must be idempotent and concurrency capped.
- *How do you test a DAG before deploy?* → DagBag import test, unit tests with mocked hooks, `dags test`, staging run.

**Signals interviewers look for:** idempotence, blast-radius thinking, knowing top-level code costs, and distinguishing grouping (TaskGroup) from isolation (separate DAG).

### 11. Check yourself
- [ ] Wire a diamond-shaped DAG three different ways.
- [ ] Explain what makes a task idempotent and rewrite an append-load to be idempotent.
- [ ] List the trade-offs between one large DAG and several dataset-linked DAGs.
- [ ] Explain why SubDAGs were replaced by TaskGroups.
- [ ] Describe a CI check that prevents broken DAGs reaching production.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../_index.md>)
- [Python](<../../Python/_index.md>)

## Practice questions

- [Basic] [What does DAG stand for and why must an Airflow workflow be acyclic?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02128_What_does_DAG_stand_for_and_why_must_an_Airflow_workflow_be_.md>)
- [Basic] [How do you initialize the Airflow metadata database and what does airflow db init do?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02218_How_do_you_initialize_the_Airflow_metadata_database_and_what.md>)
- [Intermediate] [What is a TaskGroup in Airflow 2 and how does it help organize a large DAG?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02203_What_is_a_TaskGroup_in_Airflow_2_and_how_does_it_help_organi.md>)
- [Intermediate] [If a DAG is not appearing in the Airflow UI what steps would you take to troubleshoot?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02225_If_a_DAG_is_not_appearing_in_the_Airflow_UI_what_steps_would.md>)
- [Deep] [How would you make one DAG run automatically when another DAG finishes producing a dataset?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02019_How_would_you_make_one_DAG_run_automatically_when_another_DA.md>)
- [Deep] [How would you structure connections and variables across dev staging and prod so a DAG promotes cleanly through environments without code changes?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02066_How_would_you_structure_connections_and_variables_across_dev.md>)
- [Deep] [How would you integrate Great Expectations or dbt tests as blocking tasks in an Airflow DAG and where do you draw the line between Airflow and the transformation tool?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02099_How_would_you_integrate_Great_Expectations_or_dbt_tests_as_b.md>)
- [Deep] [How do you design a DAG so that a partial failure leaves the warehouse in a consistent state — think transactional boundaries and compensation for a multi-step load?](<../../../PRACTICE/TOOLS/Apache_Airflow/Airflow_Core/Q-02134_How_do_you_design_a_DAG_so_that_a_partial_failure_leaves_the.md>)

## Related concepts

- [Airflow DAGs and Backfills](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Airflow_DAGs_and_Backfills.md>) *(Pipeline Orchestration)*
- [Idempotent Airflow Tasks](<../../Python/Python_Language/Idempotent_Airflow_Tasks.md>) *(Python Language)*
- [Orchestration vs Processing](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>) *(Pipeline Orchestration)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — A Modern Data Infrastructure › Directed Acyclic Graphs
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Directed Acyclic Graphs
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › Building Airflow DAGs
- *Data Pipelines Pocket Reference* — James Densmore — Orchestrating Pipelines › An ELT Pipeline DAG
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Managing Data Quality with Apache Airflow
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Installing Circuit Breakers with Apache Airflow
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — B › D
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Anatomy of an Airflow DAG
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Defining dependencies between tasks
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Testing
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Best practices
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Building End-to-End Data Pipelines › 13. Building End-to-End Data Pipelines
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Define default arguments
- *Building Medallion Architectures* — Piethein Strengholt — D
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 5Data Orchestration and Workflows › CHAPTER 5Data Orchestration and Workflows(#contents.xhtmlch05a)

