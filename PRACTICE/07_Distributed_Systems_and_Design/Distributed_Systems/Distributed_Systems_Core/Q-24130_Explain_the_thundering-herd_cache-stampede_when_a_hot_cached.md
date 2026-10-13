---
type: "practice"
difficulty: "Deep"
---

# Explain the thundering-herd/cache-stampede when a hot cached query result expires and 10k requests hit the database at once. Compare request coalescing, probabilistic early expiration, and a lock, and which you'd use at 100k QPS on a shared Postgres.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a cached query result expires in a distributed system, and many requests hit the database simultaneously to retrieve the same data, a thundering herd or cache-stampede can occur. This happens because all these requests simultaneously realize that the cache is expired and rush to fetch the latest data from the database, overwhelming it with a surge of load.

#### Triage

The question asks us to compare three strategies: request coalescing, probabilistic early expiration, and using a lock, in the context of mitigating thundering herd effects at high QPS (queries per second). We need to evaluate which strategy would be best for handling 100k QPS on a shared Postgres database.

Step-by-Step Solution
Request Coalescing

Description: This approach involves delaying or combining multiple requests that come in close succession, ensuring they are processed as one. For example, if two simultaneous requests hit the system, only one is allowed to proceed while others wait until it completes.

#### Pros

Reduces the number of database hits.
Simplifies implementation and can be easily integrated into existing systems.

#### Cons

Introduces latency in responses for subsequent requests that are delayed.
May not handle all use cases effectively, as delays may lead to inconsistent state if the underlying data changes while waiting.
Probabilistic Early Expiration

Description: This technique involves setting a slightly earlier expiration time on cache entries. Instead of expiring at an exact moment, it expires with some probability. For instance, instead of expiring exactly when 10k requests come in, it might expire early enough to ensure that only a few requests hit the database.

#### Pros

Reduces the likelihood of thundering herd by spreading out requests over time.
Can be implemented without changing much of the existing cache logic.

#### Cons

Requires careful tuning and monitoring to avoid premature expiration.
May still cause occasional spikes in load, especially if the early expiration isn't finely tuned.
Using a Lock

Description: This involves using a database lock mechanism to ensure that only one request can hit the database at any given time. The other requests will block until the lock is released.

#### Pros

Ensures data integrity and consistency.
Prevents multiple simultaneous reads from conflicting with writes.

#### Cons

Can lead to significant performance bottlenecks, especially under high QPS.
Introduces contention that can degrade overall system throughput and responsiveness.

#### Closing Summary

Given the need to handle 100k QPS on a shared Postgres database, probabilistic early expiration is likely the most effective strategy. It provides a balance between reducing the risk of thundering herd effects and maintaining acceptable performance. Request coalescing can be used in conjunction with probabilistic early expiration or as an additional layer for further optimization.

#### Common Mistakes/Follow-Ups
Ignoring Latency: Failing to account for the increased latency introduced by request coalescing.
Over-Reliance on Locks: Assuming that a lock can provide sufficient performance without understanding its impact on overall system throughput.
Underestimating Early Expiration: Not recognizing the importance of tuning early expiration probabilities carefully.

**Code:**

```sql
-- Example of probabilistic early expiration in PostgreSQL
ALTER TABLE cache_entries SET CACHE_POLICY ('LRU', 'TTL=10s, GRACE=5s');
```

**Example input / output:**

| Request  | Time     | Cache Expiration Check Result |
|----------|----------|--------------------------------|
| R1       | 9:00 AM  | Expired                        |
| R2       | 9:00 AM  | Expired                        |
| R3       | 9:05 AM  | Not Expired                    |

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Request Arrival
6. Is Cache Expiration CheckDone?
7. Read from Database
8. Is Early Expiration ProbabilityMet?
9. Read from Database
10. Wait for Next Batch
11. Update Cache and Respond

