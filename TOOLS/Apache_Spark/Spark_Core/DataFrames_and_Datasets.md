---
id: "TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/Spark_Core"
tools: ["Apache_Spark"]
---

# DataFrames and Datasets

**L4 Advanced** · [↑ Spark Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A DataFrame is a distributed table with named, typed columns whose operations are described declaratively and optimised by Catalyst before running; RDDs are the lower-level typed-object API underneath, and Datasets add compile-time types (Scala/Java only).

### 1. The big picture
Spark's original abstraction was the **RDD**: an immutable, partitioned collection of arbitrary objects with functional operations (`map`, `filter`, `reduceByKey`). Spark can't see inside your lambdas, so it can't optimise them. The **structured APIs** — DataFrame, Dataset and Spark SQL — give Spark the *schema* and a declarative expression tree, so the **Catalyst** optimiser can reorder filters, prune columns, push predicates into Parquet and choose join strategies, and **Tungsten** can run on compact binary rows with generated code. That is why you write DataFrame/SQL code for almost everything and drop to RDDs only for unusual low-level control.

Spark SQL and the DataFrame API are two front ends to the same engine: they compile to the same logical plan and give the same performance, so choose by readability and team skills.

### 2. Core concepts and vocabulary
- **RDD** — Resilient Distributed Dataset: immutable, partitioned, with lineage for recomputation; no schema.
- **DataFrame** — Dataset of `Row` with a schema (`Dataset[Row]` in Scala); in PySpark this is *the* structured API.
- **Dataset** — typed `Dataset[T]` (Scala/Java) with compile-time type safety via encoders; not available in Python.
- **Schema / StructType** — column names, types and nullability.
- **Transformation** — lazy, returns a new DataFrame (`select`, `filter`, `groupBy`, `join`).
- **Action** — triggers execution (`count`, `show`, `collect`, `write`, `take`).
- **Catalyst** — rule-based/cost-based optimiser (analysis → logical optimisation → physical planning).
- **Tungsten** — off-heap binary format and whole-stage code generation.
- **Temp view** — name registered for a DataFrame so SQL can query it (`createOrReplaceTempView`).
- **Narrow / wide** — whether an operation needs a shuffle.
- **DataFrameReader / Writer** — `spark.read` / `df.write` interfaces for sources and sinks.

### 3. How it works, step by step
For `df.filter("amount > 0").groupBy("country").agg(sum("amount"))`:
1. **Parse/build** an unresolved logical plan from your calls (or SQL text).
2. **Analyse**: resolve column names and types against the catalog/schema.
3. **Optimise logically**: push filters down, prune unused columns, constant-fold, combine projects.
4. **Physical planning**: choose operators (hash aggregate, broadcast vs. sort-merge join) using statistics.
5. **Code generation**: fuse operators into a single JVM function per stage.
6. **Execution**: only when an action runs; tasks process partitions in parallel; wide ops insert shuffles.

Everyday operations map as: `select/selectExpr/withColumn/drop/withColumnRenamed` (projection), `filter/where` (same), `groupBy().agg()`, `join`, `orderBy/sort` (same; global sort needs a shuffle with range partitioning), `dropDuplicates`, `na.fill/drop`, `cast`, window functions, `when/otherwise` (conditional logic).

### 4. Worked example
```python
from pyspark.sql import SparkSession, functions as F, Window
from pyspark.sql.types import *

spark = SparkSession.builder.getOrCreate()

schema = StructType([
    StructField("order_id", LongType()),  StructField("customer_id", LongType()),
    StructField("country",  StringType()), StructField("amount",     DoubleType()),
    StructField("ts",       TimestampType())])

orders = spark.read.schema(schema).option("header", True).csv("s3://lake/raw/orders/")

clean = (orders
    .where(F.col("amount").isNotNull() & (F.col("amount") > 0))
    .withColumn("amount", F.col("amount").cast("decimal(18,2)"))
    .withColumn("size", F.when(F.col("amount") >= 1000, "large").otherwise("small"))
    .dropDuplicates(["order_id"]))

w = Window.partitionBy("customer_id").orderBy(F.col("ts").desc())
latest = clean.withColumn("rn", F.row_number().over(w)).where("rn = 1").drop("rn")

clean.createOrReplaceTempView("orders_clean")
spark.sql("""
   SELECT country, COUNT(*) AS n, COUNT(DISTINCT customer_id) AS customers, SUM(amount) AS revenue
   FROM orders_clean GROUP BY country ORDER BY revenue DESC""").show()

clean.write.mode("overwrite").partitionBy("country").parquet("s3://lake/silver/orders/")
```
What to notice: nothing runs until `.show()`/`.write`; `explain()` shows the filter merged below the cast and only needed columns scanned; `COUNT(DISTINCT)` is exact and shuffle-heavy, `approx_count_distinct` is cheap (HyperLogLog, ~2–5 % error).

### 5. Under the hood
- **Lazy evaluation + lineage**: each DataFrame is a plan, not data. A lost partition is recomputed from lineage. Immutability is what makes this safe — every transformation returns a new plan.
- **Encoders vs. Java serialization**: Datasets use encoders to convert JVM objects ↔ Tungsten binary rows; DataFrame ops on built-in functions never create JVM objects at all, which is why they beat typed lambdas (`map`) and Python UDFs.
- **Python overhead**: the PySpark driver sends the plan to the JVM; only Python UDFs/`mapInPandas`/RDD lambdas move data between JVM and Python workers (pickle or Arrow). Built-in functions run entirely in the JVM.
- **Actions**: `count` is cheap (can use Parquet metadata), `collect` pulls *everything* to the driver (OOM risk), `take(n)`/`limit(n)` stop early.
- **`sort` vs `orderBy`**: aliases; both are a global sort (range-partition shuffle). `sortWithinPartitions` sorts locally with no shuffle.
- **Null semantics**: SQL three-valued logic — `col = NULL` is never true, use `isNull()`; `countDistinct` ignores nulls; `dropna(subset=...)`.

### 6. Performance and design implications
- Define schemas explicitly for big CSV/JSON (avoids a full inference pass).
- Prefer Parquet/ORC/Delta: column pruning, predicate pushdown, splittable, typed.
- Select needed columns early; avoid `SELECT *` in long pipelines.
- Prefer built-ins over UDFs; if needed, pandas UDFs (Arrow).
- Cache only when a DataFrame is reused and expensive; unpersist afterwards.
- Avoid chaining hundreds of `withColumn` calls (plan blow-up) — use a single `select`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.read.format(...).schema(s).load()` | Read with explicit schema | Production ingestion |
| `df.printSchema()`, `df.explain()` | Inspect schema/plan | Debugging |
| `df.createOrReplaceTempView("t")` | Expose to SQL | Mix SQL and API |
| `df.select / selectExpr / withColumn` | Projection and derived columns | Everyday transforms |
| `df.dropDuplicates(cols)` | Dedup by key | Cleaning (shuffles) |
| `df.na.fill / df.na.drop` | Null handling | Cleaning |
| `df.write.partitionBy(...).parquet(...)` | Write partitioned files | Lake layouts |
| `df.rdd` / `spark.sparkContext` | Drop to RDD | Rare low-level control |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| DataFrame/SQL | You need arbitrary per-record object logic (then RDD / mapInPandas) |
| Spark SQL text | Dynamic, programmatically-built pipelines (API composes better) |
| Dataset[T] (Scala) | Python (not available); columnar built-ins suffice |
| RDD | Unstructured data, custom partitioners, fine control of partitioning |
| `approx_count_distinct` | Exact count needed for finance/audit |

### 9. Common mistakes
- Assuming transformations execute immediately → confusion about timing/cost → think "plan then action".
- `collect()` on large data → driver OOM → `limit`, aggregate first, or write out.
- `col == None` instead of `isNull()` → empty results → use null-safe tests.
- Using Python UDF for a built-in operation → 10× slower → look for `F.*`.
- Reading CSV with inference in production → slow and unstable types → explicit schema.

### 10. 🎯 Interview kit
**30-second answer:** "A DataFrame is a distributed table with a schema; operations are lazy and compiled into a plan that Catalyst optimises, then Tungsten executes. RDDs are the untyped-object layer underneath; Datasets add typed objects in Scala. I use DataFrames/SQL by default because the optimiser can see what I'm doing."

**Structure a longer answer:** RDD → structured APIs → Catalyst/Tungsten → lazy evaluation → when to drop down.

**Likely follow-ups:**
- *DataFrame vs RDD?* → schema + optimiser vs. opaque functions; DataFrames faster and more compact.
- *Dataset vs DataFrame?* → compile-time types and encoders; DataFrame = `Dataset[Row]`.
- *Spark SQL vs API?* → same engine/plan; pick for clarity.
- *`count` vs `countDistinct`?* → rows incl. nulls vs. unique non-null values (shuffle).
- *Remove duplicates keeping latest?* → `row_number()` over partition by key order by ts desc.

**Signals interviewers look for:** you explain *why* DataFrames are faster, know lazy vs. action, and handle nulls correctly.

### 11. Check yourself
- [ ] Explain each Catalyst phase with one example optimisation.
- [ ] Write dedup-keep-latest in both DataFrame API and SQL.
- [ ] Name three things that force a shuffle in the pipeline above.
- [ ] Explain why a Python UDF is slower than `F.upper`.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)

## Practice questions

- [Basic] [Can you give examples of common transformations like map filter and flatMap?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02247_Can_you_give_examples_of_common_transformations_like_map_fil.md>)
- [Basic] [What is Spark SQL and why would you use it over the DataFrame API?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02272_What_is_Spark_SQL_and_why_would_you_use_it_over_the_DataFram.md>)
- [Basic] [How do you filter rows in a DataFrame using the filter or where method?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02280_How_do_you_filter_rows_in_a_DataFrame_using_the_filter_or_wh.md>)
- [Basic] [How do you remove duplicate rows from a DataFrame in Spark?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02287_How_do_you_remove_duplicate_rows_from_a_DataFrame_in_Spark.md>)
- [Basic] [How do you cast a column from one data type to another in Spark?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02324_How_do_you_cast_a_column_from_one_data_type_to_another_in_Sp.md>)
- [Intermediate] [What is the difference between orderBy and sort in Spark and are there sort side effects?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02289_What_is_the_difference_between_orderBy_and_sort_in_Spark_and.md>)
- [Intermediate] [What are window functions in Spark and give an example use case?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02321_What_are_window_functions_in_Spark_and_give_an_example_use_c.md>)
- [Intermediate] [How would you use Spark to deduplicate and clean a raw dataset before loading it to a warehouse?](<../../../PRACTICE/TOOLS/Apache_Spark/Spark_Core/Q-02338_How_would_you_use_Spark_to_deduplicate_and_clean_a_raw_datas.md>)

## Related concepts

- [Spark DataFrames and Lazy Evaluation](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation.md>) *(Pipeline Orchestration)*
- [Spark DataFrames](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_DataFrames.md>) *(Big Data Processing)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — handling. Instead of failing immediately on a bad input line (or worse, failing silently and
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Spark SQL
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › In Python
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Schemas and Creating DataFrames
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Two ways to define a schema
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Chapter 4. Transforming Data with Apache Spark
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Querying JSON Format
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Parsing JSON into Struct Type
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › DataStreamReader
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Define Schemas with the DataFrame API
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Define DataFrame with Schema
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Create the DataFrame
- *Building Medallion Architectures* — Piethein Strengholt — Streamline the Gold Layer › Create the table
- *Hadoop: The Definitive Guide* — Tom White — Spark › Note
- *Hadoop: The Definitive Guide* — Tom White — Spark › A Java Example
- *Hadoop: The Definitive Guide* — Tom White — Spark › Functions
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — What is Apache Spark › What is Apache Spark?
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — What is Apache Spark › Spark SQL
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Partitioning in Spark
- *Databricks Certified Associate Developer for Apache Spark* — Saba Shah — Spark DataFrames and their Operations

