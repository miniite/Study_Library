---
id: "TOOLS/Apache_Spark/02_Spark_Tuning/Spark_Joins"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Spark Join Strategies

**L4 Advanced** · [↑ Spark Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark picks among five join strategies; the big wins come from broadcasting the small side to avoid a shuffle, bucketing/co-partitioning to avoid exchanges for repeated joins, and letting AQE or hints correct bad plan choices.

### 1. The big picture
Joins are the most expensive common operation in distributed analytics because matching rows must be on the same machine, so data is moved across the network (the *shuffle*) and often sorted. Everything in join tuning is a way to move less data or sort less data. Spark has five strategies: **broadcast hash join (BHJ)**, **shuffle hash join (SHJ)**, **shuffle sort-merge join (SMJ)**, **broadcast nested loop join (BNLJ)** and the **Cartesian product join**. BHJ and SMJ cover most real workloads; the others are fallbacks for non-equi conditions or special cases.

The same ideas appear in data-warehouse design: denormalise or broadcast small dimensions, and co-partition large facts on the join key.

### 2. Core concepts and vocabulary
- **Shuffle** — redistribution of rows by key across executors; map side writes files, reduce side fetches them.
- **Exchange** — the physical-plan node for a shuffle (or broadcast).
- **Broadcast** — send a complete copy of a small table to every executor.
- **Build side / probe side** — in hash joins, the smaller input is hashed (build) and the other streamed against it (probe).
- **Equi-join** — join on equality of keys; required for hash and sort-merge strategies.
- **Bucketing** — pre-hashing and pre-sorting data into a fixed number of files by key when writing a table.
- **Skew** — a few join keys with disproportionate rows, giving straggler tasks.
- **AQE** — Adaptive Query Execution: re-plans at runtime from actual stage statistics.
- **Join hint** — `BROADCAST`, `MERGE`, `SHUFFLE_HASH`, `SHUFFLE_REPLICATE_NL`.

### 3. How it works, step by step
**Broadcast hash join**
1. The driver (or executors) collects the small side and broadcasts it to every executor.
2. Each executor builds an in-memory hash table.
3. Each partition of the large side streams through and probes the table. No shuffle of the large side.

**Shuffle sort-merge join**
1. Both inputs are hash-partitioned by join key (`Exchange hashpartitioning`).
2. Each partition is sorted by key.
3. A merge pass walks both sorted streams and emits matching rows. Can spill to disk, so it scales to inputs larger than memory.

**Shuffle hash join** — both sides are shuffled, but the smaller partition is hashed instead of sorted; faster than SMJ when partitions fit in memory, risky otherwise.

**Strategy selection (equi-join, simplified):** hint? → use it; else if one side's estimated size is below `spark.sql.autoBroadcastJoinThreshold` (10 MB default) → BHJ; else if SMJ is preferred (`spark.sql.join.preferSortMergeJoin=true`, default) → SMJ; non-equi join → BNLJ, or Cartesian if no condition and not broadcastable.

| Strategy | Needs equi-join | Shuffle | Memory risk | Use when |
|---|---|---|---|---|
| BHJ | yes (BNLJ for non-equi) | none for big side | small side must fit on each executor and driver | One side small |
| SMJ | yes | both sides + sort | low (spills) | Two large tables |
| SHJ | yes | both sides | partition must fit memory | Medium vs large, no sort wanted |
| BNLJ / Cartesian | no | broadcast / replicate | very high cost | Non-equi or cross joins, unavoidable |

### 4. Worked example
```python
from pyspark.sql import functions as F

orders = spark.table("orders")        # large fact
users  = spark.table("users")         # medium/small dimension

# 1) Default: look at what Spark chose
orders.join(users, orders.users_id == users.uid).explain()
#   SortMergeJoin [users_id], [uid]   with Exchange hashpartitioning on both sides

# 2) Broadcast the small side
orders.join(F.broadcast(users), orders.users_id == users.uid).explain()
#   BroadcastHashJoin ... BuildRight   + BroadcastExchange, no Exchange on orders

# 3) SQL hint
spark.sql("SELECT /*+ BROADCAST(u) */ * FROM orders o JOIN users u ON o.users_id = u.uid")
```
```scala
// 4) Bucket both sides on the join key to remove the Exchange for repeated joins
usersDF.orderBy("uid").write.format("parquet").bucketBy(8, "uid").mode("overwrite").saveAsTable("UsersTbl")
ordersDF.orderBy("users_id").write.format("parquet").bucketBy(8, "users_id").mode("overwrite").saveAsTable("OrdersTbl")
spark.table("OrdersTbl").join(spark.table("UsersTbl"), $"users_id" === $"uid").explain()
// SortMergeJoin with no Exchange and no Sort: bucketing paid for the shuffle once, at write time
```
Reading the result: in the first plan both branches of the `SortMergeJoin` have `Exchange hashpartitioning(key, 200)` then `Sort`; the Spark UI SQL tab shows shuffle write/read sizes on those edges. In the broadcast plan the large side's scan flows straight into the join. In the bucketed plan the `Exchange` disappears because both tables are already hash-partitioned and sorted with the same bucket count.

### 5. Under the hood
- **BHJ memory path:** the small table is collected to the driver, serialised, and broadcast (torrent-like BitTorrent distribution) to executors, where it is held as a hash relation. Driver memory and `spark.sql.autoBroadcastJoinThreshold`/`spark.driver.maxResultSize` therefore limit it. Broadcast timeouts (`spark.sql.broadcastTimeout`, 300 s) kill jobs when the build side is slow to compute.
- **Sort-merge** relies on an external sorter that spills sorted runs and merges them; the merge phase is streaming and cache-friendly, and with whole-stage codegen the sort and merge are fused into generated code.
- **Bucketing** metadata lives in the metastore; the planner recognises matching `bucketSpec`s (same key, same bucket count; Spark 3 can also reconcile multiples) and skips the exchange. Bucketed tables must be written via `saveAsTable`.
- **AQE (Spark 3.x):** after the map stage completes, real shuffle sizes are known; AQE can (1) coalesce small shuffle partitions, (2) convert a planned SMJ into a BHJ when a side turns out small, (3) split skewed partitions in SMJ (`spark.sql.adaptive.skewJoin.enabled`) and replicate the other side's matching partition. Enable with `spark.sql.adaptive.enabled` (default true from 3.2).
- **Skew handling by hand:** salt the hot key (add a random suffix, replicate the small side across suffixes), isolate hot keys into a separate broadcast join, or filter out null keys that all land in one partition.
- **Join type matters:** broadcast can be used for the non-preserved side only (e.g. the right side of a left outer join), so a full outer join cannot be broadcast.

### 6. Performance and design implications
- A shuffle costs disk writes, network transfer and sorting: reducing shuffled bytes is the top lever (filter and project before joining, use columnar formats with pushdown).
- Broadcast when one side is a few hundred MB at most and fits comfortably in every executor and the driver; raising the threshold blindly causes OOM.
- Bucket tables that are joined or aggregated on the same key repeatedly (large fact–fact joins in a warehouse); remember bucketing helps joins on exactly the bucket keys.
- Statistics (`ANALYZE TABLE`) improve size estimates; unknown sizes default to "large", so a small filtered table might not be broadcast until AQE intervenes.
- Diagnose with: `explain`, Spark UI SQL tab (strategy, rows, shuffle sizes), stage task-duration distribution (skew), spill metrics.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `broadcast(df)` / `/*+ BROADCAST(t) */` | Force broadcast of one side | Known-small dimension |
| `/*+ MERGE(t) */`, `SHUFFLE_HASH`, `SHUFFLE_REPLICATE_NL` | Force other strategies | Planner chose badly |
| `spark.sql.autoBroadcastJoinThreshold` | Max size for auto-broadcast (-1 disables) | Raise carefully / disable to force SMJ |
| `spark.sql.join.preferSortMergeJoin` | Prefer SMJ over SHJ | Rarely changed |
| `spark.sql.adaptive.enabled`, `skewJoin.enabled`, `localShuffleReader.enabled` | Runtime re-planning | Skewed or misestimated joins |
| `bucketBy(n, col)` + `sortBy` + `saveAsTable` | Pre-partition tables | Repeated joins on same key |
| `ANALYZE TABLE … COMPUTE STATISTICS` | Table/column stats | Better strategy choice |
| `spark.sql.broadcastTimeout` | Wait for broadcast build | Slow build side |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| BHJ: small dimension or heavily filtered side | Build side near memory limits; many concurrent queries each broadcasting |
| SMJ: two large inputs, equi-join | Skewed keys → straggler task and spill |
| SHJ: partitions fit memory and sort is wasteful | Large partitions → OOM (no spill build side) |
| Bucketing: stable join keys reused many times | Over-bucketing small tables, key changes, extra write cost |
| Salting: severe, known skew | Added complexity; try AQE skew handling first |

### 9. Common mistakes
- Raising the broadcast threshold to GBs → executor/driver OOM → keep it moderate; use hints for specific cases.
- Joining on a column with many nulls or a default value → one giant partition → filter or handle nulls separately.
- Joining then filtering → shuffles unnecessary rows → filter/project before joins (Catalyst helps but not through UDFs).
- Writing bucketed tables with `.save()` → bucket info lost → use `saveAsTable`.
- Cartesian joins by accident (missing condition) → explosion → check plan for `CartesianProduct`.
- Ignoring data type mismatches on keys (int vs string) → implicit casts, no pushdown, wrong matches → align types.

### 10. 🎯 Interview kit
**30-second answer:** Spark joins by shuffling both sides by key and sort-merging, unless one side is small enough to broadcast, in which case a broadcast hash join avoids shuffling the big side. Tune by broadcasting small tables, bucketing repeated join keys, collecting statistics, and enabling AQE to switch strategies and split skewed partitions at runtime.
**Structure a longer answer:** clarify sizes and key distribution → name the strategies and how each moves data → choose using size/equi/skew → tools: hints, bucketing, AQE, salting → verify with `explain` and UI.
**Likely follow-ups:**
- *When does Spark choose broadcast automatically?* → estimated size below threshold, no conflicting hint.
- *SMJ vs SHJ?* → sort vs hash per partition; SMJ spills safely.
- *How do you fix a skewed join?* → AQE skew join, salting, isolate hot keys, broadcast.
- *What does bucketing buy you?* → removes exchange and sort for joins on bucket keys.
- *Why a non-equi join is slow?* → falls back to nested loop; no hashing or merge.
**Signals interviewers look for:** you reason in terms of bytes moved, read plans, know AQE capabilities, and have practical skew remedies.

### 11. Check yourself
- [ ] Given two table sizes and a threshold, predict the strategy and confirm with `explain`.
- [ ] Explain build side/probe side in a hash join.
- [ ] Show a plan where bucketing removed an `Exchange`.
- [ ] Describe three AQE behaviours and the config that controls each.
- [ ] Salt a skewed key and explain the cost to the small side.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › When to use a broadcast hash join
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › When to use a shuffle sort merge join
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › Broadcast hash join (BHJ)
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Optimizing the shuffle sort merge join
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Broadcast join
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › A Family of Spark Joins
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › SQL Join Hints
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 12. Epilogue: Apache Spark 3.0 › Adaptive Query Execution
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Optimize your JOIN strategy and schema
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — B

