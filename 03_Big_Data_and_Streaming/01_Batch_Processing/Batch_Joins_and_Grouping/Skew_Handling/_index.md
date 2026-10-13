---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/Batch_Joins_and_Grouping/Skew_Handling"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Handling Skew in Batch Jobs

**L5 Expert** · [↑ Joins and Grouping in Batch Jobs](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Skew is when a few "hot" keys carry a disproportionate share of the data so one task becomes a straggler; you fix it by spreading the hot keys over several tasks (salting, sampling-based skewed joins, map-side handling for hot keys, two-stage aggregation).

### 1. The big picture

The batch recipe — "bring every record with the same key to the same place" — assumes keys have roughly similar volume. Real data is not like that: a celebrity with millions of followers, a default value like `NULL` or `-1`, a bot IP, one very large customer. All records for such a key land on a single reducer or partition, creating a **hot spot**. Because a stage finishes only when its slowest task does — and downstream jobs wait for it — one overloaded task can dominate the runtime of a whole cluster-sized job.

Hashing the key does not solve this: identical keys hash identically. Most data systems cannot compensate automatically; the application (or the query author) must reduce skew. This lesson covers the standard techniques in batch joins and group-bys and the same randomisation idea used to relieve hot spots in partitioned databases.

### 2. Core concepts and vocabulary

- **Skew / hot spot** — one partition or reducer processing far more data than others.
- **Hot key / linchpin object** — a key with disproportionately many associated records.
- **Straggler** — a task that finishes long after its siblings; determines stage time.
- **Salting** — appending a random suffix (or prefix) to a hot key so its records spread across partitions.
- **Skewed join (Pig)** — sampling job finds hot keys; hot-key records go to a random one of several reducers; the other input's matching records are replicated to all of them.
- **Sharded join (Crunch)** — like skewed join but hot keys are given explicitly.
- **Hive skewed join** — hot keys declared in table metadata, stored in separate files and joined map-side.
- **Two-stage aggregation** — partial aggregate by (key, random salt), then combine partials per key.
- **Replication (of the small side)** — copying the other join input to every task that handles a split hot key.

### 3. How it works, step by step

**Diagnosing**

1. Look at per-task metrics: input size, records and duration of each reducer/partition. One or a few tasks much larger than the median indicate skew.
2. Identify the key: sample or `GROUP BY key ORDER BY COUNT(*) DESC LIMIT 20`.

**Skewed join (sample + randomise)**

1. A sampling job finds the hot keys.
2. For hot keys, mappers send each record from the large input to a *randomly chosen* one of N reducers instead of a hash-determined one.
3. For the other input, records matching a hot key are *replicated to all N* reducers that handle that key.
4. Each reducer joins its slice; the hot key's work is parallelised at the cost of duplicating the smaller side's hot records.

**Hot keys in aggregation**

1. Stage 1: key by `(key, random 0..N-1)`; each reducer computes a partial aggregate (count, sum, min, max).
2. Stage 2: key by the original key; combine the N partials into the final value.

**Alternative: isolate hot keys**

1. Split the dataset into hot-key records and the rest.
2. Join the hot subset with a map-side (broadcast) join, since few keys means a small other side.
3. Join the rest normally and union the results.

| Technique | Needs to know hot keys? | Cost | Works for |
|---|---|---|---|
| Salting both sides | Yes (or all keys) | Replicates small side × N | Joins |
| Pig skewed join | Auto (sampling job) | Extra sampling job; replication | Joins |
| Crunch sharded join | Explicit list | Replication | Joins |
| Hive skewed join | Explicit in metadata | Separate files; map join for hot keys | Joins |
| Two-stage aggregation | Optional | Extra stage | Group-by |
| Combiner / map-side pre-agg | No | Little | Algebraic aggregates |

### 4. Worked example

Counting events per customer where one customer, `acme`, has 80% of the records.

```sql
-- Stage 1: spread each key over 16 salts
WITH partial AS (
  SELECT customer_id,
         CAST(FLOOR(RAND(42) * 16) AS INT) AS salt,      -- fixed seed => deterministic on retry
         COUNT(*) AS c
  FROM events
  GROUP BY customer_id, CAST(FLOOR(RAND(42) * 16) AS INT)
)
-- Stage 2: combine partial counts
SELECT customer_id, SUM(c) AS events
FROM partial
GROUP BY customer_id;
```

For a skewed join in Spark SQL, you can salt the large table and explode the small one:

```python
from pyspark.sql import functions as F
N = 16
big   = big.withColumn("salt", (F.rand(42) * N).cast("int"))
small = small.withColumn("salt", F.explode(F.array([F.lit(i) for i in range(N)])))
joined = big.join(small, ["customer_id", "salt"])
```

The hot key's work now spreads over 16 tasks. The price: the small table is 16× larger in the shuffle and the query is more complex. Check the effect in the stage task-duration distribution: max task time should drop close to the median. (Modern Spark also offers adaptive query execution skew-join handling that splits oversized partitions automatically; verify in your version's plan.)

### 5. Under the hood

