---
id: "TOOLS/Databricks/Databricks_Platform/Delta_Lake_in_Databricks"
level: 4
type: "concept"
topic: "TOOLS/Databricks/Databricks_Platform"
tools: ["Databricks", "Apache_Spark"]
---

# Delta Lake in Databricks

**L4 Advanced** · [↑ Databricks Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Delta Lake is a transaction log stored next to ordinary Parquet files in cloud object storage; that log is what turns a folder of immutable files into a table with ACID commits, versions, time travel and schema control — and it is the default table format in Databricks.

### 1. The big picture
A plain data lake is a folder of Parquet files. Object storage (S3, ADLS, GCS) can only write whole objects and cannot rename or update them atomically across many files, so a plain folder has no notion of "this write either fully happened or did not", no way to update one row, and no safe way for a reader and a writer to work at the same time. A job that dies half-way leaves a half-written folder that readers happily consume.

Delta Lake fixes this with a small idea: keep the data files immutable, and record *which files make up the table at each version* in an ordered transaction log (`_delta_log/`). The table is whatever the log says it is. Readers pick a version of the log and read exactly the files it lists; writers add new files and then append one log entry that makes them visible all at once. Everything else Databricks users value — `UPDATE`/`DELETE`/`MERGE`, time travel, change data feed, streaming in and out of the same table, `OPTIMIZE` — is built on that log.

This lesson is the foundation for the other Databricks lessons: ingestion (Auto Loader writes Delta), Delta Live Tables (declarative Delta pipelines), and streaming tuning (checkpoints, triggers, retention) all assume the mechanics below.

### 2. Core concepts and vocabulary
- **Delta table** — a directory of Parquet data files plus a `_delta_log/` subdirectory; one table = one directory (or a Unity Catalog managed table backed by one).
- **Transaction log (`_delta_log`)** — ordered JSON commit files (`00000000000000000000.json`, `…001.json`, …) each describing actions: `add` file, `remove` file, `metaData` (schema, partition columns), `protocol`, `commitInfo`.
- **Checkpoint file** — every 10 commits a Parquet snapshot of the log state, so readers don't replay thousands of JSON files.
- **Version** — the commit number; version *N* of the table = the result of applying commits 0…N.
- **Snapshot** — the set of live files for one version.
- **Optimistic concurrency control** — writers don't lock; they write data files, then try to commit; if another writer committed a conflicting change first, the commit fails and is retried or raises an error.
- **Time travel** — reading an earlier version with `VERSION AS OF n` or `TIMESTAMP AS OF '…'`.
- **Schema enforcement** — a write whose schema doesn't match the table is rejected.
- **Schema evolution** — explicit opt-in to change the schema (`mergeSchema`, `overwriteSchema`, `ALTER TABLE`).
- **`OPTIMIZE`** — compacts many small files into larger ones (optionally with `ZORDER BY` or liquid clustering).
- **`VACUUM`** — physically deletes data files no longer referenced by the log and older than the retention threshold.
- **Change Data Feed (CDF)** — row-level record of inserts, updates (pre/post image) and deletes between versions.

### 3. How it works, step by step
How an `UPDATE orders SET status='shipped' WHERE id=7` commits on object storage:

1. **Read the latest snapshot** — find the newest checkpoint, replay the JSON commits after it, and get the list of live files.
2. **Find affected files** — use per-file min/max statistics in the log to find which files might contain `id = 7`.
3. **Write new files** — rewrite each affected file with the change applied. Delta never edits a Parquet file in place; the old file is untouched.
4. **Attempt the commit** — write the next numbered JSON file (`N+1.json`) containing `remove` actions for the old files and `add` actions for the new ones. The storage layer guarantees that only one writer can create that exact file name (put-if-absent semantics; on S3 Delta uses a coordination mechanism for this).
5. **Conflict check** — if someone else already created `N+1.json`, re-read the new commits and check whether they touched the same files. Disjoint changes (e.g. appends vs. an update to other files) are reconciled and retried; real conflicts raise `ConcurrentModificationException`-style errors.
6. **Readers** — a query that started at version *N* keeps reading version *N* files even while the commit lands: snapshot isolation without locks.

```
orders/
  part-0001.snappy.parquet      <- removed in v4, still on disk until VACUUM
  part-0002.snappy.parquet
  part-0007.snappy.parquet      <- added in v4
  _delta_log/
    00000000000000000003.json
    00000000000000000004.json   <- remove 0001, add 0007
```

Delta isolation level for writes is **Serializable by default** for most operations and `WriteSerializable` for others (Databricks default for `UPDATE/DELETE/MERGE`): a write may be ordered differently from the real-time order as long as the table state is explainable by *some* serial order.

### 4. Worked example
```sql
-- 1) create and write
CREATE TABLE sales.orders (
  order_id BIGINT, customer_id BIGINT, status STRING, amount DECIMAL(12,2), order_ts TIMESTAMP
) USING DELTA;                       -- DELTA is the default in Databricks

INSERT INTO sales.orders VALUES (1, 10, 'placed', 25.00, current_timestamp());

-- 2) change data: each statement is one atomic commit
UPDATE sales.orders SET status = 'shipped' WHERE order_id = 1;
DELETE FROM sales.orders WHERE status = 'cancelled';

-- 3) look at history
DESCRIBE HISTORY sales.orders;
-- version | timestamp | operation | operationMetrics (numFiles, numOutputRows, …)

-- 4) bad load corrupted today's data? read or restore the good version
SELECT * FROM sales.orders VERSION AS OF 12;
SELECT * FROM sales.orders TIMESTAMP AS OF '2026-10-11 23:00:00';
RESTORE TABLE sales.orders TO VERSION AS OF 12;   -- itself a new commit (v+1)

-- 5) maintenance
OPTIMIZE sales.orders;                            -- compact small files
VACUUM sales.orders RETAIN 168 HOURS;             -- delete unreferenced files > 7 days
```
Read the output of `DESCRIBE HISTORY`: every row is a commit; `operationMetrics` tells you how many files were added/removed — a `MERGE` that rewrote 4,000 files for 10 changed rows is the classic sign of a poorly clustered table.

**Schema evolution** (`mergeSchema` vs `overwriteSchema`):
```python
# new column "channel" appears in the incoming data
(df.write.format("delta")
   .mode("append")
   .option("mergeSchema", "true")      # ADD new columns; existing rows get NULL
   .saveAsTable("sales.orders"))

(df.write.format("delta")
   .mode("overwrite")
   .option("overwriteSchema", "true")  # REPLACE the schema (only with overwrite mode)
   .saveAsTable("sales.orders"))
```
Without either option the append fails with a schema-mismatch error — that is enforcement doing its job.

### 5. Under the hood
- **Log replay cost** — the table state is not stored anywhere but derived from checkpoint + trailing JSON commits. Very frequent tiny commits (streaming at 1-second triggers) inflate the log and file count; `delta.checkpointInterval` and compaction control this.
- **Statistics** — each `add` action carries min/max/null-count for the first 32 columns by default (`delta.dataSkippingNumIndexedCols`). Data skipping uses these to prune files; this is why column order and clustering matter (see the query-performance lesson).
- **Protocol versions** — `minReaderVersion`/`minWriterVersion` gate features (column mapping, deletion vectors, CDF). Upgrading the protocol is one-way; old clients may no longer read the table.
- **Deletion vectors** — newer Databricks runtimes can mark rows as deleted in a small side file instead of rewriting the whole Parquet file; `OPTIMIZE`/`REORG` later physically applies them. This makes `DELETE`/`UPDATE`/`MERGE` much cheaper on large files.
- **Retention knobs** — `delta.logRetentionDuration` (default 30 days) bounds how far back *metadata* history is kept; `delta.deletedFileRetentionDuration` (default 7 days) is the minimum age before `VACUUM` may delete a removed file. Time travel to a version works only if **both** the log entry and the data files still exist.
- **Managed vs external tables** — in Unity Catalog, a managed table lives in storage the catalog controls and is deleted with `DROP TABLE`; an external table points at a path you own and `DROP` leaves the files.

### 6. Performance and design implications
- Many small files are the number-one Delta performance problem: listing and opening files dominates. Schedule `OPTIMIZE` (or enable auto-compaction / optimized writes) and aim for files in the hundreds-of-MB range.
- `MERGE` cost = cost of the *match scan* + cost of rewriting touched files. Add a partition/clustering predicate to the `ON` clause so the scan is pruned.
- Don't over-partition. Partition only on low-cardinality columns that are almost always filtered (date); otherwise prefer `ZORDER`/liquid clustering.
- `VACUUM` with a retention shorter than the longest-running reader or stream can delete files that reader still needs. Keep ≥ 7 days unless you know exactly what is reading.
- Measure with `DESCRIBE DETAIL table` (numFiles, sizeInBytes), `DESCRIBE HISTORY` (operationMetrics) and the Spark UI's "files read / files pruned".

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DESCRIBE HISTORY t` | list commits, operations, metrics | audit, find the bad version |
| `SELECT … VERSION AS OF n` / `TIMESTAMP AS OF` | read a past snapshot | debugging, reproducing a report |
| `RESTORE TABLE t TO VERSION AS OF n` | roll the table back (new commit) | undo a bad load |
| `OPTIMIZE t [ZORDER BY (c)]` | compact files, co-locate data | small files, selective filters |
| `VACUUM t RETAIN n HOURS` | delete unreferenced files | storage cost, GDPR physical delete |
| `CONVERT TO DELTA parquet.\`path\`` | in-place conversion of a Parquet directory | migrating legacy tables |
| `CREATE TABLE … DEEP/SHALLOW CLONE` | copy / zero-copy snapshot | testing, backups |
| `delta.enableChangeDataFeed = true` | record row-level changes | incremental downstream loads |
| `mergeSchema` / `overwriteSchema` | schema evolution on write | new source columns / full redesign |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Delta table over raw Parquet — you need updates, deletes, concurrent readers/writers, history | write-once, read-by-another-engine-only archives where transactions add no value |
| `mergeSchema` — upstream adds columns and you want them to flow through | the upstream is untrusted; silently widening the schema hides breaking changes (quarantine instead) |
| Long retention (30+ days) for audits and recovery | storage cost matters and the data is large; GDPR deletes need `VACUUM` to actually remove bytes |
| `CONVERT TO DELTA` for in-place migration | consumers read the old Parquet path directly and must not see log files — use a new Delta path and cut over |
| `SHALLOW CLONE` for dev/test | you need the clone to survive `VACUUM` on the source (it won't — use deep clone) |

### 9. Common mistakes
- Calling Delta "a database file format" and forgetting it is Parquet + a log → can't explain why `VACUUM` breaks time travel → remember data files live until vacuumed, history ends when either log or files expire.
- Running `VACUUM RETAIN 0 HOURS` to "save cost" → deletes files active queries/streams need and ends time travel → keep the default or set a safe floor.
- Assuming `DELETE` removes bytes → the data is still in older files until `VACUUM` → for GDPR/RTBF run delete **then** vacuum after retention, and account for copies in clones/CDF.
- Appending with a drifted schema and "fixing" it by setting `overwriteSchema` → wipes the old schema → use `mergeSchema` for additive changes only.
- Partitioning by a high-cardinality column (user_id) → millions of tiny files → partition by date or use clustering.
- Mixing `OPTIMIZE` and streaming writers without understanding conflicts → occasional commit failures → rely on row-level concurrency/optimized writes or schedule compaction in quiet windows.

### 10. 🎯 Interview kit
**30-second answer:** Delta Lake is Parquet files plus a transaction log in `_delta_log`. A write adds new immutable files and then atomically appends a log entry listing adds/removes, which gives ACID commits, snapshot-isolated reads, time travel by version, schema enforcement, and efficient `MERGE`/`UPDATE`/`DELETE` on top of cheap object storage.

**Structure a longer answer:** clarify (object storage has no atomic multi-file ops) → core mechanism (immutable files + ordered log + optimistic concurrency + checkpoints) → trade-offs (small files, vacuum vs time travel, conflict retries) → close with a scenario (restoring after a bad load with `RESTORE`).

**Likely follow-ups:**
- *How does Delta achieve ACID on S3?* → atomic creation of the next numbered log file (put-if-absent or a coordinator), optimistic concurrency, snapshot reads.
- *What happens on concurrent writers?* → both write files; first commit wins; the second checks logical conflicts and either reconciles or fails and retries.
- *`VACUUM` vs `OPTIMIZE`?* → OPTIMIZE rewrites small files into big ones (creates new files, marks old as removed); VACUUM deletes the *removed* files that are past retention.
- *Migrate Parquet to Delta with zero downtime?* → `CONVERT TO DELTA` or write a new Delta copy, keep the old path read-only, switch consumers via the metastore/view, then retire the old path.
- *How would you do GDPR deletes with 30 days of history?* → delete from the table, shorten retention for that table, `VACUUM`, and handle clones/backups/CDF copies.

**Signals interviewers look for:** you can say *where* ACID comes from (the log, not Parquet), you know time travel has a retention limit tied to VACUUM, and you can reason about small files and conflicts.

### 11. Check yourself
- [ ] Draw what `_delta_log` contains after an insert, an update and an `OPTIMIZE`.
- [ ] Explain why a reader never sees a half-finished `MERGE`.
- [ ] State exactly when `VERSION AS OF 3` stops working.
- [ ] Choose between `mergeSchema` and `overwriteSchema` for two given scenarios.
- [ ] Describe how you would roll back a table that a bad job overwrote this morning.
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../_index.md>)
- [Apache Spark](<../../Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What is Delta Lake and why is it the default storage format in Databricks?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21760_What_is_Delta_Lake_and_why_is_it_the_default_storage_format_.md>)
- [Intermediate] [What advantages does a Delta table give you over a plain Parquet file?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21761_What_advantages_does_a_Delta_table_give_you_over_a_plain_Par.md>)
- [Intermediate] [How does Delta Lake give you ACID transactions on files stored in cloud object storage?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21762_How_does_Delta_Lake_give_you_ACID_transactions_on_files_stor.md>)
- [Intermediate] [What is the Delta transaction log (_delta_log) and what does it store?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21763_What_is_the_Delta_transaction_log_delta_log_and_what_does_it.md>)
- [Intermediate] [What is time travel in Delta Lake and give an example of when you would use VERSION AS OF or TIMESTAMP AS OF?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21764_What_is_time_travel_in_Delta_Lake_and_give_an_example_of_whe.md>)
- [Intermediate] [How would you read a specific older version of a Delta table if a bad load corrupted todays data?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21765_How_would_you_read_a_specific_older_version_of_a_Delta_table.md>)
- [Intermediate] [What is schema evolution in Delta Lake and how do mergeSchema and overwriteSchema options behave?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21841_What_is_schema_evolution_in_Delta_Lake_and_how_do_mergeSchem.md>)
- [Deep] [Explain how you'd migrate a set of legacy Parquet tables to Delta with zero downtime for downstream consumers still reading the old paths.](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21860_Explain_how_you_d_migrate_a_set_of_legacy_Parquet_tables_to_.md>)

## Related concepts

- [Table Format Internals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>) *(Table Formats)*
- [Lakehouse Fundamentals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [File and Table Formats for Features](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*

## References

- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Mapping Data Flow Advanced Topics › Delta Lake
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Delta Lake
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › The Delta Ecosystem
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Delta Lake Storage
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Delta Connectors
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › Chapter 2. Managing Data with Delta Lake
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › What Is Delta Lake?
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › Delta Lake Transaction Log
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › Understanding Delta Lake Functionality
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Databricks Delta Lake › Chapter 6: Databricks Delta Lake
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › Delta Lake
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Introduction to the Delta Lake Lakehouse Format › Chapter 1. Introduction to the Delta Lake Lakehouse Format
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Introduction to the Delta Lake Lakehouse Format › What Is Delta Lake?
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Introduction to the Delta Lake Lakehouse Format › Key Features
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Introduction to the Delta Lake Lakehouse Format › Anatomy of a Delta Lake Table

