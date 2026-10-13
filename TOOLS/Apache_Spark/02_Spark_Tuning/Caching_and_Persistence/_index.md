---
id: "TOOLS/Apache_Spark/02_Spark_Tuning/Caching_and_Persistence"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Caching and Persistence

**L4 Advanced** · [↑ Spark Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `cache()`/`persist()` keep a computed dataset's partitions in executor memory (and optionally disk) so later actions reuse them instead of recomputing the whole lineage — worth it only for data that is reused and affordable to hold.

### 1. The big picture
Spark is lazy, so two actions on the same DataFrame compute everything twice, from the source. MapReduce had the same problem permanently: every new computation reloaded input from disk. Spark's answer is a cross-cluster in-memory cache. It makes interactive exploration fast and is the natural fit for iterative algorithms (machine learning, graph processing) where each iteration reuses the previous result.

Caching is not free: it uses executor memory that execution (joins, sorts, shuffles) also needs, costs serialisation time, and can evict itself. Good tuning is knowing *what* to cache, *at which storage level*, *when it is materialised*, and *when to release it*.

### 2. Core concepts and vocabulary
- **Cache / persist** — synonyms in Spark: `cache()` is `persist()` with the default level; `persist(level)` gives you control.
- **Storage level** — where and how partitions are held: memory, disk, serialised or not, replicated or not.
- **Materialisation** — the first action after `cache()` that actually computes and stores partitions; `cache()` itself only sets a flag.
- **Block manager** — per-executor component that stores cached partitions as blocks (named like `rdd_4_0` = RDD 4, partition 0).
- **Eviction** — least-recently-used removal of cached blocks when storage memory is needed; evicted partitions are recomputed from lineage.
- **Unified memory** — execution and storage share one region and can borrow from each other; execution can evict cached blocks down to a protected storage fraction.
- **Checkpoint** — writes data to reliable storage and truncates lineage; different from cache.

### 3. How it works, step by step
1. You mark a dataset: `df.cache()` or `df.persist(StorageLevel.MEMORY_AND_DISK)`. No work happens.
2. The first **action** that touches every partition runs the lineage; as each task finishes a partition, the executor's block manager stores it at the chosen level.
3. Later jobs ask the block manager first: "Found block rdd_4_0 locally" in the logs means a cache hit; otherwise the partition is recomputed.
4. Partitions are cached **whole**. If only 4.5 of 8 partitions fit, 4 are cached and the rest are recomputed on every use (for `MEMORY_ONLY`).
5. When no longer needed, `df.unpersist()` frees space; otherwise LRU eviction handles pressure, and cached data disappears when the application ends.

Typical storage levels:

| Level | Memory | Disk | Serialised | Notes |
|---|---|---|---|---|
| `MEMORY_ONLY` | ✓ | | no (objects/Tungsten for DataFrames) | Fastest to read; recomputes what does not fit |
| `MEMORY_AND_DISK` | ✓ | spill | no | Safer; DataFrame `cache()` default |
| `MEMORY_ONLY_SER` / `MEMORY_AND_DISK_SER` | ✓ | | yes | Smaller footprint, CPU cost to read (JVM APIs) |
| `DISK_ONLY` | | ✓ | yes | Slow but avoids recompute of expensive lineages |
| `*_2` variants | | | | Replicate on two nodes; faster recovery |
| `OFF_HEAP` | off-heap | | yes | Avoids GC pressure |

### 4. Worked example
```python
from pyspark.sql import functions as F
from pyspark import StorageLevel

df = spark.range(10_000_000).withColumn("square", F.col("id") * F.col("id"))

df.cache()          # only marks the plan
df.count()          # ACTION: reads all partitions -> materialises cache (e.g. ~5 s)
df.count()          # served from memory (e.g. ~0.4 s) -> roughly an order of magnitude faster

df.take(1)          # WARNING pattern: if this is the first action only ONE partition gets cached

df.unpersist()      # release
df2 = df.persist(StorageLevel.MEMORY_AND_DISK)
```
Check it in the **Storage** tab of the Spark UI: you see the RDD/relation name, storage level, "Fraction Cached", size in memory and on disk, and the number of cached partitions. In the SQL tab the plan shows `InMemoryTableScan` where the cached data is read. The book's example shows a ~12× speed-up for a 10M-row DataFrame; the exact ratio depends on how expensive the lineage is.

The classic RDD form from the Hadoop book: `tuples.cache()` followed by two different `reduceByKey` jobs (max then min temperature). The first job logs `Added rdd_4_0 in memory`; the second logs `Found block rdd_4_0 locally` — no re-read of the input.

### 5. Under the hood
- DataFrame/Dataset `cache()` uses the in-memory **columnar** cache with compression (Tungsten-style batches), which is why it is more compact than caching Java objects. RDD `cache()` stores deserialised objects at `MEMORY_ONLY`.
- Caching is **keyed by the logical plan**: if another query has an identical plan fragment, Spark reuses the cache automatically (cache lookup via the cache manager). Changing the plan slightly (different projection) can bypass it.
- Caching is lazy and **per partition**; partial caches are normal and `take()` / `limit()` only fill what they touch.
- Cached data lives in the executor JVM storage memory fraction (`spark.memory.storageFraction` is the part protected from eviction by execution). Executor loss loses its blocks; they are recomputed (or fetched from a replica at `_2` levels).
- Caching differs from *checkpointing*: a checkpoint is written to reliable storage and cuts lineage (recovery from files), a cache is a volatile accelerator and keeps lineage.
- Related, outside Spark: buffer pools in databases use LRU/clock eviction and pinned pages; the same trade-offs (hot working set, cost of recompute, write-back) apply, and external caches like Redis or Memcached trade durability for speed.

### 6. Performance and design implications
- **Cache when**: a DataFrame is reused several times (iterative ML training, branching ETL, repeated interactive queries) and recomputing it is expensive (wide shuffle, expensive UDF, slow source).
- **Do not cache when**: it is used once, is cheap to recompute, or is larger than available memory (you will thrash and evict useful execution memory).
- Cache **after** filters/projections so you store the smallest useful data; cache before the fan-out point of the pipeline.
- Watch GC: large `MEMORY_ONLY` object caches on the heap lengthen GC pauses; serialised or off-heap levels reduce that.
- Measure: time before/after, Storage tab fraction cached, task "input" metrics showing memory vs. disk reads.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.cache()` | Persist at default level (DataFrame: memory+disk) | Quick reuse |
| `df.persist(StorageLevel.X)` | Choose level | Control memory vs disk vs serialisation |
| `df.unpersist(blocking=True)` | Release cached blocks | After last reuse |
| `spark.catalog.cacheTable("t")` / `CACHE TABLE t` / `uncacheTable` | Cache SQL tables/views | SQL-centric workflows |
| `CACHE LAZY TABLE t` | Cache on first use | Avoid eager full scan |
| `spark.catalog.clearCache()` | Drop everything | Notebook hygiene |
| `df.checkpoint()` / `localCheckpoint()` | Truncate lineage | Very long iterative lineages |
| `spark.memory.fraction`, `spark.memory.storageFraction` | Memory split | When caches evict too much or execution spills |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `MEMORY_AND_DISK` for expensive lineages that may not fit | Disk reads can be slower than recompute for cheap lineages |
| Serialised levels for big caches with GC pain | Extra CPU on each read; no benefit for already-compact DataFrame cache |
| `_2` replication when executor loss is costly | Doubles memory use |
| Checkpoint for 100s of iterations or streaming state | Writes to storage; cache alone will not shorten lineage |
| Table-level `CACHE TABLE` for shared BI queries | Stale data if the source changes; remember to refresh |

### 9. Common mistakes
- Calling `cache()` and assuming it is done → nothing computed yet, first query still slow → trigger with `count()`.
- Materialising with `take(1)`/`show()` → only one partition cached → use a full-scan action.
- Caching everything "just in case" → memory pressure, eviction churn, slower jobs → cache only reused, expensive data.
- Never unpersisting in long-running apps or notebooks → stale hogs of memory → `unpersist()`.
- Caching a DataFrame then changing it (new `withColumn`) and expecting the cache to apply → new plan, new computation → cache the final reused form.
- Expecting the cache to survive executor loss or a new application → recomputed/lost → use persistent storage (Parquet/Delta) for durable intermediates.

### 10. 🎯 Interview kit
**30-second answer:** `cache()`/`persist()` mark a DataFrame to be stored in executor memory (optionally disk) the first time an action computes it, so later actions skip recomputation. Use it for reused, expensive data that fits; release with `unpersist()`; remember it is lazy and partition-granular.
**Structure a longer answer:** clarify the access pattern → laziness and materialisation → storage levels and the memory/CPU/disk trade-off → when not to cache → verify in the UI → alternative: checkpoint or write to Parquet.
**Likely follow-ups:**
- *Difference between cache and persist?* → cache = persist with default level.
- *Cache vs checkpoint?* → cache keeps lineage and is volatile; checkpoint writes reliably and cuts lineage.
- *Why only one partition cached after `take(1)`?* → Spark computes only what the action needs.
- *What happens if cache does not fit?* → whole partitions are dropped/spilled; the rest are recomputed or read from disk.
- *Where do you see it?* → Storage tab, `InMemoryTableScan`.
**Signals interviewers look for:** understanding laziness, measuring rather than guessing, awareness of memory contention with execution, and knowing alternatives.

### 11. Check yourself
- [ ] Explain why the first `count()` after `cache()` is slow and the second fast.
- [ ] Pick a storage level for a 200 GB dataset on a cluster with 100 GB of cache memory and justify it.
- [ ] Show where cache hits appear in logs, Storage tab and the physical plan.
- [ ] Describe what happens to a cached DataFrame when one executor dies.
- [ ] Decide whether to cache, checkpoint or write to Parquet for a 500-iteration ML loop.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › When to Cache and Persist
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Caching and Persistence of Data
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Caching
- *Hadoop: The Definitive Guide* — Tom White — Spark › Persistence
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Example: Memcached and lightweight object caching
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Caching Semantics
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Cache Eviction
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — B › C
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Buffer Management
- *Hadoop: The Definitive Guide* — Tom White — Spark › Note

