---
type: "practice"
difficulty: "Expert"
---

# You run replicas of a model server behind a load balancer and see uneven GPU memory usage because of KV-cache. Explain why round-robin load balancing is wrong for LLM serving and what a KV-cache-aware or least-loaded router does instead.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Load Balancing Stateful Inference](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Load_Balancing_Stateful_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of uneven GPU memory usage due to KV-cache, it's essential first to understand why round-robin load balancing is not suitable for LLM serving and what a more sophisticated routing mechanism can do instead.

TL;DR

Round-robin load balancing distributes requests evenly across servers but ignores stateful data like the KV-cache. This leads to uneven GPU memory usage because each replica might have different cache states, causing some to be overloaded while others are underutilized. A KV-cache-aware or least-loaded router can mitigate this by routing requests based on the current cache state of each server, ensuring more balanced use of resources.

#### Triaging the Question

The question is probing your understanding of load balancing strategies in the context of serving large language models (LLMs) where stateful data like a key-value (KV) cache plays a critical role. The core idea here is to recognize that traditional round-robin load balancing does not take into account the stateful nature of certain applications, which can lead to inefficiencies and imbalances.

#### Assumptions
We are dealing with multiple model replicas behind a load balancer.
These replicas maintain their own copies of a KV-cache.
The goal is to serve requests in an efficient manner while keeping GPU memory usage balanced across all replicas.

#### First-Principles Solution
Why Round-Robin Load Balancing Fails
Statefulness Ignored: In round-robin load balancing, each replica is treated as independent of the others. Requests are distributed evenly regardless of their state, such as the current contents of the KV-cache.
Cache Misses and Hits Variance: Different replicas might have different cache hit rates due to varying access patterns or recent updates in the data they serve. This means that while one replica might be experiencing frequent cache hits (low memory usage), another could be dealing with many cache misses (high memory usage).
Resource Imbalance: As a result, some replicas will end up handling more requests than others, leading to uneven GPU memory usage and potential performance bottlenecks.
Least-Loaded Router
Cache-Aware Routing: Instead of distributing requests strictly based on the round-robin pattern, a least-loaded router routes each request to the replica with the smallest current cache size or highest free memory.
Dynamic Load Balancing: This approach ensures that replicas with more available resources handle more requests, thereby maintaining better balance in GPU memory usage and overall performance.
Reduced Cache Invalidation Overhead: By minimizing the number of cache misses across all replicas, the least-loaded router can reduce the overhead associated with invalidating stale data.

#### Implementation
Example

Here’s a high-level flow for how a least-loaded router might work:

FLOWCHART · MERMAID
graph TD;
    A[User Request] --> B1(Router);
    B1 --> C1(Remote Replica 1) when "Free Memory High";
    B1 --> C2(Remote Replica 2) when "Free Memory Low";
    C1 --> D1(Processing Request);
    C2 --> D2(Processing Request);
    D1 --> E[Response] | Update Free Memory;
    D2 --> F[Response] | Update Free Memory;

#### Closing Summary

In summary, round-robin load balancing is inappropriate for LLM serving due to its inability to account for stateful data like the KV-cache. A more effective approach is a least-loaded router that routes requests based on the current free memory of each replica, ensuring balanced GPU usage and efficient resource utilization.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to recognize the importance of state in load balancing.

#### Follow-Up
Discussing how this solution can be further optimized using machine learning techniques to predict cache behavior or using hybrid strategies that combine static and dynamic routing.

**Comparison:**

```
Replicas	Free Memory
Replica 1	50
Replica 2	80
```

**Code:**

```python
# Example code for a simple least-loaded router
class LeastLoadedRouter:
    def __init__(self, replicas):
        self.replicas = replicas

    def route_request(self):
        min_free_memory = float('inf')
        selected_replica = None
        for replica in self.replicas:
            if replica.free_memory < min_free_memory:
                min_free_memory = replica.free_memory
                selected_replica = replica
        return selected_replica

class Replica:
    def __init__(self, free_memory):
        self.free_memory = free_memory

# Example usage
replica1 = Replica(50)
replica2 = Replica(80)
router = LeastLoadedRouter([replica1, replica2])
selected_replica = router.route_request()
print(f"Request routed to {selected_replica}")
```

**Example input / output:**

```text
# Example input and output for the least-loaded router
replica1 = Replica(50)
replica2 = Replica(80)

router = LeastLoadedRouter([replica1, replica2])
selected_replica = router.route_request()

assert selected_replica.id == 1 or selected_replica.id == 2
print(f"Request routed to {selected_replica}")
```

**Data:**

```json
{
  "replicas": [
    {
      "id": 1,
      "free_memory": 50
    },
    {
      "id": 2,
      "free_memory": 80
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Router
5. Free Memory < 70?
6. Select Replica 1
7. Select Replica 2

