---
type: "practice"
difficulty: "Deep"
---

# Design a read-heavy service with the CQRS pattern where writes go to a normalized store and reads to a denormalized projection — how do you keep the projection eventually consistent and handle projection rebuild?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a read-heavy service using the Command Query Responsibility Segregation (CQRS) pattern, where writes go to a normalized store and reads are served from a denormalized projection, ensuring eventual consistency between them is crucial. This involves understanding how changes propagate from the write-store to the projection, handling failure scenarios effectively, and managing performance optimization.

#### Triaging the Question

The key here is to ensure that updates made to the normalized store are reflected in the denormalized projections efficiently and reliably. The CQRS pattern separates command handlers (writes) from query handlers (reads), allowing us to optimize each for their specific purpose. However, the challenge lies in maintaining consistency between these two stores.

Key Assumptions
Write-Store: A transactional database that ensures ACID properties.
Projection Store: Typically a NoSQL or read-optimized store.
Event Sourcing: Writes are recorded as events in an event log, which can be replayed to update the projection.

#### First Principles Solution

Event Sourcing and Projections:

When a write operation occurs (e.g., updating user data), it is logged as an event.
These events are stored in an event store.
Queries for reads rely on projections, which are updated asynchronously by replaying these events.

Rebuilding Projections:

Projections can be rebuilt from scratch when needed (e.g., during system restarts or upgrades).
A background process can periodically scan the event log and apply all past events to update the projection.

Handling Partial Consistency:

During periods of high load, it’s acceptable for projections to be slightly out-of-date.
Use versioning and timestamps in events to ensure correct reordering if the system is unavailable during a period where events are not fully processed.
Detailed Steps

Event Logging:

Each write operation generates an event (e.g., UserUpdated).
Events include necessary metadata like timestamp, primary key, payload, and version.

Event Store:

The event store is durable and reliable to ensure no events are lost.
Use a distributed system like Apache Kafka for high availability.

Projection Update Process:

A background service (projection worker) continuously reads from the event log.
It applies each event to update the projection state, ensuring idempotency and atomicity.
Events may be retried if processing fails due to system issues.

Rebuilding Projections:

During a deployment or maintenance window, trigger a full rebuild of projections.
This ensures that all events are processed up-to-date before serving reads.

Handling Concurrency and Failures:

Use distributed locks to prevent race conditions when applying events in parallel.
Implement retries with exponential backoff for failed updates.
Monitor the event backlog to detect bottlenecks or system issues early.

Performance Optimization:

Cache frequently accessed projections to reduce load on the event store and speed up reads.
Use caching strategies like Redis or a distributed cache for read-heavy queries.

#### Closing Summary

To maintain eventual consistency between write-store and projection, leverage an event sourcing strategy where events are logged, replayed, and used to update projections. This approach ensures robustness against failures while optimizing performance during high load periods. Regular monitoring and tuning of the system help in maintaining a balance between consistency and availability.

#### Common Mistakes/Follow-Ups
Failing to Handle Event Backlog: Ensure that the event backlog is managed properly to avoid delays.
Not Using Versioning: Use version numbers or timestamps in events to handle out-of-order processing correctly.
Ignoring Concurrency Issues: Implement distributed locks and retries to manage concurrent updates.

**Comparison:**

```
Event Type	Payload Example
UserUpdated	{ "user_id": 1, "new_data": {"email": ...} }
OrderCreated	{ "order_id": 2345, "product": ... }
```

**Code:**

```python
# Example code snippet for event handling in a background worker

def process_event(event):
    # Update the projection based on the event type
    if event.type == 'UserUpdated':
        update_user_projection(event.payload)
    elif event.type == 'OrderCreated':
        update_order_projection(event.payload)

def update_user_projection(payload):
    user_id = payload['user_id']
    new_data = payload['new_data']
    # Apply changes to the projection store
```

**Example input / output:**

```text
Input Event:
{
  "type": "UserUpdated",
  "timestamp": "2023-10-05T12:00:00Z",
  "payload": {
    "user_id": 1,
    "new_data": {"email": "updated@example.com"}
  }
}

Output (Projection Update):
{
  "user_id": 1,
  "email": "updated@example.com"
}
```

**Data:**

```json
{
  "event": {
    "type": "UserUpdated",
    "timestamp": "2023-10-05T12:00:00Z",
    "payload": {
      "user_id": 1,
      "new_data": {"email": "updated@example.com"}
    }
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Event Occurs
5. Is Event in Event Store?
6. Record Event
7. Background Worker Readsfrom Event Log
8. Process Event?
9. Update Projection
10. Projection Updated
11. Handle Failure, Retry Later

