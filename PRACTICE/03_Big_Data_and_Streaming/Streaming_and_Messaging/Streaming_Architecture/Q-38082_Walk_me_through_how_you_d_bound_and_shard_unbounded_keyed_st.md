---
type: "practice"
difficulty: "Expert"
---

# Walk me through how you'd bound and shard unbounded keyed state (e.g., per-user session state) when a small fraction of keys hold most of the data. What's your hot-key mitigation and state-TTL strategy, and how do you avoid unbounded memory growth?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Graceful Degradation in Streaming](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To bound and shard unbounded keyed state in a real-time streaming architecture, we need to address the challenge of uneven data distribution where a small fraction of keys (e.g., per-user session states) hold most of the data. This problem can lead to hotspots and memory issues if not properly managed. The key is to implement an effective hot-key mitigation strategy combined with state TTLs to control memory usage.

First, let's break down what the question is asking:

Hot-Key Mitigation: How do we handle keys that have disproportionately high volumes of data or traffic?
State TTL Strategy: How do we manage the lifecycle of states to ensure efficient memory usage and avoid unbounded growth?

We'll start with a high-level overview, then dive into the first principles.

#### Overview

To solve this problem, we will use a combination of sharding, caching, and time-to-live (TTL) mechanisms. The goal is to distribute the load evenly across shards while ensuring that hot keys are managed effectively to prevent memory bloat in any single shard.

#### Triage

The question is probing your ability to design scalable and efficient systems for real-time streaming data. We need to consider trade-offs between performance, memory usage, and fault tolerance. The assumptions include:

Data Distribution: Keys are unevenly distributed.
Hot Key Impact: A small number of keys hold most of the data or traffic.

#### First Principles Solution

Sharding Strategy:

Hash-based Sharding: Use a hash function to map keys to shards. For per-user session states, we can use the user ID as the key.
User
User ID
Shard
State Data
Dynamic Sharding: If the number of shards is not fixed, dynamically add or remove shards based on current load. This can be managed using a service like Apache Zookeeper.

Caching:

Implement in-memory caching for frequently accessed keys to reduce database I/O and improve read performance.
PYTHON
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_user_state(user_id):
    # Fetch user state from the shard
    return get_state_from_shard(user_id)

State TTLs:

Set a Time-to-Live (TTL) for each key to automatically expire and clean up old data.
JSON
{
  "user_states": {
    "user123": {"state_data": "data", "ttl": 600}  // Expiry in 10 minutes
  }
}
Use a cleanup service that periodically checks for expired states and removes them from memory.

Hot-Key Mitigation:

Detection: Monitor shard usage and identify hot keys using metrics like read/write counts.
Load Balancing: Move hot keys to multiple shards if necessary.
Tiered Storage: For very hot keys, use a tiered storage approach where the data is stored in both memory and disk. This ensures performance while managing memory usage.

Fault Tolerance:

Ensure that shard failures do not cause data loss or unavailability by implementing replicas and failover mechanisms.
Shard 1
Replica

**Data:**

```
Shard 2
Replica
```

**Data:**

```
Shard 3
Replica
```

**Data:**

```
Closing Summary and Common Mistakes
Summary: By implementing a sharded, caching, and TTL-based strategy, we can effectively manage unbounded keyed state in real-time streaming architectures. This ensures that hot keys are mitigated, and memory usage remains under control.
Common Mistakes:
Overlooking the impact of hot keys on shard performance.
Ignoring the importance of timely cleanup through TTLs.
Failing to implement adequate fault tolerance mechanisms.
```

**Comparison:**

| Shard | Load | State Count |
|-------|------|-------------|
| 1     | High | 500         |
| 2     | Low  | 10          |
| 3     | Medium | 200       |

**Code:**

```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_user_state(user_id):
    # Fetch user state from the shard
    return get_state_from_shard(user_id)
```

**Example input / output:**

```text
# Example input-output for a hot key detection and load balancing mechanism
def monitor_shard_usage(shards):
    # Simulate monitoring of shard usage
    for shard in shards:
        if shard.get_load() > THRESHOLD_LOAD:
            move_hot_keys_to_multiple_shards(shard)

def move_hot_keys_to_multiple_shards(hot_key_shard):
    # Move hot keys to multiple shards
    for key in hot_key_shard.keys:
        distribute_key(key, other_shards)
```

**Data:**

```json
{
  "user_states": {
    "user123": {"state_data": "data", "ttl": 600}  // Expiry in 10 minutes
  }
}
```

**Flow:**

1. User
2. User ID
3. Shard
4. State Data
5. Hot Key Detection
6. Monitor Shard Load
7. Move Hot Keys to MultipleShards
8. Periodic Cleanup
9. Remove Expired States fromMemory

