---
id: "TOOLS/dbt/dbt_Core/dbt_Fundamentals"
level: 4
type: "concept"
topic: "TOOLS/dbt/dbt_Core"
tools: ["dbt"]
---

# dbt Fundamentals

**L4 Advanced** · [↑ dbt Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** dbt is the "T" of ELT: you write `SELECT` statements as version-controlled files, and dbt compiles them (with Jinja), orders them by their `ref()` dependencies, and materialises them as tables/views inside your warehouse — adding tests, docs and lineage on top.

### 1. The big picture
In **ELT**, raw data is first *extracted and loaded* into the warehouse or lakehouse, then *transformed there* using the warehouse's own compute. dbt (data build tool) does only the transformation step. It does **not** move data in, and it is not a processing engine: it generates SQL and sends it to your warehouse (Snowflake, BigQuery, Redshift, Databricks, Postgres, DuckDB, Spark …) through an **adapter**. This lets analytics engineers apply software-engineering practice — Git, code review, modular code, tests, CI/CD, documentation — to SQL transformations.

In a typical modern stack: an ingestion tool or custom job loads raw tables → dbt builds staging, intermediate and mart models → an orchestrator (Airflow, Argo, dbt Cloud scheduler) triggers `dbt build` → BI tools read the marts. dbt sits in the middle and is scheduled by something else.

### 2. Core concepts and vocabulary
- **Project** — a folder with `dbt_project.yml` and subfolders (`models/`, `tests/`, `seeds/`, `snapshots/`, `macros/`, `analyses/`).
- **Model** — a `.sql` file containing one `SELECT`; the file name is the model name; dbt wraps it in the DDL/DML needed to create the object.
- **Materialization** — how the model is built: `view`, `table`, `incremental`, `ephemeral` (plus `materialized_view`, snapshot strategies).
- **`ref('model')`** — reference another model; dbt resolves the real schema/table name for the current environment and records a dependency edge.
- **`source('src','table')`** — reference a raw table that dbt does not build; declared in YAML, can have freshness checks.
- **Seed** — a small CSV in the repo loaded as a table (lookup/mapping data).
- **Snapshot** — captures slowly-changing history (SCD type 2) of a mutable source.
- **Test** — an assertion about data (`unique`, `not_null`, `accepted_values`, `relationships`, or custom SQL).
- **Macro** — a reusable Jinja function.
- **Jinja** — the templating language mixed into SQL (`{{ }}` expressions, `{% %}` statements) evaluated at compile time.
- **Package** — a shared dbt project (e.g. `dbt_utils`) installed via `packages.yml` and `dbt deps`.
- **Profile / `profiles.yml`** — connection credentials and target (dev/prod) settings; kept *outside* the repo or filled from environment variables because it holds secrets.
- **Target** — the active environment inside a profile.
- **Artifacts** — JSON files in `target/` (`manifest.json`, `run_results.json`, `catalog.json`) describing the project and the last invocation.
- **Lineage graph (DAG)** — the dependency graph of sources, models, tests and exposures.

### 3. How it works, step by step
1. **Parse.** dbt reads `dbt_project.yml`, all `.sql` and `.yml` files, resolves macros/packages, and builds an in-memory manifest.
2. **Build the DAG.** Every `ref()` and `source()` call becomes an edge. Cycles are an error.
3. **Compile.** Jinja is rendered: `ref()` becomes a fully-qualified name, `{% if is_incremental() %}` branches are resolved, macros expand. Compiled SQL is written to `target/compiled/`.
4. **Wrap.** For each selected model, dbt generates DDL/DML from its materialization — e.g. `create or replace table x as (<your select>)` or a `merge` for incrementals. This wrapped statement is written to `target/run/`.
5. **Execute in dependency order**, in parallel across independent branches up to `threads`.
6. **Record.** Results (status, timing, rows affected) go to `target/run_results.json`; the manifest to `target/manifest.json`.

### 4. Worked example
```yaml
# models/staging/_sources.yml
version: 2
sources:
  - name: shop
    schema: raw_shop
    tables:
      - name: orders
```
```sql
-- models/staging/stg_orders.sql
{{ config(materialized='view') }}
select
    id          as order_id,
    customer_id,
    cast(created_at as timestamp) as ordered_at,
    amount_cents / 100.0          as amount
from {{ source('shop', 'orders') }}
```
```sql
-- models/marts/fct_daily_revenue.sql
{{ config(materialized='table') }}
select date_trunc('day', ordered_at) as day, sum(amount) as revenue
from {{ ref('stg_orders') }}
group by 1
```
```bash
dbt debug            # verify profile, connection, project
dbt deps             # install packages
dbt build            # run + test (+ seed + snapshot) in DAG order
dbt docs generate && dbt docs serve   # browse docs and lineage
```
What you will see: `stg_orders` is created as a view first, then `fct_daily_revenue` as a table, because the `ref()` makes it downstream. If you change the target from `dev` to `prod`, the same code writes to a different schema with no edit to the model.

### 5. Under the hood
- dbt is mostly a **compiler + dependency scheduler**. The warehouse does all the data work; dbt only holds results metadata.
- Because models are `SELECT`s, dbt does the DDL: for `table` it typically creates a temp/intermediate relation and swaps it in (adapter-specific; on Snowflake `create or replace`), so downstream readers see old or new, not half.
- The **manifest** is the project's complete graph and metadata. `state:modified` selection (Slim CI) compares the current manifest to a stored production manifest.
- `ref()` is also what makes environments work: it uses the active target's database/schema and the `generate_schema_name` macro, so each developer gets an isolated schema.
- `run_results.json` has one entry per node with `status`, `execution_time`, `adapter_response` (rows, bytes billed) — useful for performance monitoring and alerting.
- Jinja can only produce SQL text at compile time; it cannot see data except via `run_query()`/`execute` during the execution phase, so avoid data-dependent branching that is not deterministic.

### 6. Performance and design implications
- Cost and speed are those of your warehouse; dbt's job is to structure and limit what runs (selection, incrementals, `threads`).
- Layer models: **staging** (1:1 with sources, light cleaning, views) → **intermediate** (joins/logic) → **marts** (business-facing facts/dimensions, tables). This keeps logic reusable and each model small and testable.
- Set `threads` to fit warehouse concurrency; more is not always faster.
- Use variables (`--vars '{"start_date": "2026-01-01"}'`) with `env_var()` for parameterising runs (e.g. backfill ranges) rather than editing code.
- Orchestration: Airflow can call `dbt build` (BashOperator/Cosmos, or dbt Cloud API). Team choice trade-off: Airflow gives unified scheduling with ingestion tasks and custom alerting; dbt Cloud gives integrated CI, hosting and docs with less ops work.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `dbt debug` | checks profile, connection, dependencies | first-time setup, connection errors |
| `dbt deps` | installs packages | after changing `packages.yml` |
| `dbt compile` | renders SQL only | inspect Jinja output |
| `dbt run` | builds models only | models without tests |
| `dbt test` | runs data tests | after models exist |
| `dbt build` | seeds + models + snapshots + tests in DAG order | default in CI/prod |
| `dbt seed`, `dbt snapshot` | load CSVs / capture history | reference data, SCD2 |
| `dbt docs generate/serve` | docs site with lineage | onboarding, review |
| `dbt source freshness` | check source recency | SLA on raw data |
| `profiles.yml` | connection + targets | keep secrets out of Git |
| `dbt_project.yml` | project config, default materializations | project-wide settings |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| dbt for SQL-centred transformations in a warehouse/lakehouse | you need row-level streaming or heavy Python/ML processing (use Spark/Flink; dbt Python models are limited) |
| `view` for light staging | the view is heavy and queried often |
| `table` for expensive, widely-used marts | data is huge and mostly unchanged (use incremental) |
| dbt Cloud for turnkey CI/scheduling | you already operate Airflow and want one control plane |
| Packages like `dbt_utils` | adding many third-party packages with unpinned versions |

### 9. Common mistakes
- Hardcoding `schema.table` instead of `ref()`/`source()` → breaks lineage and environments → always use the functions.
- Committing `profiles.yml` credentials → secret leak → use env vars / secrets store.
- Treating dbt as an ingestion tool → no data arrives → load raw data first with an EL tool.
- Business logic in staging, or one giant model → untestable → use layers and small models.
- Running `dbt run` only → tests never execute → use `dbt build`.
- Confusing `run` with `build`: `run` executes models only; `build` executes models, tests, seeds and snapshots in dependency order, stopping downstream of a failed test.

### 10. 🎯 Interview kit
**30-second answer:** dbt does the T in ELT. You write SELECT statements with Jinja, dbt resolves `ref()` dependencies into a DAG, compiles to SQL, and runs it in your warehouse in order, materialising tables or views, with built-in tests, docs and lineage.

**Structure a longer answer:** clarify (ELT vs ETL, which warehouse) → compile/DAG/execute flow → materializations and layering → tests/docs/CI → trade-offs and orchestration.

**Likely follow-ups:**
- *ref() vs source()?* → `ref` points to dbt-built models and creates dependencies; `source` points to raw tables dbt doesn't build, with freshness and docs.
- *What does dbt run do?* → compiles and executes selected models in dependency order.
- *Why not commit profiles.yml?* → contains credentials; use env vars.
- *What's in run_results.json?* → per-node status, timing, adapter response.
- *How to run dbt from Airflow?* → a task calling `dbt build` with selectors, or dbt Cloud API; one task per layer or per model with Cosmos.

**Signals interviewers look for:** knowing dbt does not load or process data itself, comfort with the compile → DAG → execute model, and practical environment/secrets handling.

### 11. Check yourself
- [ ] Explain ELT and exactly what dbt does and does not do.
- [ ] Walk through what happens between `dbt build` and the first SQL sent to the warehouse.
- [ ] Contrast `ref()` and `source()` and `run` and `build`.
- [ ] Sketch a staging → intermediate → mart layout for orders data.
- [ ] Explain how dev and prod use the same code safely.
<!-- explainer:end -->

## Implementations (tools)

- [dbt](<../_index.md>)

## Practice questions

- [Basic] [What is dbt and what problem does it solve in a modern data stack?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22648_What_is_dbt_and_what_problem_does_it_solve_in_a_modern_data_.md>)
- [Basic] [What does the command dbt run actually do?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22657_What_does_the_command_dbt_run_actually_do.md>)
- [Basic] [What is Jinja and how does dbt combine it with SQL?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22684_What_is_Jinja_and_how_does_dbt_combine_it_with_SQL.md>)
- [Basic] [What does the dbt debug command check and when would you run it?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22741_What_does_the_dbt_debug_command_check_and_when_would_you_run.md>)
- [Intermediate] [What is the dbt lineage graph and why is it valuable for a data team?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22691_What_is_the_dbt_lineage_graph_and_why_is_it_valuable_for_a_d.md>)
- [Intermediate] [What does the run_results.json artifact contain after a dbt run?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22723_What_does_the_run_results_json_artifact_contain_after_a_dbt_.md>)
- [Intermediate] [How do you use vars and env_var together to parameterize a run for backfill date ranges without editing model code?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22839_How_do_you_use_vars_and_env_var_together_to_parameterize_a_r.md>)
- [Deep] [How can you orchestrate dbt runs from Airflow, and why might a team choose that over dbt Cloud scheduling?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22734_How_can_you_orchestrate_dbt_runs_from_Airflow_and_why_might_.md>)

## Related concepts

- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Linting YAML](<../../YAML/YAML_Core/Linting_YAML.md>) *(YAML Core)*
- [Versioning SQL and Migrations](<../../Git_and_GitHub/Git_Core/Versioning_SQL_and_Migrations.md>) *(Git Core)*
- [Schema Migration Risks](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*
- [Pipeline Design Patterns](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*

## References

- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 7Scaling Data Solutions › CHAPTER 7Scaling Data Solutions(#contents.xhtmlch07a)
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › dbt Unit Testing
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering › Operations Core Overview
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Glossary
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Data Transformation Frameworks and Data Quality Tools
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › User-Defined Data Types (UDTs)

