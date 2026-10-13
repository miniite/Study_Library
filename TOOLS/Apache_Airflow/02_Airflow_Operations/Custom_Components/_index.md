---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Custom_Components"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Custom Operators Hooks and Sensors

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When no built-in operator fits, you extend Airflow with your own hooks (connection logic), operators (task logic), sensors, and packaged Python modules so the DAG files stay short and the code reusable and testable.

### 1. The big picture
Airflow ships with hundreds of provider operators, but real systems have in-house APIs, odd file formats and company-specific rules. The temptation is to write a `PythonOperator` with 80 lines of `requests` calls, copy it into five DAGs, and watch the copies drift. The better path is to build **custom components**:

- a **Hook** — one class that knows how to talk to an external system (auth, session, pagination, retries);
- an **Operator** — uses hooks to do one unit of work, with templated fields and clean `execute()`;
- a **Sensor** — waits for a condition via the same hook;
- a **Python package** — distributable, versioned, unit-tested and installable on every worker.

Book example: a movie-ratings API. A `MovielensHook` handles the base URL, credentials from an Airflow *connection*, pagination and date-window queries; a `MovielensFetchRatingsOperator` writes the ratings of an interval to JSON; a `MovielensSensor` waits for data to be available.

### 2. Core concepts and vocabulary
- **Hook** — subclass of `BaseHook`; wraps connection to one external system; reads host/login/password from `get_connection(conn_id)`.
- **Connection** — named credentials/endpoints stored in Airflow (DB, secrets backend or `AIRFLOW_CONN_*` env).
- **Operator** — subclass of `BaseOperator`; implements `execute(self, context)`.
- **`template_fields`** — tuple of attribute names Jinja-rendered before execute.
- **`template_ext`** — file extensions loaded as templates.
- **`apply_defaults`** (legacy) — decorator for default args, no longer required in 2.x.
- **Sensor** — subclass of `BaseSensorOperator`; implements `poke(context)`.
- **Provider package** — pip-installable Airflow extension exposing hooks/operators and metadata.
- **Plugins** — `AirflowPlugin` mechanism (macros, views); today most extensions are plain packages.
- **Custom XCom backend** — store large values externally while Airflow keeps a reference.

### 3. How it works, step by step
1. **Identify reusable logic** — repeated API calls, file handling, credential logic.
2. **Write the hook** — constructor takes `conn_id`; lazily builds a session on first use; methods return Python data (iterators/lists).
3. **Write the operator** — constructor takes parameters (dates, paths, conn ids) and stores them; `execute()` creates the hook, calls it, writes outputs; mark parameters in `template_fields`.
4. **Package** — put in `my_company_airflow/` with `pyproject.toml`; version and publish to an internal index.
5. **Install** the package into every environment (scheduler, workers, webserver/triggerer images).
6. **Use** in DAGs by import; the DAG file now only declares configuration and dependencies.
7. **Test** hooks with mocked HTTP and operators with mocked hooks (see Testing).

### 4. Worked example
```python
# my_company_airflow/hooks/movielens.py
import requests
from airflow.hooks.base import BaseHook

class MovielensHook(BaseHook):
    def __init__(self, conn_id: str, retries: int = 3):
        super().__init__()
        self._conn_id, self._retries, self._session = conn_id, retries, None

    def get_conn(self):
        if self._session is None:
            c = self.get_connection(self._conn_id)         # runs at execute time only
            self._session = requests.Session()
            self._session.auth = (c.login, c.password)
            self._base = f"{c.schema or 'http'}://{c.host}:{c.port or 80}"
        return self._session, self._base

    def get_ratings(self, start_date, end_date, batch_size=100):
        session, base = self.get_conn()
        offset, total = 0, None
        while total is None or offset < total:
            r = session.get(f"{base}/ratings",
                            params={"start_date": start_date, "end_date": end_date,
                                    "offset": offset, "limit": batch_size})
            r.raise_for_status()
            data = r.json()
            yield from data["result"]
            offset += batch_size
            total = data["total"]
```
```python
# my_company_airflow/operators/movielens.py
import json, os
from airflow.models import BaseOperator
from my_company_airflow.hooks.movielens import MovielensHook

class MovielensFetchRatingsOperator(BaseOperator):
    template_fields = ("_start_date", "_end_date", "_output_path")

    def __init__(self, conn_id, start_date="{{ data_interval_start | ds }}",
                 end_date="{{ data_interval_end | ds }}", output_path=None, **kwargs):
        super().__init__(**kwargs)
        self._conn_id, self._start_date = conn_id, start_date
        self._end_date, self._output_path = end_date, output_path

    def execute(self, context):
        hook = MovielensHook(self._conn_id)
        ratings = list(hook.get_ratings(self._start_date, self._end_date))
        self.log.info("Fetched %d ratings", len(ratings))
        os.makedirs(os.path.dirname(self._output_path), exist_ok=True)
        tmp = self._output_path + ".tmp"
        with open(tmp, "w") as f:
            json.dump(ratings, f)
        os.replace(tmp, self._output_path)                  # atomic output
```
Use in a DAG:
```python
MovielensFetchRatingsOperator(task_id="fetch", conn_id="movielens",
                              output_path="/data/{{ ds }}/ratings.json")
```
Verify with `airflow tasks render` (templated fields) and a unit test that patches the hook.

