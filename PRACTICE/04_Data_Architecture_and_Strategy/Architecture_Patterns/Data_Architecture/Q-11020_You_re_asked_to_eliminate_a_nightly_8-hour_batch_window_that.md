---
type: "practice"
difficulty: "Deep"
---

# You're asked to eliminate a nightly 8-hour batch window that no longer fits the business's global operating hours. Walk me through converting a batch-dependent warehouse to continuously-loaded micro-batches or streaming, and where downstream consumers' assumptions about 'a complete day' break.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of eliminating an 8-hour nightly batch window by transitioning from a batch-dependent warehouse to a continuously-loaded micro-batch or streaming architecture, we need to rearchitect our data pipeline and carefully consider how downstream consumers handle 'complete day' data. This involves understanding the current system's constraints, identifying new challenges, and implementing robust solutions.

#### Triaging the Question

The interviewer is probing your ability to:

Transition from a batch processing model to a continuous micro-batch or streaming architecture.
Manage changes in how downstream systems process and consume data.
Identify potential issues with existing assumptions about 'complete day' data.
Ensure minimal disruption during the transition.

#### Assumptions
The current system uses a nightly batch window of 8 hours (e.g., from midnight to 8 AM) for loading data into a warehouse.
Downstream consumers rely on this batch processing model, expecting complete and consistent daily data.
We aim to maintain or improve the quality and timeliness of data while reducing operational constraints.

#### Solution
Step 1: Assess Current System

The first step is to understand the current system's architecture. We need a detailed view of:

Data sources (e.g., transaction logs, external APIs).
Batch processing logic.
Warehouse schema and storage.
Downstream consumers (e.g., BI tools, data analytics).
Step 2: Define Micro-Batches or Streams

To move to a micro-batch or streaming model, we need to define smaller time windows for data processing. For example:

Micro-Batch: Process data in 15-minute intervals.
Streaming: Process data as it arrives.
Step 3: Implement Data Pipeline Changes

We will use a combination of tools and techniques such as Apache Kafka for streaming, Apache Flink or Spark for micro-batching, and Delta Lake for storage. The key steps are:

Data Ingestion: Set up real-time ingestion from sources using tools like Kafka Connect.
Processing Layer: Use Apache Flink or Spark Streaming to process data in micro-batches (e.g., every 15 minutes).
Storage: Store processed data in a continuously-updated table, such as Delta Lake, which supports ACID transactions and incremental updates.
Step 4: Update Warehouse Schema

The warehouse schema will need to be updated to handle continuous writes:

Use append-only tables for new data.
Implement upsert logic to update existing records based on primary keys or unique identifiers.
Step 5: Modify Downstream Consumers

Downstream consumers will require updates to their queries and processing logic to handle streaming data. We can address this by:

Incremental Aggregation: Update aggregations in real-time using incremental methods.
Materialized Views: Create materialized views for common queries that update with each micro-batch or stream.
Step 6: Testing

Ensure thorough testing of the new system, particularly focusing on:

Data consistency across different time windows.
Performance and latency.
Handling edge cases (e.g., partial batches).

#### Closing Summary

Transitioning from a batch-dependent warehouse to continuously-loaded micro-batches or streaming involves a comprehensive rearchitecturing effort. The key is to carefully manage the transition, ensuring that downstream consumers can adapt to new data processing paradigms without disruption.

#### Common Mistakes
and Follow-Ups
Failing to Update Downstream Systems: Ensuring all downstream systems are updated is crucial but often overlooked.
Ignoring Data Consistency Issues: Micro-batches or streams may introduce consistency issues; proper schema design and transactional support are essential.
Overlooking Performance Impact: Real-time processing can have significant performance implications, requiring careful tuning and monitoring.

**Code:**

```
MERMAID
Copy
graph LR;
    A[Data Ingestion] --> B[Kafka Connect]
    B --> C[Processing Layer (Flink/Spark)]
    C --> D[Storage (Delta Lake)]
    D --> E[Materialized Views]
    E --> F[Downstream Consumers]
```

**Example input / output:**

```text
Input (every 15 minutes):
- At 00:00: Data [id: 1, value: 10.5]
- At 00:15: Data [id: 2, value: 20.7]

Output:
- After ingestion and processing:
  - Materialized View:
    | id | value |
    |----|-------|
    | 1  | 10.5  |
    | 2  | 20.7  |
```

**Data:**

```json
{
  "schema": {
    "id": "integer",
    "value": "double"
  },
  "data": [
    {"id": 1, "value": 10.5},
    {"id": 2, "value": 20.7}
  ]
}
```

