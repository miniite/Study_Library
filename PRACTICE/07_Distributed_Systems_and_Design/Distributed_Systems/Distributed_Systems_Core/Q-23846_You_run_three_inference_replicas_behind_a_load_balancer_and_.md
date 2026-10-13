---
type: "practice"
difficulty: "Deep"
---

# You run three inference replicas behind a load balancer and round-robin routing causes uneven KV-cache memory because sessions have different context lengths. Explain why round-robin is wrong for LLM serving and what a KV-cache-aware or least-loaded router does instead.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of uneven KV-cache memory across three inference replicas due to round-robin routing, we need to understand why round-robin is inappropriate for serving large language models (LLMs) and explore a more effective approach—KV-cache-aware or least-loaded routing.

Triaging the Problem
What's Being Probed:

The core of this question revolves around understanding the nuances of load balancing in distributed systems, specifically for LLMs. The interviewer is likely probing your knowledge on cache coherence, resource utilization, and the implications of different routing strategies.

Key Assumptions:
Inference Replicas: There are three inference replicas behind a load balancer.
KV-cache Memory: Sessions have varying context lengths, affecting their memory footprint in the KV-cache.
Uneven Distribution: The round-robin distribution leads to an uneven cache utilization.

#### First-Principles Solution
Why Round-Robin is Wrong for LLMs:

Round-robin routing assigns requests to replicas based on a simple cyclic order, which can lead to inefficient use of resources, especially when dealing with varying session contexts. In the context of LLMs:

Memory Fragmentation: Sessions with longer contexts will consume more memory in the KV-cache. Round-robin might distribute such sessions unevenly across replicas, leading to some being underutilized while others are overburdened.
Cache Coherence: Each session has a unique context that needs to be cached, and this requires coordination among replicas to ensure consistent results. Round-robin doesn't account for the actual memory usage of each session.
KV-cache-aware or Least-loaded Routing:

To address these issues, a more sophisticated routing strategy is required:

Least-loaded Routing: This approach routes requests to the replica with the least cache load. By monitoring the current state of each replica's cache, this method ensures that no single replica becomes overloaded while others remain underutilized.
Cache-aware Load Balancing: The system can maintain a record of which sessions are currently being served and their respective memory footprints. This information is used to make informed decisions about where new requests should be directed.

#### Implementation
Steps:
Monitoring Cache Usage: Each replica continuously monitors its cache usage and reports this information back to the load balancer.
Load Balancer Logic: The load balancer then selects the replica with the least cached sessions for each incoming request, ensuring a more balanced distribution of load.
Dynamic Adjustment: As session contexts change over time, the system can dynamically adjust its routing decisions to maintain optimal cache usage.

#### Closing Summary
and Common Mistakes

In summary, round-robin is not ideal for LLMs because it does not account for varying memory footprints caused by different session contexts. A more effective solution involves using a least-loaded or cache-aware routing strategy that monitors and balances the load based on current cache states. Common mistakes include failing to recognize the importance of cache coherence and resource utilization, or over-simplifying the problem as just a matter of distributing requests evenly.

**Code:**

```python
# Example pseudo-code for least-loaded routing

class LeastLoadedBalancer:
    def __init__(self):
        self.replicas = [Replica1(), Replica2(), Replica3()]

    def get_least_loaded_replica(self):
        min_load = float('inf')
        chosen_replica = None
        for replica in self.replicas:
            if replica.get_cache_load() < min_load:
                min_load = replica.get_cache_load()
                chosen_replica = replica
        return chosen_replica

# Example Replica class
class Replica:
    def __init__(self):
        self.cache_usage = 0

    def get_cache_load(self):
        return self.cache_usage

balancer = LeastLoadedBalancer()
chosen_replica = balancer.get_least_loaded_replica()
```

**Example input / output:**

```text
Input:
- Request to distribute sessions among replicas

Output:
- Replica with the least cache load is chosen (Replica 2 in this case)
```

**Data:**

```json
{
  "replicas": [
    {"id": 1, "cache_usage": 50},
    {"id": 2, "cache_usage": 30},
    {"id": 3, "cache_usage": 70}
  ]
}
```

