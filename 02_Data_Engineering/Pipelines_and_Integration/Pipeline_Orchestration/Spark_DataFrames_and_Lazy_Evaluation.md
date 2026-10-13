---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark"]
---

# Spark DataFrames and Lazy Evaluation

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark DataFrame transformations only *describe* a computation; nothing runs until an action asks for a result, which lets Spark optimize the whole plan at once.

### 1. The big picture
Spark represents data as RDDs, DataFrames, or Datasets — immutable, partitioned collections spread across a cluster. A **DataFrame** is a table-like abstraction with named, typed columns, and it is the main API for data engineering because Spark can understand its structure and optimize it. You work with two kinds of operations: **transformations**, which build a new DataFrame from an old one, and **actions**, which produce a result (count, collect, write).

Because transformations are *lazy*, Spark records a lineage of steps and waits. When an action finally arrives, the optimizer sees the entire chain, reorders and prunes it, and executes the cheapest physical plan. This is why Spark pipelines can read hundreds of columns' worth of files but only touch the three you used, and why a bug in line 10 may only appear when line 50 runs.

### 2. Core concepts and vocabulary
- **RDD** — the low-level resilient distributed dataset; no schema, no query optimizer.
- **DataFrame** — distributed table with a schema; operations are expressed as columns/expressions and optimized.
- **Dataset** — typed variant (Scala/Java); in Python you effectively use DataFrames.
- **Transformation** — returns a new DataFrame, lazy (`select`, `filter`, `withColumn`, `join`, `groupBy`).
- **Action** — triggers execution (`count`, `show`, `collect`, `write`, `take`).
- **Narrow transformation** — each output partition depends on one input partition (`filter`, `select`, `map`); no shuffle.
- **Wide transformation** — needs data from many partitions (`groupBy`, `join`, `distinct`, `orderBy`); causes a shuffle and a new stage.
- **Lineage / logical plan** — the recorded sequence of transformations; enables fault recovery by recomputation.
- **Catalyst** — Spark SQL's optimizer (logical plan → optimized logical plan → physical plan).
- **Immutability** — a DataFrame never changes; every transformation yields a new one.
- **Cache / persist** — keep a computed DataFrame in memory/disk for reuse.

### 3. How it works, step by step
1. **Build:** `spark.read...` creates a DataFrame backed only by a plan to read files — no data is loaded yet (schema inference excepted, which may sample the files).
2. **Chain transformations:** each call returns a new DataFrame with a longer logical plan.
3. **Trigger:** an action such as `write` or `count` submits a **job**.
4. **Optimize:** Catalyst applies rules — predicate pushdown, column pruning, constant folding, join reordering — then picks physical operators (e.g., broadcast hash join vs sort-merge join).
5. **Split into stages:** at every wide dependency (shuffle) the plan breaks into a new stage.
6. **Run tasks:** each stage runs one task per partition on executors; results of shuffles are written to local disk then fetched.
7. **Return or write** the result. If an executor dies, Spark recomputes lost partitions from lineage.

The Fundamentals of Data Engineering example shows this end to end: three JSON sources are read, joined (a shuffle), filtered with SQL, and written as Parquet/Delta. Inputs are held in cluster memory, spilling to disk when too large; the join redistributes by key; the filter happens in memory; the final write compresses and stores. Airflow's DAG and Spark's DAG are separate objects.

### 4. Worked example
```python
from pyspark.sql import functions as F

orders = spark.read.parquet("s3://lake/silver/orders/")      # no data read yet
users  = spark.read.parquet("s3://lake/silver/users/")

recent = orders.filter(F.col("order_date") >= "2026-10-01")  # transformation
joined = recent.join(users, "user_id")                       # transformation (wide)
daily  = (joined.groupBy("country", "order_date")            # transformation (wide)
                .agg(F.sum("amount").alias("revenue")))

daily.explain("formatted")     # shows the plan; still no execution
daily.write.mode("overwrite").parquet("s3://lake/gold/daily_revenue/")   # ACTION → job runs
```
Reading `explain`: you will see `PushedFilters: [GreaterThanOrEqual(order_date,2026-10-01)]` on the orders scan — the filter was pushed to the file reader, so only matching partitions/row groups are read. `Exchange hashpartitioning(user_id)` marks the shuffle for the join; a second `Exchange` marks the groupBy. If `users` is small enough (below `spark.sql.autoBroadcastJoinThreshold`, 10 MB by default) Spark will plan a `BroadcastHashJoin` and the first Exchange disappears.

