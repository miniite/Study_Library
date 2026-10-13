---
type: "practice"
difficulty: "Deep"
---

# You cache derived data that depends on a config value. When the config changes, thousands of derived entries must recompute. How do you spread that recompute over time to protect the origin?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage the recompute burden when a configuration change triggers recomputation of numerous derived data entries, we need to design a strategy that spreads the load evenly and efficiently across time. The core idea is to avoid overwhelming the origin (the source of truth) by spreading the recomputation tasks over multiple intervals. This approach involves several steps: identifying the key components, estimating the workload, and implementing a dynamic scheduling mechanism.

Firstly, we need to understand the dependencies between the config value and derived data entries. Each change in the config may affect different groups of derived data entries, so it's crucial to categorize these entries based on their dependency rules. For instance, if changing a config value A affects 10% of all entries, while another config B affects 50%, we can handle them differently.

Secondly, estimating the workload is essential for creating a realistic schedule. We need to know how long it takes to recompute each group of derived data and any potential bottlenecks in the origin system. This helps us determine the optimal interval between recomputations to avoid overwhelming the origin while ensuring timely updates.

To implement this strategy effectively, we can use a distributed task queue like Redis or Kafka to manage the recomputation tasks. Here's a step-by-step approach:

Categorize Dependencies: Group derived data entries based on their dependency rules.
Estimate Workload: Calculate the time required for each group and identify any potential bottlenecks.
Create a Schedule: Determine the optimal interval between recomputations to balance load with timeliness.
Queue Tasks: Use a distributed task queue to manage the recomputation tasks dynamically.

**Comparison:**

```
Dependency	Affected Entries	Recomputation Time
config_key_A	10 entries	30 seconds
config_key_B	20 entries	60 seconds
```

**Code:**

```python
import redis

# Initialize Redis client
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def recompute_derived_data(config_key):
    # Recompute logic for derived data entries depending on config key
    pass

def schedule_recomputations(dependencies):
    for config_key, affected_entries in dependencies.items():
        recomputation_time = estimate_recomputation_time(affected_entries)
        interval = determine_optimal_interval(recomputation_time)

        redis_client.zadd('recompute_queue', {config_key: time.time() + interval})

def estimate_recomputation_time(entries):
    # Estimate the total time required to recompute affected entries
    return sum(entry.computation_time for entry in entries) / len(entries)

def determine_optimal_interval(recomputation_time):
    # Calculate the optimal interval based on recomputation time and current load
    return recomputation_time * 1.5 + (time.time() - last_updated_time)
```

**Example input / output:**

```text
Input:
- Config change event for 'config_key_A'

Output:
- Redis queue entry added with timestamp: {time.time() + interval}
```

**Data:**

```json
{
  "dependencies": {
    "config_key_A": ["entry_1", "entry_2"],
    "config_key_B": ["entry_3", "entry_4"]
  }
}
```

**Flow:**

1. Receive Config Change Event
2. Identify Affected Entries
3. Estimate Recomputation Time
4. Calculate Optimal Interval
5. Schedule Task in Queue
6. Recompute Derived Data

