---
type: "practice"
difficulty: "Deep"
---

# You're running online schema-change tooling (gh-ost/pt-osc/native) on a 2TB table in a sharded MySQL fleet during peak. Explain how it avoids long locks, why replication lag can still spike, and the coordination needed so the change applies consistently across all shards.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of applying schema changes to a 2TB sharded MySQL fleet during peak usage, we need to ensure minimal downtime and consistent application of changes across all shards. This involves understanding how distributed systems handle locks, replication lag, and coordination mechanisms.

Firstly, let's clarify what gh-ost (Guillermo Rauch Online Schema Transform) is designed for: it performs online schema changes without taking the database offline by creating a temporary tablespace on the fly, copying data from the old to new tables, and then switching them over. This approach minimizes locking by only holding locks temporarily.

However, during peak traffic, ensuring that replication lag does not spike is crucial because any delay can lead to inconsistencies across shards. Replication lag can be exacerbated due to various factors such as network latency, resource contention, or the inherent complexity of handling large datasets in real-time.

To coordinate schema changes effectively, we need a robust strategy that includes:

Lock Management: Using minimal locking techniques like row-level locks instead of table-level locks.
Replication Monitoring and Optimization: Ensuring replication is optimized to handle high write loads.
Shard Coordination: Implementing mechanisms to synchronize schema changes across shards.

#### Triaging the Question

The key points to address are:

How gh-ost or similar tools manage locking during schema changes.
Why replication lag might spike despite using distributed systems.
The coordination required to ensure consistent application of schema changes across all shards.

#### Assumptions
include:

We are dealing with a highly available, sharded MySQL database.
Schema changes need to be applied without significant downtime.
Peak traffic is a critical consideration for performance and consistency.

#### First Principles Solution

Lock Management: gh-ost minimizes locking by performing schema changes in stages, often using row-level locks instead of table-level locks. This allows other transactions to continue running while the change is being made. However, during peak traffic, even these minimal locks can cause contention and delays.

Replication Lag:

Network Latency: During high traffic periods, network latency increases, leading to increased replication lag.
Resource Contention: High write loads on the primary database can increase resource contention, slowing down the replication process.
Replication Mechanism: Binary logging and semi-sync replication are used in MySQL. These mechanisms introduce additional overhead during peak times.

Shard Coordination:

Synchronization Mechanisms: Implementing a synchronization mechanism like distributed locks or a coordination service (e.g., Zookeeper) to ensure that schema changes are applied simultaneously across all shards.
Conflict Resolution: Handling conflicts that might arise if different shards try to apply the same change at slightly different times.
Rollback and Retry Strategies: Ensuring that if a shard fails to apply the change, it can be retried or rolled back gracefully.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, distributed schema changes require careful management of locks, replication lag, and coordination across shards. gh-ost minimizes locking but cannot completely eliminate contention during peak traffic. Replication lag spikes due to network latency and resource contention. To ensure consistency, we need robust mechanisms for shard synchronization and conflict resolution.

#### Common mistakes
include:

Ignoring the impact of peak traffic on replication performance.
Underestimating the complexity of synchronizing schema changes across multiple shards.
Failing to implement effective rollback and retry strategies.

#### Follow-ups
could involve discussing specific tools and technologies used for synchronization, such as distributed locks or consensus algorithms like Raft. Additionally, exploring alternative approaches like partial sharding or using more advanced replication mechanisms (e.g., MySQL Group Replication) might be beneficial depending on the use case.

**Comparison:**

| Shard | Last Update Time (UTC) | Expected Consistency Time |
|-------|------------------------|--------------------------|
| 1     | 2022-04-25T08:00:00Z   | 2022-04-25T09:00:00Z     |
| 2     | 2022-04-25T08:00:10Z   | 2022-04-25T09:00:10Z     |

**Code:**

```sql
-- Example of a simple SQL query to check replication lag
SHOW SLAVE STATUS\G;
```

**Example input / output:**

```text
Input:
- Schema change request for a 2TB table in a sharded MySQL fleet during peak traffic

Output:
- Temporary tablespace created on the fly
- Data copied from old to new tables without locking
- Synchronization mechanisms ensure consistent application across all shards
```

**Data:**

```json
{
  "example_shard": {
    "shard_id": 1,
    "replica_set": ["primary", "secondary1", "secondary2"],
    "last_update_time": 1640995200,
    "expected_consistency_time": 1640995300
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Schema Change Request
6. Is Peak Traffic?
7. Minimize Locking withRow-Level Locks
8. Synchronize Schema ChangesAcross Shards UsingDistributed Locks
9. Apply Schema ChangesNormally
10. Replication Lag Spike?
11. Implement Retries andRollbacks
12. Schema Change AppliedSuccessfully

