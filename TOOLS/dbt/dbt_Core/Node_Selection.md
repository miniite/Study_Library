---
id: "TOOLS/dbt/dbt_Core/Node_Selection"
level: 4
type: "concept"
topic: "TOOLS/dbt/dbt_Core"
tools: ["dbt"]
---

# Node Selection

**L4 Advanced** · [↑ dbt Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** dbt's node-selection syntax (`--select`, `--exclude`, graph operators, tags, state) lets you run exactly the slice of the DAG you mean — one model, a model plus everything it feeds, or only what changed — instead of the whole project.

### 1. The big picture
Every dbt command that touches nodes (`run`, `test`, `build`, `seed`, `snapshot`, `ls`, `compile`, `docs generate`) accepts a **selection**. Selecting is what makes dbt usable in development (build only my model and its parents), in CI (only modified models and dependants) and in production (hourly vs daily subsets). A precise mental model of the selection operators prevents both "I rebuilt the world" and "I forgot to rebuild a downstream model".

### 2. Core concepts and vocabulary
- **Node** — any resource in the graph: model, seed, snapshot, test, source, exposure.
- **`--select` / `-s`** — which nodes to include.
- **`--exclude`** — nodes to remove from the selected set.
- **Graph operators** — `+` (ancestors/descendants), `@` (node, descendants, and the ancestors of those descendants), `n+` (limit depth).
- **Methods** — `tag:`, `path:`, `package:`, `config:`, `resource_type:`, `source:`, `test_type:`, `state:`, `result:`, `fqn` by name.
- **Intersection / union** — comma `a,b` = intersection (AND); space `a b` = union (OR).
- **`selectors.yml`** — named, reusable selection definitions.
- **State** — comparison with a previous run's `manifest.json` (`--state path`).
- **Defer** — resolve unselected upstream `ref()`s to another environment's relations (`--defer`).

### 3. How it works, step by step
1. dbt parses the project and builds the full DAG.
2. It evaluates each selection argument into a set of nodes, applying graph operators against that DAG.
3. It unions space-separated criteria, intersects comma-separated ones, then subtracts `--exclude`.
4. The remaining nodes are filtered by resource type for the command (`run` → models; `test` → tests; `build` → all runnable types).
5. They execute in dependency order.

| Syntax | Meaning |
|---|---|
| `--select orders` | just model `orders` |
| `--select +orders` | `orders` and all its ancestors (everything it needs) |
| `--select orders+` | `orders` and all descendants (everything that depends on it) |
| `--select +orders+` | ancestors, node, descendants |
| `--select 1+orders` / `orders+2` | limit to one parent level / two child levels |
| `--select @orders` | node + descendants + the other ancestors those descendants need (CI "build everything this change affects") |
| `--select tag:daily` | everything tagged `daily` |
| `--select path:models/marts` or `models/marts` | folder |
| `--select tag:daily,config.materialized:incremental` | AND |
| `--select orders customers` | OR |
| `--select state:modified+ --state ./prod-artifacts` | changed nodes and descendants |
| `--exclude tag:slow` | remove tagged nodes |

### 4. Worked example
```bash
# list first, without running anything
dbt ls --select stg_orders+ --resource-type model

# daily job, skip slow nightly models
dbt build --select tag:daily --exclude tag:nightly

# development: build my model and what it needs
dbt build --select +fct_orders

# CI: only modified nodes and descendants, parents taken from prod
dbt build --select state:modified+ --defer --state ./prod-artifacts

# retry only what failed in the previous invocation
dbt build --select result:error+ --state ./target
```
`dbt ls` is the safe way to preview a selector. In the CI example, unchanged parents are not rebuilt: with `--defer`, `ref('dim_customers')` resolves to the production table, so a PR schema contains only what changed.

### 5. Under the hood
- Selection happens on the manifest graph, not on files: a file path is just one *method* to match node attributes.
- `+` walks edges in the compiled DAG, so it follows `ref()` and `source()` edges and also test edges (tests attach to the nodes they test, so `model+` includes its tests when running `build`).
- `state:modified` compares each node's checksum/config (and macros it depends on) between the current manifest and the reference manifest; with `state:modified+` descendants of changed nodes are included.
- `--exclude` is applied last, so it can remove nodes pulled in by `+`. A pitfall is that excluding an upstream of a selected node leaves the downstream node building on *stale* relations.
- `selectors.yml` allows `union`/`intersection`/`exclude` composition and a `default: true` selector for jobs.

### 6. Performance and design implications
- Cost control: in a large project a full `dbt build` can take hours; selection (and incrementals) bounds cost.
- Cadence design: tag by freshness need (`hourly`, `daily`), and select with the graph operators so required parents are included — or run parents on their own cadence and accept staleness knowingly.
- CI speed: Slim CI (`state:modified+` + `--defer`) typically cuts PR builds from the whole DAG to a handful of models.
- Safety: always `dbt ls --select …` before running a destructive selection such as `--full-refresh`.
- Keep tags/paths meaningful and few; selectors in YAML are easier to review than ad-hoc shell strings.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `dbt ls --select X` | preview selection | always before big runs |
| `--select` / `-s` | include nodes | all commands |
| `--exclude` | subtract nodes | skip slow/flaky |
| `+`, `@`, `n+` | graph traversal | parent/child scope |
| `tag:`, `path:`, `config:` | attribute methods | cadence/folder/config |
| `state:modified`, `result:` | change- and outcome-based | CI, retries |
| `--defer --state` | borrow other env's parents | PR/dev builds |
| `selectors.yml` | named selection | production jobs |
| `--resource-type` | filter types | `ls`, `build` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `+model` in dev to ensure parents exist | warehouse cost matters and parents are huge (use defer) |
| `model+` after changing logic | you only want to inspect one node |
| `@model` in CI for complete correctness | the project is huge — it can pull in many ancestors |
| `state:modified+` with defer | you have no stored production manifest |
| tags for cadences | tags drift out of date; prefer folder structure plus a few tags |

### 9. Common mistakes
- Believing `run` vs `build` differ in compile vs execute → both execute; `build` additionally runs tests, seeds and snapshots in DAG order.
- Confusing `--select` and `--exclude` → they are include and subtract, applied in that order, not "compile vs skip".
- Space vs comma → `a b` is OR, `a,b` is AND.
- Running `model` after changing a parent but not `model+` → stale downstream tables → select with `+`.
- Excluding an upstream and expecting downstream to be unaffected → builds on old data.
- Running selectors without `dbt ls` → rebuilding more (or less) than intended.

### 10. 🎯 Interview kit
**30-second answer:** `--select` chooses nodes by name, path, tag, config, state and graph operators; `--exclude` subtracts. `+model` adds ancestors, `model+` adds descendants, comma is intersection and space is union. `state:modified+` with `--defer` powers fast CI.

**Structure a longer answer:** clarify the use case (dev, CI, prod cadence) → show the operators → explain state/defer → note pitfalls (stale parents) → close with a concrete command set.

**Likely follow-ups:**
- *run vs build?* → build = run + test + seed + snapshot in DAG order; failures skip dependants.
- *select vs exclude?* → include set vs subtraction applied afterwards.
- *How do you run only what changed?* → `state:modified+` with a prod manifest.
- *How do you re-run failures?* → `result:error+` against the previous `run_results`.
- *How to preview?* → `dbt ls`.

**Signals interviewers look for:** fluent use of graph operators, awareness of stale-parent risk, and a CI strategy that avoids full rebuilds.

### 11. Check yourself
- [ ] Write selectors for: a model and its parents; its children; only tests for one model; all incrementals tagged daily.
- [ ] Explain the difference between `a b` and `a,b`.
- [ ] Describe how `state:modified+` and `--defer` work together.
- [ ] Say what `dbt build --select +orders --exclude stg_orders` does.
- [ ] Explain why `dbt ls` should precede a large run.
<!-- explainer:end -->

## Implementations (tools)

- [dbt](<../_index.md>)

## Practice questions

- [Basic] [What is the difference between dbt run and dbt build?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22658_What_is_the_difference_between_dbt_run_and_dbt_build.md>)
- [Basic] [What is the difference between the --select and --exclude flags?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22707_What_is_the_difference_between_the_--select_and_--exclude_fl.md>)
- [Intermediate] [How do you select which models to run using the --select flag in dbt?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22703_How_do_you_select_which_models_to_run_using_the_--select_fla.md>)
- [Intermediate] [What are dbt tags and how can you use them to run a subset of models?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22706_What_are_dbt_tags_and_how_can_you_use_them_to_run_a_subset_o.md>)
- [Intermediate] [How does dbt decide the order to build models when one depends on another?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22736_How_does_dbt_decide_the_order_to_build_models_when_one_depen.md>)
- [Intermediate] [Explain how a selectors.yml file helps you maintain complex, named model selections for different scheduled runs instead of long CLI strings.](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22803_Explain_how_a_selectors_yml_file_helps_you_maintain_complex_.md>)
- [Deep] [What does the graph operator + mean in a dbt selector, for example my_model+ versus +my_model?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22704_What_does_the_graph_operator_mean_in_a_dbt_selector_for_exam.md>)
- [Deep] [Explain the difference between generate_schema_name and generate_database_name overrides, and why teams override the default schema-prefixing behavior.](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22768_Explain_the_difference_between_generate_schema_name_and_gene.md>)

## Related concepts

- [SQL vs NoSQL](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL.md>) *(SQL Fundamentals)*
- [Data Vault](<../../../02_Data_Engineering/Modeling_and_Warehousing/Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*

