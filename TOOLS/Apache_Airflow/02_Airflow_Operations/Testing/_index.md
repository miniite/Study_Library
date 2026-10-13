---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Testing"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Testing Airflow DAGs

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Test Airflow code in layers — DAG integrity (does it load?), unit tests of task logic with mocks, integration tests against real-like services, and whole-DAG tests — and run them in CI before deploying.

### 1. The big picture
DAGs are code, but they are awkward to test: they run on a schedule, depend on external systems and are loaded by the framework rather than called by you. Without tests, mistakes show up as failed production runs at 3 a.m. A practical strategy separates what you can verify cheaply and quickly (does the file import? are there cycles? do all tasks have owners/retries?) from what needs real dependencies (does this SQL work? can this connection reach the API?).

The book's testing chapter walks through: setting up **pytest** with Airflow, testing **operators** with mocked connections/hooks, using **`airflow tasks test`**, running a **test database** (e.g., Postgres in Docker) for integration, and **emulating production** with containers.

### 2. Core concepts and vocabulary
- **DAG integrity test** — loads all DAG files with `DagBag` and asserts no import errors, no cycles, unique ids and required conventions.
- **Unit test** — checks a function/operator `execute()` in isolation with mocked hooks.
- **Integration test** — runs a task against real or containerised dependencies (Postgres, S3-compatible store).
- **`DagBag`** — Airflow class that parses a folder and exposes `dags` and `import_errors`.
- **`dag.test()` / `airflow dags test`** — executes a full DAG run in a single process without scheduler/executor.
- **`airflow tasks test`** — runs one task for a given date, no DB state saved.
- **Mock / patch** — replace a hook/operator method (`unittest.mock.patch`) to avoid external calls.
- **Fixture** — pytest setup/teardown helper (e.g., a temporary DB).
- **`pytest-mock`, `pytest-docker-tools`** — common helper libraries.
- **Test context** — dict you build to call `execute(context)` manually.

### 3. How it works, step by step
1. **Local environment:** install the same Airflow version and constraints as production; set `AIRFLOW_HOME` and `AIRFLOW__CORE__LOAD_EXAMPLES=False`; use an isolated SQLite metadata DB.
2. **Integrity tests:** a parametrised pytest that builds a `DagBag(dag_folder, include_examples=False)` and asserts `len(dagbag.import_errors) == 0`, then loops over `dag.tasks`.
3. **Unit tests:** instantiate the operator with a `dag` and call `operator.execute(context={})`, patching hooks. Assert on calls/results.
4. **Task logic outside Airflow:** keep business logic in plain Python modules with no Airflow imports, test with ordinary pytest.
5. **Integration tests:** spin up a service container, point a connection to it through an environment variable (`AIRFLOW_CONN_MY_DB=postgres://...`), run the task or `dag.test()`.
6. **CI:** run all of this on every pull request, and the integrity tests again against the target environment's DAG folder on deploy.

### 4. Worked example
```python
# tests/test_dag_integrity.py
import glob, pytest
from airflow.models import DagBag

DAG_FILES = glob.glob("dags/**/*.py", recursive=True)

def test_no_import_errors():
    bag = DagBag(dag_folder="dags", include_examples=False)
    assert bag.import_errors == {}, f"Import errors: {bag.import_errors}"

def test_dag_conventions():
    bag = DagBag(dag_folder="dags", include_examples=False)
    for dag_id, dag in bag.dags.items():
        assert dag.tags, f"{dag_id} needs tags"
        assert dag.default_args.get("retries", 0) >= 1, f"{dag_id} needs retries"
        assert dag.catchup is False or dag_id.startswith("hist_"), f"{dag_id} catchup"
```
```python
# tests/test_load_operator.py
from unittest.mock import patch
from datetime import datetime
from airflow import DAG
from my_plugin.operators import PostgresToS3Operator

def test_operator_writes_expected_key(tmp_path):
    dag = DAG("t", start_date=datetime(2024, 1, 1))
    op = PostgresToS3Operator(task_id="x", query="select 1", s3_key="out/{{ ds }}.csv", dag=dag)
    with patch("my_plugin.operators.PostgresHook") as pg, \
         patch("my_plugin.operators.S3Hook") as s3:
        pg.return_value.get_pandas_df.return_value = __import__("pandas").DataFrame({"a": [1]})
        op.execute(context={"ds": "2024-01-01"})
        s3.return_value.load_string.assert_called_once()
        assert s3.return_value.load_string.call_args.kwargs["key"] == "out/2024-01-01.csv"
```
Run locally:
```bash
pytest -q tests/
airflow tasks test my_dag load_sales 2024-01-01      # one task, no state saved
python dags/my_dag.py                                 # catches syntax/import errors
```
Interpretation: the integrity test fails *before* deployment on import errors and policy violations; the unit test proves the operator uses the right key without touching real services.

