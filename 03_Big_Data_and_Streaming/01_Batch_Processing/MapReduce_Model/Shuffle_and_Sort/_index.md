---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/MapReduce_Model/Shuffle_and_Sort"
level: 6
type: "curriculum"
curriculum: "data_engineering"
---

# Shuffle and Sort Internals

**L6 Internals** · [↑ The MapReduce Programming Model](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The shuffle is the machinery between map and reduce that partitions, sorts, spills, merges and copies every map output so each reducer receives all the values of its keys in sorted order — and it is where most MapReduce (and Spark) time and tuning effort goes.

### 1. The big picture

MapReduce promises one thing to the programmer: **the input to every reducer is sorted by key**, and all values of a key arrive at the same reducer. Nothing in your `map` or `reduce` code does that. The framework does it in the phase between them, called the **shuffle** (often "shuffle and sort"). It moves data from every map task's machine to every reduce task's machine — an all-to-all network exchange — and sorts it on the way.

Two facts make the shuffle the heart of the model:

- It is the only point where data crosses the network between tasks, so it dominates the cost of most jobs.
- It is where "free" behaviours such as grouping, sorted output, secondary sort and sort-merge joins come from.

The same idea reappears everywhere: Hive's `GROUP BY` and `ORDER BY`, HBase region splits, and Spark's `Exchange` operator in a query plan are all shuffles. Understanding the Hadoop version (buffer → spill → merge → copy → merge) gives you the vocabulary for Spark tuning too.

This lesson sits under the MapReduce model: the *Job Anatomy* lesson covers who runs what; here we open the box between map and reduce.

### 2. Core concepts and vocabulary

- **Shuffle** — the process of sorting map output and transferring it to reducers as their input.
- **Partition** — the slice of map output destined for one particular reducer. By default `hash(key) mod numReducers`.
- **Circular buffer** — the in-memory area a map task writes its output into before anything touches disk (100 MB by default, `mapreduce.task.io.sort.mb`).
- **Spill** — writing the buffer contents to a file on the map task's *local* disk (not HDFS) when it reaches the threshold (`mapreduce.map.sort.spill.percent`, default 0.80).
- **Combiner** — an optional mini-reducer run on map output to shrink it before it is written or sent; must be safe to apply zero, one or many times.
- **Merge factor** — the maximum number of sorted files merged in one pass (`mapreduce.task.io.sort.factor`, default 10).
- **Copy phase** — reducers fetching their partition from every map task over HTTP, as soon as each map finishes.
- **Spill file / segment** — a sorted, partitioned file produced by one spill; segments are merged into larger ones.
- **Skiplist / sorted structure** — in-memory ordered data structure used in other engines (e.g. memtables in LSM stores) for the same "keep it sorted as it arrives" job; MapReduce instead sorts the buffer in bulk at spill time.
- **Skew** — one key (or a few) carries far more records than others, so one reducer receives most of the shuffled data.

### 3. How it works, step by step

![Shuffle and sort in MapReduce](<../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0704.png>)

**Map side**

1. The `map` function emits `(key, value)` pairs. Each pair is serialized into the task's **circular in-memory buffer**; the task does *not* write straight to disk.
2. When the buffer is 80% full, a **background thread starts a spill**. The map keeps writing into the remaining 20%; if that fills before the spill ends, the map **blocks**.
3. Before writing, the spill thread **partitions** the records (one partition per reducer) and **sorts each partition by key** in memory.
4. If a **combiner** is configured, it runs on the sorted output, so less data is written and later transferred.
5. The sorted, partitioned data is written to a **new spill file** on local disk. A task with a lot of output produces several spill files.
6. When the map finishes, all spill files are **merged into one partitioned, sorted output file**, at most `io.sort.factor` (10) streams at a time.
7. If there are at least three spills (`mapreduce.map.combine.minspills`), the combiner runs once more during the merge. It does not run for one or two spills because the benefit would not pay for the work.
8. Map output may be **compressed** (setting `mapreduce.map.output.compress=true`, with a fast codec like LZ4/Snappy) — a cheap and common win because both disk and network get less data.
9. The partitions are made available to reducers over HTTP by the node's shuffle service.

**Reduce side**

10. A reduce task cannot start sorting until it has **every** map's partition, but it begins **copying as soon as each map completes** (the copy phase runs in parallel, with several fetcher threads — `mapreduce.reduce.shuffle.parallelcopies`, default 5).
11. Fetched data lands in the reducer's JVM memory if it is small enough (`mapreduce.reduce.shuffle.input.buffer.percent`, default 70% of heap); otherwise it is written to disk.
12. When the in-memory buffer reaches a threshold (`mapreduce.reduce.shuffle.merge.percent`, 66%) or a number of map outputs (`mapreduce.reduce.merge.inmem.threshold`, 1000), the data is **merged and spilled to disk**. The combiner runs here too if configured.
13. As copies accumulate on disk, a background thread merges them into larger sorted files to keep the final merge short.
14. After the last map output is copied, the **sort (merge) phase** begins: all segments are merged, preserving key order, in rounds of `io.sort.factor` files.
15. The **final merge round is fed directly into the reduce function** — it is never written back to disk. The reducer sees one sorted stream, calls `reduce(key, values)` once per key group, and writes results to HDFS (first replica goes to the local disk since the node manager also runs a datanode).

**Why the odd merge arithmetic?** The final round must merge exactly 10 streams straight into the reduce. With 40 segments, the naive plan (four rounds of 10) would write 40 segments' worth of data back to disk. Instead:

![Efficiently merging 40 file segments with a merge factor of 10](<../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0705.png>)

- Round 1 merges only **4** files into 1.
- Rounds 2–4 merge 10 files each.
- Remaining: 4 merged outputs + 6 untouched files = 10 for the final round that feeds the reduce.

The number of rounds is unchanged; only the amount of data rewritten to disk is minimized.

### 4. Worked example

Word count over 10 TB of text with 2,000 maps and 100 reducers.

```bash
hadoop jar wc.jar WordCount \
  -D mapreduce.job.reduces=100 \
  -D mapreduce.task.io.sort.mb=256 \
  -D mapreduce.map.output.compress=true \
  -D mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec \
  -D mapreduce.reduce.shuffle.parallelcopies=20 \
  /data/books /out/wc
```

What happens to one map task:

1. It reads a 128 MB split and emits `("the",1)`, `("data",1)`, … about 20M pairs, roughly 400 MB of serialized output.
2. With a 256 MB buffer and 0.80 threshold, spills start at ~205 MB: two spill files.
3. In each spill, pairs are bucketed into 100 partitions (by `hash("the") % 100`), sorted by key inside each bucket, and the combiner collapses `("the",1)×50,000` into `("the",50000)`. Output shrinks from 400 MB to maybe 30 MB.
4. The two spills are merged into one file with 100 sorted partitions.

Where to look in the job counters afterwards:

| Counter | What it tells you |
|---|---|
| `MAP_OUTPUT_RECORDS` vs `COMBINE_OUTPUT_RECORDS` | how much the combiner helped |
| `SPILLED_RECORDS` | records written to disk more than once; ideally ≈ map output records (one spill) |
| `REDUCE_SHUFFLE_BYTES` | actual bytes pulled over the network |
| `SHUFFLED_MAPS` / `MERGED_MAP_OUTPUTS` | count of segments fetched/merged |

If `SPILLED_RECORDS` is several times `MAP_OUTPUT_RECORDS`, the buffer is too small or the merge factor too low: records are being rewritten to disk repeatedly.

### 5. Under the hood

**Why sort at all?** Sorting groups equal keys adjacently so the reducer can stream through values with constant memory (no hash table of all keys). It also gives sorted output for free, and external merge sort is a well-understood algorithm that degrades gracefully when data exceeds memory.

**Sort inside the buffer.** The buffer holds serialized key/values plus an index of `(partition, key offset, value offset)` records. Sorting moves only the small index entries (quick-sort on partition then key bytes, using a raw comparator when possible), not the data — so it is cache-friendly and avoids deserializing keys. This is why `RawComparator` implementations on custom Writables matter for speed.

**Partitioner contract.** Records with the same key must produce the same partition number, or the grouping guarantee breaks. A custom `Partitioner` is used for total order sorting (range partitioning from a sampled key distribution, as in TeraSort), for secondary sort (partition on natural key only, sort on natural+secondary key, group on natural key), or to spread skew (salting a hot key).

**Combiner rules.** Because the framework may run it 0, 1 or many times, the function must be commutative and associative and have the same input and output types. Sum and max qualify; *mean* does not unless you carry `(sum,count)` pairs.

**Memory vs disk trade-off.** The best case is one spill per map and all map outputs fitting in reducer memory — the data then goes disk (once, map side) → network → reduce memory, with no extra disk passes. Every extra spill or on-disk merge is an extra full read/write of that data.

**Data skew.** All records of a key go to one reducer, so one slow reducer ("straggler") determines job completion. Fixes: combiners, key salting with a second aggregation, sampling-based partitioners, or (in Hive/Spark) skew-join hints.

**The same idea in Spark.** Spark's shuffle writes map-side files per task (sorted by partition id — the sort-based shuffle), then reducers fetch blocks. A **shuffle sort-merge join** shuffles both tables on the join key, sorts each partition, and merges. In the physical plan this appears as `Exchange hashpartitioning(key)` followed by `Sort` and `SortMergeJoin`. If both tables were saved **bucketed and sorted by the join key**, the plan has no `Exchange` and no `Sort` — the shuffle is skipped entirely. That is the Spark equivalent of removing the biggest cost in the MapReduce pipeline.

### 6. Performance and design implications

- **Shuffle volume = the cost driver.** Reduce it before it exists: filter and project early, use combiners, use map-side joins (broadcast the small table) instead of reduce-side joins.
- **Compress map output** — nearly always beneficial for network-bound jobs.
- **Avoid disk re-spills**: raise `io.sort.mb` so each map spills once, but stay under the task's container memory.
- **Right-size reducers.** Too few → huge partitions, long merges, no parallelism. Too many → tiny files and per-task startup overhead. A common target is a few GB (or ~1 container-minute of work) per reducer.
- **Check for skew** by comparing reducer input bytes: if the max is 10× the median, the plan is wrong, not the cluster.
- **Slow start**: `mapreduce.job.reduce.slowstart.completedmaps` (default 0.05) controls when reducers start copying; starting too early wastes slots, too late delays the copy phase.
- **Local disks matter**: spills and merges use local (non-HDFS) disks, so slow or few disks directly hurt the shuffle.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `mapreduce.task.io.sort.mb` | map output buffer size (default 100) | many spills; raise within container limits |
| `mapreduce.map.sort.spill.percent` | buffer fill level that triggers a spill (0.80) | rarely; lower if maps block |
| `mapreduce.task.io.sort.factor` | files merged per pass (10) | many spills or many map outputs |
| `mapreduce.map.combine.minspills` | min spills before combiner reruns at merge (3) | tuning combiner overhead |
| `mapreduce.map.output.compress` (+codec) | compress map output | almost always for large shuffles |
| `mapreduce.reduce.shuffle.parallelcopies` | fetcher threads per reducer (5) | many small map outputs |
| `mapreduce.reduce.shuffle.input.buffer.percent` | heap share for fetched data (0.70) | reducers spilling needlessly |
| `mapreduce.job.reduces` | number of reducers | sizing partitions |
| `job.setCombinerClass` / `setPartitionerClass` | plug in combiner / partitioner | aggregation / skew / total order |
| Spark `spark.sql.shuffle.partitions` | partitions after a shuffle (200) | tune post-shuffle parallelism |
| Spark `df.write.bucketBy(n, col).sortBy(col)` | pre-shuffle data for later joins | repeated joins on one key |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| A **combiner** when the reduce is associative/commutative aggregation | The operation is not (mean, distinct counts) — wrong answers |
| **Larger sort buffer** for fewer spills | Containers are memory-tight → OOM kills |
| **Map-output compression** | CPU is the bottleneck and data barely compresses |
| **More reducers** for parallelism | Output files become tiny; NameNode pressure |
| **Custom partitioner** to fix skew or get total order | Key distribution unknown — sample first |
| **Bucketed + sorted tables** (Spark/Hive) to skip the shuffle | Data is written once and joined once, or key changes often |

### 9. Common mistakes

- **Non-associative combiner** (e.g. averaging averages) → results differ depending on how many times it ran → emit sum and count, divide in the reducer.
- **Using one reducer for a global sort** → single-node bottleneck → use a sampled range partitioner (TeraSort-style).
- **Ignoring skew** → 99 reducers finish in a minute, one takes an hour → salt the hot key or handle it separately.
- **Huge `io.sort.mb` without raising container memory** → the container is killed by YARN → size both together.
- **Treating the shuffle as free** → joins and group-bys at the end of a long pipeline dominate runtime → push filters/projections before them.
- **Assuming reducer input is sorted by value too** → only keys are sorted; for sorted values use secondary sort.

### 10. 🎯 Interview kit

**30-second answer:** The shuffle is everything between a map's output and a reducer's input. Map output is buffered in memory, partitioned by reducer, sorted by key, optionally combined, and spilled to local disk; spills are merged into one file. Reducers then fetch their partition from every map, merge the sorted segments, and stream the result into the reduce function. It guarantees keys arrive grouped and sorted, and it is the main cost of a job.

**Structure a longer answer:** clarify (Hadoop MapReduce or Spark?) → map-side buffer/spill/merge → reduce-side copy/merge → where the combiner fits → what to tune → skew → relate to Spark `Exchange`.

**Likely follow-ups:**
- *Why does the map write to local disk, not HDFS?* → Intermediate data is temporary and recomputable; replicating it would be wasteful. On failure the map is simply rerun.
- *What is a combiner and what are its constraints?* → Map-side pre-aggregation; must be associative, commutative and type-preserving because the framework may run it any number of times.
- *How do you handle a skewed key?* → combiner, salting then a second aggregation stage, custom partitioner, or isolate the hot key.
- *Why does the merge not use 10-file rounds evenly?* → To minimize bytes written; the last round streams straight into reduce.
- *How is this different in Spark?* → Same concept (shuffle files, `Exchange`), plus bucketing can eliminate it for joins.

**Signals interviewers look for:** you can name the buffer/spill/merge/copy sequence, explain why intermediate data is local, connect combiner correctness to associativity, and talk about skew and shuffle volume as the thing to optimize.

### 11. Check yourself

- [ ] Draw the path of one record from `map()` to `reduce()`, labelling every disk and network hop.
- [ ] Explain why the combiner might not run at all, and what that implies for correctness.
- [ ] Given 40 spill segments and merge factor 10, state the merge schedule and why.
- [ ] Read a job's counters and decide whether to raise the sort buffer or add reducers.
- [ ] Explain how bucketing removes the `Exchange` in a Spark sort-merge join.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Shuffle and Sort
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Optimizing the shuffle sort merge join
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Task counters
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › Shuffle-and-replicate nested loop join (SNLJ)
- *Database Internals* — Alex Petrov — Log-Structured Storage › Skiplist
- *Hadoop: The Definitive Guide* — Tom White — Hive › Sorting and Aggregating
- *Hadoop: The Definitive Guide* — Tom White — HBase › Load distribution
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Several different sort orders
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Note
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Built-in Functions for Complex Data Types

