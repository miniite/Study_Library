---
id: "TOOLS/Hadoop/03_Ecosystem/Hive"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Hive

**L4 Advanced** · [↑ Hadoop Ecosystem](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Hive lets you write SQL (HiveQL) over files that sit in HDFS; it translates each query into a distributed job (MapReduce, Tez or Spark) and keeps only the table *definitions* in a small relational database called the metastore.

### 1. The big picture
Raw files in HDFS have no structure: a directory of tab-separated text is just bytes. Analysts, however, think in tables, columns and SQL. Hive bridges the two. You tell Hive "this directory is a table with columns `year`, `temperature`, `quality`", and from then on you can query it with `SELECT … GROUP BY …` without writing a single line of Java.

Hive was built at Facebook for exactly this reason: thousands of people who knew SQL but not MapReduce needed to analyse data in Hadoop. It is a **batch data-warehouse layer**, not an OLTP database: each query starts a distributed job, so latency is seconds to minutes, not milliseconds. It sits above HDFS and YARN (see the parent topic, the Hadoop ecosystem) and below BI tools that connect over JDBC/ODBC. Its two children in this curriculum go deeper: the **metastore and services** (where table definitions live and how clients connect) and **tables, partitions, buckets and storage formats** (how the data is laid out on disk).

### 2. Core concepts and vocabulary
- **HiveQL** — Hive's SQL dialect: a blend of SQL-92, MySQL and Oracle flavours, plus later features such as window functions, plus MapReduce-inspired extras (multi-table insert, `TRANSFORM`/`MAP`/`REDUCE`).
- **Table** — *data* (files in HDFS or another Hadoop filesystem such as S3) plus *metadata* (schema, location, format) kept in the metastore.
- **Metastore** — the relational database (Derby by default, MySQL/PostgreSQL in production) that stores table definitions, never the data itself.
- **Warehouse directory** — where managed tables live; controlled by `hive.metastore.warehouse.dir`, default `/user/hive/warehouse`.
- **Schema on read** — the schema is checked when you *query*, not when you *load*. Loading is just a file copy/move.
- **Schema on write** — the traditional database approach: data is validated and converted to an internal format at load time.
- **Execution engine** — what actually runs the plan: MapReduce (original), Tez, or Spark.
- **Hive shell / CLI** — the interactive prompt (`hive>`); `beeline` is the modern JDBC-based client.

### 3. How it works, step by step
1. You submit a HiveQL statement (shell, `hive -e`, `hive -f script.q`, JDBC, Beeline…).
2. The **parser** turns the text into a syntax tree; the **semantic analyzer** looks up table and column definitions in the **metastore**.
3. The **optimizer** builds a logical plan (predicate pushdown, partition pruning, join ordering) and then a physical plan: a DAG of map and reduce stages.
4. The plan is submitted to the cluster (YARN) using the chosen engine. Mappers read the table's files straight from HDFS, applying the table's *SerDe* (serializer/deserializer) to turn bytes into columns.
5. Results are written to a result location or returned to the client.

| Aspect | Traditional RDBMS | Hive |
|---|---|---|
| Schema enforcement | On write | On read |
| Load speed | Slow (parse, index, compress) | Fast (copy/move files) |
| Query latency | Milliseconds–seconds | Seconds–minutes (job start-up) |
| Updates / transactions | Native | Added later (0.13/0.14), delta files + compaction, needs ACID tables |
| Indexes | Rich | Limited (compact, bitmap; since removed in Hive 3) |
| Scale | Single/few nodes | Petabytes on a cluster |

**Schema on read vs write trade-off.** Schema on read gives a very fast load and flexibility — two different schemas can sit over the same files (via external tables). Schema on write gives faster queries because the engine can index and compress, but loads are slower and you must know the queries up front. Hive shines when the schema is not known, or not final, at load time.

### 4. Worked example
```sql
-- 1. define a table over tab-delimited weather data
CREATE TABLE records (year STRING, temperature INT, quality INT)
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t';

-- 2. load: this is only a file move into /user/hive/warehouse/records/
LOAD DATA LOCAL INPATH 'input/ncdc/micro-tab/sample.txt'
OVERWRITE INTO TABLE records;

-- 3. query: becomes a MapReduce/Tez job behind the scenes
SELECT year, MAX(temperature)
FROM records
WHERE temperature != 9999 AND quality IN (0, 1, 4, 5, 9)
GROUP BY year;
```
Walk-through: `CREATE TABLE` writes a row into the metastore; nothing happens in HDFS yet except the directory being created. `LOAD DATA` copies the file *verbatim* — Hive does not parse it, so a malformed line is only discovered at query time (it shows up as `NULL`). `OVERWRITE` deletes existing files in the table directory first; without it new files are simply added. The query runs as a map stage (filter, partial max) and a reduce stage (final max per year). Run `EXPLAIN` in front of the query to see the stage plan.

### 5. Under the hood
- **Files stay as-is.** Hive never rewrites your data on load; the table directory *is* the data. That is why `DROP TABLE` on a managed table removes the files and on an external table does not (see the tables lesson).
- **Updates on an immutable filesystem.** HDFS has no in-place update, so `INSERT/UPDATE/DELETE` on transactional tables write small **delta files**; a background compactor (run by the metastore) merges them into base files. Readers get a consistent snapshot.
- **Locks** (disabled by default) are coordinated via ZooKeeper and prevent, e.g., dropping a table while it is being read; `SHOW LOCKS` lists them.
- **Execution engines.** Original MapReduce wrote intermediate results to disk between stages. Tez (Hortonworks' "Stinger" initiative) runs a DAG without that overhead and adds a vectorized query engine; Hive on Spark is the third option. Note: *Spark SQL* is a different product — a separate SQL engine with partial Hive compatibility — whereas *Hive on Spark* is Hive itself using Spark as engine.

### 6. Performance and design implications
- Job start-up dominates for tiny queries; Hive is for scans over big data, not point lookups.
- Biggest wins: **partition pruning**, a **columnar format** (ORC/Parquet), compression, a faster engine (Tez), vectorization and sensible join strategy (map joins for small tables).
- Parsing text with a SerDe is CPU-heavy; keep long-term data in ORC or Parquet.
- Measure with `EXPLAIN`, the job counters, and the YARN UI.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hive -e 'SQL'` / `hive -f file.q` | Run SQL inline / from a file, non-interactive | Scripts and schedulers |
| `SHOW TABLES; DESCRIBE FORMATTED t;` | List tables; show schema, location, format | Inspect a table |
| `SET prop=value;` / `SET prop;` / `SET -v;` | Change/see/list settings for the session | Per-query tuning |
| `hive.metastore.warehouse.dir` | Location of managed tables | Cluster setup |
| `hive.execution.engine` | `mr`, `tez` or `spark` | Pick the engine |
| `EXPLAIN [EXTENDED] query` | Show the plan | Before tuning |

Precedence of settings (highest first): `SET` command → `-hiveconf` option → `hive-site.xml` / Hadoop site files → Hive and Hadoop defaults. For multi-user clusters make `/tmp` and the warehouse directory group-writable.

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large batch SQL over data already in HDFS/S3 | You need sub-second or row-level OLTP |
| Schema is flexible or not yet fixed (schema on read) | You need strict constraints enforced at load |
| Many analysts know SQL, few know Java | Interactive dashboards need low latency — consider Impala, Presto/Trino or Spark SQL, which reuse the Hive metastore |
| Need SQL on HBase → Phoenix is a separate option | Heavy row-level updates on non-transactional tables |

### 9. Common mistakes
- Assuming `LOAD DATA` validates the file → bad rows silently become `NULL` → check with a `SELECT … WHERE col IS NULL` count after loading.
- Leaving data as CSV text forever → slow scans → convert to ORC/Parquet with `CREATE TABLE … STORED AS ORC AS SELECT …`.
- Forgetting `OVERWRITE` (or using it by accident) → duplicate data or deleted data.
- Running with the default embedded metastore on a shared cluster → each user sees only their own tables → use a remote metastore.

### 10. 🎯 Interview kit
**30-second answer:** Hive is a SQL-on-Hadoop data-warehouse layer: table definitions live in a metastore, data stays as files in HDFS, and HiveQL queries are compiled into distributed jobs (MapReduce, Tez or Spark). It uses schema on read, so loads are fast but queries are batch-oriented.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Hive vs an RDBMS?* → schema on read vs write, latency, scale, limited transactions.
- *Where is the data and where is the metadata?* → data in HDFS, metadata in the metastore RDBMS.
- *Hive vs Impala/Presto/Spark SQL?* → faster interactive engines; share the metastore and formats.
- *How does Hive support UPDATE on HDFS?* → delta files + compaction on ACID tables.

**Signals interviewers look for:** you separate metadata from data, understand why loads are cheap, and know Hive is not OLTP.

### 11. Check yourself
- [ ] Explain schema on read vs schema on write and name one benefit of each.
- [ ] Describe what physically happens on `LOAD DATA … OVERWRITE INTO TABLE`.
- [ ] Trace a `GROUP BY` query from HiveQL to cluster jobs.
- [ ] State two ways to make Hive queries faster without changing the SQL.
<!-- explainer:end -->

## In this section (study in order)

1. [Hive Partitions Buckets and Storage Formats](<Hive_Tables_Partitions_Buckets/_index.md>) (L5 Expert)
2. [Hive Metastore and Services](<Hive_Metastore/_index.md>) (L5 Expert)

## References

- *Hadoop: The Definitive Guide* — Tom White — Hive › Managed Tables and External Tables
- *Hadoop: The Definitive Guide* — Tom White — Hive › Installing Hive
- *Hadoop: The Definitive Guide* — Tom White — Hive › Tables
- *Hadoop: The Definitive Guide* — Tom White — Hive › Note
- *Hadoop: The Definitive Guide* — Tom White — Hive › Comparison with Traditional Databases
- *Hadoop: The Definitive Guide* — Tom White — Hive › SQL-on-Hadoop Alternatives
- *Hadoop: The Definitive Guide* — Tom White — Hive › Dropping Tables
- *Hadoop: The Definitive Guide* — Tom White — Comparison with Databases › Using Hive tables with HCatalog
- *Hadoop: The Definitive Guide* — Tom White — Hive › The Hive Shell
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Managed Versus UnmanagedTables

