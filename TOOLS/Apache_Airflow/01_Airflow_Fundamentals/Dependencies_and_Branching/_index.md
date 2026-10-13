---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals/Dependencies_and_Branching"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Dependencies Branching and Triggers

**L4 Advanced** · [↑ Airflow Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Dependencies (`>>`) define order; trigger rules, branching and conditional tasks decide *whether* a task runs given what happened upstream; the Taskflow API (`@task`) makes passing Python values between tasks natural.

### 1. The big picture
Real pipelines are not straight lines. Some steps run in parallel and then join, some run only for certain inputs (e.g., "use the old API before a cut-over date"), and some must run even if something upstream failed (cleanup, alerts). Airflow gives you four levers:

1. **Linear and fan-out/fan-in dependencies** — arrows between tasks.
2. **Branching** — a task *chooses* which downstream path(s) run; the others are `skipped`.
3. **Conditional tasks** — a task decides inside its callable (or via `ShortCircuitOperator`/`AirflowSkipException`) to skip itself and what follows.
4. **Trigger rules** — a per-task rule about how upstream states are combined (default `all_success`).

Add the Taskflow API (Airflow ≥ 2.0), which lets you declare Python tasks with decorators and wires dependencies and XCom automatically from function calls.

### 2. Core concepts and vocabulary
- **Dependency operator** — `a >> b` (a upstream of b), `b << a`, also `a.set_downstream(b)`; lists: `a >> [b, c]`, `[b, c] >> d`.
- **`chain()`** — helper for long sequences or pairwise list wiring (`chain(a, [b, c], [d, e], f)`).
- **Fan-out / fan-in** — one task feeding many; many feeding one.
- **Branching** — `BranchPythonOperator` (or `@task.branch`): callable returns the task id(s) to follow.
- **Skipped** — task state when its branch isn't chosen; propagates through default trigger rules.
- **Trigger rule** — when a task may run: `all_success`, `all_failed`, `all_done`, `one_success`, `one_failed`, `none_failed`, `none_failed_min_one_success`, `none_skipped`, `always`.
- **`ShortCircuitOperator`** — skips all downstream tasks if its callable returns false.
- **`LatestOnlyOperator`** — skips downstream unless the run is the most recent interval (guards notifications during catch-up).
- **Dummy/Empty operator** — `EmptyOperator`, a no-op node used as a join or start marker.
- **Taskflow API** — `@dag` / `@task` decorators; return values become XComs and arguments between tasks.
- **XCom** — key/value messages between tasks in the metadata DB.

### 3. How it works, step by step
1. You declare edges; the scheduler stores them as upstream/downstream id sets.
2. At run time, for each task instance the scheduler evaluates its **trigger rule** against the states of upstream instances.
3. If the rule is satisfied, the task becomes runnable; if the rule can no longer be satisfied (e.g., `all_success` but one upstream failed), the task is marked `upstream_failed`; if upstream was skipped, it is `skipped`.
4. A branching task returns one or more task ids; the scheduler marks every *other* direct downstream of the branch task as `skipped`, and that skip propagates.
5. A join task after a branch must use a rule such as `none_failed` (or `none_failed_min_one_success`) — otherwise the skipped branch also skips the join.
6. Taskflow: calling a decorated function inside a DAG returns an XCom reference, not data; passing it to another decorated call creates both the dependency and the XCom pull.

### 4. Worked example
```python
from datetime import datetime
from airflow.decorators import dag, task
from airflow.operators.empty import EmptyOperator
from airflow.utils.trigger_rule import TriggerRule

@dag(start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False)
def sales_pipeline():

    start = EmptyOperator(task_id="start")

    @task.branch
    def pick_source(ds=None):
        # logical date decides the source system
        return "extract_legacy" if ds < "2024-03-01" else "extract_new"

    @task
    def extract_legacy():
        return {"rows": 1200, "src": "legacy"}

    @task
    def extract_new():
        return {"rows": 1300, "src": "new"}

    # join must tolerate the skipped branch
    @task(trigger_rule=TriggerRule.NONE_FAILED_MIN_ONE_SUCCESS)
    def load():
        print("load whichever branch ran")

    @task(trigger_rule=TriggerRule.ALL_DONE)
    def cleanup():
        print("always clean temp files")

    branch = pick_source()
    legacy, new = extract_legacy(), extract_new()
    start >> branch >> [legacy, new] >> load() >> cleanup()

sales_pipeline()
```
What to expect:
- For a run before March 2024: `extract_legacy` succeeds, `extract_new` is `skipped`, `load` still runs (because of `NONE_FAILED_MIN_ONE_SUCCESS`), and `cleanup` runs even if `load` failed (`ALL_DONE`).
- With the default `all_success` on `load`, it would be skipped on every run — the classic branching bug.
- In the grid view, skipped tasks are pink; `upstream_failed` are orange.

### 5. Under the hood   *(optional)*
- **Trigger rule evaluation** lives in `TriggerRuleDep`. It counts upstream states (success, skipped, failed, upstream_failed, done) and checks the rule's predicate; the result decides *schedulable*, *skip* or *upstream_failed*.
- **Branch implementation** — `BranchPythonOperator` is a `SkipMixin`: after the callable returns ids it calls `skip_all_except(ti, branch_ids)`, which sets `skipped` on the unchosen direct downstream tasks.
- **Taskflow desugaring** — `@task` creates a `_PythonDecoratedOperator`; the return value is pushed with key `return_value`; arguments that are `XComArg` objects add the upstream dependency automatically and are pulled at runtime.
- **`depends_on_past` and `wait_for_downstream`** — cross-run dependencies: a task instance waits for the same task (or its downstream) in the previous run.
- **Multiple outputs** — `@task(multiple_outputs=True)` stores each dict key as its own XCom key.
- **Dynamic task mapping (≥ 2.3)** — `task.expand(arg=list)` creates N mapped task instances at run time; the number of instances is determined by upstream output, unlike loops in the DAG file.

### 6. Performance and design implications
- **Fan-out width** is limited by `parallelism`, pools, and `max_active_tasks`; a 500-way fan-out will queue.
- **Branching adds skipped task instances** — cheap, but large DAGs with many branches inflate the UI and DB.
- **XCom size** — Taskflow makes it too easy to return big DataFrames. XComs live in the metadata DB; keep them small or use a custom XCom backend.
- **Prefer explicit trigger rules** over try/except hacks inside tasks to implement cleanup and alerting.
- **Short-circuit early** (cheap validation task) to avoid launching expensive downstream work.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `a >> [b, c] >> d` | Fan-out then fan-in | Parallel branches |
| `chain(a, b, c)` | Wire long chains | Programmatically generated DAGs |
| `@task.branch` / `BranchPythonOperator` | Choose path at run time | Conditional workflows |
| `trigger_rule=...` | Redefine eligibility | Joins after branches, cleanup |
| `ShortCircuitOperator` | Skip downstream if false | Data-quality gate |
| `LatestOnlyOperator` | Run only for latest interval | Notifications during catch-up |
| `depends_on_past=True` | Wait for previous run's task | Sequential state loads |
| `.expand()` / `.partial()` | Dynamic task mapping | Unknown number of files/items |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Branch operator — different paths with different tasks | Only one task needs to be skipped (use conditional logic or short-circuit) |
| Conditional logic in callable | The decision should be visible in the UI as skipped tasks |
| `ALL_DONE` for cleanup | You want cleanup only on success |
| Taskflow decorators | You have many classic operators; mixing is fine but keep one style per DAG |
| Dynamic mapping | Fan-out is static and known — a loop is simpler |

### 9. Common mistakes
- **Join after a branch with default rule** → join is skipped every time → use `none_failed_min_one_success`.
- **Cleanup task with `all_success`** → never runs after a failure, leaking temp resources → `all_done`.
- **Creating cycles via programmatic wiring** → import error → verify loops, use `chain`.
- **Big return values with Taskflow** → slow DB, UI timeouts → pass references to storage.
- **Branch callable returns a task id that doesn't exist or isn't downstream** → run errors → return exact ids of direct downstream tasks.
- **Using `depends_on_past` everywhere** → one failure blocks all future runs → use only where order is semantically required.

### 10. 🎯 Interview kit
**30-second answer:** "Dependencies are edges between tasks; with fan-out/fan-in you get parallelism and joins. Branching operators return the task ids to follow and skip the rest, so joins need a lenient trigger rule. Trigger rules express policies like 'run cleanup always' and the Taskflow API wires Python functions and XComs automatically."

**Structure a longer answer:** clarify the control-flow need → pick tool (branch, short-circuit, trigger rule) → trade-offs (skipped propagation, XCom size) → scenario (legacy vs new source, cleanup).

**Likely follow-ups:**
- *Why did my join task skip?* → Default `all_success` + a skipped upstream branch; use `none_failed*`.
- *Difference between branch and short-circuit?* → Branch picks among paths; short-circuit stops everything downstream.
- *How to run a task when upstream failed?* → `trigger_rule=all_done` or `one_failed`/`all_failed` for alerts.
- *Static loop vs dynamic mapping?* → Loop is parse-time; mapping decides at run time from upstream data.

**Signals interviewers look for:** correct trigger-rule vocabulary; awareness of skip propagation; XCom limits; knowledge of dynamic mapping.

### 11. Check yourself
- [ ] Draw a branch with two paths and a join; state the join's trigger rule.
- [ ] List four trigger rules and a use for each.
- [ ] Convert a PythonOperator pair using XCom to Taskflow.
- [ ] Explain the difference between `skipped` and `upstream_failed`.
- [ ] Say when dynamic task mapping beats a `for` loop in the DAG file.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Defining dependencies between tasks
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 33

