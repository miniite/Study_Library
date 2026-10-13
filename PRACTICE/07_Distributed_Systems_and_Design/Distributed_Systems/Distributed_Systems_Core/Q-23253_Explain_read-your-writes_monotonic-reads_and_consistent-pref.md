---
type: "practice"
difficulty: "Deep"
---

# Explain read-your-writes, monotonic-reads, and consistent-prefix consistency. For a social feed served from read replicas, which of these do users notice violating, and how do you provide them cheaply?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question in a backend architect/principal capacity with deep understanding, we'll break down the consistency models—read-your-writes (RYW), monotonic-reads (MR), and consistent-prefix (CP)—and then discuss their implications for a social feed served from read replicas. We’ll also explore how to provide these cheaply.

TL;DR

Read-your-writes (RYW) ensures that writes are visible to the writer, but not necessarily others. Monotonic-reads (MR) guarantees that reads follow an increasing order of timestamps. Consistent-prefix (CP) is a trade-off between RYW and MR, where updates within a prefix are consistent while still allowing some delayed visibility.

In a social feed scenario using read replicas, users will most likely notice violations in the form of out-of-order posts or stale content, which aligns with monotonic-reads. To provide these cheaply, we can use techniques like read repair and selective caching to ensure consistency without excessive overhead.

#### Triage

The question asks us to explain three different consistency models (RYW, MR, CP) and determine which one users are most likely to notice violating in a social feed scenario using read replicas. It also seeks strategies for providing these consistency guarantees cheaply.

#### Assumptions
The system uses distributed databases with read replicas.
The primary focus is on user experience (UX).
Cost efficiency is a key consideration.

#### First-Principles Solution

Read-Your-Writes (RYW): RYW ensures that the writer sees their own writes immediately, but other nodes might see them after some delay. This model provides strong write consistency but can lead to stale reads for other clients. For example, if user A posts a status update, they will see it immediately, but others might not until replication occurs.

Monotonic-Reads (MR): MR guarantees that the order of writes is preserved across time. In simpler terms, all clients should eventually see the same order of updates as the writer. However, there can be delays in visibility due to replica lag or network issues. This model ensures that users experience a consistent feed but may see stale content temporarily.

#### Cons
istent-Prefix (CP): CP is a hybrid approach where updates within a certain prefix are guaranteed to be consistent. For example, if user A writes a status update with an ID between 1-1000, all clients will eventually see this update in the correct order, but updates outside this range might not be visible immediately.

#### Closing Summary
and Common Mistakes

In the context of a social feed served from read replicas, users are most likely to notice violations in the form of out-of-order posts or stale content. This aligns with monotonic-reads (MR) because it ensures that all clients eventually see the same order but may experience some temporary staleness.

To provide these consistency guarantees cheaply:

Read Repair: Replicas can proactively fetch missing data from other replicas to ensure up-to-date information.
Selective Caching: Cache updates in a way that minimizes stale content without overloading the system with redundant reads.

#### Common Mistakes
Overlooking Trade-offs: Users might not notice RYW violations as much, but they can lead to inconsistent experiences.
Ignoring Cost: Implementing MR or CP fully can be expensive. Balancing consistency and cost is crucial.
Neglecting Network Latency: Assuming that all replicas will stay in sync without addressing network delays can lead to inconsistent user experiences.
Closing

By understanding the nuances of RYW, MR, and CP, we can design a system that ensures users have a consistent experience with minimal overhead. While full consistency is ideal, practical considerations like read repair and selective caching allow us to strike a balance between cost and user satisfaction.

**Comparison:**

```
Consistency Model	Key Characteristics	User Experience Impact
RYW	Writer sees writes immediately	Users may not notice immediate changes
MR	Orders of writes preserved	Out-of-order posts are visible
CP	Updates within prefix consistent	Some delay in full consistency
```

**Code:**

```python
# Example code for simple read-repair mechanism
def read_repair(replica_ids, primary_data):
    for replica_id in replica_ids:
        if not is_up_to_date(replica_id, primary_data):
            fetch_and_update(replica_id, primary_data)
```

**Example input / output:**

```text
Input (Primary Data):
{
  "posts": [
    {"id": 1, "user_id": 100, "content": "Hello world", "timestamp": 1632456000},
    {"id": 2, "user_id": 101, "content": "Nice day", "timestamp": 1632457000}
  ]
}

Output (Replica Data After Read Repair):
{
  "posts": [
    {"id": 1, "user_id": 100, "content": "Hello world", "timestamp": 1632456000},
    {"id": 2, "user_id": 101, "content": "Nice day", "timestamp": 1632457000}
  ]
}
```

**Data:**

```json
{
  "replicas": [
    {"id": 1, "data": { /* ... */ }},
    {"id": 2, "data": { /* ... */ }},
    {"id": 3, "data": { /* ... */ }}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Read from Primary?
4. No
5. Write Update
6. Is Writer?
7. Update Primary
8. Replicate to Replicas
9. Fetch and Update Replicas ifNecessary
10. Monitor Replica Lag
11. Read Request
12. Primary Data
13. Read from Replicas (withrepair)

