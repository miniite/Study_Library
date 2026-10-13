---
type: "practice"
difficulty: "Deep"
---

# A distributed query fans out to 100 shards and its p99.9 is dominated by the single slowest shard. Explain tail-latency amplification for scatter-gather queries and how hedged/backup requests to replicas mitigate it, plus the extra load they create.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Tail-latency amplification in distributed systems occurs when a query or operation needs to gather results from multiple shards, and the slowest shard significantly impacts the overall performance. In your scenario, where 100 shards are involved and p99.9 is dominated by the single slowest shard, this issue becomes critical. Hedged requests, which involve sending multiple requests to replicas, can mitigate tail latency but introduce additional load on the system.

Let's break down the concept of tail-latency amplification in scatter-gather queries, explain how hedging works to mitigate it, and discuss the trade-offs involved.

#### Triaging the Question

The question is asking you to:

Explain tail-latency amplification.
Describe how hedged/backup requests can mitigate this issue.
Discuss the additional load that these backup requests create.

#### Assumptions

We are dealing with a distributed system where data is sharded across 100 shards.
The query in question is scatter-gather, meaning it requires results from all shards to complete.
p99.9 refers to the 99.9th percentile of response times, which means that 99.9% of queries are faster, and only 0.1% take longer.

#### First Principles Solution
Tail-Latency Amplification

In a scatter-gather query, each shard must process its part of the request before the result can be aggregated. If one or more shards respond extremely slowly, it can significantly increase the overall latency for the entire operation. This is known as tail-latency amplification.

Imagine you are playing a game where 100 players need to complete a task before you can proceed. If one player takes an unusually long time, everyone else has to wait until that single player finishes. In distributed systems, this "single slowest shard" effect causes the entire operation to be delayed.

Hedged Requests

To mitigate tail latency, we use hedged requests, which involve sending multiple requests to replicas of a shard. Replicas are typically secondary copies of data that can serve read-only queries faster than the primary shard. By having backup requests, we ensure that even if one request takes longer, another might complete quickly and contribute to the overall response.

For example, imagine you have three replicas for each shard. If sending a request to the primary shard does not respond in time, the system can send additional requests to two or more of the replicas. This increases the chances that at least one replica will respond quickly, thus reducing the overall latency.

#### Trade-offs
Additional Load

While hedging reduces tail latency, it comes with an increased load on the system:

Replica Server Overload: Replicas are typically less optimized for write operations and may not be as performant under heavy read loads. Sending multiple requests to them can overload these servers.
Network Utilization: Additional requests increase network traffic, which can introduce latency and consume more bandwidth.
Resource Consumption: More server resources are utilized due to the need to handle additional queries.

#### Closing Summary

Tail-latency amplification in distributed systems is a critical issue that needs addressing. Hedged requests help mitigate this by ensuring that even if one shard is slow, others can contribute quickly. However, this approach introduces additional load and resource consumption, which must be carefully managed.

#### Common Mistakes/Follow-Ups
Misunderstanding of Tail Latency: Failing to recognize how the slowest shard affects overall performance.
Overlooking Resource Constraints: Not considering the potential for replicas to become overloaded or network congestion.
Ignoring Scalability Implications: Failing to evaluate long-term scalability and resource usage implications.

#### Example Scenario

#### Cons
ider a scenario where you have 100 shards, each with three replicas. If p99.9 is dominated by one slow shard, sending hedged requests can significantly reduce the overall latency. However, this also means that if multiple queries are sent to the replicas, they might experience increased load.

Summary Confirmation

The answer covers tail-latency amplification, hedging for mitigation, and trade-offs clearly.

**Comparison:**

```
Shard 50
Replica B
Response Time: 400ms
```

**Code:**

```sql
-- Example of a distributed query in SQL
SELECT * FROM global_table WHERE id IN (1, 2, 3);
```

**Example input / output:**

| Query | Target Shard | Response Time (ms) |
|-------|--------------|-------------------|
| 1     | 0            | 200               |
| 2     | 50           | 8000              |
| 3     | 50           | 400               | # Hedged request to replica B

**Data:**

```yaml
# Sample data schema for shards and replicas
shards:
- shard_id: 0
  replicas:
  - replica_id: A
  - replica_id: B
  - replica_id: C

queries:
- query_id: 1
  target_shard: 0
  response_time_ms: 200
- query_id: 2
  target_shard: 50
  response_time_ms: 8000 # This is the slow shard
```

**Flow:**

1. A--"Send initial query to Shard 50"-->B["8000 ms"]
2. A--"Send hedged request to Replica B of Shard 50"-->C["400 ms"]

