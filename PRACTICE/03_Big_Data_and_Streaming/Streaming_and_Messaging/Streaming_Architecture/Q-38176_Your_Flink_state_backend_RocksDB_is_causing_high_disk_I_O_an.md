---
type: "practice"
difficulty: "Expert"
---

# Your Flink state backend (RocksDB) is causing high disk I/O and slow checkpoints for a large-cardinality anomaly-baseline job. Walk me through tuning options and when you'd shard the job instead.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Graceful Degradation in Streaming](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we have a real-time streaming application using Apache Flink with RocksDB as the state backend that is experiencing high disk I/O and slow checkpointing issues due to large cardinality anomaly detection jobs. To address these problems, we need to optimize our setup through various tuning options before considering job sharding. Let's walk through each of these steps in detail.

Firstly, it’s important to understand the core issue: RocksDB is a persistent key-value store that supports efficient state management for Flink applications. However, its performance can degrade with large cardinality states due to the overhead of maintaining multiple versions and compaction operations. High disk I/O and slow checkpoints are typical symptoms when these operations become a bottleneck.

#### Triage
Identify the Problem: High disk I/O suggests that RocksDB is frequently reading or writing data, which can be caused by frequent state changes or large cardinality states.
Understand Checkpointing: Slow checkpoints indicate that the process of saving the current state to persistent storage (e.g., RocksDB) takes too long, which can impact overall system performance and fault tolerance.
Tuning Options

Optimize State Management:

Compact State: Enable automatic compaction in RocksDB by adjusting parameters like compaction.large_compactor, which helps reduce the number of files and improve I/O efficiency.
Persistency Level: Use different persistency levels (EXACT ONCE, AT LEAST ONCE) depending on your application requirements. EXACT ONCE can help manage state more efficiently but might impact performance.

Checkpoint Configuration:

Increase Checkpoint Interval: This allows for fewer frequent checkpoints, reducing the overhead of checkpointing.
Parallel Checkpoints: Utilize parallelism by increasing the number of concurrent checkpoints (num.tasks in Flink). This can help distribute the I/O load and improve overall performance.

Storage Optimization:

Use SSDs: Replace traditional HDDs with SSDs, which offer significantly lower latency and higher throughput for disk operations.
Data Layout: Optimize data layout within RocksDB to minimize read/write patterns that cause fragmentation or hotspots.

Flink Configuration:

State Backend Settings: Tune Flink’s state backend settings such as state.backend.rocksdb.block-cache-size and state.backend.rocksdb.write-buffer-usage, which can improve the performance of state operations.
Checkpointing Strategies: Use checkpointing strategies like incremental checkpoints to reduce the amount of data written during each checkpoint.
When to Shard the Job

Job Complexity: If tuning options do not yield satisfactory results and the job complexity is high, sharding might be necessary. Sharding can distribute the workload across multiple instances, reducing the load on any single node.

Resource Utilization: Monitor resource utilization metrics (CPU, memory, network) to ensure that individual shards are not overloaded. If a shard becomes a bottleneck, consider adding more shards or redistributing the load.

Scalability: Sharding can significantly improve scalability and fault tolerance. By distributing the job across multiple instances, you can handle larger volumes of data and maintain consistent performance under heavy loads.

#### Closing Summary

In summary, tuning options such as optimizing state management, checkpointing configurations, storage optimizations, and Flink settings should be explored before resorting to sharding. Sharding is a viable option when these optimizations are insufficient or when dealing with extremely complex jobs that require more distributed processing capabilities.

#### Common Mistakes/Follow-Ups
A common mistake is underestimating the impact of state management on performance. Another potential issue is not monitoring and tuning regularly, which can lead to suboptimal configurations over time. Follow-ups might include discussing specific Flink configuration parameters or exploring alternative state backends if RocksDB continues to be a bottleneck.

**Code:**

```java
// Example of setting up RocksDB state backend with Flink
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new RocksDBStateBackend("hdfs://path/to/rocksdb/state", true));
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify High I/O and SlowCheckpoints
6. Is State ManagementOptimizable?
7. Tune State Backend Settings,Enable Compaction
8. Tune Checkpoint Interval,Parallel Checkpoints
9. Monitor Performance andResource Utilization
10. Sharding Needed?
11. Job Tuned Successfully
12. Shard Job Across MultipleInstances
13. Consider Alternative StateBackends or Sharding

