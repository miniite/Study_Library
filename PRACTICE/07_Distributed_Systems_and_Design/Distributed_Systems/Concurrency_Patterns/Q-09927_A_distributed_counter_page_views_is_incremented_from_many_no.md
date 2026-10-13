---
type: "practice"
difficulty: "Expert"
---

# A distributed counter (page views) is incremented from many nodes and you need approximate-but-fast reads. Compare CRDT counters, sharded Redis counters, and async aggregation. What consistency do you give up?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-service Invalidation Races](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system where we are incrementing a counter (page views) from multiple nodes and need fast approximate reads, we must carefully choose between CRDT counters, sharded Redis counters, and async aggregation techniques. Each approach has its own trade-offs in terms of consistency, availability, and performance. Let’s break down each option, understand the consistency model they offer, and compare them.

#### Triage
Understanding the Question

The question is essentially asking us to evaluate three different strategies for managing a distributed counter that must be incremented from multiple nodes while providing fast approximate reads. We need to consider how each approach handles concurrency issues, race conditions, and overall system consistency.

#### First-Principles Solution
1. CRDT (Conflict-free Replicated Data Types) Counters

CRDTs are a family of data structures that can be updated independently on multiple nodes and then merged without loss of data or order. This approach is particularly useful for achieving eventual consistency in distributed systems.

#### Pros

Eventual Consistency: CRDTs ensure that, over time, all nodes will converge to the same state.
Concurrency Handling: They handle concurrent writes gracefully by merging operations and resolving conflicts automatically.

#### Cons

Complexity: Implementing CRDTs can be complex and error-prone.
Approximate Reads: While reads are eventually consistent, they may not reflect the latest value immediately.
2. Sharded Redis Counters

Redis is a popular in-memory data store that supports atomic increment operations via the INCR command. Sharding can be used to distribute the load across multiple Redis instances.

#### Pros

High Performance: Redis has excellent performance for simple key-value operations.
Simplicity: Easy to implement and manage with well-documented APIs.
Partition Tolerance: Redis shards provide a natural way to handle partition tolerance, ensuring that each shard can operate independently.

#### Cons

Read Consistency: Reads from a sharded system may not reflect the latest value until all shards have updated. This is because reads and writes are eventually consistent but not strongly consistent.
Complexity in Sharding: Proper sharding and load balancing require careful design to avoid hotspots and ensure balanced traffic.
3. Async Aggregation

In this approach, we periodically aggregate the values from multiple nodes using a leader-election mechanism or consensus algorithm (like Raft).

#### Pros

Strong Consistency: Periodic aggregation can provide strong consistency if done with high frequency.
Controlled Reads: Reads are less likely to show outdated data since they reflect the latest aggregated value.

#### Cons

Latency: Aggregation introduces latency, and fast approximate reads might not be as responsive.
Complexity in Leader Election: Implementing a robust leader-election mechanism can add complexity to the system.

#### Closing Summary
+ Common Mistakes/Follow-Ups

When choosing between these options, it’s crucial to understand the specific requirements of your application. For most use cases where fast approximate reads are needed and eventual consistency is acceptable, sharded Redis counters might be the simplest and most performant solution. CRDTs offer a robust approach for handling concurrency but come with higher complexity. Async aggregation provides strong consistency at the cost of increased latency.

#### Common mistakes
include overestimating the need for strict consistency, underestimating the complexity of CRDT implementations, and not properly sharding Redis to avoid hotspots.

Common Follow-Ups
What are the trade-offs in terms of performance?
How do you ensure that sharded Redis is correctly implemented with proper load balancing?
Can we use a hybrid approach combining elements from CRDTs and async aggregation?

**Comparison:**

| Consistency Model | CRDT Counters        | Sharded Redis     | Async Aggregation |
|-------------------|---------------------|------------------|------------------|
| Eventual          | Yes                 | Eventually       | Strong           |
| Approximate Reads | Yes (Eventually)    | No               | Yes              |
| Complexity        | High                | Low              | Moderate         |
| Performance       | Medium - High       | High             | Moderate - High  |

**Code:**

```python
# Example of Redis Sharding
import redis

def get_redis_client(node):
    return redis.Redis(host=f'node_{node}', port=6379, db=0)

# Increment counter using sharded Redis
for node in range(10):  # Assuming 10 shards
    client = get_redis_client(node)
    client.incr('page_views')
```

**Example input / output:**

```text
Input:
- Increment 'page_views' on each node in parallel

Output:
- Each node increments its local Redis instance with `INCR page_views`

Dry Run:
Node 0 -> INCR page_views = 1
Node 1 -> INCR page_views = 1
...
Node 9 -> INCR page_views = 1
```

**Data:**

```json
{
    "nodes": ["node_0", "node_1", ..., "node_9"],
    "shards_per_node": 1,
    "counter_name": "page_views"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. CRDT Counters
8. Concurrency?
9. Handle Concurrent WritesGracefully
10. Implement Custom MergeLogic
11. Eventually Converge to CorrectState
12. Ensure Correctness via CRDTAlgorithms
13. Sharded Redis
14. Read Consistency?
15. Use High-Performance APIs
16. Implement Sharding Logic
17. Provide Approximate Readswith Lower Latency
18. Ensure Balanced LoadDistribution
19. Async Aggregation
20. Leader Election?
21. Periodically Aggregate Values
22. Use Simple Periodic Checks
23. Provide Strong Consistency atHigher Latency
24. Reduce Latency but SacrificeSome Consistency

