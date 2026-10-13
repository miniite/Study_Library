---
id: "TOOLS/Apache_Spark/01_Spark_Fundamentals/Spark_SQL_and_Catalyst/Catalyst_and_Tungsten"
level: 6
type: "curriculum"
curriculum: "data_engineering"
---

# Catalyst Optimizer and Tungsten

**L6 Internals** · [↑ Spark SQL and Data Sources](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Catalyst turns your query into an optimized physical plan through four phases (analysis, logical optimization, physical planning, code generation), and Tungsten makes executing that plan fast with off-heap binary memory and whole-stage code generation.

### 1. The big picture
Whether you write SQL, the DataFrame API in Python, Scala or R, Spark SQL sends your query through the **same pipeline**. That is why PySpark DataFrame code is about as fast as Scala: the optimizer produces equivalent plans and bytecode. Two projects share the work:

- **Catalyst** — the *query optimizer*: a rule-based (plus cost-based) framework that rewrites trees representing the query.
- **Tungsten** — the *execution engine improvements*: memory layout, cache-friendly operators, and whole-stage code generation.

The only case where language matters is code outside the plan: Python UDFs or RDD lambdas run row by row in a Python process and are opaque to Catalyst.

### 2. Core concepts and vocabulary
- **Unresolved logical plan** — parsed query with names not yet bound to tables/columns.
- **Analyzed logical plan** — names/types resolved using the **Catalog**.
- **Optimized logical plan** — after rule-based rewrites.
- **Physical plan** — chosen operators (e.g., `BroadcastHashJoin`, `HashAggregate`, `Exchange`).
- **Rule / rule batch** — a transformation applied to a tree until fixed point.
- **Predicate pushdown / column pruning / constant folding** — classic optimizations.
- **Cost-based optimizer (CBO)** — uses table/column statistics to choose join order and strategy.
- **AQE (adaptive query execution)** — re-optimizes at stage boundaries using runtime stats.
- **Whole-stage code generation** — fuses a pipeline of operators into one generated Java function.
- **UnsafeRow** — Tungsten's compact binary row format in off-heap/on-heap memory.
- **Vectorized reader** — reads Parquet/ORC in column batches.

### 3. How it works, step by step
**Catalyst's four phases**
1. **Analysis** — resolve table and column names, check types and functions using the Catalog; produce the analyzed logical plan.
2. **Logical optimization** — apply rule batches: constant folding, predicate pushdown, column pruning, combining projections, null propagation, join reordering (CBO), subquery rewrites.
3. **Physical planning** — generate one or more physical plans using *strategies*, pick the cheapest by cost model (e.g., broadcast vs sort-merge join).
4. **Code generation** — whole-stage codegen compiles the stage's operator pipeline to Java bytecode using Janino.

**Tungsten's execution**
1. Data is held in binary `UnsafeRow`s, avoiding JVM object overhead and GC pressure.
2. Operators are fused (`WholeStageCodegen`) so there are no virtual calls per row/operator.
3. Hot loops keep values in CPU registers/caches.

### 4. Worked example
```python
q = (spark.read.parquet("/data/sales")
        .join(spark.read.parquet("/data/stores"), "store_id")
        .filter("year = 2024 and region = 'EMEA'")
        .groupBy("category").sum("amount"))
q.explain(True)
```
`explain(True)` prints four sections: *Parsed*, *Analyzed*, *Optimized* and *Physical*. What to look for:

- In the **optimized** plan the `Filter` has moved *below* the `Join` (predicate pushdown), and `Project` keeps only used columns (column pruning).
- In the **physical** plan, the Parquet scan lists `PushedFilters: [IsNotNull(year), EqualTo(year,2024)]` and `ReadSchema` containing only needed columns; the small `stores` table is broadcast: `BroadcastHashJoin ... BuildRight`.
- Operators prefixed with `*(1)`, `*(2)` are inside whole-stage codegen stages.

```text
*(3) HashAggregate(keys=[category], functions=[sum(amount)])
+- Exchange hashpartitioning(category, 200)
   +- *(2) HashAggregate(keys=[category], functions=[partial_sum(amount)])
      +- *(2) Project [category, amount]
         +- *(2) BroadcastHashJoin [store_id], [store_id], Inner, BuildRight
            :- *(2) Filter (isnotnull(year) AND (year = 2024))
            :  +- *(2) FileScan parquet [store_id,year,category,amount] PushedFilters: [...]
            +- BroadcastExchange ...
```
Compare against a version using a Python UDF in the filter: pushdown disappears and a `BatchEvalPython` node appears.

### 5. Under the hood   *(L6)*
- **Trees and rules:** plans are immutable `TreeNode`s; rules are partial functions `plan => plan` executed by a `RuleExecutor` in batches until the tree stops changing (fixed point) or an iteration limit hits.
- **Physical plan selection:** `SparkStrategies` pattern-match logical nodes to physical ones (`JoinSelection` decides broadcast hash / shuffle hash / sort-merge / nested loop based on size hints, join type and config thresholds like `spark.sql.autoBroadcastJoinThreshold`, default 10 MB).
- **Exchange insertion:** `EnsureRequirements` adds shuffles/sorts so children satisfy required distribution and ordering.
- **AQE (3.x):** after each shuffle stage completes, real sizes are known; the plan can switch sort-merge → broadcast, coalesce small shuffle partitions, and split skewed partitions.
- **Codegen limits:** methods over 8 KB bytecode aren't JIT-compiled by HotSpot; Spark falls back to interpretation when the generated function is too big (`spark.sql.codegen.hugeMethodLimit`) or a stage has too many fields (`spark.sql.codegen.maxFields`).
- **Memory:** Tungsten manages memory explicitly (pages from the unified memory manager, `sun.misc.Unsafe` access), uses cache-aware sorting (prefix sort of key + pointer), and compact column batches for caching/reading.
- **Statistics:** `ANALYZE TABLE … COMPUTE STATISTICS [FOR COLUMNS …]` populates sizes, distinct counts, min/max, histograms for CBO.

### 6. Performance and design implications
- **Prefer built-in functions** over UDFs so Catalyst can optimise and codegen can fuse them; if you need Python, use pandas UDFs (Arrow batches).
- **Store columnar, partitioned data** (Parquet/ORC/Delta) so pushdown and pruning work.
- **Compute statistics** for large tables and enable CBO when join order matters.
- **Read the plan, not the code:** look for `Exchange` count, join strategy, scan filters, spill.
- **Avoid wide schemas in one stage** (hundreds of columns) that disable codegen.
- **Use AQE** (on by default from 3.2) for skew and partition coalescing.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.explain(True)` / `explain("formatted")` | Show all plan phases | Always when tuning |
| `EXPLAIN COST SELECT …` | Plan with statistics | Check CBO inputs |
| `ANALYZE TABLE t COMPUTE STATISTICS FOR ALL COLUMNS` | Collect stats | Before CBO tuning |
| `spark.sql.cbo.enabled` | Enable cost-based optimizer | Complex joins |
| `spark.sql.autoBroadcastJoinThreshold` | Max size to broadcast | Join strategy |
| `spark.sql.adaptive.enabled` | AQE | Runtime re-optimization |
| `spark.sql.codegen.wholeStage` | Toggle codegen | Debugging codegen issues |
| `df.queryExecution.debug.codegen()` (Scala) | Print generated Java | Deep dives |
| Hints: `broadcast(df)`, `/*+ BROADCAST(t) */` | Force strategy | When stats are missing/wrong |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Built-in SQL functions | A UDF is unavoidable — use pandas UDF |
| CBO + statistics | Statistics are stale — plans can get worse |
| Join hints | You can fix stats instead; hints can become wrong as data grows |
| Whole-stage codegen | Debugging or huge generated methods (fallback) |
| AQE | Very short queries — overhead is negligible but the benefit is small |

### 9. Common mistakes
- **Python UDF in a filter** → no pushdown, row-by-row serialization → rewrite with built-ins.
- **Casting partition/filter columns** (`WHERE CAST(dt AS INT)`) → blocks pushdown → compare in native type.
- **Expecting different speed for SQL vs DataFrame** → they share a plan; compare `explain` instead.
- **Stale or missing statistics** → bad join order/broadcast → `ANALYZE TABLE`.
- **Reading the unoptimized plan** → mistaken diagnosis → look at the *physical* plan or the UI SQL tab.

### 10. 🎯 Interview kit
**30-second answer:** "Catalyst parses and analyzes the query, applies rule-based and cost-based logical optimizations like predicate pushdown and column pruning, picks a physical plan such as a broadcast join, and Tungsten executes it using binary memory and whole-stage code generation."

**Structure a longer answer:** four phases → one example of each optimization → how to verify with `explain` → what breaks it (UDFs, casts, stale stats) → AQE.

**Likely follow-ups:**
- *Why is PySpark DataFrame code as fast as Scala?* → Same optimizer and engine; only UDFs leave it.
- *What is whole-stage codegen?* → Fusing operators into one generated function to avoid virtual calls and keep data in registers.
- *How does Spark choose a join?* → Size estimates/hints → broadcast, shuffle hash, sort-merge.
- *What does AQE add?* → Runtime re-planning with actual shuffle statistics.

**Signals interviewers look for:** correct phase names; reading `explain`; knowing UDF penalty; awareness of statistics and AQE.

### 11. Check yourself
- [ ] Name the four Catalyst phases and what each outputs.
- [ ] Show a query whose filter is pushed down and one where it cannot be.
- [ ] Explain Tungsten's two main ideas.
- [ ] Interpret `*(2)` markers and `Exchange` in a physical plan.
- [ ] List three ways to give the optimizer better information.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › The Catalyst Optimizer
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Phase 3: Physical planning

