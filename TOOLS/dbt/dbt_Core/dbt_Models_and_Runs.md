---
id: "TOOLS/dbt/dbt_Core/dbt_Models_and_Runs"
level: 4
type: "concept"
topic: "TOOLS/dbt/dbt_Core"
tools: ["dbt"]
---

# dbt Models and Runs

**L4 Advanced** · [↑ dbt Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A dbt model is a SELECT whose *materialization* (view, table, incremental, ephemeral) decides how and how expensively it is built, and dbt orders and runs models by the dependency graph that `ref()` creates.

### 1. The big picture
After the basics, day-to-day dbt work is deciding **how each model is stored and refreshed**, **how dependencies are tracked**, and **how runs are structured** (selection, cadence, CI, performance). Dependencies matter because, as the pipeline books stress, handling dependencies at scale must not be left to manual checks: dbt derives the graph from the code itself so a model can never be built before the models it reads.

### 2. Core concepts and vocabulary
- **Materialization** — build strategy for a model:
  - `view` — a stored query; no data copied, always fresh, cost paid at read time.
  - `table` — full rebuild each run; fast reads, cost paid at build.
  - `incremental` — only new/changed rows are processed and merged/appended into an existing table.
  - `ephemeral` — inlined as a CTE into downstream models; no object created.
  - `materialized_view` — warehouse-maintained result where supported.
- **`is_incremental()`** — Jinja test true when the target exists, the model is incremental, and no full refresh is requested.
- **`{{ this }}`** — the relation being built; used to find the current max timestamp.
- **`unique_key`** — columns that identify a row so incrementals can update rather than duplicate.
- **Incremental strategy** — `append`, `merge`, `delete+insert`, `insert_overwrite` (adapter-dependent).
- **Snapshot** — SCD2 history table with `dbt_valid_from`/`dbt_valid_to`.
- **Tags / selectors** — labels used to run subsets (e.g. hourly vs daily).
- **Contracts and constraints** — enforced column names/types on a model.
- **Exposure** — declared downstream consumer (dashboard) in the graph.

### 3. How it works, step by step
**Dependency graph**
1. dbt scans each model for `ref()`/`source()` calls (by rendering Jinja at parse time).
2. Each call adds an edge; the result is a DAG with sources at the left.
3. At run time dbt topologically sorts it and runs independent nodes in parallel, up to `--threads`.
4. If a model fails, its downstream models are skipped; tests with `dbt build` stop dependants of failing data.

**Incremental run**
1. First run: table does not exist → whole SELECT runs → table created.
2. Later runs: `is_incremental()` is true → the `where` filter limits input to new rows (e.g. `updated_at > (select max(updated_at) from {{ this }})`).
3. dbt builds a temp relation from the filtered SELECT, then applies the strategy: `merge` on `unique_key` (upsert), `delete+insert`, or append.
4. `--full-refresh` drops/rebuilds the table from scratch.

### 4. Worked example
```sql
-- models/marts/fct_events.sql
{{
  config(
    materialized='incremental',
    unique_key='event_id',
    incremental_strategy='merge',
    on_schema_change='append_new_columns'
  )
}}

select
    event_id,
    user_id,
    event_type,
    event_ts
from {{ ref('stg_events') }}

{% if is_incremental() %}
  -- look back 3 days to catch late-arriving rows
  where event_ts >= (select dateadd('day', -3, max(event_ts)) from {{ this }})
{% endif %}
```
```bash
dbt build --select fct_events+            # model and everything downstream
dbt build --select fct_events --full-refresh
dbt compile --select fct_events           # inspect rendered SQL
```
Reading it: on the first run the `where` is omitted and all history is loaded. On later runs only the last three days of `stg_events` are scanned and merged by `event_id`, so late rows are corrected without a full rebuild. The 3-day window is a deliberate trade-off between correctness for late data and cost.

### 5. Under the hood
- The **ref graph** is also used for lineage and state-based selection. `dbt run` does not "know" data; it knows SQL text and edges.
- With `merge`, the adapter generates a `MERGE INTO target USING temp ON unique_key WHEN MATCHED UPDATE … WHEN NOT MATCHED INSERT …`. Without a `unique_key` and with `append`, reruns duplicate rows — so incrementals are not automatically idempotent; *you* make them so with the filter and key.
- `insert_overwrite` (BigQuery/Spark) replaces whole partitions, which is cheap and naturally idempotent for partition-aligned data.
- Ephemeral models are text substitution: every consumer repeats the CTE, which can duplicate work and make debugging harder.
- Snapshots use either `timestamp` (compare an `updated_at`) or `check` (compare columns) strategies; they must run on a schedule — history is only as granular as the run frequency.
- Debugging a run: compiled SQL in `target/compiled`, wrapped SQL in `target/run`, warehouse query history for the actual statement and plan.

### 6. Performance and design implications
- **Profile the slowest node**: `run_results.json` gives per-model timing; then inspect the warehouse query profile of that model. A window-function-heavy intermediate model can often be fixed by filtering earlier, pre-aggregating, avoiding repeated sorts, changing materialization to `table`, or clustering/partitioning the output.
- Prefer incrementals for large append-mostly fact tables; keep dimensions and small aggregates as tables.
- Split cadences with **tags** (`dbt build --select tag:hourly`) and run `tag:daily` separately; keep graph consistency by selecting with `+` (upstream) or using `state:modified+` and `--defer` so a daily model that reads hourly models does not run against stale or missing parents.
- **CI on pull requests**: build modified models and their children into a per-PR schema with `--select state:modified+ --defer --state <prod-artifacts>`, run tests, and fail the check on errors (Slim CI).
- **Metric drift**: define each KPI once in a mart (or semantic layer / metrics), have dashboards read it, and use `exposures` plus tests so changes are visible.
- **Legacy migration**: move stored procedures one at a time — reproduce output as a dbt model, compare results (e.g. `audit_helper`), switch consumers, retire the proc; no big bang.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `materialized=` | pick build strategy | per model or folder in `dbt_project.yml` |
| `unique_key`, `incremental_strategy` | upsert behaviour | incremental tables |
| `on_schema_change` | `ignore`/`append_new_columns`/`sync_all_columns`/`fail` | evolving schemas |
| `--full-refresh` | rebuild incrementals | logic change, drift |
| `--threads N` | parallel nodes | tune to warehouse |
| `--select`, `--exclude`, tags | subset runs | cadence, dev |
| `--defer --state path` | use prod relations for unbuilt parents | CI, dev |
| `dbt snapshot` | capture SCD2 history | tracking status changes |
| `dbt ls` | list selected nodes | verifying selectors |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `view` for staging/light logic | many downstream queries repeatedly pay the cost |
| `table` for moderate data and heavy reads | table is huge and mostly static — rebuild cost dominates |
| `incremental` for large, append/late-update data | source has no reliable change column, or deletes must propagate |
| `ephemeral` for tiny reusable fragments | it is referenced many times or needs debugging |
| dbt snapshot for source history you don't retain | you already have CDC/event history (build SCD2 from it, incrementally) |

### 9. Common mistakes
- Incremental with no `unique_key` and append → duplicates on rerun → add key/merge or partition overwrite.
- Filtering with `>` max timestamp only → late rows lost → use lookback window or reprocess partitions.
- Forgetting `--full-refresh` after changing logic → old rows keep old logic → refresh or backfill.
- Everything as `table` → needless cost; everything as `view` → slow dashboards → choose per layer.
- Same KPI computed in multiple models → drift → single definition in one mart/metric.
- Treating `dbt build` as "compile only": it executes; use `dbt compile` to only render SQL.

### 10. 🎯 Interview kit
**30-second answer:** dbt builds a DAG from `ref()` calls and executes models in order, using each model's materialization — view, table, incremental or ephemeral — to decide how it's persisted. Incrementals process only new data and merge on a unique key; you control correctness with the filter and key.

**Structure a longer answer:** clarify data volume and freshness → choose materialization by layer → incremental design (key, filter, late data, schema change) → selection/cadence/CI → monitoring and cost.

**Likely follow-ups:**
- *How does dbt create the DAG?* → parses `ref`/`source` calls; topological sort.
- *Test failing unexpectedly — debug?* → `dbt test --select`, inspect compiled test SQL / `--store-failures`, check upstream data and recent model changes.
- *Slowest model?* → timing from `run_results.json`, warehouse profile, restructure/materialize/partition.
- *Snapshot vs incremental SCD2?* → snapshot is simplest for sources with no history; incrementals over CDC give control and replay.
- *Hourly vs daily cadences?* → tags + selection with `+`/state/defer to keep dependencies consistent.

**Signals interviewers look for:** idempotence reasoning for incrementals, awareness of late data, cost/perf thinking, CI strategy.

### 11. Check yourself
- [ ] Write an incremental model with merge and a 3-day lookback.
- [ ] Explain what happens on run 1 vs run 2 and on `--full-refresh`.
- [ ] Choose materializations for staging, intermediate and marts and justify.
- [ ] Describe a Slim CI setup for pull requests.
- [ ] Explain how to avoid KPI drift across dashboards.
<!-- explainer:end -->

## Implementations (tools)

- [dbt](<../_index.md>)

## Practice questions

- [Basic] [How does dbt build a dependency graph (DAG) between models?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22656_How_does_dbt_build_a_dependency_graph_DAG_between_models.md>)
- [Intermediate] [How would you debug a dbt test that is unexpectedly failing?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22740_How_would_you_debug_a_dbt_test_that_is_unexpectedly_failing.md>)
- [Intermediate] [An analyst wrote a window-function-heavy intermediate model that's now the slowest node in the DAG. How do you profile and optimize it while keeping logic readable?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22820_An_analyst_wrote_a_window-function-heavy_intermediate_model_.md>)
- [Deep] [How would you set up CI so dbt models and tests run automatically on a pull request?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22732_How_would_you_set_up_CI_so_dbt_models_and_tests_run_automati.md>)
- [Deep] [How do you decide between a dbt snapshot and an incrementally-built SCD Type 2 fact for tracking customer status history, considering reprocessing and auditability?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22758_How_do_you_decide_between_a_dbt_snapshot_and_an_incrementall.md>)
- [Deep] [How do you prevent metric drift where the same KPI is computed three different ways across dashboards, using dbt's structure and governance features?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22781_How_do_you_prevent_metric_drift_where_the_same_KPI_is_comput.md>)
- [Deep] [How would you split your DAG into hourly and daily job cadences using tags, and how do you keep dependencies consistent so an hourly model never reads a stale daily parent?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22804_How_would_you_split_your_DAG_into_hourly_and_daily_job_caden.md>)
- [Deep] [How would you migrate a legacy warehouse of stored procedures into dbt incrementally without a big-bang cutover?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22828_How_would_you_migrate_a_legacy_warehouse_of_stored_procedure.md>)

## Related concepts

- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Modeling Relationships and Schemas](<../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>) *(Schema Design)*
- [Incremental Rollups](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>) *(SQL Fundamentals)*
- [Idempotent Incremental Loads](<../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>) *(Idempotent Incremental Loads)*
- [Data Warehouse Fundamentals](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Best Practices for Maintaining Pipelines › Ensuring Dependency Integrity
- *Data Engineering with Medallion Architecture* — Miki Eto — Building Scalable Data Pipelines › Concrete example
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › dbt Unit Testing
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Conclusion
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data-Driven Decision Making › Wrapping Up
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Overview of Principles › Run Experiments in Production

