---
type: "practice"
difficulty: "Deep"
---

# Design the end-to-end architecture for a 'time travel' debugging capability where support can replay exactly what a user saw and did (frontend state, network calls, backend responses) for a reported bug, without recording PII or ballooning storage. Where's the seam?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an end-to-end architecture for time travel debugging that allows replaying exactly what a user saw and did in real-time applications, while avoiding the recording of personally identifiable information (PII) and minimizing storage overhead, we need to carefully consider several components and their interactions. The key idea is to ensure data integrity without compromising privacy by leveraging event sourcing and snapshotting techniques.

#### Triage

The question is asking for a detailed system design that captures and stores application state changes in a way that allows precise replay while ensuring PII protection. We'll need to address the following:

Data Integrity: Ensure accurate state recording.
Privacy Compliance: Prevent PII from being recorded or stored.
Efficient Storage: Minimize storage overhead without compromising on replay capabilities.
Step-by-Step Solution
1. Event Sourcing with Snapshotting

Event sourcing is a technique where the history of all changes to an entity's state are stored as a sequence of events, rather than simply storing the final state. This allows us to reconstruct any point in time by applying these events sequentially.

Key Components:

Events: Represent actions or states that change the application’s state.
Snapshots: Periodic snapshots that store the current state at certain intervals to reduce storage requirements and improve performance during replay.
2. PII Masking

To ensure privacy, we can mask sensitive data in events before storing them. This involves defining a policy for what constitutes PII and implementing mechanisms like tokenization or hashing to protect this information.

#### Example

PYTHON
def mask_pii(event):
    if 'user_id' in event:
        event['user_id'] = hash(event['user_id'])
3. Event Storage

We need a reliable, scalable storage mechanism for events and snapshots. A distributed system like Cassandra or a time-series database like InfluxDB can be effective choices.

#### Example
Schema:

SQL
CREATE TABLE events (
    id UUID PRIMARY KEY,
    user_id UUID,
    timestamp TIMESTAMP,
    event_data JSONB
);
4. Replay Mechanism

To replay the application state, we need a mechanism to retrieve and apply stored events in chronological order. This can be achieved using stream processing frameworks like Kafka or AWS Kinesis.

Replay Process:

Identify the starting point (e.g., timestamp).
Fetch relevant events from storage.
Apply each event to reconstruct the state.
Handle snapshots by applying them when necessary to optimize performance.
5. Frontend Integration

The frontend must be able to request specific timestamps and receive the reconstructed state. This involves maintaining a consistent API layer that abstracts away the complexity of event replay.

#### Example
Request/Response:

HTTP
POST /api/debug/replay/{userId}/{timestamp}
Headers:
Content-Type: application/json

Request Body:
{
    "timestamp": 1634502000,
    "user_id": "someUserId"
}

Response Body:
{
    "frontend_state": {
        // Detailed state of the frontend at the requested timestamp
    },
    "network_calls": [
        // Network requests and responses as they occurred at the time
    ]
}

#### Closing Summary

By leveraging event sourcing, snapshotting, PII masking, efficient storage mechanisms, robust replay mechanisms, and well-integrated frontends, we can build a highly scalable and privacy-compliant time travel debugging system. The key is to ensure that all components work seamlessly together while maintaining strict controls over data access and storage.

#### Common Mistakes/Follow-Ups
Mistake: Failing to properly mask PII, leading to potential breaches.

#### Follow-Up
Discussing alternative masking techniques or compliance frameworks like GDPR for handling sensitive data.
Mistake: Overlooking the performance implications of snapshotting and event storage.

#### Follow-Up
Exploring optimization strategies such as incremental snapshots or more efficient event storage formats.

**Code:**

```python
def mask_pii(event):
    if 'user_id' in event:
        event['user_id'] = hash(event['user_id'])
```

**Example input / output:**

```

Example request and response for replay:

HTTP
Copy
POST /api/debug/replay/{userId}/{timestamp}
Headers:
Content-Type: application/json

Request Body:
{
    "timestamp": 1634502000,
    "user_id": "someUserId"
}

Response Body:
{
    "frontend_state": {
        // Detailed state of the frontend at the requested timestamp
    },
    "network_calls": [
        // Network requests and responses as they occurred at the time
    ]
}
```

**Data:**

```sql
CREATE TABLE events (
    id UUID PRIMARY KEY,
    user_id UUID,
    timestamp TIMESTAMP,
    event_data JSONB
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. Event Source
7. Is event a PII?
8. mask_pii
9. Store Event in DB
10. snapshot?
11. Take Snapshot
12. Store Snapshot in DB
13. Stream Events to ReplaySystem
14. User Request for Replay
15. Is replay required at a specifictimestamp?
16. Retrieve and Apply Eventsfrom DB
17. Replay from latest snapshot

