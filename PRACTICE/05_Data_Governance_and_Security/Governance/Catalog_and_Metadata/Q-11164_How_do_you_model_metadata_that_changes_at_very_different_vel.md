---
type: "practice"
difficulty: "Deep"
---

# How do you model metadata that changes at very different velocities in one system — schema (rare), freshness/row-counts (per-run), ownership (occasional), popularity (continuous) — without either a bloated event stream or a stale snapshot?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data catalog and metadata engineering context, managing metadata with varying update velocities can be challenging. The key challenge here is to maintain a balance between freshness and completeness while avoiding either bloated event streams or stale snapshots. Let's explore how we can model such metadata effectively.

The question asks us to manage four types of metadata: schema changes (rare), freshness/row-counts per run (per-run updates), ownership (occasional changes), and popularity (continuous updates). We need a robust system that avoids either overloading with too much data or under-providing by relying on outdated information.

First, let's break down the requirements:

Schema Changes: These are infrequent but critical updates.
Freshness/Row-Counts: Need to track these per run and can be quite frequent.
Ownership: Ownership details change occasionally.
Popularity: This data is updated continuously.

To address this, we'll design a system that leverages a combination of real-time streams and historical snapshots with efficient querying mechanisms.

#### Triaging the Question
Schema Changes: We need to ensure these changes are captured but don't overwhelm our systems.
Freshness/Row-Counts: This data needs frequent updates but doesn't require immediate processing after each update.
Ownership: Occasional updates, so we can handle them in a batch manner.
Popularity: Continuous updates, requiring real-time handling.

#### Solution

Event Sourcing for Schema Changes:

Use an event sourcing approach to capture schema changes as domain events. These events are stored in a dedicated event store with a low retention period (e.g., 30 days). This ensures that we retain history without bloating our system.

Incremental Data Pipelines for Freshness/Row-Counts:

Implement incremental data pipelines to capture freshness and row-counts per run. These updates can be aggregated into a periodic batch job or stored in a time-series database, like Apache Cassandra, which supports efficient range queries.

Batch Processing for Ownership Changes:

Ownship changes are rare but important. We can implement a batch processing system that runs periodically (e.g., daily) to update ownership metadata. This ensures we don’t miss any changes while keeping the update frequency reasonable.

Real-Time Streams for Popularity:

Use a real-time stream processing framework like Apache Flink or Kafka Streams to continuously track and update popularity metrics in near-real time. This allows us to react quickly to changes and maintain up-to-date information without overwhelming our systems with frequent updates.

#### First-Principles Solution

Event Store for Schema Changes:

Event Store: Use a distributed event store like Apache Kafka or Amazon Kinesis.
PLAINTEXT
Event: "SCHEMA_CHANGE"
Payload: {
  schema_id: 123,
  old_schema: ...,
  new_schema: ...
}

Time-Series Database for Freshness/Row-Counts:

Use a time-series database like Apache Cassandra to store freshness and row-counts.
PLAINTEXT
Metric: "ROW_COUNT"
Payload: {
  run_id: 456,
  table_name: "sales",
  count: 10000
}

Batch Processing for Ownership:

Use a batch processing framework like Apache Beam to handle ownership changes.
PLAINTEXT
Pipeline: "OWNERSHIP_UPDATE"
Operation: "UpdateOwnershipMetadata"
Payload: {
  user_id: 789,
  table_name: "sales",
  new_owner: "john_doe"
}

Real-Time Stream Processing for Popularity:

Use Apache Flink to continuously update popularity metrics.
PLAINTEXT
Stream: "USER_INTERACTIONS"
Operation: "UpdatePopularityMetric"
Payload: {
  user_id: 101,
  item_id: 202,
  timestamp: "2023-10-15T14:30:00Z",
  interaction_type: "view"
}

#### Closing Summary
and Common Mistakes

In summary, we have designed a system that leverages event sourcing for schema changes, incremental data pipelines for freshness/row-counts, batch processing for ownership updates, and real-time stream processing for popularity. This approach ensures we maintain the right balance between freshness and completeness without overloading our systems.

#### Common mistakes
include:

Overcomplicating the architecture by introducing too many streams or databases.
Failing to consider edge cases where schema changes might impact existing metadata.
Not properly tuning batch jobs and real-time streams for optimal performance.

**Comparison:**

| Event Type | Description               |
|------------|---------------------------|
| SCHEMA_CHANGE | Captures schema updates infrequently  |
| ROW_COUNT   | Tracks row counts per run         |
| OWNERSHIP_UPDATE | Handles ownership changes occasionally |
| USER_INTERACTIONS | Updates popularity metrics continuously |

**Code:**

```
MERMAID
Copy
graph TD;
    subgraph Event Store
        SCHEMA_CHANGE[Schema Change]
    end

    subgraph Time-Series Database
        ROW_COUNT[Row Count]
    end

    subgraph Batch Processing Framework
        OWNERSHIP_UPDATE[Ownership Update]
    end

    subgraph Real-Time Stream Processing
        USER_INTERACTIONS[User Interactions]
    end

    SCHEMA_CHANGE -->|Store in event store| Event Store
    ROW_COUNT -- Aggregated per run --> Time-Series Database
    OWNERSHIP_UPDATE -- Batch job --> Batch Processing Framework
    USER_INTERACTIONS -->|Processed by Flink| Real-Time Stream Processing
```

**Example input / output:**

```text
Input:
- Schema Change Event: {"type": "SCHEMA_CHANGE", "payload": {"schema_id": 123, "old_schema": ..., "new_schema": ...}}
- Row Count Metric: {"type": "ROW_COUNT", "payload": {"run_id": 456, "table_name": "sales", "count": 10000}}
- Ownership Update Operation: {"type": "OWNERSHIP_UPDATE", "payload": {"user_id": 789, "table_name": "sales", "new_owner": "john_doe"}}
- User Interaction Stream: {"type": "USER_INTERACTIONS", "payload": {"user_id": 101, "item_id": 202, "timestamp": "2023-10-15T14:30:00Z", "interaction_type": "view"}}

Output:
- Store schema change in event store
- Update row count metric in time-series database
- Process ownership update operation via batch job
- Update popularity metric using real-time stream processing
```

**Data:**

```yaml
# Example of an event for schema change
event:
  type: SCHEMA_CHANGE
  payload:
    schema_id: 123
    old_schema: '{"fields": [{"name": "id", "type": "int"}, {"name": "name", "type": "string"}]}'
    new_schema: '{"fields": [{"name": "id", "type": "int"}, {"name": "name", "type": "string"}, {"name": "age", "type": "int"}]}'

# Example of a row count metric
metric:
  type: ROW_COUNT
  payload:
    run_id: 456
    table_name: sales
    count: 10000

# Example of an ownership update operation
operation:
  type: OWNERSHIP_UPDATE
  payload:
    user_id: 789
    table_name: sales
    new_owner: john_doe

# Example of a real-time interaction stream
stream:
  type: USER_INTERACTIONS
  payload:
    user_id: 101
    item_id: 202
    timestamp: "2023-10-15T14:30:00Z"
    interaction_type: view
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Schema Change Event
8. Is it a frequent update?
9. Store in event store
10. Update ownership metadatavia batch job
11. Row Count Metric
12. Is it per run data?
13. Aggregate and store intime-series DB
14. Store as a regular metricupdate
15. User Interaction Stream
16. Is it real-time data?
17. Process using Apache Flink
18. Store and process in batchmode

