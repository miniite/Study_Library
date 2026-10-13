---
type: "practice"
difficulty: "Deep"
---

# A team benchmarked RocksDB at 3x their InnoDB write throughput and wants to migrate immediately. As the reviewing architect, list the ways a synthetic benchmark misleads at production scale (compaction debt, cache warmup, real key distribution) and what you'd require before approving.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether to migrate from InnoDB to RocksDB, it's crucial to understand that synthetic benchmarks can often mislead when applied directly to production workloads. This is particularly true with databases like RocksDB, which uses a Log-Structured Merge (LSM) tree for storage and compaction processes that aren't as straightforward as traditional B-trees used in InnoDB.

#### Triaging the Question

The team has benchmarked RocksDB at 3x their current write throughput using synthetic tests. As an architect reviewing this migration, I need to ensure that these benchmarks accurately reflect real-world performance. I'll focus on explaining how synthetic benchmarks can be misleading and what additional requirements are needed before approving a full migration.

Core Concepts
Benchmarks: Synthetic vs. Real Workloads
Compaction Debt: Accumulation of small SST files over time
Cache Warmup: Initial data load performance
Real Key Distribution: Skewed or uniform distribution impact

#### First Principles Solution

Compaction Debt:

Explanation: RocksDB compacts its data to maintain performance, but during periods of high write activity, small SST files (compaction units) build up. This "debt" isn't visible in synthetic benchmarks which typically use a steady-state setup.
Impact on Production: In production, this debt will eventually need to be addressed with compaction runs, which can cause performance drops or even pauses if not managed properly.

Cache Warmup:

Explanation: Synthetic tests often run in memory where cache is fully warm from the start. This doesn't account for initial write amplification and performance hit during data loading.
Impact on Production: Real-world applications might experience significant performance hits as they first populate the database, only to stabilize later.

Real Key Distribution:

Explanation: Synthetic benchmarks often use a uniform or simple key distribution, which can obscure issues related to skewed workloads or specific access patterns.
Impact on Production: Workloads with highly skewed keys (e.g., popular news articles) could perform poorly in RocksDB due to its write-heavy nature compared to InnoDB.

#### Closing Summary
and Common Mistakes/Follow-Ups

To ensure a successful migration, the team must address these potential issues:

Conduct detailed production simulations that include compaction debt handling.
Perform cache warmup tests under realistic workloads.
Test with realistic key distributions from their actual application.

#### Common mistakes
include relying solely on synthetic benchmarks without considering real-world complexities and not adequately planning for compaction and write amplification. Follow-ups might involve setting up a test environment to simulate the production workload more accurately, or using tools like perfetto for detailed performance profiling during the migration process.

Triaging Common Mistakes
Misleading Metrics: Relying too heavily on synthetic benchmarks.
Ignoring Compaction Debt: Not understanding how compaction debt affects long-term performance.
Overlooking Cache Warmup: Failing to account for initial write amplification and its impact on performance.
Skewed Data Assumptions: Assuming uniform key distribution when real workloads are highly skewed.

**Comparison:**

```
Workload Type	Write Throughput (ops/sec)
Uniform	100
Skewed	80
Real World vs Synthetic Benchmark
Real World
Synthetic
```

**Code:**

```python
# Example of a basic compaction simulation function (not executable)
def simulate_compaction(workload, db_type='rocksdb'):
    if db_type == 'rocksdb':
        # Simulate compaction process with RocksDB behavior
        pass
    else:
        # Handle InnoDB or other database types
        pass
```

**Example input / output:**

```text
Input:
- Simulate RocksDB compaction under a uniform write workload.
Output:
- Compaction frequency and impact on write throughput.

Request for input-output data of real-world workloads to validate benchmarks.
```

**Data:**

```json
{
  "workload": {
    "type": "uniform",
    "keys_per_second": 100,
    "writes_per_key": 5
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the workload uniform?
7. Simulate with uniform keys
8. Simulate with skewed keys?
9. Simulate with skewed keys
10. Simulate with uniform keys
11. Analyze results for compactionimpact

