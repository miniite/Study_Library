---
type: "practice"
difficulty: "Deep"
---

# What is your strategy for exactly-once versus at-least-once delivery from ingest to lake and why does it matter for alert counting accuracy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once into Observability Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure accurate alert counting in an Observability Data Lake, we need to carefully consider our strategy between exactly-once and at-least-once delivery mechanisms from ingest to lake storage. The core challenge lies in ensuring that each event is processed only once and accurately, without duplicates or missing events. This is crucial for reliable alerting systems.

In this scenario, the question probes your understanding of data processing semantics, reliability guarantees, and trade-offs between exactly-once versus at-least-once delivery mechanisms. We will reason through the problem from first principles to arrive at a comprehensive solution that balances reliability with performance and cost.

Firstly, let's define our terms:

Exactly-once Delivery: Ensures that each event is processed exactly once.
At-least-once Delivery: Guarantees that an event is processed at least once but might be duplicated.

#### Triage
What Matters for Alert Counting Accuracy

Alert counting accuracy depends on the integrity of the data. If events are not processed correctly, we risk miscounting alerts, leading to incorrect business decisions or missed incidents. For example, if a user logs in and this event is counted twice due to duplicate processing, it could lead to an inaccurate login rate metric, affecting operational insights.

#### First-Principles Solution

To ensure exactly-once delivery, we need to leverage advanced techniques like idempotent operations, deduplication strategies, and consistent timestamps. Here's a step-by-step solution:

Event ID Generation: Generate a unique identifier for each event during ingestion. This can be done via UUIDs or sequence IDs.
Timestamps and Sequence Numbers: Include a timestamp and/or sequence number with each event to help in deduplication logic downstream.
Deduplication Logic: Implement robust deduplication checks at the lake storage level. For example, maintain a separate metadata store that tracks seen events using their identifiers.
Transaction Management: Use transactional semantics where possible, such as Kafka's exactly-once processing guarantees.
Error Handling and Retry Mechanisms: Implement retry logic with backoff strategies to handle transient failures without causing duplicates.

#### Example
Architecture
Ingest Node
Event ID Generation &Metadata Storage
Lake Storage
Deduplication Check
Alert Counting Logic
Ingest Node: This node generates a unique event identifier and metadata.
Metadata Storage: A database or a cache that stores information about seen events to prevent duplicates.
Lake Storage: The storage layer where events are stored after passing through deduplication checks.
Deduplication Check: Ensures each event is processed only once by comparing its identifier against the metadata store.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Implementing exactly-once delivery requires careful consideration of all layers in the data pipeline, from ingestion to storage and processing. Using idempotent operations and maintaining a metadata store can help achieve this. However, these solutions come with increased complexity and potential performance overhead.

#### Common mistakes
include:

Overlooking deduplication checks, leading to duplicate events.
Failing to implement transactional semantics properly.
Not handling errors and retries effectively, which can result in missed processing or duplicates.

#### Follow-up
questions might include the trade-offs between exactly-once versus at-least-once delivery, specific tools used for implementation (e.g., Kafka, Kinesis), and how to handle edge cases like network partitions during data ingestion.

**Comparison:**

```
Event ID	Timestamp	Processed?
1	2023-10-01T14:30Z	Yes
2	2023-10-01T15:00Z	No
```

**Code:**

```python
# Example of a simple deduplication check in Python

def is_duplicate(event_id, seen_events):
    return event_id in seen_events

seen_events = set()
event_id = "1234567890abcdef"
is_duplicate(event_id, seen_events)  # Returns True or False based on whether the event has been seen before
```

**Example input / output:**

```text
Input:
Event 1 (ID: 1)
Event 2 (ID: 2)

Metadata Store:
Event 1 processed

Output:
Alert Count: 1 (Event 1 counted once, Event 2 not seen before)
```

**Data:**

```json
{
    "events": [
        { "id": "1", "timestamp": "2023-10-01T14:30:00Z" },
        { "id": "2", "timestamp": "2023-10-01T15:00:00Z" },
        { "id": "3", "timestamp": "2023-10-01T16:00:00Z" },
        { "id": "4", "timestamp": "2023-10-01T17:00:00Z" }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Ingest Event
4. Is Event ID in Metadata Store?
5. Mark as Processed
6. Process Event, Add toMetadata Store
7. Update Alert Count