Lazy-evaluation consequence: a typo in a column name raises an `AnalysisException` immediately (the plan is analyzed eagerly), but a bad data value, such as a failed cast in an expression, appears only when the action runs, possibly far from the line that caused it.

### 5. Under the hood
- **Three plan levels:** parsed → analyzed (names resolved against the catalog/schema) → optimized (rules) → physical (strategies, cost-based choices). `explain("extended")` prints them.
- **Whole-stage code generation** compiles a pipeline of narrow operators into a single Java method to avoid virtual calls per row.
- **Tungsten** manages memory off-heap/binary formats, reducing GC cost and enabling cache-friendly processing.
- **Adaptive Query Execution** re-plans at shuffle boundaries using runtime statistics: coalesces small shuffle partitions, switches to broadcast joins, and splits skewed partitions.
- **Caching is lazy too:** `df.cache()` only marks the plan; the first action materializes it. `unpersist()` frees it.
- **Python UDFs** are black boxes to Catalyst: no pushdown through them, plus serialization overhead between JVM and Python. Built-in functions are optimizable.

### 6. Performance and design implications
- Prefer built-in functions and column expressions over UDFs; use pandas (Arrow) UDFs when Python logic is unavoidable.
- Filter and select early — though Catalyst often pushes them, it can't through opaque functions.
- Avoid `collect()` on big data (driver OOM); use `limit`, `take`, or write to storage.
- Repeated use of the same expensive DataFrame in several actions recomputes it each time — cache or write an intermediate table.
- Too many wide steps = too many shuffles; combine aggregations and choose join strategies deliberately.
- Measure with the SQL tab (per-operator metrics) and stage summaries.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.explain("formatted")` | Show physical plan | Verify pushdown and joins |
| `df.cache()` / `persist(level)` | Keep result for reuse | Multiple actions on same data |
| `df.repartition(n, col)` / `coalesce(n)` | Change partition count (shuffle vs no-shuffle) | Balance or reduce files |
| `broadcast(df)` hint | Force broadcast join | Small dimension table |
| `spark.sql.shuffle.partitions` | Post-shuffle parallelism | Tune per workload |
| `spark.sql.adaptive.enabled` | AQE | Keep on |
| `df.printSchema()` / `df.schema` | Inspect types | After reads and joins |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| DataFrame API/Spark SQL — optimizable | Fine-grained custom row logic that needs RDD control |
| `cache()` — reused multiple times | Used once; consumes memory |
| `repartition` — fix skew/parallelism | Just reducing files (use `coalesce`) |
| Broadcast join — small side | Side is large or growing |
| Persist intermediate table — long lineage, failures costly | Cheap recomputation |

### 9. Common mistakes
- Assuming code ran because the line executed → no action, no work → know which calls are actions.
- Calling an action repeatedly (`count()`, `show()`, `write`) on an unpersisted expensive DataFrame → recomputed each time → cache or write once.
- `collect()` large results → driver OOM → aggregate first.
- Reading with `inferSchema` on huge data → extra pass and unstable types → explicit schema.
- Using Python UDFs for simple string/date logic → slow → built-in functions.

### 10. 🎯 Interview kit
**30-second answer:** Spark transformations are lazy: they build a logical plan on immutable DataFrames. An action triggers Catalyst to optimize the entire plan, split it into stages at shuffles, and run tasks per partition. Laziness enables pushdown, pruning, and recovery via lineage.

**Structure a longer answer:** clarify (batch/stream, data size) → core mechanism (transformations vs actions, narrow vs wide, Catalyst) → trade-offs (cache, UDFs, join strategies) → scenario (job computed the same input three times).

**Likely follow-ups:**
- *Narrow vs wide?* → shuffle or not; stage boundaries.
- *When does caching help?* → reuse across actions; it's lazy.
- *Why are Python UDFs slow?* → serialization and opaque to the optimizer.
- *How do you see the plan?* → `explain`, SQL tab.

**Signals interviewers look for:** correctly distinguishing actions/transformations, plan reading, practical tuning.

### 11. Check yourself
- [ ] List five transformations and five actions.
- [ ] Explain why a stage boundary appears at a groupBy.
- [ ] Read an `explain` output and find the pushed filter and Exchange.
- [ ] Explain why `cache()` doesn't run anything by itself.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What is Apache Spark and why is it faster than traditional MapReduce for processing large log data?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13354_What_is_Apache_Spark_and_why_is_it_faster_than_traditional_M.md>)
- [Basic] [What is a groupBy in Spark and how would you use it to aggregate error counts per host?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13386_What_is_a_groupBy_in_Spark_and_how_would_you_use_it_to_aggre.md>)
- [Basic] [What is the difference between a transformation returning a new DataFrame versus modifying data in place in Spark?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13574_What_is_the_difference_between_a_transformation_returning_a_.md>)
- [Basic] [What is the difference between an inner join and a left join in Spark and when would you pick each?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13583_What_is_the_difference_between_an_inner_join_and_a_left_join.md>)
- [Intermediate] [What is the difference between narrow and wide transformations in Spark?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13362_What_is_the_difference_between_narrow_and_wide_transformatio.md>)
- [Intermediate] [What is the difference between orderBy and sort in Spark and why can sorting be expensive?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13408_What_is_the_difference_between_orderBy_and_sort_in_Spark_and.md>)
- [Intermediate] [What is a user-defined function (UDF) in Spark and why is it usually slower than built-in functions?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13589_What_is_a_user-defined_function_UDF_in_Spark_and_why_is_it_u.md>)
- [Intermediate] [What are the pitfalls of collecting a Spark DataFrame of clustered incidents to the driver and how would you avoid needing to?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13509_What_are_the_pitfalls_of_collecting_a_Spark_DataFrame_of_clu.md>)

## Related concepts

- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*
- [Spark Execution Caching and Debugging](<../../../TOOLS/Apache_Spark/Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*
- [DataFrames and Datasets](<../../../TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>) *(Spark Core)*
- [Spark DataFrames](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_DataFrames.md>) *(Big Data Processing)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Spark
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Streaming Data Transformation
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Streaming Data Transformation › Data Transformation Using AWS Glue
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Example: When to avoid SQL for batch transformations in Spark
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Example: Data transformation in Spark
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Materialized Views, Federation, and Query Virtualization › Streaming Transformations and Processing
- *Data Algorithms with Spark* — Mahmoud Parsian — **PART I** › PART I
- *Data Algorithms with Spark* — Mahmoud Parsian — **Spark Usage** › Spark Usage
- *Data Algorithms with Spark* — Mahmoud Parsian — **Spark Usage** › Chapter 2(#indexsplit008.htmlpage59)). A high-level view of PySpark's architecture is presented in Figure 1-7(#indexsplit006.htmlpage40).
- *Data Algorithms with Spark* — Mahmoud Parsian — **Serverless Spark** › CHAPTER 2
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Ease of Use
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Extensibility
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Spark MLlib
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › The Developer's Experience
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Desynchronization
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Aggregation › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — R › S
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Spark Structured Streaming
- *Building Medallion Architectures* — Piethein Strengholt — Building a Medallion Foundation with Microsoft Fabric › Data Engineering with Spark
- *Building Medallion Architectures* — Piethein Strengholt — Build the Silver Layer › Infer base parameters from the pipeline context
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Create the schema

