---
type: "practice"
difficulty: "Deep"
---

# You need predictable read latency but your LSM's compaction causes periodic p99 jumps. Would you switch to a B-Tree engine, tune compaction, or add a caching layer? Reason through the trade-offs and how you'd measure success.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of unpredictable read latency due to periodic p99 jumps caused by LSM compaction, we need to evaluate three potential strategies: switching to a B-Tree engine, tuning compaction parameters, or adding a caching layer. Each approach has its trade-offs and requirements for measurement.

Firstly, let's break down why the current situation with LSM is problematic. An LSM (Log-Structured Merge) database writes data in sequential, immutable SSTables, which are periodically merged through compaction to reduce the number of files and improve read performance. However, this process can cause significant latency spikes when compactions occur.

#### Triaging the Question

The interviewer is probing your understanding of database internals, specifically B-Trees vs LSMs, and how these databases handle data durability and performance. They are also interested in your ability to diagnose problems and make informed decisions based on trade-offs. We need to consider:

Current Latency: The current p99 latency is unacceptable.
Compaction Impact: Compactions cause periodic spikes that affect read performance.
Alternatives: Evaluate B-Trees, compaction tuning, and caching layers.

#### First-Principles Solution
1. Switch to a B-Tree Engine

B-Trees are inherently balanced trees that maintain sorted data in memory and on disk. This means they provide predictable read latency because every access path is roughly the same length, reducing variance. However, this comes at the cost of higher write amplification due to frequent splits and rebalancing.

#### Trade-offs

Read Performance: Predictable; no compaction-induced spikes.
Write Performance: Higher overhead due to constant tree balancing.
Memory Usage: Requires more memory for maintaining balanced structures.
2. Tune Compaction Parameters

We can try adjusting the compaction parameters to mitigate latency spikes without changing the database engine.

#### Trade-offs

Compaction Frequency: Lower frequency reduces spikes but may increase storage usage and write amplification.
Compaction Size: Smaller compactions reduce spike size but increase overhead.
Write Buffer Size: Larger buffers can absorb more writes before starting compaction, smoothing out spikes.
3. Add a Caching Layer

A caching layer can significantly reduce the impact of compactions on read latency by keeping frequently accessed data in memory.

#### Trade-offs

Cache Hit Rate: High hit rate improves performance but requires tuning and maintenance.
Eviction Strategy: Effective eviction strategies are critical to avoid cache thrashing.
Latency Trade-offs: Cache lookup can introduce additional latency if it fails or hits the disk.
Measuring Success

To determine which approach is best, we need clear metrics:

P99 Latency: Measure before and after changes.
Write Amplification: Monitor to ensure no significant degradation in write performance.
Storage Usage: Track total storage consumption to avoid uncontrolled growth.
Hit Rate: For caching, measure cache hit rates and eviction efficiency.

#### Closing Summary

Switching to B-Trees or tuning compaction are viable options if read predictability is critical. However, both come with trade-offs in write performance and storage overhead. Adding a caching layer can provide immediate benefits but requires careful design and monitoring.

In practice, we might start by trying to tune the current LSM system before considering more drastic changes like switching engines or implementing complex caching layers. The key will be continuous monitoring and adjustment based on real-world metrics.

#### Common Mistakes/Follow-Ups
Overlooking Write Amplification: In B-Trees, frequent splits can cause higher write amplification.
Ignoring Cache Efficiency: Caching without an effective eviction strategy can lead to cache thrashing.
Not Measuring Performance Holistically: Only focusing on read latency while ignoring other metrics like storage usage and write performance.

We would need to measure these aspects thoroughly before making a decision, ensuring the chosen approach provides the best overall balance of read and write performance.