**Why hashing cannot fix it.** Hash partitioning spreads *different* keys evenly; it cannot split *one* key. In the extreme, all requests for one key hit one partition. The same phenomenon appears in partitioned databases: a celebrity's user ID or a trending post receives a storm of writes. The remedy there is the same: add a random number (e.g., two decimal digits → 100 sub-keys) to hot keys. Reads then must query all 100 sub-keys and combine them, and you need bookkeeping on which keys are split — splitting every key would be pointless overhead.

**Cost model of the skewed join.** The hot key's large-side records are partitioned randomly (balanced), but the other side's records for that key must be copied to every participating reducer. Total network volume grows by roughly (N−1) × hot-key records of the small side. That is fine when the hot key is hot in only one input.

**Hive's alternative** avoids shuffle for hot keys entirely: records for declared hot keys are written to separate files and handled with a map-side join, since the hot subset on the other side is small.

**Straggler amplification.** A MapReduce job completes only when all tasks do, and the next job waits. With 1,000 reducers and one slow one, 999 slots idle. Speculative execution doesn't help skew: a duplicate of the same task has the same amount of data.

**Determinism.** Random salts must be reproducible on retry; use a fixed seed or derive the salt from other record content (e.g., hash of a row ID) so a re-executed task assigns the same keys.

### 6. Performance and design implications

- Detect skew early: log per-task input bytes and records, and alert when max/median > 5×.
- Treat NULLs and default values specially — filter them out before the join or handle them separately; they are the most common hidden hot key.
- Pre-aggregate before joining where possible (reduce the big side).
- Prefer broadcast of the small side for hot keys; avoid shuffling the hot subset at all.
- Choose the salt factor N to balance parallelism against replication cost (e.g., N = 8–32, or sized so a hot key's share is near the median).
- Re-evaluate: skew changes over time (new big customers, viral events), so static hot-key lists need maintenance.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Pig `JOIN a BY k, b BY k USING 'skewed';` | Sampling-based skewed join | Unknown hot keys in Pig |
| Hive `SET hive.optimize.skewjoin=true;` / `SKEWED BY (...) ON (...)` | Skew join optimisation / declared skewed values | Hot keys in Hive |
| Spark `spark.sql.adaptive.enabled` / `skewJoin.enabled` | Adaptive splitting of skewed partitions | Spark 3+ SQL joins |
| `F.rand(seed)`, salt column | Manual salting | Custom joins and aggregations |
| `df.groupBy(k).count().orderBy(desc("count"))` | Finds hot keys | Diagnosis |
| Spark UI stage task metrics / YARN counters | Shows per-task duration and size | Detecting stragglers |
| `mapreduce.job.reduces` | Number of reducers | Increasing parallelism (doesn't split a single key) |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Two-stage aggregation for algebraic aggregates on hot keys | Aggregate is not decomposable (exact median, distinct count) |
| Salting + replication for joins with one hot side | Both sides are hot on the same key (replication explodes) |
| Isolate hot keys + broadcast | The hot-key set is large or unstable |
| Adaptive engine features | Older engine versions or non-SQL APIs |
| Filter NULL/default values | Business logic needs those rows (handle separately, then union) |

### 9. Common mistakes

- Increasing the number of reducers to fix skew → the hot key still goes to one reducer → salt or isolate the key.
- Salting non-deterministically (`rand()` without seed) → inconsistent results on task retry → use a fixed seed or a content-derived salt.
- Salting every key → needless replication → salt only hot keys (join against a small hot-key list).
- Ignoring NULL keys → millions of rows shuffled to one task → filter or branch.
- Trusting averages → missing the straggler → check max and percentile task times.

### 10. 🎯 Interview kit

**30-second answer:** Skew means a few hot keys concentrate data on a single task, making that straggler determine job time. Hashing can't split one key, so you spread the hot key manually: salt it and replicate the other side, use a sampling-based skewed join, handle hot keys with a map-side/broadcast join, or aggregate in two stages (partial by salted key, then final by key).

**Structure a longer answer:** symptom (one slow task) → diagnose key → choose technique by operation (join vs aggregate) → state cost (replication, extra stage) → monitor.

**Likely follow-ups:**
- *Why not just add reducers?* → One key still maps to one reducer.
- *How does two-stage aggregation work?* → Salt for partials, then re-aggregate on the true key; requires decomposable aggregates.
- *Where else does hot-key skew appear?* → Partitioned databases and streaming keyed state: same salting fix, with read fan-out.
- *What does Spark AQE do?* → Detects oversized shuffle partitions at runtime and splits them.

**Signals interviewers look for:** you diagnose from task metrics, understand the replication trade-off, and watch determinism of the random salt.

### 11. Check yourself

- [ ] Explain why hash partitioning can't resolve a single hot key.
- [ ] Describe the Pig skewed join algorithm, including what is replicated.
- [ ] Rewrite a `COUNT(DISTINCT)`-free group-by as two-stage aggregation with salt 16.
- [ ] State two risks of non-deterministic salting.
- [ ] Explain how the same idea applies to a key-value store with a celebrity key (and the read-side cost).
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Reduce-Side Joins and Grouping › Handling skew
- *Designing Data-Intensive Applications* — Martin Kleppmann — Partitioning › Skewed Workloads and Relieving Hot Spots
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — G › H

