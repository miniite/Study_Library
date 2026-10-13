---
type: "practice"
difficulty: "Intermediate"
---

# You store per-user long-term memory for an agent in a sharded KV store keyed by user id, and a few power users create hot shards. Explain how you'd detect the hot key and three mitigations, and how memory-write ordering matters for correctness.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AI system where per-user long-term memory is stored in a sharded key-value (KV) store, detecting and managing "hot keys" (user IDs that generate a disproportionate amount of writes) is crucial to ensure performance and maintain the integrity of the system. This problem involves understanding how to identify these hot keys and implementing mitigations to handle them effectively. Additionally, we must address write ordering for correctness.

To tackle this, we'll first define what constitutes a "hot key" and then discuss three mitigation strategies. We'll also explore why memory-write ordering matters in such systems.

#### Triage
What the Question is Asking
Identifying Hot Keys: The question probes our ability to detect which user IDs generate excessive write operations.
Mitigations for Hot Keys: We need to suggest solutions that can handle the high load from these hot keys without compromising system performance or correctness.
Memory-Write Ordering: This aspect examines how writes are managed in a distributed system, ensuring data consistency and avoiding issues like stale reads.

#### Assumptions
The sharded KV store is horizontally scalable with multiple shards.
Writes to the user memory are time-sensitive but not immediately critical (e.g., not real-time).
We can monitor write operations at various points in the system, such as before and after write requests.
Step-by-Step Solution from First Principles

Identifying Hot Keys

Monitoring: Implement a monitoring system to track write operations per shard/user ID over time.
Thresholds: Define thresholds for what constitutes a "hot key" based on the number of writes or the rate of writes.

Mitigation Strategies

Shard Rebalancing: Periodically redistribute shards across nodes to ensure no single node bears an excessive load.
Write Throttling: Implement throttling mechanisms to limit write operations from hot keys, ensuring that other user data isn’t adversely affected.
Warm-Sharding: Proactively move hot shards to more powerful or dedicated nodes to handle the increased traffic.

Memory-Write Ordering

Atomicity and Consistency: Ensure that writes are atomic (cannot be interrupted) and consistent across all replicas of a shard.
Replication Strategy: Use strong consistency models where each write operation is acknowledged by multiple replicas before it’s considered complete.
Conflict Resolution: Implement conflict resolution strategies for concurrent writes to avoid data corruption.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Detecting hot keys involves monitoring and setting thresholds, while mitigations include rebalancing shards, throttling writes, and optimizing shard distribution. Write ordering ensures data integrity but can introduce latency.

#### Common Mistakes
Failing to implement robust monitoring for early detection of hot keys.
Ignoring the impact of write ordering on system performance and consistency.
Not considering the trade-offs between performance, cost, and complexity in mitigation strategies.
Triangular Question Follow-Ups
How would you balance between real-time and historical data storage?
What are some common failure modes in your proposed solutions, and how do you address them?
How would you scale this system to handle an increase in the number of users?

**Comparison:**

```
Shard	User ID	Write Count
shard1	user1234	3
	user5678	2
shard2	user9012	4
	user3456	2
```

**Code:**

```python
# Example code for monitoring write operations
def monitor_write_operations(shards):
    hot_keys = {}
    threshold = 1000  # Threshold for identifying hot keys (adjust as necessary)

    for shard, data in shards.items():
        for user_id, writes in data.items():
            if user_id not in hot_keys:
                hot_keys[user_id] = len(writes)
            else:
                hot_keys[user_id] += len(writes)

    # Identify and log hot keys
    for key, value in hot_keys.items():
        if value > threshold:
            print(f"Hot Key Detected: {key} with writes: {value}")
```

**Example input / output:**

```text
Input:
Monitoring data for shards as shown above.

Output:
Hot Key Detected: user9012 with writes: 4
Hot Key Detected: user1234 with writes: 3
```

**Data:**

```json
{
  "shards": {
    "shard1": {"user1234": [1, 2, 3], "user5678": [1, 2]},
    "shard2": {"user9012": [1, 2, 3, 4], "user3456": [1, 2]}
  }
}
```

**Flow:**

1. Yes
2. No
3. User Writes to Shard
4. Is Write Rate ExceedingThreshold?
5. Identify Hot Key and Log
6. Monitor Continuous WriteOperations
7. Continue Monitoring

