---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/Batch_Joins_and_Grouping"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Joins and Grouping in Batch Jobs

**L4 Advanced** · [↑ Batch Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In batch systems a join or group-by is solved by bringing all records with the same key to the same place — either in the reducer (sort-merge, any inputs) or in the mapper (broadcast, partitioned or merge joins, when the data allows).

### 1. The big picture

A join is needed wherever one record references another: a foreign key in a relational model, a document reference, a graph edge. A database answers a small query with index lookups. A batch job has no indexes: it reads all its input and resolves *every* association at once. So batch join algorithms are about moving data so that matching keys meet, with as little network, sorting and disk work as possible.

Two families exist. **Reduce-side joins** make no assumptions about the data but pay for sorting, copying and merging. **Map-side joins** skip the shuffle entirely but need the data to be small, or pre-partitioned and sorted. Grouping (`GROUP BY`, sessionisation) uses the same "bring related data together" mechanism. This knowledge carries straight into Spark and SQL engines, where the same strategies appear as sort-merge join, shuffle hash join and broadcast hash join.

### 2. Core concepts and vocabulary

- **Reduce-side join** — mappers tag/emit records by join key; the shuffle brings them together; the reducer does the join.
- **Sort-merge join** — reduce-side join in which sorted mapper outputs from both sides are merged at the reducer.
- **Secondary sort** — ordering the values within a key (e.g., user record first, then events by timestamp).
- **Map-side join** — join in the mapper; no reducers, no sort.
- **Broadcast hash join** — small input loaded entirely into an in-memory hash table in every mapper (Pig "replicated join", Hive "MapJoin", Spark broadcast).
- **Partitioned hash join** — both inputs partitioned identically (same key and hash function and partition count); each mapper loads one partition (Hive "bucketed map join").
- **Map-side merge join** — both inputs partitioned *and sorted* on the key; mappers merge them incrementally.
- **Shuffle hash join** — hash-partition both sides across the cluster, then hash-join locally.
- **Sessionization** — grouping events by a session/user key to reconstruct a user's sequence of actions.
- **Metastore / HCatalog** — metadata service recording how datasets are partitioned and sorted.

### 3. How it works, step by step

**Reduce-side sort-merge join** (activity events ⨝ user database on user ID)

1. One set of mappers reads events and emits `(user_id, event)`; another set reads users and emits `(user_id, date_of_birth)`.
2. The framework partitions by hash of `user_id` and sorts, so all records for one user arrive at the same reducer, adjacent.
3. With a secondary sort, the user record comes first, then events by timestamp.
4. The reducer is called once per user ID: it keeps the date of birth in a local variable, iterates events and emits `(viewed_url, viewer_age)`.
5. Only one user record is in memory at a time and no network requests are made.

**Broadcast hash join**

1. Each mapper starts by reading the small dataset from the filesystem into a hash table.
2. It scans its block of the large input and looks up each key.
3. Output has the same partitioning/order as the large input; there is no reduce phase.

**Choosing a strategy**

| Strategy | Assumptions | Cost | Output layout |
|---|---|---|---|
| Sort-merge (reduce-side) | None | Sort + shuffle + merge; multiple disk writes | Partitioned and sorted by join key |
| Broadcast hash | One side fits in memory | Small side copied to every mapper | Same as large input |
| Partitioned hash | Same partition count, key and hash | Each mapper loads one partition | Same as large input |
| Map-side merge | Same partitioning and sorted on key | Streaming merge, minimal memory | Same as input |

### 4. Worked example

Joining a large `players` dataset to a small `clubs` dataset in Spark: the broadcast hash join.

```scala
import org.apache.spark.sql.functions.broadcast
val joinedDF = playersDF.join(broadcast(clubsDF), playersDF("key1") === clubsDF("key2"))
joinedDF.explain("formatted")   // look for BroadcastHashJoin in the physical plan
```

By default Spark broadcasts when the small side is under `spark.sql.autoBroadcastJoinThreshold` (10 MB); with enough executor and driver memory you can raise it (up to ~100 MB) or force it with `broadcast()`. In Hive, the equivalent is `SET hive.auto.convert.join=true;` or a `MAPJOIN` hint. The plan should show no exchange (shuffle) for the large side.

Grouping/sessionisation in SQL:

```sql
SELECT user_id, COUNT(*) AS events, MIN(ts) AS session_start
FROM clicks
GROUP BY user_id;   -- mappers key by user_id; shuffle collates each user's events
```

### 5. Under the hood

**Mappers as message senders.** In a sort-merge join the key acts like a destination address: every pair with that key is delivered to a single reducer call. This separates the *physical* problem (moving data to the right machine, surviving crashes via task retry) from the *application logic* (the reducer). Contrast this with application code that queries a database mid-computation.

**Grouping is joining's twin.** Setting the mapper key to the grouping column makes partition + sort produce groups; the reducer then counts, sums or takes top-*k*. Sessionisation collates scattered events from many web servers' logs by user ID or session cookie, spreading different users across partitions.

**Why map-side joins are faster.** They avoid sorting, copying to reducers and merging reducer inputs, which can write data to disk several times. The price is assumptions:

- *Broadcast*: the small side fits in memory — or in a read-only on-disk index served from the OS page cache, which gives near-memory lookup speed without holding it all in RAM.
- *Partitioned*: both sides use the same number of partitions, same key and same hash function — natural if both were produced by earlier jobs doing the same grouping.
- *Merge*: additionally sorted on the key, so the mapper performs the reducer's merge incrementally without memory concerns.

**Physical layout matters.** To pick a join strategy you must know not only a dataset's format and directory but its number of partitions and the keys by which it is partitioned and sorted. In Hadoop this is metadata in the Hive metastore/HCatalog. Hadoop's `CompositeInputFormat` (`org.apache.hadoop.mapreduce.join`) runs map-side merges when inputs have the same number of reducers, keys and non-splittable files.

**Spark equivalents.** Broadcast hash join (no shuffle, fastest, memory-limited), shuffle hash join, and sort-merge join (default for large-large) — selected by size statistics and hints.

### 6. Performance and design implications

- **Prefer broadcast** when one side is small; verify in `EXPLAIN` that it actually happened.
- **Pre-partition and pre-sort** datasets that are joined repeatedly (bucketing in Hive/Spark) so later joins are map-side.
- **Join keys with skew** hurt reduce-side joins; see *Skew Handling*.
- **Filter and project before the join** to shrink shuffled data.
- **Know your output layout:** reduce-side output is partitioned and sorted by join key, which can feed a subsequent merge join; map-side output follows the large input.
- Avoid joining on high-cardinality strings when an integer surrogate key is available.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `broadcast(df)` / `/*+ BROADCAST(t) */` | Forces a broadcast hash join | Small dimension tables |
| `spark.sql.autoBroadcastJoinThreshold` | Max size for automatic broadcast (10 MB default) | Raise cautiously with enough memory |
| `df.explain("formatted")` | Shows the join algorithm chosen | Verifying plans |
| `hive.auto.convert.join` / `MAPJOIN` | Hive map-side join | Small tables in Hive |
| `CLUSTERED BY (key) INTO n BUCKETS` | Buckets/sorts a table | Enabling bucketed map joins |
| `CompositeInputFormat` | Map-side merge join in Hadoop MapReduce | Pre-partitioned, sorted inputs |
| `Pig USING 'replicated'` | Pig replicated (broadcast) join | Small right-hand relation |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Sort-merge (reduce-side) — inputs of any size/shape | Data is huge and skewed on the key; sorting cost dominates |
| Broadcast hash — one side comfortably in memory | The "small" side grows past executor memory (OOM) |
| Partitioned hash / bucketed — repeated joins on same key | Partition counts or hash functions differ between datasets |
| Map-side merge — both sides sorted and partitioned | Data layout can't be guaranteed |

### 9. Common mistakes

- Broadcasting a table that has grown → executor out-of-memory → monitor sizes; set thresholds; fall back to sort-merge.
- Assuming a join is map-side without checking the plan → a surprise full shuffle → read `EXPLAIN`.
- Mismatched bucket counts → no bucketed join → align `n` and the hash key.
- Joining on nullable keys → all nulls land on one reducer → filter or salt nulls.
- Re-sorting data that was already sorted by a previous job → wasted work → persist and reuse layout metadata.

### 10. 🎯 Interview kit

**30-second answer:** In batch, joins bring matching keys to the same place. A reduce-side sort-merge join emits records keyed by the join key from both sides, shuffles and sorts, then merges in the reducer; it works on any data but is expensive. Map-side joins avoid the shuffle: a broadcast hash join loads the small side into memory in every mapper; partitioned and merge joins exploit pre-partitioned (and sorted) inputs.

**Structure a longer answer:** state the problem (no indexes, full scans) → reduce-side algorithm → map-side variants and assumptions → how the engine picks → skew and layout.

**Likely follow-ups:**
- *Broadcast vs shuffle join?* → Broadcast moves only the small side; shuffle moves both and sorts or hashes.
- *What is a secondary sort for?* → Guarantees the order of values for a key (dimension first, then facts).
- *How is GROUP BY related?* → Same shuffle by key; aggregation happens in the reducer.
- *How do you know which join Spark chose?* → `explain()`.

**Signals interviewers look for:** you reason from data size, layout and skew; you can name each strategy's assumptions and its Hive/Spark equivalent.

### 11. Check yourself

- [ ] Walk through a reduce-side join of events and users, including the secondary sort.
- [ ] List the preconditions for a partitioned hash join and a map-side merge join.
- [ ] Explain why a reduce-side join needs no assumptions but a broadcast join does.
- [ ] Read an `explain()` output and spot `BroadcastHashJoin` vs `SortMergeJoin`.
- [ ] Show how sessionisation reduces to a group-by on a session key.
<!-- explainer:end -->

## In this section (study in order)

1. [Handling Skew in Batch Jobs](<Skew_Handling/_index.md>) (L5 Expert)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › Partitioned hash joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reduce-Side Joins and Grouping
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › Broadcast hash joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Broadcast Hash Join
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › Map-side merge joins
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Map-Side Joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — M
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › MapReduce workflows with map-side joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reduce-Side Joins and Grouping › Sort-merge joins

