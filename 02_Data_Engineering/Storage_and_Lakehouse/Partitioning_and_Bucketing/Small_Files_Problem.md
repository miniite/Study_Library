---
id: "02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Small_Files_Problem"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing"
tools: ["Apache_Spark", "Hadoop"]
---

# Small Files Problem

**L4 Advanced** · [↑ Partitioning and Bucketing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Too many tiny files make every read pay a fixed open/list/metadata cost per file, so analytical tables must be written and periodically compacted into files of roughly 128 MB to 1 GB.

### 1. The big picture
Distributed engines read data in units of *files* (and blocks inside them). Opening a file, reading its footer, listing a directory and scheduling a task each carry a roughly fixed overhead — small on a local disk, large on HDFS NameNodes and on object stores such as S3 or ADLS where every list and open is a network request. When a table consists of millions of files of a few KB each, query time is dominated by that overhead instead of useful reading, planning slows, the catalog and NameNode are stressed, and cost rises (more requests, bigger clusters to compensate).

The books describe the problem as one that "sneaks up over time": streaming micro-batches, frequent small appends, over-partitioning, and many parallel writers each produce small files; nothing fails, queries just get slower and more expensive. Delta's guide loosely defines small files as under about 64 KB for the worst cases, but in practice anything well below the target size (tens of MB) is a concern.

### 2. Core concepts and vocabulary
- **Small file** — file much smaller than the engine's ideal read size (block size, ~128 MB).
- **Open cost** — fixed time per file (list, open, read footer, create task).
- **Block / split** — unit of parallel reading; `spark.sql.files.maxPartitionBytes` default 128 MB.
- **Compaction (bin-packing)** — rewriting many small files into fewer larger ones.
- **Write amplification of partitions** — files ≈ partitions × writer tasks.
- **Over-partitioning** — partition scheme so fine that each partition is tiny.
- **Coalesce / repartition** — Spark operations controlling output partitions (and so files).
- **Auto-optimize / optimized writes** — platform features that merge small outputs at write time.
- **NameNode pressure** — HDFS metadata held in memory per file/block; millions of files strain it.
- **Streaming micro-batch** — frequent small commits producing many small files.

### 3. How it works, step by step
**How small files arise**
1. A Spark job has `spark.sql.shuffle.partitions = 200` and writes to a table partitioned by 365 dates; each task writes a file for each date it holds → up to 73,000 files for one run.
2. A streaming job commits every few seconds, each commit adding at least one file per partition.
3. A source delivers thousands of tiny objects (logs, IoT) that are copied as-is.
4. Over-partitioning by hour/customer yields partitions of kilobytes.

**Detect**
1. Count files and bytes per partition/table (`DESCRIBE DETAIL`, file listing, catalog statistics).
2. Check average file size; investigate partitions with large file counts.
3. Look at job UIs: many tiny tasks, long scheduling time relative to compute.
4. Check source: is ingestion itself producing small files, or the batch writers?

**Fix**
1. Reduce parallel writers before writing: `coalesce(n)` or `repartition(n)` / `repartitionByRange` by partition columns so each partition gets few files; Spark 3 provides partitioning hints for SQL (`/*+ REPARTITION(...) */`).
2. Turn on adaptive query execution coalescing and optimized writes where available.
3. Batch ingestion: land bigger files, increase trigger interval for streaming.
4. Compact existing data on a schedule (`OPTIMIZE`, Iceberg `rewrite_data_files`, Hudi compaction, or a Spark rewrite job).
5. Re-evaluate partitioning (coarser granularity).

### 4. Worked example
```sql
-- Delta: inspect
DESCRIBE DETAIL sales.events;
-- numFiles, sizeInBytes → average size = sizeInBytes / numFiles

-- Per-partition file count (Spark)
SELECT event_date, count(*) AS files, sum(size)/1024/1024 AS mb
FROM (SELECT input_file_name() AS f, event_date, 1 AS x FROM sales.events) -- illustrative;
GROUP BY event_date ORDER BY files DESC;

-- Compact
OPTIMIZE sales.events WHERE event_date >= current_date - 7;
-- Iceberg equivalent
CALL catalog.system.rewrite_data_files(table => 'sales.events',
     options => map('target-file-size-bytes','536870912'));
```
```python
# Spark: avoid creating small files in the first place
(df.repartition("event_date")                 # one shuffle partition per date
   .write.partitionBy("event_date")
   .mode("append").parquet(path))
# or: df.coalesce(8) when the output is tiny
```
What to look at after compaction: `numFiles` drops, average file size rises toward target, and a representative query's tasks and wall-clock time fall. Old small files remain until `VACUUM`/snapshot expiry removes them after the retention period.

Trade-off of `repartition`: it adds a shuffle. It is worth it when the shuffle cost is smaller than the repeated open cost saved across all future reads of the table; for a rarely read staging table it may not be.

### 5. Under the hood
- **HDFS** keeps namespace metadata in NameNode memory — about 150 bytes per file/block object — so hundreds of millions of files can exhaust it, and each file occupies at least a block entry even if tiny.
- **Object stores** have no real directories; listing is paginated (1,000 keys per request on S3) and has throttled request rates per prefix, so enormous file counts under a prefix slow listing and can cause throttling.
- **Parquet** per-file footers contain schema and row-group stats; reading ten thousand footers is far costlier than reading ten. Tiny files also mean tiny row groups, hurting compression and statistics usefulness.
- **Spark** creates roughly one task per ~128 MB split but small files are packed together up to `spark.sql.files.openCostInBytes` (default 4 MB cost per file) — Spark deliberately *charges* a file as if it were 4 MB when packing, which is itself a sign of how heavy opening is.
- **Delta/Iceberg** keep the file list in metadata, avoiding directory listing, but metadata itself grows with file count; checkpoints and manifests must be rewritten.
- **Compaction conflicts:** compaction rewrites files while writers append; table formats handle this with optimistic commits, but compaction of a partition being heavily updated can fail and retry.

### 6. Performance and design implications
- Target 128 MB–1 GB files (smaller for tables with highly selective queries and Z-order; larger for pure scans).
- Compaction costs compute; schedule it on the *cold* tail (partitions no longer receiving writes) and keep hot partitions lightly optimised.
- Streaming: choose trigger intervals balancing latency with file count, or write to a staging table then compact into Silver.
- Partition granularity is the biggest lever: a few thousand well-sized partitions beat millions of tiny ones.
- Metrics: average file size, files per partition, number of tasks per stage, listing time, planning time, S3 request counts, cost per query.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTIMIZE` (Delta) | Bin-pack small files, optional Z-order | Scheduled compaction |
| `rewrite_data_files` (Iceberg) | Compaction with target size | Iceberg tables |
| `delta.autoOptimize.optimizeWrite` | Larger files at write time | Frequent appends |
| `coalesce(n)` / `repartition(n, cols)` | Control output files | Batch writes |
| `spark.sql.shuffle.partitions`, AQE coalesce | Reduce tiny tasks | Spark SQL jobs |
| `spark.sql.files.maxPartitionBytes` | Read split size | Reading many files |
| `VACUUM` / `expire_snapshots` | Delete unreferenced files | After compaction |
| `maxRecordsPerFile` | Cap file size | Prevent giant files |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Write-time repartition | Shuffle cost exceeds benefit (rarely read tables) |
| Scheduled compaction | Table rewritten constantly — compaction fights writers |
| Larger streaming trigger | Latency requirement is seconds |
| Coarser partitions | Need partition-level deletes/reprocessing at fine granularity |
| Very large files (1 GB+) | Selective point queries need fine skipping |

### 9. Common mistakes
- One file per partition per task with 5,000 partitions → millions of files → repartition by partition columns before writing.
- Compacting but never vacuuming → storage doubles → expire old files after retention.
- Setting target file size without checking read patterns → skip effectiveness lost → test with real queries.
- Fixing symptoms with bigger clusters → rising cost → fix the layout.
- Ignoring the ingestion stage that creates the tiny objects → problem recurs → batch at source or in landing.

### 10. 🎯 Interview kit
**30-second answer:** Small files make each read pay open, list and metadata overhead per file, slowing queries, stressing NameNode/object-store listing and wasting tasks. They come from over-partitioning, many parallel writers and frequent streaming commits. I fix them by controlling output partitions at write time, enlarging batches, scheduling compaction toward 128 MB–1 GB, and revisiting partition granularity.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Spark job with 5,000 partitions writing one file each?* → 5,000 tiny files per write; use repartition by the partition key or coalesce.
- *Is repartition worth the shuffle?* → compare shuffle cost with cumulative read savings.
- *Good target size?* → ~128 MB–1 GB depending on engine and query selectivity.
- *Kafka partitions vs consumers?* → parallelism capped by partition count (different "partition", same sizing idea).
- *What is skew?* → uneven data per partition; one task becomes the bottleneck.

**Signals interviewers look for:** root-cause thinking (ingestion vs batch), quantified targets, maintenance plan.

### 11. Check yourself
- [ ] Calculate files created by a job with 200 tasks and 30 partitions.
- [ ] List three detection signals and the command or UI for each.
- [ ] Explain why compaction does not immediately reduce storage.
- [ ] Choose between repartition-before-write and post-write compaction for a streaming job.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Intermediate] [What is the 'small files problem' and how can bad partitioning cause it? Explain simply.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35321_What_is_the_small_files_problem_and_how_can_bad_partitioning.md>)
- [Intermediate] [What is data skew in the context of partitions, and why is it a problem?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35413_What_is_data_skew_in_the_context_of_partitions_and_why_is_it.md>)
- [Intermediate] [What does 'over-partitioning' mean and how would you recognize it in a data lake?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35423_What_does_over-partitioning_mean_and_how_would_you_recognize.md>)
- [Intermediate] [Why might reading from thousands of tiny partitions be slower than reading from a few larger ones?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35424_Why_might_reading_from_thousands_of_tiny_partitions_be_slowe.md>)
- [Intermediate] [If your Spark job writes one file per partition and you have 5000 partitions, what goes wrong?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35427_If_your_Spark_job_writes_one_file_per_partition_and_you_have.md>)
- [Intermediate] [In Kafka, how does the number of partitions relate to how many consumers can read in parallel?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35435_In_Kafka_how_does_the_number_of_partitions_relate_to_how_man.md>)
- [Intermediate] [If your Spark output has too many small files, how could adjusting partitions before write help?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35454_If_your_Spark_output_has_too_many_small_files_how_could_adju.md>)
- [Intermediate] [What is a good target file size for files inside a partition, and why do we care?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Q-35455_What_is_a_good_target_file_size_for_files_inside_a_partition.md>)

## Related concepts

- [Partitioning Strategy](<../../../01_Databases_and_SQL/Database_Operations/Partitioning/Partition_Pruning/Partitioning_Strategy.md>) *(Partitioning)*
- [Lakehouse Fundamentals](<../Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Partitioning and Data Skipping in Lakes](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Small Files in a Big Data World A Data Engineer's Biggest Nightmare › Why Does It Happen?
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Small Files in a Big Data World A Data Engineer's Biggest Nightmare › Detect and Mitigate
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › How partitions are created
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › Note
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Maintaining Your Delta Lake › The Problem with Big Tables and Small Files
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Maintaining Your Delta Lake › Creating the small file problem
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Performance Tuning Optimizing Your Data Pipelines with Delta Lake › File sizes
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Performance Tuning › Compact Files
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Data Lakes › Optimize file size

