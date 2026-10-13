---
type: "practice"
difficulty: "Expert"
---

# Probe: you propose storing all metadata as immutable events (event-sourced). What are the operational downsides — replay time, storage growth, snapshotting — and when would you regret not just mutating a relational store?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In proposing to store all metadata as immutable events in an event-sourced system, we aim to achieve strong consistency and full auditability. However, this approach comes with significant operational downsides such as replay time, storage growth, and the need for efficient snapshotting strategies. The decision to use an immutable event store over a relational database is highly context-dependent; certain scenarios might not justify the trade-offs involved.

The question is probing into the practical implications of using an event-sourced architecture for metadata management in contrast to traditional relational databases. We need to explore the downsides and identify situations where these downsides make the immutable event store unsuitable or costly. The key assumptions are that we have a choice between immutability and mutability, and the goal is to understand when one might be more advantageous than the other.

Let's break down each downside:

Replay Time: When making changes to metadata through immutable events, any update requires replaying all prior events leading up to the current state. This process can become computationally expensive as the number of historical events increases over time.
Storage Growth: As we continuously append new events without removing old ones, storage requirements balloon. Each event contributes to this growth, making it necessary to manage space more carefully.
Snapshotting Strategy: To mitigate some of the downsides, snapshotting techniques are often employed. However, efficient and optimal strategies for creating these snapshots can be complex and may introduce additional overhead.

Now, let's evaluate when we might regret choosing an immutable event store over a relational database:

Performance Requirements:
If real-time data updates or fast query performance is critical, the overhead of replaying events could make the system unacceptably slow.
Storage Constraints:
In environments with strict storage limits, the exponential growth of event data can quickly exhaust available space unless managed through sophisticated snapshotting and archiving techniques.

In summary, while immutable event sourcing offers robustness and auditability, it is not universally superior to relational databases. The choice should be guided by specific use cases where strong consistency and immutability are paramount. For scenarios requiring high throughput updates or real-time analytics, a traditional relational store might be more appropriate despite its potential for eventual consistency.

#### Common mistakes
include underestimating the storage growth implications, ignoring the performance overhead of event replay, and failing to implement effective snapshotting strategies. Follow-ups could involve discussing specific use cases where each solution excels or exploring hybrid approaches that leverage both models.

**Comparison:**

```
Event Type	Description
metadata.created	Initial creation of metadata entry
metadata.updated	Update of an existing metadata entry
```

**Code:**

```sql
-- Example schema for a simple relational metadata store
CREATE TABLE Metadata (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    value TEXT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**Example input / output:**

```text
Request to update metadata:
Input: { "name": "user_count", "newValue": 1000 }

Response (relational store):
- Updated "user_count" to 1000 in real-time.
- Query performance optimal for such operations.

Response (event-sourced store):
- Created new event with type "metadata.updated".
- Replayed all events from creation to current state, then applied the update.
```

**Data:**

```json
{
  "events": [
    { "id": 1, "type": "metadata.created", "data": { "name": "user_count", "value": 0 } },
    { "id": 2, "type": "metadata.updated", "data": { "name": "user_count", "value": 500 } }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Metadata Change Request
6. Relational Store
7. Immediate Update
8. Real-time Performance
9. Query Optimization
10. Event-Sourced Store
11. Replay Events
12. Full State Recomputation
13. Incremental Update Only