### 5. Under the hood   *(optional)*
- **Why `execute(context)`** — the framework calls it with the rendered context; calling it directly tests logic but skips rendering, so test templated fields separately with `op.render_template_fields(context)` or `dag.test()`.
- **`airflow tasks test`** builds a TaskInstance without persisting its final state and ignores dependencies, so downstream not required.
- **`dag.test()`** (2.5+) runs every task in topological order using the `DebugExecutor`-like local runner inside one process — breakpoints work.
- **Connections in tests** — environment variables named `AIRFLOW_CONN_<ID>` override the DB; avoids needing a populated metadata DB.
- **Testing DAG-level Python** (dynamic generation) — assert the generated structure (`dag.task_ids`, `task.upstream_task_ids`).
- **Test pyramid for data** — logic tests (fast), schema/contract tests, and data-quality checks that run *in* the DAG (row counts, nulls) complement code tests.

### 6. Performance and design implications
- Integrity tests parse every DAG — keep files light so the test suite stays under seconds.
- Avoid real network in unit tests; slow and flaky tests get disabled.
- Containers make integration tests realistic but cost minutes; run them nightly or on merge, not on every save.
- Separate **pure logic** from **Airflow glue** to test 90 % of behaviour without Airflow at all.
- Pin versions: tests that pass on Airflow 2.5 may fail on 2.9; mirror production images.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pytest` | Run test suite | CI, local |
| `airflow tasks test <dag> <task> <date>` | Run one task ad hoc | Debug a task body |
| `airflow dags test <dag> <date>` | Run a DAG in-process | Pre-merge check |
| `airflow dags list-import-errors` | Show parse failures | Quick triage |
| `DagBag(...)` | Load DAGs in tests | Integrity tests |
| `AIRFLOW_CONN_X=...` | Define connections via env | Integration tests |
| `unittest.mock.patch` | Replace hooks | Unit tests |
| `AIRFLOW__CORE__UNIT_TEST_MODE=True` | Test-mode settings | pytest sessions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Mocked unit tests | Behaviour depends on real SQL dialect or API semantics |
| Containerised integration tests | CI minutes are limited; reserve for critical pipelines |
| `dag.test()` | You need executor-specific behaviour (Kubernetes pod templates) |
| Staging environment | Data is sensitive or copying production is costly |
| Data-quality tasks in DAG | You need to *prevent* bad code from deploying — keep code tests too |

### 9. Common mistakes
- **No integrity test** → one import error hides a DAG in production → add `DagBag` test to CI.
- **Testing against production connections** → accidental writes → use env-var connections/containers.
- **Mock everything** → tests pass but code breaks on real service → add a few integration tests.
- **Logic buried in DAG files** → untestable → move to importable modules.
- **Not pinning Airflow/provider versions in tests** → "works on my machine" → use constraints files.
- **Ignoring templating** → `{{ ds }}` bug appears only at run time → render and assert.

### 10. 🎯 Interview kit
**30-second answer:** "I test Airflow in layers: a DagBag integrity test for imports, cycles and conventions; unit tests of operators and pure Python logic with mocked hooks; integration tests against containerised dependencies; and `dag.test()` for end-to-end — all wired into CI with the production Airflow version pinned."

**Structure a longer answer:** clarify risk (import vs logic vs data) → test layers → trade-offs (speed vs realism) → CI/CD scenario.

**Likely follow-ups:**
- *How do you test a templated field?* → Render it with a context and assert on the result.
- *What does `airflow tasks test` skip?* → Dependencies and persisting state.
- *How do you give a test a DB connection?* → `AIRFLOW_CONN_*` environment variable pointing to a container.
- *What about data correctness?* → Add data-quality tasks/checks and contract tests.

**Signals interviewers look for:** layered approach, separating logic from glue, CI integration, version pinning.

### 11. Check yourself
- [ ] Write a pytest that fails if any DAG has an import error.
- [ ] Unit-test a custom operator with a patched hook.
- [ ] Explain differences between `tasks test`, `dags test` and a real scheduled run.
- [ ] Describe how to supply a test database connection without touching the metadata DB.
- [ ] Say what you would run on every PR vs nightly.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Testing
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 31

