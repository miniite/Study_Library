---
id: "TOOLS/Apache_Airflow/01_Airflow_Fundamentals/Templating"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Templating and the Task Context

**L4 Advanced** · [↑ Airflow Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Templating lets an Airflow task use Jinja expressions like `{{ ds }}` that are rendered at run time with that run's context (logical date, data interval, params), so the same code processes a different slice of data each run.

### 1. The big picture
A daily pipeline must read *different data every day*. Hard-coding a date would make every run identical; calling `datetime.now()` breaks reruns and backfills. Airflow's solution is **templating**: you write placeholders in operator arguments, and just before the task executes Airflow replaces them using the **task context** — a dictionary describing *this* run (its logical date, the data interval, the task instance, DAG params…).

Typical use: download the pageview file for the hour being processed, query rows where `date = '{{ ds }}'`, write output to a per-run path. Because the placeholder resolves from the run's own date, rerunning last Tuesday's run produces last Tuesday's file again — reproducibility for free.

The mechanism is Jinja2, the templating engine used by Flask; Airflow adds variables, macros and filters. This lesson uses the StockSense example from the book: hourly Wikipedia pageview dumps that must be loaded incrementally.

### 2. Core concepts and vocabulary
- **Jinja template** — text containing `{{ expression }}` (value), `{% statement %}` (logic) and `{# comment #}`.
- **Task context** — dict of values available at render time (`ds`, `ts`, `data_interval_start`, `params`, `task_instance`, `dag_run`…).
- **`template_fields`** — the set of operator attributes that Airflow renders. Only these are templated; other arguments are taken literally.
- **`ds` / `ds_nodash`** — logical date as `YYYY-MM-DD` / `YYYYMMDD`.
- **`data_interval_start` / `data_interval_end`** — pendulum datetimes bounding the slice.
- **`macros`** — helper library (`macros.ds_add`, `macros.datetime`, …).
- **`params`** — user-supplied constants/defaults attached to the DAG or task, available as `{{ params.x }}`.
- **Rendering** — the substitution step; happens on the worker right before `execute()`.
- **`templates_dict`, `op_kwargs`, `op_args`** — templated PythonOperator arguments.
- **`user_defined_macros`, `user_defined_filters`** — your own helpers registered on the DAG.

### 3. How it works, step by step
1. A DAG author writes an operator argument containing Jinja, e.g. `bash_command="echo {{ ds }}"`.
2. At parse time nothing is rendered; the string is stored as-is.
3. When the task instance starts, Airflow builds the context for *this* run.
4. For each attribute listed in the operator's `template_fields`, Airflow runs Jinja over the string (or recursively over lists/dicts) with the context.
5. The rendered values replace the attributes and `execute()` runs with real values.
6. The rendered result is stored and visible in the UI under **Task Instance → Rendered Template** (and `airflow tasks render`).

Which operator arguments are templatable:

| Operator | Templated arguments (examples) |
|---|---|
| `BashOperator` | `bash_command`, `env` |
| `PythonOperator` | `op_args`, `op_kwargs`, `templates_dict` |
| `PostgresOperator` / SQL operators | `sql` (also `.sql` files) |
| `S3` / transfer operators | keys, prefixes, buckets (varies) |

### 4. Worked example
```python
from datetime import datetime
from urllib import request
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

def _get_data(year, month, day, hour, output_path):
    url = (f"https://dumps.wikimedia.org/other/pageviews/"
           f"{year}/{year}-{month:0>2}/pageviews-{year}{month:0>2}{day:0>2}-{hour:0>2}0000.gz")
    request.urlretrieve(url, output_path)

with DAG("stocksense", start_date=datetime(2024, 1, 1), schedule_interval="@hourly",
         catchup=False) as dag:

    get_data = PythonOperator(
        task_id="get_data",
        python_callable=_get_data,
        op_kwargs={
            "year": "{{ data_interval_start.year }}",
            "month": "{{ data_interval_start.month }}",
            "day": "{{ data_interval_start.day }}",
            "hour": "{{ data_interval_start.hour }}",
            "output_path": "/tmp/wikipageviews_{{ ds_nodash }}_{{ data_interval_start.hour }}.gz",
        },
    )

    extract = BashOperator(
        task_id="extract",
        bash_command="gunzip --force /tmp/wikipageviews_{{ ds_nodash }}_{{ data_interval_start.hour }}.gz",
    )

    get_data >> extract
```
Notes:
- Note the *strings*: the values arrive in the callable as already-rendered strings (`"2024"`, `"3"`), hence the `:0>2` padding in the f-string.
- Alternatively, the PythonOperator can receive the whole context: `def _get_data(**context): start = context["data_interval_start"]`. Airflow inspects the signature and passes only the arguments you name (`def f(ds, **_)`).
- Check the output with:
```bash
airflow tasks render stocksense get_data 2024-01-01T10:00:00
```
or open the *Rendered Template* tab in the UI to confirm what the worker actually saw.

### 5. Under the hood   *(optional)*
- **Why only `template_fields`?** Rendering every string would be expensive and could mangle values that contain `{{` legitimately. Each operator class declares `template_fields` (and `template_ext` such as `.sql`, `.sh`, which makes Airflow load the file contents and render them).
- **Jinja environment** — built per DAG; its `jinja_environment_kwargs` and `template_searchpath` (extra directories for template files) can be tuned. `render_template_as_native_obj=True` returns Python objects (lists, ints) instead of strings.
- **The context** — `TaskInstance.get_template_context()` assembles keys like `ds`, `ts`, `prev_ds`, `next_ds`, `run_id`, `dag`, `task`, `conf`, `var.value`, `var.json`, `conn`, `ti`, `params`.
- **Variables and connections** — `{{ var.value.my_key }}` and `{{ conn.my_conn.host }}` fetch from the metadata DB *at render time*, which is better than fetching at top level (which would run every parse).
- **Custom operators** — subclass `BaseOperator` and set `template_fields = ("sql", "output_path")` to inherit the behaviour.
- **XCom** — `{{ ti.xcom_pull(task_ids='upstream') }}` pulls a value from an upstream task during rendering.

### 6. Performance and design implications
- Rendering is cheap; the real cost is **what you do at parse time instead**. Putting dynamic logic in templates keeps the DAG file light and the scheduler fast.
- Templated SQL files keep long queries out of Python and make them testable.
- Keep templates short; complex Jinja logic is hard to debug — move logic into a Python callable.
- Always render-check new templates with `airflow tasks render` before a big backfill.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `{{ ds }}`, `{{ ds_nodash }}` | Logical date string | File names, SQL filters |
| `{{ data_interval_start }}` / `_end` | Slice boundaries | Incremental loads |
| `{{ macros.ds_add(ds, -7) }}` | Date arithmetic | Look-back windows |
| `{{ params.name }}` | Per-DAG/task parameters | Reusable DAG factories |
| `{{ var.value.key }}` | Airflow Variable | Config without hard-coding |
| `airflow tasks render <dag> <task> <date>` | Show rendered fields | Debugging |
| `template_searchpath` | Folder with `.sql` templates | Keep SQL in files |
| `render_template_as_native_obj` | Native Python objects | Passing lists/dicts |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Templated operator arguments | Value is needed at *parse* time (e.g., to decide how many tasks to create) |
| `**context` in a Python callable | You only need one or two values (explicit args are clearer) |
| `.sql` template files | Query is a one-liner |
| Variables via `{{ var.value }}` | The value changes rarely and a constant/param would do |
| `params` | You need runtime overrides in "Trigger DAG w/ config" |

### 9. Common mistakes
- **Templating a non-template field** → placeholder printed literally (`{{ ds }}` in the command) → check the operator's `template_fields` or pass via a templated argument.
- **Using `datetime.now()` instead of `{{ ds }}`** → reruns and backfills read the wrong data → use the context.
- **Forgetting that rendered values are strings** → `int` arithmetic fails → convert, or enable native-object rendering.
- **Mistaking the logical date for "today"** → off-by-one interval → recall that `ds` = start of the interval being processed.
- **Fetching Variables/Connections at top level** → DB hit every parse → use `{{ var.value.key }}` or fetch inside the callable.
- **Escaping issues with Bash** (`${var}`, `{{`) → Jinja eats braces → use `{% raw %}…{% endraw %}` when needed.

### 10. 🎯 Interview kit
**30-second answer:** "Airflow renders Jinja templates in an operator's `template_fields` at execution time using that run's context — logical date, data interval, params. This makes each run process its own slice and keeps reruns/backfills reproducible."

**Structure a longer answer:** clarify (what changes between runs) → mechanism (template_fields, context, render before execute) → trade-offs (when logic belongs in Python) → scenario (incremental hourly download, backfill last week).

**Likely follow-ups:**
- *Why is my `{{ ds }}` not substituted?* → The argument is not in `template_fields`; or it's in a non-templated field/file extension.
- *How do I see what a task received?* → UI *Rendered Template* tab or `airflow tasks render`.
- *Difference between `ds` and `execution_date`?* → `ds` is the string form of the logical date; execution_date is the datetime (deprecated name for logical_date).
- *How do you pass data from one task to another?* → XCom for small values; storage path for big data, optionally via `ti.xcom_pull` in a template.

**Signals interviewers look for:** you explain parse vs render time; you tie templating to idempotency and backfill; you know how to debug rendering.

### 11. Check yourself
- [ ] Write a templated Bash command that writes to `/data/{{ ds }}/out.csv`.
- [ ] Explain why `template_fields` exists and how to add one to a custom operator.
- [ ] Use `data_interval_start`/`end` to build a SQL `WHERE` clause for an hourly slice.
- [ ] Show how to inspect rendered output without running the DAG.
- [ ] Describe why a `{{ ds }}` in a non-templated argument appears literally.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Templating tasks using the Airflow context
- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › Launching a Job
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Materialization
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Importing Large Objects
- *Designing Data-Intensive Applications* — Martin Kleppmann — W › Y
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Warning
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Tip
- *Hadoop: The Definitive Guide* — Tom White — Crunch › PObject
- *Hadoop: The Definitive Guide* — Tom White — G
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Java code

