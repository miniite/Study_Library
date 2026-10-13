---
id: "TOOLS/Apache_Spark/01_Spark_Fundamentals/Spark_SQL_and_Catalyst"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Spark SQL and Data Sources

**L4 Advanced** · [↑ Spark Fundamentals](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark SQL is the engine under every DataFrame and SQL query: it reads many file formats and tables through a metastore, and its Catalyst optimiser turns your query into an efficient physical plan.

### 1. The big picture
Spark SQL does three jobs. It is a **SQL engine** (ANSI-style SQL plus HiveQL) usable from notebooks, a shell, JDBC/ODBC BI tools or a Thrift server. It is the **foundation of the DataFrame and Dataset APIs**: whatever you write is converted into the same logical plan. And it is the **data-source layer**: one reader/writer interface for Parquet, ORC, Avro, JSON, CSV, text, JDBC databases and lakehouse formats.

You can freely mix styles: read with the DataFrame API, register a temp view, query with SQL, and continue with the DataFrame API. The generated code and performance are the same in Python, Scala, Java, R or SQL, because all of them build the same plan that Catalyst optimises.

### 2. Core concepts and vocabulary
- **Catalog / metastore** — stores table metadata: schema, name, database, partitioning, location. By default a Hive-compatible metastore; the warehouse directory is `spark.sql.warehouse.dir`.
- **Managed table** — Spark owns both metadata and data; `DROP TABLE` deletes the files.
- **Unmanaged (external) table** — Spark owns only metadata; data stays at its path and survives `DROP TABLE`.
- **View** — a stored query; **temporary view** lives only in the session, **global temporary view** is shared across sessions of the application (`global_temp.<name>`).
- **DataFrameReader / DataFrameWriter** — `spark.read` / `df.write` with `format`, `option`, `schema`, `load`/`save`.
- **Data source** — pluggable format/connector; Parquet is the default.
- **Catalyst** — extensible optimiser working on trees with rules.
- **Tungsten / whole-stage codegen** — physical execution with binary memory layout and generated Java bytecode.
- **Predicate pushdown / column pruning** — pushing filters and projections into the scan so less data is read.

### 3. How it works, step by step
1. **Parse** — SQL text or a DataFrame expression becomes an *unresolved* logical plan (names unchecked).
2. **Analyse** — Catalyst resolves tables, columns and types against the catalog → *analysed* logical plan. A missing column fails here.
3. **Logical optimisation** — rule batches rewrite the plan: constant folding, filter and projection pushdown, combining filters, boolean simplification, removing unneeded columns, reordering joins.
4. **Physical planning** — Spark generates candidate physical plans (e.g. broadcast vs sort-merge join) and picks one with a cost model, using table statistics when available.
5. **Code generation** — operators in a stage are fused into a single generated function (whole-stage codegen) running over Tungsten binary rows.
6. **Execute** — the physical plan becomes RDD stages/tasks as described in the architecture lesson. With Adaptive Query Execution, the plan can be re-optimised between stages using runtime statistics.

Built-in data sources at a glance:

| Source | Nature | Notes |
|---|---|---|
| Parquet | columnar, schema in footer | Default; pushdown, pruning, compression; recommended for cleansed data |
| ORC | columnar | Common with Hive; similar benefits |
| Avro | row-oriented, schema-based | Native for Kafka serialisation; since Spark 2.4 built in |
| JSON | semi-structured text | Schema inference costs a pass |
| CSV | delimited text | Options: `header`, `inferSchema`, `mode` (`PERMISSIVE`/`DROPMALFORMED`/`FAILFAST`) |
| JDBC | external RDBMS | Partition the read with `partitionColumn`, bounds, `numPartitions` |

### 4. Worked example
```python
# Read Parquet (schema comes from the file footer), register a view, query with SQL
df = spark.read.format("parquet").load("/data/flights/2010-summary.parquet")
df.createOrReplaceTempView("us_delay_flights_tbl")

top = spark.sql("""
    SELECT DEST_COUNTRY_NAME, SUM(count) AS flights
    FROM us_delay_flights_tbl
    WHERE ORIGIN_COUNTRY_NAME = 'United States'
    GROUP BY DEST_COUNTRY_NAME
    ORDER BY flights DESC
    LIMIT 10
""")
top.explain("formatted")
```
```sql
-- Same source as an unmanaged table / view created directly in SQL
CREATE OR REPLACE TEMPORARY VIEW us_delay_flights_tbl
USING csv
OPTIONS (path "/data/flights/csv/*", header "true", inferSchema "true", mode "FAILFAST");

-- Managed table in the metastore
CREATE DATABASE IF NOT EXISTS learn;
CREATE TABLE learn.flights USING parquet AS SELECT * FROM us_delay_flights_tbl;
SHOW TABLES IN learn;
```
Reading the plan: `FileScan parquet [...]` with `PushedFilters` and `ReadSchema` proves pushdown and pruning; `HashAggregate` appears twice (partial before the shuffle, final after `Exchange hashpartitioning`); `TakeOrderedAndProject` implements `ORDER BY … LIMIT` without sorting everything. `mode "FAILFAST"` makes bad CSV rows abort the job rather than turning into nulls.

### 5. Under the hood
- **Trees and rules.** A plan is a tree of case-class nodes; each Catalyst rule pattern-matches subtrees and returns a rewritten tree; batches run until a fixed point. Developers can inject custom rules via `spark.experimental.extraOptimizations`.
- **Cost-based optimisation (CBO).** With `ANALYZE TABLE … COMPUTE STATISTICS [FOR COLUMNS …]` and `spark.sql.cbo.enabled`, join ordering and strategy use row counts and column statistics. Without stats, join choice relies on file size estimates (e.g. broadcast threshold of 10 MB).
- **Parquet** stores column chunks within row groups plus footer metadata (schema, min/max statistics), allowing row-group skipping for pushed-down predicates and reading only needed columns; files are written as `part-*.snappy.parquet` plus `_SUCCESS` markers.
- **Hive compatibility.** Spark can use a Hive metastore, Hive SerDes and UDFs, and understands Hive partitioned layout (`dt=2026-10-01/`); partition pruning removes whole directories before scanning.
- **UDFs** are black boxes to Catalyst: no pushdown through them and extra serialisation; built-in functions and higher-order functions (`transform`, `filter`, `aggregate` on arrays) stay inside the optimiser.
- **Thrift JDBC/ODBC server** runs a long-lived Spark application so BI tools (Tableau, Power BI) and `beeline` share one engine and cache.

### 6. Performance and design implications
- Store curated data as Parquet (or a lakehouse format); keep raw CSV/JSON only at the edge.
- Provide schemas and partitions that match query filters; avoid millions of tiny files.
- Check `explain` for `PushedFilters`, `PartitionFilters` and `ReadSchema` after changes.
- Collect statistics on large dimension/fact tables used in joins so CBO and broadcast decisions are good.
- Use JDBC partitioned reads (`partitionColumn`, `lowerBound`, `upperBound`, `numPartitions`) to avoid a single-task read, but cap parallelism to protect the source database.
- Prefer built-in SQL functions over Python UDFs; if needed, use vectorised Pandas UDFs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.sql("…")`, `%sql` | Run SQL, get a DataFrame | Mixed workflows |
| `df.createOrReplaceTempView` / `createGlobalTempView` | Name a DataFrame | Query it from SQL |
| `CREATE TABLE … USING parquet` / `saveAsTable` | Managed table | Persistent curated data |
| `CREATE TABLE … LOCATION '…'` / `option("path",…)` | Unmanaged table | Data owned elsewhere |
| `EXPLAIN [EXTENDED|FORMATTED]` / `df.explain(True)` | Show plans | Verify optimisations |
| `ANALYZE TABLE t COMPUTE STATISTICS FOR COLUMNS c` | Collect stats | CBO, join strategy |
| `spark.sql.adaptive.enabled` | Runtime re-optimisation | Default on in modern versions |
| `spark.sql.shuffle.partitions` | Post-shuffle partitions | Tune per data size |
| `spark.sql.warehouse.dir` | Warehouse location | Metastore configuration |
| `spark.sql.sources.default` | Default format | Usually `parquet` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| SQL for analyst-facing, declarative transformations | Complex iterative logic is clearer and more testable in the DataFrame API |
| Managed table for data Spark fully owns | External data that other systems also manage → unmanaged table |
| Parquet/ORC for analytics | Streaming serialisation → Avro; row-level lookups → a database |
| CBO with fresh statistics | Stale stats mislead the optimiser; re-analyse after big loads |
| Views for reuse | Deep view stacks hide cost and make plans hard to read |

### 9. Common mistakes
- Dropping a managed table expecting data to remain → files deleted → use an unmanaged table for shared data.
- Using `inferSchema` on huge CSVs in production → double read, wrong types → explicit schema.
- Reading a big JDBC table with default settings → one task, one connection → set partitioning options.
- Forgetting that lookups of case-sensitive names depend on `spark.sql.caseSensitive` → intermittent "column not found".
- Wrapping a filter column in a UDF or cast → pushdown lost → filter on the raw column.
- Assuming SQL is slower than the DataFrame API → they share the optimiser → choose by readability.

### 10. 🎯 Interview kit
**30-second answer:** Spark SQL is Spark's structured engine: queries (SQL or DataFrame) are parsed, resolved against a catalog, optimised by Catalyst (pushdown, pruning, join selection), and compiled to code over Tungsten binary rows. It reads many formats through one data-source API, with Parquet as default.
**Structure a longer answer:** clarify (formats, catalogs, BI access) → pipeline of parse/analyse/optimise/physical/codegen → data sources and tables (managed vs unmanaged) → optimiser features and their prerequisites (stats, pushdown) → a scenario like fixing a full scan.
**Likely follow-ups:**
- *Managed vs unmanaged table?* → who owns the data and what `DROP` does.
- *What does Catalyst do?* → rule-based and cost-based plan rewriting; list the four plan stages.
- *Why Parquet?* → columnar, statistics, compression, schema in footer.
- *Are SQL and DataFrame API equally fast?* → yes, same plan.
- *How to speed a JDBC read?* → partitioned reads with bounds and `numPartitions`.
**Signals interviewers look for:** you can read an `explain` plan, mention pushdown and statistics, treat UDFs as optimisation barriers, and select formats deliberately.

### 11. Check yourself
- [ ] Create the same table as a managed and an unmanaged table and predict what `DROP` does.
- [ ] List the four plan stages and what happens in each.
- [ ] Show a query where `PushedFilters` is empty because of a function wrapped around the column, then fix it.
- [ ] Read a CSV with `FAILFAST` and explain the behaviour with `PERMISSIVE`.
- [ ] Explain how Parquet footer statistics enable row-group skipping.
<!-- explainer:end -->

## In this section (study in order)

1. [Catalyst Optimizer and Tungsten](<Catalyst_and_Tungsten/_index.md>) (L6 Internals)

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Parquet
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › DataFrameReader
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Spark SQL
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Reading a CSV file into a Spark SQL table
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — S
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Chapter 4. Spark SQL and DataFrames: Introduction to Built-in Data Sources
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › SQL Tables and Views
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › In Python
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Reading an ORC file into a Spark SQL table
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Avro

