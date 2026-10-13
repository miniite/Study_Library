---
id: "TOOLS/Hadoop/03_Ecosystem/Pig_and_Crunch"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Pig Crunch and Cascading

**L4 Advanced** · [↑ Hadoop Ecosystem](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Pig, Crunch and Cascading are higher-level ways to write MapReduce data flows — Pig with a scripting language (Pig Latin), Crunch and Cascading with Java/Scala pipeline APIs — so you describe *what* to transform and the framework plans the chain of jobs.

### 1. The big picture
Raw MapReduce forces every problem into one map and one reduce function; real analyses (joins, multi-step aggregations) need several chained jobs, plus the boilerplate of writing, compiling, packaging and submitting each. Higher-level abstractions raise the level: you express a **dataflow** and the framework compiles it into an optimized series of jobs. That also shortens the develop-test loop, which the book names as MapReduce's main criticism.

These tools predate and inspired Spark's DataFrame/RDD pipelines; today Pig is mainly legacy, but the ideas (lazy dataflow, logical vs physical plan, user-defined functions) are everywhere in modern engines.

### 2. Core concepts and vocabulary
- **Pig Latin** — the data-flow language: a sequence of named transformations on *relations*.
- **Relation / bag / tuple / field** — Pig's nested data model: a relation is a bag (collection) of tuples; tuples hold fields, which can themselves be bags, maps or tuples.
- **Grunt** — Pig's interactive shell; scripts run with `pig script.pig`.
- **Execution modes** — local (single JVM) and MapReduce (cluster); later Tez/Spark.
- **Logical / physical / MapReduce plan** — the three stages Pig compiles to; inspect with `EXPLAIN`.
- **UDF** — user-defined function in Java/Python for load, eval, filter or store.
- **Crunch** — Java/Scala library (modelled on Google's FlumeJava) with `PCollection`, `PTable`, `PGroupedTable` abstractions, written against ordinary Java types.
- **Cascading** — Java API where you wire **pipes**, **taps** (sources/sinks) and **flows** into assemblies; the basis for Scalding and Cascalog.

### 3. How it works, step by step
1. You write Pig Latin statements; nothing executes until `DUMP` or `STORE` (lazy evaluation).
2. Pig parses them into a **logical plan** (operators such as LOAD, FILTER, GROUP).
3. The plan is optimized (e.g. pushing filters earlier) and turned into a **physical plan**.
4. Physical operators are grouped into one or more **MapReduce jobs** (each `GROUP`/`JOIN`/`ORDER` introduces a reduce phase).
5. Jobs are submitted and results written to HDFS.

| Category | Operators |
|---|---|
| Load/store | `LOAD`, `STORE`, `DUMP` |
| Filtering | `FILTER`, `DISTINCT`, `FOREACH … GENERATE`, `SAMPLE`, `STREAM` |
| Grouping/joining | `JOIN`, `COGROUP`, `GROUP`, `CROSS` |
| Sorting | `ORDER`, `LIMIT` |
| Combining | `UNION`, `SPLIT` |

Crunch and Cascading apply the same idea in Java: you chain `parallelDo`, `groupByKey`, `combineValues` (Crunch) or `Each`, `GroupBy`, `Every` (Cascading), and the planner fuses them into minimal MapReduce jobs.

### 4. Worked example
```pig
-- max temperature per year, cleaning bad readings
records = LOAD 'input/ncdc/sample.txt'
          AS (year:chararray, temperature:int, quality:int);
filtered = FILTER records BY temperature != 9999 AND quality IN (0,1,4,5,9);
grouped  = GROUP filtered BY year;
max_temp = FOREACH grouped GENERATE group, MAX(filtered.temperature);
DUMP max_temp;
EXPLAIN max_temp;
```
`records` declares a schema inline (or use HCatalog to read it from the Hive metastore: `LOAD 'records' USING org.apache.hcatalog.pig.HCatLoader();` after `pig -useHCatalog`). `GROUP` creates a relation `(group, filtered:bag)`; `FOREACH … GENERATE` applies `MAX` to each bag. `EXPLAIN` shows the logical, physical and MapReduce plans — here, one MapReduce job with a combiner for `MAX`.

Parallelism: by default Pig uses one reducer per 1 GB of input up to 999 (`pig.exec.reducers.bytes.per.reducer`, `pig.exec.reducers.max`); override per operator with `PARALLEL 30`.

### 5. Under the hood
- Pig's data model is **nested**, unlike flat relational rows; this is why Pig fits semi-structured logs.
- Schemas are optional ("pigs eat anything"); with no schema, fields are untyped bytearrays and operations may fail at runtime.
- Joins have strategies: replicated (map-side, small table in memory), skewed, and merge joins — choose with `USING 'replicated'` etc.
- **UDFs**: a `LoadFunc` uses a Hadoop `InputFormat`/`RecordReader`, repeatedly calling `getNext()` to produce `Tuple`s and returning `null` at end of split; decide whether short lines throw or yield null fields.
- Crunch's type system allows plain Java objects, Avro records or Writables, and its planner defers execution until the pipeline is run.

### 6. Performance and design implications
- Filter and project early; use combiner-friendly (algebraic) functions like `SUM`, `COUNT`, `MAX`.
- Count reduce phases in `EXPLAIN`: every one is a new MapReduce job with disk I/O between jobs.
- Prefer replicated joins when one side is small; watch for key skew.
- Today, prefer Spark or Hive/Tez for new work; keep Pig for existing pipelines.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pig -x local script.pig` | Run locally | Debugging small data |
| `pig script.pig` / `grunt>` | Run on cluster / interactive | Normal use |
| `DESCRIBE rel;` `ILLUSTRATE rel;` | Show schema / sample data trace | Developing |
| `EXPLAIN rel;` | Show plans, number of jobs | Tuning |
| `PARALLEL n` | Set reducers for an operator | Control parallelism |
| `-param name=value` | Parameterise scripts | Scheduled runs |
| `REGISTER udfs.jar;` | Load UDFs | Custom logic |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Multi-step ETL dataflows without writing Java (Pig) | New projects where Spark/Hive are the standard |
| Type-safe Java pipelines with unit tests (Crunch/Cascading) | Team has no Java skills |
| Nested/semi-structured data | Purely relational SQL workloads (use Hive) |

### 9. Common mistakes
- Forgetting laziness — nothing runs without `STORE`/`DUMP` → add output.
- Cross-joining accidentally with `CROSS` → explosion → use `JOIN` with keys.
- Not filtering NULLs before grouping/joining → silent skew or dropped rows.
- Leaving default reducers → too few/many tasks → set `PARALLEL`.

### 10. 🎯 Interview kit
**30-second answer:** Pig Latin is a data-flow language that compiles lazily into a chain of MapReduce jobs, with a nested data model, built-in relational operators and UDFs; Crunch and Cascading give the same abstraction as Java pipeline APIs.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Pig vs Hive?* → procedural dataflow vs declarative SQL; schema optional vs metastore-based.
- *How do you see how many MR jobs a script makes?* → `EXPLAIN`.
- *What replaced these?* → Spark and SQL engines.

**Signals interviewers look for:** you understand compilation to jobs and why each reduce step costs I/O.

### 11. Check yourself
- [ ] Translate a word-count into Pig Latin.
- [ ] Explain the three plans shown by `EXPLAIN`.
- [ ] State when a replicated join is appropriate.
- [ ] Compare Pig, Crunch and Cascading in one sentence each.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Crunch › Chapter 18. Crunch
- *Hadoop: The Definitive Guide* — Tom White — Chapter 16. Pig
- *Hadoop: The Definitive Guide* — Tom White — Composable Data at Cerner › Enter Apache Crunch
- *Hadoop: The Definitive Guide* — Tom White — Comparison with Databases › Note
- *Hadoop: The Definitive Guide* — Tom White — User-Defined Functions › Note
- *Hadoop: The Definitive Guide* — Tom White — Chapter 24. Cascading
- *Hadoop: The Definitive Guide* — Tom White — Pig in Practice
- *Designing Data-Intensive Applications* — Martin Kleppmann — Part III. Derived Data › Overview of Chapters
- *Hadoop: The Definitive Guide* — Tom White — K › M
- *Hadoop: The Definitive Guide* — Tom White — Chapter 24. Cascading › Operations