### 5. Under the hood   *(optional)*
- **Why store params with underscores / in `__init__`:** Airflow serialises operators for the UI; keep constructor args simple, JSON-serialisable and assign them to attributes named in `template_fields`.
- **Execution location:** `execute()` runs on the worker, so every dependency must be installed there.
- **Hook lifecycle:** no connection should be opened at import or in `__init__` — only in `get_conn()` — to keep DAG parsing cheap.
- **Operator extra links, `on_kill`:** override `on_kill()` to clean up when a task is killed; `operator_extra_links` adds UI buttons.
- **Packaging providers:** a proper provider declares entry points (`apache_airflow_provider`) so Airflow discovers connection types and UI fields.
- **Subclassing vs composing:** subclass existing operators to override small behaviours; compose hooks inside new operators for new systems.
- **Custom sensors:** `poke()` should be cheap and side-effect-free; `deferrable` variants add an async trigger.

### 6. Performance and design implications
- **Parse time:** importing heavy libraries in operator module top-level slows every DAG file that imports it — import lazily.
- **Reuse sessions** within a task; don't create one per request.
- **Stream, don't list:** hooks yielding batches keep memory bounded.
- **Version the package** and pin it in worker images so DAG and component versions never skew.
- **Keep operators thin:** business logic in plain functions you can test without Airflow.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `class X(BaseHook)` / `get_connection(id)` | Connection wrapper | External systems |
| `class Y(BaseOperator)` + `execute()` | New task type | Reusable work |
| `template_fields = (...)` | Enable Jinja | Date/path params |
| `class Z(BaseSensorOperator)` + `poke()` | Wait condition | Event arrival |
| `pip install -e .` | Develop package locally | Dev loop |
| `AIRFLOW_CONN_<ID>=...` | Define connection by env | Tests, containers |
| `airflow connections add/test` | Manage connections | Setup |
| `self.log.info(...)` | Task logging | Always use over `print` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Custom hook + operator | Same integration is used by several DAGs/teams |
| `PythonOperator` with a function | One-off, single DAG |
| Subclass a provider operator | Only a small behaviour change is needed |
| Separate package | Components change at a different pace from DAGs and need CI |
| Put code in `dags/` folder | Quick prototypes — not for shared logic (parse overhead, packaging) |

### 9. Common mistakes
- **Connecting in `__init__`** → DB/network hit on each parse → connect in `execute()`.
- **Not templating parameters** → fixed dates → add to `template_fields`.
- **Hard-coded credentials** → leaks → use connections.
- **Code installed on scheduler but not workers** → ImportError at run time → bake into all images.
- **Writing non-atomic output** → partial files on failure → temp file + rename.
- **Big in-memory results** → worker OOM → stream or page.

### 10. 🎯 Interview kit
**30-second answer:** "I wrap external-system access in a Hook that reads an Airflow connection, build an Operator on top with templated parameters and an idempotent `execute()`, and ship them as a versioned package so DAGs stay declarative and the logic is unit-testable."

**Structure a longer answer:** clarify repetition/pain → design (hook vs operator vs sensor) → trade-offs (package vs inline, subclass vs compose) → example (REST API with pagination).

**Likely follow-ups:**
- *Hook vs operator?* → Hook = how to talk to a system; operator = what to do in a task.
- *Why not do the work in `__init__`?* → Runs on every DAG parse.
- *How do you deploy custom code?* → Versioned package in all component images.
- *How do templated fields work in a custom operator?* → List attribute names in `template_fields`.

**Signals interviewers look for:** parse-time awareness, separation of concerns, idempotent outputs, packaging discipline.

### 11. Check yourself
- [ ] Write a hook that reads credentials from a connection and exposes a paged iterator.
- [ ] Add a templated `output_path` to a custom operator.
- [ ] Explain why custom code must be installed on workers.
- [ ] Decide between subclassing a provider operator and writing a new one.
- [ ] Describe how you would unit-test the operator without network access.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Building custom components
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 31

