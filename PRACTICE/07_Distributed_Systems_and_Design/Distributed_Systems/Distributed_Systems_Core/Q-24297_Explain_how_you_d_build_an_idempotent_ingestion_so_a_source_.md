---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd build an idempotent ingestion so a source that retries and delivers the same record twice (at-least-once) doesn't double-count in the warehouse. What dedup key storage and window fit a high-throughput event stream?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Idempotent ingestion is crucial when dealing with at-least-once delivery guarantees, especially in distributed systems where retries can lead to double-counting issues. To ensure that records are processed exactly once, even if they are delivered multiple times, we need a well-designed system that captures deduplication keys and manages the processing window effectively.

#### Triaging the Question

The core of this question is about designing an idempotent ingestion mechanism for event streams where retries can lead to duplicate processing. The key aspects include:

Deduplication Key Storage: How do we uniquely identify records in a way that allows us to track and reject duplicates?
Processing Window Management: How long should we retain information about processed records to handle retries properly?

#### First-Principles Solution

To build an idempotent ingestion system, we need to ensure two main components: a robust deduplication key storage mechanism and a well-defined processing window.

Dedup Key Storage

The first step is to define what makes each record unique. Commonly used dedup keys include primary keys from the source system, combinations of fields (e.g., order ID + timestamp), or custom hash values derived from the payload. Once we have a reliable dedup key, we store it in a highly available and scalable storage layer.

Processing Window

A processing window is essential to manage retries within a defined timeframe. This ensures that even if a record is delivered multiple times, it is only processed once during its valid time window.

Detailed Steps

Extract Dedup Key:

From the event payload, extract a unique identifier (dedup key) such as an order ID or a combination of fields.

Store Dedup Key in Storage:

Use a distributed database like Cassandra or a message queue with built-in deduplication capabilities to store these keys.
Implement a read-write lock mechanism to ensure thread safety when checking for existing records.

Write Event to Pipeline:

Write the event to an event pipeline (e.g., Kafka) where it will be processed by multiple consumers.

Process Event in Consumer:

When a consumer receives an event, check if the dedup key already exists in storage.
If the key does not exist or is outside the processing window, process the event and update the storage to mark the key as seen.
If the key exists within the processing window, discard the event.

Manage Processing Window:

Define a sliding window for processing records. For example, if a record should only be processed once in 24 hours, set up a timer that removes old records from storage after this period.
Sample Data Schema

Here's an example of how we might define the schema for storing dedup keys in Cassandra:

YAML
# Cassandra Table Schema
CREATE TABLE event_dedup (
    order_id TEXT,
    timestamp TIMESTAMP,
    PRIMARY KEY (order_id, timestamp)
);

#### Common Mistakes
and Follow-Ups
Failing to Implement Read-Write Locks: Without proper synchronization, multiple consumers might process the same record.
Ignoring Retention Policies: Not setting up retention policies can lead to storage bloat or data loss.
Not Handling Partial Processing: If an event is partially processed before failing, the dedup key should still be marked as seen.

#### Closing Summary

Building an idempotent ingestion system requires careful consideration of how records are uniquely identified and stored. A combination of robust deduplication keys and well-defined processing windows ensures that even in distributed systems with retries, events are processed exactly once. The choice of storage and window management strategy will depend on the specific throughput requirements and failure modes of the event stream.

Self-Check

This solution covers all key aspects of idempotent ingestion and is complete.

**Comparison:**

| Dedup Key | Description                |
|-----------|----------------------------|
| order_id  | Unique identifier for each event |
| timestamp | Time when the event was received |

**Code:**

```python
# Example Python code for checking if a record should be processed

from cassandra.cluster import Cluster
import time

def is_record_processed(order_id, timestamp):
    cluster = Cluster(['127.0.0.1'])
    session = cluster.connect('event_dedup')

    # Check if the key exists in storage within the processing window
    query = "SELECT * FROM event_dedup WHERE order_id=%s AND timestamp=%s"
    rows = session.execute(query, (order_id, timestamp))

    return bool(rows.one())

def process_event(event):
    order_id = extract_order_id_from_event(event)
    timestamp = time.time()

    if not is_record_processed(order_id, timestamp):
        # Process the event
        print(f"Processing {event}")

        # Update storage to mark the key as seen
        update_dedup_storage(order_id, timestamp)
```

**Example input / output:**

```text
Input:
Event: {"orderId": "12345", "timestamp": 1672531200}

Output:
Processing {"orderId": "12345", "timestamp": 1672531200}
```

**Data:**

```yaml
# Cassandra Table Schema
CREATE TABLE event_dedup (
    order_id TEXT,
    timestamp TIMESTAMP,
    PRIMARY KEY (order_id, timestamp)
);
```

**Flow:**

1. No
2. Yes
3. Receive Event
4. Is Dedup Key Processed?
5. Process Event
6. Update Storage to Mark asSeen
7. Discard Event

