---
type: "practice"
difficulty: "Deep"
---

# Your event-sourced system reconstructs state from a log. How do isolation levels apply (or not) to the projection/read-model builders and where can anomalies sneak in there?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-sourced system, reconstruction of state from logs is crucial for maintaining consistency and ensuring that read operations see a consistent snapshot of the data. However, the application of isolation levels—such as RU (Read Uncommitted), RC (Read Committed), RR (Repeatable Read), and Serializable—can be complex when dealing with event-based projections. In this context, understanding how these isolation levels apply or do not apply is key to ensuring that read models are consistent and free from anomalies.

#### Triage
What the Question Is Asking

The question probes your understanding of isolation levels in the context of event-sourced systems and projection builders. Specifically, it challenges you to identify where anomalies might arise when reconstructing state and building read models. The core issue here is how different read operations interact with the immutable event log.

#### Assumptions
Event-Sourced System: Events are appended to a log in a sequential manner.
Read Model Builders: These are responsible for reconstructing the current state of entities from events.
Isolation Levels: RU, RC, RR, and Serializable define different degrees of visibility and consistency.

#### First-Principles Solution
1. Understanding Isolation Levels:
RU (Read Uncommitted): Allows reading uncommitted data. This level is not typically used in read models due to the risk of reading incomplete or inconsistent states.
RC (Read Committed): Ensures that only committed data is visible during reads. In an event-sourced system, this means waiting until all events have been committed before allowing a read operation.
RR (Repeatable Read): Ensures that repeated reads return the same result. This is challenging in an event-sourced system because the state can be constantly changing as new events are appended to the log.
Serializable: Provides the highest level of isolation, ensuring that all operations within a transaction see a consistent view of the data.
2. Applying Isolation Levels:
RC (Read Committed): This is often the most suitable for read models in an event-sourced system because it ensures that only committed events are considered during state reconstruction.
RR and Serializable: While theoretically possible, they can lead to performance issues due to the need to lock or cache data until all relevant events have been processed.
3. Anomalies and Consistency Issues:
Late-Committed Events: If an event is committed after a read operation but before it fully reconstructs the state, anomalies may occur.
Event Processing Delays: Variability in the time taken to process events can lead to inconsistent states being temporarily visible during reconstruction.
Read Operations During Reconstruction: If a read model builder performs reads while still processing events, it might encounter an incomplete state.
4. Mitigating Anomalies:
Batch Processing: Batch events together and ensure they are fully processed before allowing any read operations to proceed.
Caching: Cache the reconstructed state and invalidate caches when new events arrive, ensuring that reads always see a consistent snapshot.
Eventual Consistency: Design the system for eventual consistency where transient anomalies are acceptable within a certain timeframe.
5. Conclusion:

In an event-sourced system, using isolation levels like RC (Read Committed) is essential to ensure consistency and reliability in read operations. However, careful design and implementation of state reconstruction logic are crucial to mitigate potential anomalies and maintain the integrity of read models.

#### Common Mistakes/Follow-Ups
Ignoring Event Commitment: Failing to account for late-committed events can lead to temporary inconsistencies.
Overuse of Serializable Isolation Levels: While they provide strong guarantees, they come with significant performance overhead.
Inadequate Caching Strategies: Poorly designed caching mechanisms can introduce stale data issues.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Event Log] --> B[Reconstruction Logic];
    B --> C[Read Model Builder];
    C --> D[Consistent State];
    D --> E[Read Operation];
```

**Example input / output:**

```text
sequenceDiagram
    participant Log as EventLog
    participant Builder as ReadModelBuilder
    participant State as ReconstructedState
    participant Operation as ReadOperation

    Log->>Builder: AppendEvent(create)
    Builder->>Builder: ProcessEvent(create), UpdateReconstructedState()
    Builder-->>State: SetReconstructedState()

    Note over Builder,State: Ensuring consistency before committing state changes

    Builder->>Operation: GetConsistentState()
```

**Data:**

```json
{
  "eventLog": [
    {"id": 1, "type": "create", "timestamp": "2023-09-01T10:00:00Z"},
    {"id": 2, "type": "update", "timestamp": "2023-09-01T10:05:00Z"},
    {"id": 3, "type": "create", "timestamp": "2023-09-01T10:10:00Z"}
  ],
  "readModelBuilder": {
    "state": {},
    "reconstructedState": {}
  }
}
```

