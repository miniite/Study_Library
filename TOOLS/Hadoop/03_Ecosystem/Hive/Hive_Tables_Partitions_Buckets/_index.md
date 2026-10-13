---
id: "TOOLS/Hadoop/03_Ecosystem/Hive/Hive_Tables_Partitions_Buckets"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Hive Partitions Buckets and Storage Formats

**L5 Expert** · [↑ Hive](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Hive table is a directory of files; partitions are sub-directories that let queries skip data, buckets hash-split files for faster joins and sampling, and the storage format (text, ORC, Parquet…) decides how fast those files can be read.

### 1. The big picture
Because Hive never owns a proprietary file format, the physical layout you choose *is* your performance strategy. Three levers do almost all the work: **how the table is owned** (managed vs external), **how the directory tree is split** (partitions, then buckets), and **how each file is encoded** (row text vs columnar). Get these right and a query reads a few percent of the bytes; get them wrong and every query scans everything and parses text.

This lesson builds on the Hive overview (SQL over HDFS files) and the metastore (where this layout is recorded).

### 2. Core concepts and vocabulary
- **Managed (internal) table** — Hive owns data; `DROP TABLE` deletes metadata *and* files; files live under the warehouse directory.
- **External table** — you own the data; `DROP TABLE` removes only metadata. Declared with `EXTERNAL` and a `LOCATION`.
- **Partition** — a table subdivided by the value(s) of partition column(s); each value is a sub-directory such as `dt=2024-01-01/country=IN`.
- **Partition pruning** — the optimizer reading only directories matching the `WHERE` clause.
- **Bucket** — a hash-based split of a table or partition into a fixed number of files, by `CLUSTERED BY (col) INTO n BUCKETS`.
- **SerDe** — serializer/deserializer: converts file bytes ↔ columns (`LazySimpleSerDe`, `OpenCSVSerde`, ORC, Parquet, JSON…).
- **Storage format** — file container: `TEXTFILE`, `SEQUENCEFILE`, `AVRO`, `ORC`, `PARQUET`.
- **Complex types** — `ARRAY`, `MAP`, `STRUCT` beside primitives (numeric, `BOOLEAN`, `STRING`, `TIMESTAMP`…).

### 3. How it works, step by step
1. **Creating**: `CREATE TABLE` records schema, location and format in the metastore.
2. **Loading**: `LOAD DATA` moves files into the directory; `INSERT … SELECT` runs a job that writes new files in the table's format; `CREATE TABLE … AS SELECT` (CTAS) does both.
3. **Partitioned load**: each distinct partition value gets its own sub-directory; *static* partitions name the value in the statement, *dynamic* partitions derive it from the data.
4. **Bucketed load**: rows are routed to file `hash(col) mod n`; with `hive.enforce.bucketing=true` (automatic in newer versions) the reducer count matches the bucket count.
5. **Querying**: the planner turns `WHERE dt='2024-01-01'` into "read only that directory"; for bucketed tables a join on the bucket column can be done bucket-by-bucket (bucket map join) and `TABLESAMPLE(BUCKET 1 OUT OF 4)` reads one file.

```
/user/hive/warehouse/sales/
  dt=2024-01-01/country=IN/000000_0
  dt=2024-01-01/country=US/000000_0
  dt=2024-01-02/country=IN/000000_0
```

**Storage formats**

| Format | Layout | Splittable | Notes |
|---|---|---|---|
| TEXTFILE | Row, delimited text | Yes (uncompressed / bzip2) | Default, human-readable, slowest |
| SEQUENCEFILE | Row, binary key/value | Yes | Hadoop-native, rarely chosen now |
| AVRO | Row, binary + schema | Yes | Good for schema evolution, ingestion |
| ORC | Columnar, stripes, indexes | Yes | Best for Hive; supports ACID, predicate pushdown |
| PARQUET | Columnar, row groups | Yes | Best cross-engine (Spark, Impala, Presto) |

### 4. Worked example
```sql
-- external table over raw landing data (not owned by Hive)
CREATE EXTERNAL TABLE raw_events (
  user_id BIGINT, url STRING, ts TIMESTAMP, tags ARRAY<STRING>, props MAP<STRING,STRING>)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
LOCATION '/data/landing/events/';

-- managed, partitioned, columnar curated table
CREATE TABLE events (
  user_id BIGINT, url STRING, ts TIMESTAMP)
PARTITIONED BY (dt STRING)
CLUSTERED BY (user_id) INTO 16 BUCKETS
STORED AS ORC;

SET hive.exec.dynamic.partition.mode=nonstrict;
INSERT OVERWRITE TABLE events PARTITION (dt)
SELECT user_id, url, ts, date_format(ts,'yyyy-MM-dd') AS dt FROM raw_events;

-- only the dt=2024-01-01 directory is read
SELECT count(*) FROM events WHERE dt = '2024-01-01';
```
What to look at: `SHOW PARTITIONS events;` lists directories registered in the metastore; `EXPLAIN` shows the pruned partition list; `hadoop fs -ls` shows the sub-directory structure. If you copy files into a partition directory by hand, the metastore doesn't know — run `MSCK REPAIR TABLE events;` (or `ALTER TABLE … ADD PARTITION`).

### 5. Under the hood
- A partition column is **not stored in the data files**; it is encoded in the path and kept in the metastore. That is why it can't also be a regular column.
- Bucketing uses the same hash as the writer expects, so two tables bucketed on the join key with compatible bucket counts can join without a full shuffle.
- **ORC** files are divided into *stripes* (~64–256 MB) each with column data, lightweight indexes (min/max, bloom filters) and a footer; readers skip stripes and row groups that can't match a predicate. **Parquet** is similar with *row groups* and column chunks.
- **Transactions**: `UPDATE`/`DELETE` need transactional (ACID) tables, normally ORC and bucketed in older releases; changes are stored as delta files merged by background compaction. Hive indexes (compact, bitmap) existed in older versions but were superseded by columnar-format indexes.
- Hive's type system is narrower than most RDBMSs, so imports (e.g. Sqoop) may down-cast types and warn about lost precision.

### 6. Performance and design implications
- Partition on low-cardinality columns that appear in filters (date, region). Thousands of partitions are fine; millions overload the metastore and create small files.
- Target file sizes close to the HDFS block size; many tiny files hurt both NameNode and query start-up.
- Use a columnar format plus compression (Snappy/ZSTD) for anything queried repeatedly.
- Bucket on high-cardinality join keys and choose a power-of-two bucket count that matches typical parallelism.
- Verify with `EXPLAIN`, `DESCRIBE FORMATTED`, and counters for bytes read.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PARTITIONED BY (col type)` | Define partition columns | Large, filter-by-date tables |
| `CLUSTERED BY (c) INTO n BUCKETS` | Define buckets | Join/sampling optimisation |
| `STORED AS ORC|PARQUET` | Choose format | Always for curated tables |
| `ALTER TABLE … ADD PARTITION` / `MSCK REPAIR TABLE` | Register partitions | After external writes |
| `hive.exec.dynamic.partition(.mode)` | Enable dynamic partitioning | Bulk loads |
| `TBLPROPERTIES ('transactional'='true')` | ACID table | Row-level updates |
| `ANALYZE TABLE … COMPUTE STATISTICS` | Gather stats | Before heavy joins |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| External table over shared/landing data | You want Hive to clean up files on drop |
| Managed table for curated warehouse data | Other tools also write there |
| Partition by date | Partition by high-cardinality column (user_id) |
| ORC for Hive-centric stack, Parquet for multi-engine | TEXTFILE for large, repeatedly scanned data |
| Buckets for big-big joins | Tiny tables, or when engine ignores bucketing |

### 9. Common mistakes
- Partitioning by a near-unique column → millions of tiny directories → pick a coarser key (month, region).
- Dropping a managed table that was meant to be shared → data gone → create it `EXTERNAL`.
- Writing files to a partition path without registering it → rows invisible → `MSCK REPAIR TABLE`.
- Expecting `LOAD DATA` to convert formats → file is copied untouched → use `INSERT … SELECT` into an ORC/Parquet table.
- Loading into a bucketed table without enforcing bucketing → wrong file count → set the flag / use a recent Hive.

### 10. 🎯 Interview kit
**30-second answer:** Partitions split a table into directories by column value so queries prune data; buckets hash-split rows into a fixed number of files for efficient joins and sampling; and columnar formats like ORC/Parquet cut I/O further. External tables leave data ownership with you, managed tables with Hive.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Partition vs bucket?* → directory per value vs fixed hash files; cardinality guidance.
- *Managed vs external?* → drop semantics and ownership.
- *Why ORC/Parquet over text?* → column pruning, predicate pushdown, compression, stats.
- *Small-files problem?* → NameNode memory, task overhead; fix via compaction, fewer partitions.

**Signals interviewers look for:** you pick keys using query patterns and cardinality, and you quantify the effect on bytes scanned.

### 11. Check yourself
- [ ] Sketch the directory tree for a table partitioned by `dt` and `country`.
- [ ] Explain what `MSCK REPAIR TABLE` fixes.
- [ ] Choose a format and partition scheme for a 5 TB event table and justify it.
- [ ] Describe how ACID updates work on an immutable filesystem.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Hive › Partitions and Buckets
- *Hadoop: The Definitive Guide* — Tom White — Hive › Warning
- *Hadoop: The Definitive Guide* — Tom White — Hive › Storage Formats
- *Hadoop: The Definitive Guide* — Tom White — Hive › Buckets
- *Hadoop: The Definitive Guide* — Tom White — Hive › An Example
- *Hadoop: The Definitive Guide* — Tom White — Hive › Joins
- *Hadoop: The Definitive Guide* — Tom White — Sqoop › Note
- *Hadoop: The Definitive Guide* — Tom White — Hive › Updates, Transactions, and Indexes
- *Hadoop: The Definitive Guide* — Tom White — Hive › Using a custom SerDe: RegexSerDe
- *Hadoop: The Definitive Guide* — Tom White — Hive › Data Types

