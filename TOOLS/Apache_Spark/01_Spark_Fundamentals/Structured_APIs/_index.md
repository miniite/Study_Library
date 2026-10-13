---
id: "TOOLS/Apache_Spark/01_Spark_Fundamentals/Structured_APIs"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# DataFrame and Dataset APIs

**L4 Advanced** · [↑ Spark Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark's Structured APIs (DataFrames and Datasets) describe *what* you want over data that has a schema, so the Catalyst optimiser can decide *how* to do it, which makes them faster and more compact than raw RDDs.

### 1. The big picture
RDDs are opaque: Spark sees a collection of JVM/Python objects and an arbitrary lambda, so it cannot reorder, prune or compress anything. Structured APIs fix this by giving data a **schema** (named, typed columns) and expressing operations with recognisable operators (`select`, `filter`, `groupBy`, `agg`, `join`). Because Spark understands the intent, it can push filters down, prune columns, pick join strategies, store data in a compact binary format and generate fused code.

The same engine serves three front-ends: the **DataFrame** API (all languages), the **Dataset** API (typed, Scala/Java only) and **SQL**. They compile to the same logical plan, so equivalent queries have the same performance regardless of the language you write them in. This lesson sits between the architecture lesson (how jobs run) and the Spark SQL/Catalyst lesson (how plans are optimised).

### 2. Core concepts and vocabulary
- **DataFrame** — a distributed table of rows with a schema. Immutable; every operation returns a new DataFrame. In Scala/Java it is `Dataset[Row]`.
- **Dataset[T]** — a DataFrame whose rows are typed JVM objects (case class / JavaBean), giving compile-time type checks.
- **Row** — a generic untyped record; fields accessed by position or name.
- **Schema** — list of `StructField(name, dataType, nullable)` in a `StructType`.
- **Column** — an expression (`col("a") + 1`) evaluated per row; the building block of DataFrame operations.
- **Encoder** — converts JVM objects to Spark's internal binary format (Tungsten) and back; required by typed Datasets.
- **Lazy evaluation** — transformations build a plan; actions run it.
- **Catalyst** — the rule-based/cost-based optimiser that rewrites the plan.
- **Tungsten** — off-heap, binary row/column memory format and code generation.
- **Temp view** — a named handle (`createOrReplaceTempView`) letting you query a DataFrame with SQL.

### 3. How it works, step by step
1. **Define a schema** — either by letting Spark infer it (costs a pass over the data for CSV/JSON) or by declaring it:
   - programmatically with `StructType([...])`, or
   - with a DDL string such as `"id INT, name STRING, ts TIMESTAMP"`.
2. **Create the DataFrame** — from files (`spark.read`), tables, an existing RDD, or a local collection.
3. **Transform** — chain `select`, `withColumn`, `filter`, `groupBy().agg()`, `join`, `orderBy`. Each call returns a new immutable DataFrame.
4. **Optionally type it** (Scala/Java) — `df.as[Person]` turns rows into case-class objects; lambda-style `map`/`filter` become available.
5. **Execute** — an action (`show`, `count`, `write`) triggers Catalyst → physical plan → RDD stages.

| | RDD | DataFrame | Dataset[T] |
|---|---|---|---|
| Schema known to Spark | no | yes | yes |
| Compile-time type safety | yes | no (errors at analysis time) | yes |
| Catalyst + Tungsten | no | yes | yes (lambdas are opaque) |
| Languages | all | all | Scala, Java |

Error timing is a classic exam point: a typo in a column name in a DataFrame fails at *analysis* (runtime of the driver, before data is processed); in a Dataset a wrong field name fails at *compile* time.

### 4. Worked example
```python
from pyspark.sql import SparkSession, functions as F
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

spark = SparkSession.builder.getOrCreate()

schema = StructType([
    StructField("Id", IntegerType(), False),
    StructField("Author", StringType(), True),
    StructField("State", StringType(), True),
    StructField("Hits", IntegerType(), True),
])
# same thing as a DDL string: "Id INT, Author STRING, State STRING, Hits INT"

blogs = spark.read.schema(schema).json("/data/blogs.json")

top = (blogs
       .where(F.col("Hits") > 10_000)
       .withColumn("Big", F.col("Hits") > 25_000)
       .groupBy("State")
       .agg(F.count("*").alias("posts"), F.avg("Hits").alias("avg_hits"))
       .orderBy(F.desc("posts")))
top.show()
top.explain(True)   # parsed -> analysed -> optimised -> physical plan
```
`explain(True)` shows the four plan stages. Look in the physical plan for `PushedFilters: [GreaterThan(Hits,10000)]` — the filter was pushed into the file scan — and `ReadSchema` listing only the columns used (column pruning). Supplying `schema` avoids the extra pass to infer it and prevents surprise type changes between runs.

Scala typed Dataset equivalent:
```scala
case class Blog(Id: Int, Author: String, State: String, Hits: Int)
val ds = spark.read.schema("Id INT, Author STRING, State STRING, Hits INT")
              .json("/data/blogs.json").as[Blog]
ds.filter(_.Hits > 10000)   // compile-time checked, but lambda is opaque to Catalyst
ds.filter($"Hits" > 10000)  // column expression: optimisable (pushdown)
```

### 5. Under the hood
- **Four plan stages:** unresolved logical plan (names not yet checked) → analysed (names and types resolved against the catalog) → optimised logical plan (Catalyst rules: constant folding, predicate/projection pushdown, join reordering) → physical plan chosen by cost (join strategy, scan type), then code generation.
- **Encoders & Tungsten.** Dataset objects are stored as compact binary rows, not Java objects; sorting and hashing operate on the bytes, avoiding GC and (de)serialisation. A typed lambda forces Spark to deserialise rows into objects, which is why typed `map` is slower than the equivalent column expression.
- **Immutability** means every DataFrame is a node in a plan tree; reusing one in two branches duplicates computation unless cached.
- **Python.** DataFrame operations compile to JVM plans, so PySpark DataFrames run at roughly Scala speed; Python UDFs break this by shipping rows to Python workers (pandas/Arrow UDFs reduce the cost).
- **Nullability** in a schema is a hint, not an enforced constraint when reading files.
- Column naming and the `Row` object: `Row` is an ordered collection that can be indexed (`row[0]`) or accessed by name (`row.Hits`).

### 6. Performance and design implications
- Prefer built-in functions (`pyspark.sql.functions`) over UDFs; built-ins are optimisable and code-generated.
- Declare schemas for CSV/JSON in production: inference reads data twice and can silently choose wrong types.
- Select only needed columns early and filter early; with Parquet/ORC this reduces I/O dramatically.
- Use typed Datasets only where compile-time safety is worth losing some optimisation; mix typed and untyped by dropping to column expressions for hot paths.
- `df.printSchema()`, `df.dtypes`, `explain()`, and the SQL tab of the Spark UI are the main inspection tools.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.read.schema(s).format(...).load(path)` | Read with explicit schema | Always for CSV/JSON in pipelines |
| `df.printSchema()` / `df.schema` | Show/inspect schema | Debugging type issues |
| `df.select`, `where`, `withColumn`, `groupBy().agg()` | Core transformations | Everyday ETL |
| `df.createOrReplaceTempView("t")` + `spark.sql(...)` | Switch to SQL | Mixed SQL/API pipelines |
| `df.as[T]` (Scala/Java) | Convert to typed Dataset | Need compile-time checks |
| `df.explain(True)` | Print all plan stages | Verify pushdown/pruning |
| `spark.sql.caseSensitive` | Column-name case rules | Migration from case-sensitive sources |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| DataFrame API in Python/R/any language | You need compile-time field checks → use Dataset in Scala/Java |
| Dataset[T] for domain-object logic and strong typing | Lambdas on hot paths: they bypass Catalyst optimisations |
| SQL strings for analysts and quick exploration | Large dynamically built SQL strings are hard to test; no compile checking |
| Explicit schema | Schema drifting source files will then fail or null out → add schema-evolution handling |
| RDD only for unstructured data or fine-grained control | Using it by default loses all optimisation |

### 9. Common mistakes
- Relying on `inferSchema` for CSV → extra pass and wrong types (e.g. IDs as int that need leading zeros) → give a schema.
- Using Python UDFs for logic that exists as a built-in → big slowdown → use `functions`.
- Mutating expectations (`df.withColumn(...)` without assigning the result) → nothing changes → remember immutability.
- Assuming case classes make Dataset filters fast → opaque lambda blocks pushdown → use column expressions.
- Collecting a DataFrame to the driver to loop in Python → driver OOM and no parallelism → express as column operations.

### 10. 🎯 Interview kit
**30-second answer:** DataFrames are distributed tables with a schema; because Spark knows the schema and the operators, Catalyst can optimise the plan and Tungsten stores data compactly, so they beat RDDs. Datasets add compile-time typing in Scala/Java; SQL, DataFrames and Datasets all compile to the same plan.
**Structure a longer answer:** clarify (language, need for typing) → what the schema enables (pushdown, pruning, binary format) → the API trio and when each fits → optimisation caveats (lambdas, UDFs) → example from a pipeline.
**Likely follow-ups:**
- *DataFrame vs Dataset vs RDD?* → table above; DataFrame = Dataset[Row].
- *Why not always infer schema?* → extra pass, type surprises, non-determinism.
- *When do errors surface?* → SQL/DataFrame at analysis time, Dataset at compile time.
- *Why can a typed `filter(_.x > 1)` be slower than `filter($"x" > 1)`?* → deserialisation to objects and no predicate pushdown.
- *Are PySpark DataFrames slower than Scala?* → not for built-in ops; only UDFs/driver-side Python add cost.
**Signals interviewers look for:** naming Catalyst and Tungsten, knowing the lambda/UDF penalty, defining schemas deliberately, and understanding immutability and laziness.

### 11. Check yourself
- [ ] Define the same schema programmatically and as a DDL string.
- [ ] Explain why a column-name typo fails at a different time for DataFrame and Dataset.
- [ ] Use `explain(True)` to prove a filter was pushed into a Parquet scan.
- [ ] Rewrite a Python UDF as built-in functions.
- [ ] State what `df.as[Person]` does and when you would not use it.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Typed Objects, Untyped Objects, and Generic Rows
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › The Dataset API
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › The DataFrame API
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › DataFrames Versus Datasets
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Two ways to define a schema
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Schemas and Creating DataFrames
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Rows
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Accelerator-Aware Scheduler › Changes to the DataFrame and Dataset APIs
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Common DataFrame Operations
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and Datasets › Converting DataFrames to Datasets

