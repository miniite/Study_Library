---
type: "practice"
difficulty: "Deep"
---

# Why might you move from Lambda to Kappa architecture and how does that change your denormalization maintenance model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The transition from a Lambda architecture to a Kappa (or Strimzi) architecture involves leveraging event streaming platforms like Apache Kafka, which fundamentally changes how data is processed and stored. In this shift, denormalization maintenance becomes more complex due to the need for real-time processing and eventual consistency models. Understanding these changes helps in designing robust data architectures that can handle large volumes of events with low latency.

#### Triaging the Question

The question asks about moving from a Lambda architecture to a Kappa (or Strimzi) architecture, specifically focusing on how this transition impacts denormalization maintenance. The key elements being probed are your understanding of both architectures and the differences in their approaches to data processing and storage.

#### Assumptions

You understand the basic concepts of Lambda and Kappa architectures.
You know that denormalization is a common practice for improving query performance in large-scale systems.

#### First Principles Solution
1. Understanding Lambda Architecture:
Batch Processing Layer: Processes historical data in batch mode.
Real-time Stream Layer: Handles real-time data streaming.
Serving Layer: Provides unified view of both layers to applications, ensuring consistency and up-to-date information.
2. Understanding Kappa Architecture (Strimzi):
Event-Driven Processing: Both historical and real-time data are processed in a single stream, avoiding the need for separate batch processing.
No Batch Layer: All data is stored in a stream and can be read by both batch and streaming applications.
Eventual Consistency: There’s no guaranteed consistency between different views of the same dataset, which simplifies some aspects but introduces others.
3. Denormalization Maintenance in Lambda:
In a Lambda architecture, denormalization is primarily handled in the serving layer to improve read performance.
Schema updates and data migrations need to be carefully managed across both batch and real-time layers to ensure consistency.
4. Denormalization Maintenance in Kappa (Strimzi):
Since all data is stored in a single stream, denormalization needs to be designed into the event processing logic.
Schema evolution must account for changes that occur at any point in time, making it more complex but potentially more robust.
Detailed Solution:
Batch Processing Changes:

In Lambda, batch processing handles historical data, which is often used for long-running queries or analytics. With Kappa, this layer disappears, and all data is stored in a stream. This means that denormalization logic must be embedded directly into the event streaming process.

Real-time Data Handling:

Kappa’s real-time data handling requires more complex schema evolution strategies because you need to ensure that your state machines (or processing pipelines) handle changes correctly without disrupting ongoing operations. For example, if you add a new field or change the structure of existing events, this needs to be propagated across all consumers.

Serving Layer Reconsideration:

In Lambda, the serving layer provides a unified view by merging batch and real-time results. In Kappa, the serving layer can directly access the stream, but maintaining consistency between different views (e.g., historical vs. current) becomes more challenging due to eventual consistency.

#### Closing Summary

Transitioning from a Lambda to a Kappa architecture shifts the responsibility of denormalization maintenance entirely into event processing logic. This requires careful design and robust schema management practices to ensure that your system can handle both real-time and historical data efficiently while maintaining performance and correctness.

#### Common Mistakes/Follow-ups
Not Accounting for Eventual Consistency: Failing to understand that Kappa operates under eventual consistency principles, leading to potential data inconsistencies.
Overlooking Schema Evolution Strategies: Not having a clear plan for how schema changes will be propagated across all event consumers.
Ignoring Real-time Data Latency: Assuming that all real-time data is available instantly when it may not always be.

**Comparison:**

|Lambda Architecture| Kappa Architecture|
|---|---|
|Batch Layer| No Batch Layer, Event-Driven Processing Only|
|Real-time Stream Layer| Real-time Stream Layer, All Data|
|Serving Layer| Serving Layer Directly Accesses Streams|

**Code:**

```python
# Example of handling schema evolution in Kappa architecture using Kafka and Apache Pulsar

def process_event(event):
    # Check if event includes new field
    if 'new_field' in event:
        # Handle the addition of a new field, ensuring consistency across consumers
        pass

    # Process existing fields as normal
    do_processing(event)
```

**Example input / output:**

```text
Input:
{
  "event": {
    "timestamp": "2023-10-01T12:00:00Z",
    "old_field": "value",
    "new_field": "updated_value"
  }
}

Output:
{
  "processed_event": {
    "timestamp": "2023-10-01T12:00:00Z",
    "old_field_processed": true,
    "new_field_processed": false, # Indicates that the new field was not processed due to its novelty
  }
}
```

**Data:**

```json
{
  "event": {
    "timestamp": "2023-10-01T12:00:00Z",
    "old_field": "value",
    "new_field": "updated_value"
  }
}
```

**Flow:**

1. Event Generation
2. Batch Processing
3. Stream Processing
4. Kappa Serving Layer
5. Lambda Serving Layer
6. Unified View for Applications

