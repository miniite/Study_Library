---
type: "practice"
difficulty: "Expert"
---

# How do you handle GDPR/DPDP deletion (right to be forgotten) when a record lives simultaneously in an immutable streaming log and derived lakehouse tables? What's your unified deletion propagation design?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of GDPR/DPDP deletion when records live both in an immutable streaming log and derived lakehouse tables, we need to ensure a robust and efficient deletion mechanism that can propagate the deletion request across all affected data stores. This involves understanding the nature of the data, the schema, and the operational workflow, then designing a unified propagation strategy.

The key idea is to create a unified propagation system that ensures consistency and integrity across both the streaming log and the derived tables. We will leverage change event tracking, versioning, and a central coordination mechanism for this purpose.

#### Triage

This question primarily seeks an understanding of how you can handle deletions in a hybrid data architecture where data is stored in both immutable logs (e.g., Apache Kafka) and mutable tables (e.g., Delta Lake). The interviewer is probing your knowledge of event-driven architectures, data versioning, and the principles of change tracking. We need to assume that the data is structured and semi-structured, with a schema that allows for easy identification of relevant records.

#### First Principles Solution
Identify the Deletion Request: Start by identifying when a deletion request comes in. This could be from an external system or as part of a data governance process.
Event Tracking and Versioning: Maintain event tracking and versioning mechanisms to ensure that each record can be uniquely identified, even after it has been deleted.
Central Coordination Mechanism: Implement a central coordination mechanism (e.g., a message broker) to manage the propagation of deletion requests across all relevant data stores.
Immutable Streaming Log Processing: Process deletions in the streaming log by marking records as "deleted" or removing them entirely, depending on the design.
Derived Tables Update: Ensure that any derived tables are updated to reflect the deletion, possibly through a batch processing job.
Step-by-Step Solution

Identify Deletion Requests:

A deletion request could be in the form of an external API call or a scheduled task within your data governance system.

Event Tracking and Versioning:

Maintain a versioned log of all changes to each record, including deletions.
Use a unique identifier (UUID) for each record to track its lineage and ensure that the deletion is applied consistently across all stores.

Central Coordination Mechanism:

Use a message broker like Apache Kafka or AWS SNS/SQS to centralize the handling of deletion requests.
The coordinator system should be able to route deletion events to the appropriate data stores based on the record's metadata (e.g., stream topic, table name).

Immutable Streaming Log Processing:

For immutable logs like Apache Kafka, mark records as "deleted" by adding a flag or timestamp indicating when and why they were deleted.
Alternatively, if the log supports it, you can remove the record entirely.

Derived Tables Update:

Use batch processing jobs to update derived tables with the deletion information.
Ensure that these updates are idempotent and atomic to prevent data inconsistencies.

#### Closing Summary

By implementing a unified propagation design, we ensure that GDPR/DPDP deletions can be handled consistently across both immutable logs and derived tables. This involves using event tracking, versioning, and a central coordination mechanism to manage the deletion process effectively.

#### Common Mistakes/Follow-Ups
Ignoring Derived Tables: A common mistake is not considering how derived tables are affected by deletions in the immutable log.
Handling Concurrency Issues: Ensure that your system can handle concurrent deletion requests without causing conflicts or data corruption.
Synchronization Delay: Be aware of potential delays in updating derived tables, as these may impact real-time analytics.

**Comparison:**

| Record ID | Stream Topic | Partition | Offset |
|-----------|--------------|-----------|--------|
| 12345     | user_events  | 0         | 100    |
| 67890     | transaction_logs | 1      | 200    |

**Code:**

```python
# Example code to mark a record as deleted in an immutable log using Kafka
from kafka import KafkaProducer

def mark_record_as_deleted(topic, partition, offset, deletion_id):
    producer = KafkaProducer(bootstrap_servers=['localhost:9092'])
    delete_message = f"DELETE {deletion_id}"
    producer.send(topic, key=str(deletion_id), value=delete_message.encode('utf-8'), partition=int(partition), offset=int(offset))
```

**Example input / output:**

```text
Input:
- Deletion request for user with ID '12345' received.

Output:
- Kafka message sent to topic 'user_events' with offset 100 and partition 0.
- Record '67890' marked as deleted in transaction logs.
```

**Data:**

```json
{
  "records": [
    {
      "id": "12345",
      "stream_topic": "user_events",
      "offset": 100,
      "partition": 0
    },
    {
      "id": "67890",
      "stream_topic": "transaction_logs",
      "offset": 200,
      "partition": 1
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Deletion Request
4. Is the record in an immutablelog?
5. Kafka message sent to topic
6. Mark record as deleted inderived table
7. Maintain versioned logs fortracking
8. Update derived tables withbatch processing job
9. Central CoordinationMechanism routes request

