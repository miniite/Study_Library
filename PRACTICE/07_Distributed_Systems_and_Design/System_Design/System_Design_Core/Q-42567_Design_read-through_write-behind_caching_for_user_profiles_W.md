---
type: "practice"
difficulty: "Deep"
---

# Design read-through + write-behind caching for user profiles. What's your invalidation approach, how do you bound data loss if the cache dies before the write-behind flush, and when is write-behind too risky?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read-through and Write-behind Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design read-through + write-behind caching for user profiles, we need to ensure that reads are served from the cache while writes are eventually propagated back to a central data store. The key challenges are handling invalidation and bounding data loss if the cache dies before flushing changes. Write-behind can become risky under certain conditions, which I'll detail later.

#### Triage

The question is probing how you manage caching consistency in distributed systems, specifically focusing on invalidation strategies and potential data loss scenarios. You need to consider trade-offs between performance and reliability, as well as the risks associated with different implementation choices.

#### Assumptions
We are dealing with a high-traffic application where user profiles are frequently accessed but infrequently updated.
The central data store (e.g., RDBMS) is used for authoritative storage of user profiles.
The cache system supports read-through and write-behind caching mechanisms.

#### Solution
Invalidation Approach

Invalidation strategies can be categorized into two main types: push-based and pull-based.

Push-Based Invalidations: This approach involves sending a message to the cache when an update occurs in the central data store, instructing it to invalidate specific cache entries. Pros include minimal overhead for each write operation and immediate invalidation of stale data. Cons are that this requires additional coordination between the central data store and the cache system.

Pull-Based Invalidations: This involves the cache periodically polling the central data store to check if a cached entry is outdated. The main advantage is simpler implementation, but it can lead to delays in invalidating stale data.

For our scenario, push-based invalidation appears more suitable due to its faster response time and better consistency guarantees. We can use a publish-subscribe model where updates are pushed to all relevant cache nodes.

Bounding Data Loss

Bounding data loss if the cache dies before the write-behind flush requires careful design:

Write-Ahead Logging (WAL): Maintain a log of all writes in the central data store. When the cache fails, this log can be used to replay and ensure eventual consistency.

Replication: Replicate updates across multiple nodes so that even if one node loses its cache state, other nodes can take over.

Time-to-Live (TTL): Set a TTL for cached entries. If the cache fails, the central data store will eventually expire and recreate the entry based on the latest version in storage.

Risk of Write-Behind

Write-behind caching introduces risks that need to be carefully managed:

Latency: Synchronous writes can introduce latency issues, especially if the write-back process is slow. This can affect user experience, particularly for real-time applications.

Data Consistency: There's a risk of data inconsistency during cache flush operations, leading to stale reads or even conflicting updates.

Resource Utilization: Continuous background flushing can consume significant resources, potentially impacting system performance and stability.

#### Closing Summary

In summary, for read-through + write-behind caching, push-based invalidation is preferred due to its immediate effect. Bounding data loss requires mechanisms like WAL and replication. Write-behind caching should be used judiciously, considering the trade-offs in terms of latency and resource utilization.

#### Common Mistakes/Follow-Ups
Failing to Use Push-Based Invalidations: This can result in stale reads.
Overlooking Data Loss Risks: Without proper bounds on data loss, the system may become unreliable during cache failures.
Ignoring Performance Implications: Over-reliance on write-behind caching without adequate monitoring and optimization can lead to suboptimal performance.

Would you like more details on any specific aspect?

**Comparison:**

```
Invalidation Strategy	Push-Based	Pull-Based
Pros	Immediate invalidation, minimal overhead for writes.	Simpler implementation, reduces coordination complexity.
Cons	Requires additional messaging infrastructure.	Potential delays in invalidation, may lead to stale data.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[User Request] --> B{Is Cache Valid?};
    B -- Yes --> C[Retrieve from Cache];
    B -- No --> D[Fetch from Database];
    D --> E[Store in Cache];
    D --> F[Send Write-Behind Request to Central Store];
    C --> G[Cache Hit! Return Profile to User];
```

**Example input / output:**

```text
- **Input**: User requests profile with ID `12345`.
- **Output**:
  - Cache hit: Profile is retrieved from cache and returned to user.
  - Cache miss: Profile fetched from database, stored in cache, write-behind request sent, and then profile returned to user.
```

**Data:**

```json
{
  "user_profile": {
    "id": "12345",
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Cache Valid?
5. Retrieve from Cache
6. Fetch from Database
7. Store in Cache
8. Send Write-Behind Request toCentral Store
9. Cache Hit! Return Profile toUser

