---
id: "02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Table_Formats"
tools: ["Databricks", "Apache_Spark", "Hadoop"]
---

# Table Format Internals

**L4 Advanced** · [↑ Table Formats](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Open table formats (Delta Lake, Apache Iceberg, Apache Hudi) turn a directory of immutable Parquet files into an ACID table by keeping a metadata layer that records exactly which files make up each version of the table.

### 1. The big picture
Plain Parquet in S3 with a crawler is a *directory listing* pretending to be a table. Failures follow naturally: a reader sees half of a job's output; two writers overwrite each other; a crash leaves orphan files; there is no safe way to update or delete a single row (GDPR); schema changes are not versioned; there is no time travel; and planning requires listing millions of keys. A **table format** fixes these by making the *metadata* authoritative: a table is "the set of data files referenced by the latest committed snapshot", and a commit is an atomic metadata operation.

Iceberg tracks all files in a table and the files in each snapshot over time, giving time travel, schema evolution and petabyte scale. Delta Lake uses an ordered transaction log and adds features such as generated columns, CHECK constraints, comments and deletion vectors (merge-on-read for deletes). Hudi adds record-level indexing and timeline-based upserts. The choice among them depends on read/write patterns and ecosystem fit: Delta is strongest in Spark/Databricks estates; Iceberg is engine-agnostic (Trino, Flink, Spark, Snowflake, etc.); Hudi targets streaming upserts.

### 2. Core concepts and vocabulary
- **Data file** — immutable Parquet/ORC/Avro file holding rows.
- **Snapshot / version** — consistent view of the table after a commit.
- **Transaction log (Delta)** — ordered JSON commit files in `_delta_log/` plus periodic Parquet checkpoints.
- **Manifest file (Iceberg)** — list of data files with partition values and column statistics.
- **Manifest list** — per-snapshot list of manifests.
- **Metadata file** — table state: schema, partition spec, snapshots, properties; pointer held by the catalog.
- **Catalog** — maps table name to current metadata location; performs atomic pointer swap.
- **Copy-on-write (CoW) / merge-on-read (MoR)** — rewrite files on update vs record changes in delta/delete files and merge at read.
- **Deletion vector / delete file** — marks rows as deleted without rewriting the data file.
- **Compaction / rewrite** — combine small files, apply deletes.
- **Snapshot expiry / VACUUM** — remove unreferenced files after retention.
- **Orphan files** — files in storage not referenced by any snapshot (failed writes).

### 3. How it works, step by step
**Write (Iceberg-style):**
1. Writer reads the current metadata to learn the table state.
2. It writes new data files to object storage.
3. It writes a new manifest listing those files with statistics.
4. It writes a new manifest list and a new metadata file that includes the new snapshot.
5. It asks the catalog to atomically swap the table pointer from the old to the new metadata file (compare-and-swap).
6. If another writer swapped first, validation checks for conflict; if compatible it rebases and retries, otherwise it fails.

**Write (Delta-style):** write data files, then attempt to create the next numbered JSON commit file (`00000000000000000042.json`) using put-if-absent semantics; if it already exists, another writer won, so check conflicts and retry.

**Read / plan an Iceberg query:**
1. Catalog lookup → current metadata file.
2. Choose snapshot (latest or time-travel).
3. Read manifest list; filter manifests by partition summaries.
4. Read surviving manifests; filter data files by partition and column min/max.
5. Scan remaining data files (and apply delete files if any).
Each step can become a bottleneck: catalog latency, huge manifest lists, manifests that are too many/too small, or too many data files.

**Delete a user's data (GDPR):**
1. `DELETE FROM t WHERE user_id = ...` — with data skipping only files containing that user are touched.
2. CoW rewrites those files; MoR writes deletion vectors/delete files.
3. Data still physically exists in older files referenced by previous snapshots.
4. After retention, `VACUUM`/`expire_snapshots` plus orphan cleanup physically remove them.

### 4. Worked example
```sql
-- Delta
CREATE TABLE sales.orders (
  order_id BIGINT NOT NULL, user_id BIGINT, amount DECIMAL(12,2),
  order_date DATE, CONSTRAINT positive CHECK (amount >= 0)
) USING DELTA PARTITIONED BY (order_date);

DELETE FROM sales.orders WHERE user_id = 12345;      -- GDPR delete
DESCRIBE HISTORY sales.orders;                        -- commit log
VACUUM sales.orders RETAIN 168 HOURS;                 -- physically remove old files

-- Iceberg maintenance
CALL cat.system.rewrite_data_files(table => 'db.orders');
CALL cat.system.rewrite_manifests('db.orders');
CALL cat.system.expire_snapshots(table => 'db.orders', older_than => TIMESTAMP '2026-09-01 00:00:00');
CALL cat.system.remove_orphan_files(table => 'db.orders');

-- Inspect metadata tables (Iceberg)
SELECT count(*) FROM db.orders.files;
SELECT * FROM db.orders.snapshots ORDER BY committed_at DESC LIMIT 5;
SELECT * FROM db.orders.manifests;
```
Reading the result: if reads have slowed while data volume is flat, `manifests` and `files` counts will show manifest bloat or many small files; if `snapshots` has thousands of entries, metadata is heavy. A violated CHECK constraint makes the *entire* write batch fail atomically — nothing partial is committed.

**Why "my time travel broke":** a nightly `VACUUM` with a short retention removed files that an old snapshot (or long-running query) still referenced; reading that version fails. Retention must exceed the longest query and time-travel window.

### 5. Under the hood
- **Atomicity** comes from the single atomic step (rename/put-if-absent in the log, or catalog pointer swap). Data files are written first, invisible until the commit.
- **Delta checkpoints:** every ~10 commits a Parquet checkpoint summarises state so readers needn't replay every JSON file.
- **Iceberg hidden partitioning:** the partition spec lives in metadata with transforms (`day(ts)`, `bucket(16, id)`, `truncate`); partition spec evolution doesn't rewrite old files — old and new specs coexist per manifest.
- **Statistics:** per-file column min/max, null counts, record counts enable data skipping; Delta may collect stats on the first 32 columns by default; keep wide tables' filter columns early or configure.
- **Isolation:** snapshot isolation for readers; writers use optimistic concurrency. Delta's `OPTIMIZE` is compatible with concurrent appends, but conflicts with concurrent updates/deletes touching the same files; schedule maintenance away from heavy writers or per partition.
- **Streaming exactly-once:** Flink's Iceberg sink writes data files per checkpoint and commits them in the checkpoint-complete callback with the checkpoint id stored in snapshot summary; on recovery it skips already committed checkpoints. Duplicates can still appear from at-least-once sources upstream, non-deterministic keys, or multiple sinks writing the same table without idempotent upsert mode.
- **S3 request costs:** listing and reading many small manifests, many small data files, and frequent metadata refresh drive GET/LIST requests; fix with compaction, manifest rewrite, caching of metadata and fewer polling readers.
- **Merge-on-read:** cheap updates/deletes, slower reads until compaction merges deltas; Delta deletion vectors and Iceberg v2 delete files implement this.

### 6. Performance and design implications
- Planning cost scales with manifest and file counts; schedule compaction and manifest rewrite.
- Choose target file size (e.g., 128–512 MB); large for batch ETL, smaller if BI requires fine skipping.
- Prove pruning is happening: examine the plan for partition filters and `files read / total files`; compare bytes scanned with expected; a dashboard scanning 4 TB instead of 40 GB usually means non-sargable filters, type mismatch or poor clustering.
- Keep metadata retention sane: expire snapshots on a schedule, but longer than the longest reader.
- Engines differ: Spark has the richest write/maintenance support; Trino supports reads and many writes with its own maintenance procedures; Flink excels at streaming writes but compaction is often run separately. Catalog configuration (Glue, Hive Metastore, REST, Nessie, Unity) must match across engines.
- Metrics: snapshot count, manifest count, average file size, commit retry rate, planning time, request counts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTIMIZE` / `rewrite_data_files` | Compact files, sort/cluster | Small-file and skew control |
| `rewrite_manifests` | Consolidate manifests | Slow planning |
| `VACUUM` / `expire_snapshots` | Remove unreferenced data/metadata | Cost control, GDPR completion |
| `remove_orphan_files` | Delete files not in any snapshot | After failed writes |
| `DESCRIBE HISTORY` / `snapshots` table | Inspect commits | Audit, debug |
| CHECK / NOT NULL constraints | Reject bad batches | Table-level quality |
| `delta.deletedFileRetentionDuration`, `history.expire.max-snapshot-age-ms` | Retention | Balance time travel vs cost |
| `write.delete.mode` / deletion vectors | CoW vs MoR | Update-heavy tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Delta | Spark/Databricks-centric platform, need constraints and DV |
| Iceberg | Many engines, hidden partitioning, partition evolution, catalog neutrality |
| Hudi | Frequent record-level upserts and incremental pulls |
| Merge-on-read | Read latency sensitivity high |
| Plain Hive tables | Legacy, read-only archives, no concurrent writes |

### 9. Common mistakes
- Using raw Parquet and a crawler for mutable data → partial reads, no deletes → adopt a table format.
- No maintenance schedule → manifest/file bloat, slow reads, cost → compaction + snapshot expiry + orphan cleanup.
- Aggressive VACUUM retention → broken time travel and long-running queries → retention longer than max query duration.
- Running OPTIMIZE during heavy updates to same files → conflicts/retries → schedule or partition-scope.
- Forgetting orphan cleanup → storage leakage after failed jobs.
- Assuming delete is physical delete → privacy gap → complete with expiry/vacuum.

### 10. 🎯 Interview kit
**30-second answer:** A table format keeps a versioned metadata layer — Delta's transaction log or Iceberg's metadata → manifest list → manifests — listing the exact data files of each snapshot. Commits are atomic metadata swaps, so readers see consistent snapshots, writers use optimistic concurrency, and features like time travel, schema evolution, row-level deletes and data skipping become possible over immutable Parquet.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Why not Parquet plus crawler?* → no atomicity, no safe updates/deletes, no schema versioning, listing cost.
- *GDPR delete on 50 TB?* → predicate DELETE (touches only matched files), then expire/vacuum after retention.
- *Iceberg reads slowed with flat volume?* → manifest/file bloat; rewrite manifests, compact.
- *Query planning flow?* → catalog → snapshot → manifest list → manifests → data files.
- *Exactly-once from Flink?* → commit on checkpoint with checkpoint ids; duplicates from upstream or multiple writers.
- *VACUUM broke time travel?* → retention shorter than reader/version needs.

**Signals interviewers look for:** can narrate commit and read paths; knows maintenance duties; chooses format on ecosystem fit.

### 11. Check yourself
- [ ] Draw the Iceberg metadata tree and mark where partition pruning and column-stats pruning occur.
- [ ] Explain how two concurrent writers are reconciled.
- [ ] Write the maintenance runbook (four procedures, order, frequency).
- [ ] Explain the difference between snapshot expiry and orphan-file removal.
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../../../TOOLS/Databricks/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Intermediate] [You need to GDPR-delete a specific user's rows from a 50TB Delta table. Walk through how you'd do it efficiently and what VACUUM does afterward.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-42959_You_need_to_GDPR-delete_a_specific_user_s_rows_from_a_50TB_D.md>)
- [Intermediate] [A junior engineer asks why they can't just use Parquet directly in S3 with a Glue crawler. Give the concrete failure scenarios a table format prevents.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-43017_A_junior_engineer_asks_why_they_can_t_just_use_Parquet_direc.md>)
- [Deep] [Your Iceberg reads got slow over weeks even though data volume is flat. How do you investigate manifest bloat and what maintenance procedures fix it?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-42961_Your_Iceberg_reads_got_slow_over_weeks_even_though_data_volu.md>)
- [Deep] [Explain how a query engine plans a read against an Iceberg table: catalog lookup → snapshot → manifest list → manifest filtering → data-file scan. Where can each step become a bottleneck?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-42981_Explain_how_a_query_engine_plans_a_read_against_an_Iceberg_t.md>)
- [Deep] [A team reports S3 request costs spiking after adopting Iceberg. Which access patterns (LIST, manifest reads, small files) drive request charges and how do you cut them?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-43008_A_team_reports_S3_request_costs_spiking_after_adopting_Icebe.md>)
- [Deep] [What are the practical differences in how Spark, Trino, and Flink integrate with Iceberg (write support, maintenance, catalog config)? Where do teams get surprised?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-43029_What_are_the_practical_differences_in_how_Spark_Trino_and_Fl.md>)
- [Deep] [How does Delta's OPTIMIZE interact with an active streaming write in terms of conflict detection, and how do you schedule it to minimize retries?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-43052_How_does_Delta_s_OPTIMIZE_interact_with_an_active_streaming_.md>)
- [Expert] [Your team wants exactly-once writes from a Flink streaming job into Iceberg. How does the Iceberg Flink sink achieve this and where can duplicates still creep in?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Q-42996_Your_team_wants_exactly-once_writes_from_a_Flink_streaming_j.md>)

## Related concepts

- [Lakehouse Fundamentals](<../Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [File and Table Formats for Features](<../../Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*
- [Partitioning and Data Skipping in Lakes](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*
- [Optimistic Concurrency and Clocks](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>) *(Distributed Systems Core)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Tiered Lake Storage](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>) *(Data Architecture)*

## References

- *Data Engineering with Medallion Architecture* — Miki Eto — Medallion Architecture in Multi-cloud › Expert notes
- *Data Engineering with Medallion Architecture* — Miki Eto — Data Governance and Compliance › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Glossary
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Hybrid Serialization
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Iceberg
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Introduction to the Delta Lake Lakehouse Format › Observing the Interaction Between the Metadata and Data
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Read
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Overwriting Data in a Delta Lake Table
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Table Operations
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Scaling Massive Metadata
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Table Deletes, Updates, and Merges › Chapter 4. Table Deletes, Updates, and Merges
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Table Deletes, Updates, and Merges › Performing the DELETE Operation
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Table Deletes, Updates, and Merges › DELETE Performance Tuning Tips
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Key features
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Hudi
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Supported Features
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Apache Hudi, Apache Iceberg, and Delta Lake
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Conclusion
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Managing Delta Tables
- *Building Medallion Architectures* — Piethein Strengholt — Laying the Groundwork › Compaction and Optimized Writes

