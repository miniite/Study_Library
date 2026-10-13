---
type: "practice"
difficulty: "Deep"
---

# When is at-least-once acceptable versus exactly-once for an alerting pipeline, and how do you make downstream idempotent instead?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once into Kafka](<../../../../TOOLS/Apache_Flink/Flink_Core/Exactly_Once_into_Kafka.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In stream processing pipelines like those used in AIOps, ensuring correctness often hinges on whether at-least-once or exactly-once semantics are acceptable. At-least-once is sufficient when occasional duplicate alerts can be tolerated, whereas exactly-once is crucial for maintaining data integrity and avoiding false positives. To handle this, making the downstream system idempotent becomes a key strategy.

#### Triage
The question probes your understanding of stream processing consistency guarantees in alerting pipelines, specifically distinguishing between at-least-once and exactly-once semantics, and how to make systems idempotent to achieve precise control over data processing.

#### Assumptions

We are dealing with a real-time stream processing system.
An "alerting pipeline" processes events (e.g., logs, metrics) to generate alerts based on defined conditions.
Idempotency means that any operation can be performed multiple times without changing the result beyond the first execution.

#### First-principles solution

At-least-once vs Exactly-once: At-least-once is acceptable when occasional duplicates are tolerable, as it ensures all events will eventually be processed, albeit possibly duplicated. Exactly-once requires a mechanism to ensure no events are missed and no events are processed more than once.
Making Downstream Idempotent:
Identify Critical Sections: Determine which operations in the downstream system can tolerate duplicates (e.g., logging) versus those that must be atomic.
Idempotent Handling of Events: For stateful operations, ensure idempotency by using unique identifiers or timestamps to track event processing. For example, if an event is processed multiple times due to a duplicate, the downstream system should only act on it once based on these identifiers.
Transaction Control: Use transactional boundaries around critical sections of code where exactness matters (e.g., database updates).

#### Closing summary
Ensuring at-least-once or exactly-once processing in an alerting pipeline depends on the specific requirements and the nature of downstream systems. By making downstream operations idempotent, we can achieve precise control over data processing without requiring complex exactly-once semantics everywhere.

#### Common mistakes/follow-ups

Not understanding the trade-offs between at-least-once and exactly-once.
Failing to properly distinguish which parts of a system need idempotency.
Overcomplicating solutions by implementing exactly-once guarantees where they are not necessary, leading to unnecessary complexity and performance overhead.

**Comparison:**

```
Event ID	Processed
12345	true
67890	false
```

**Code:**

```python
# Example Python code snippet for making an operation idempotent
def process_event(event_id, event_data):
    if should_process_event(event_id):  # Check if the event has been processed before
        return
    # Perform stateful operations
    update_database(event_data)
    log_event("Processed", event_id)

# Example function to check for duplicate events
def should_process_event(event_id):
    existing_records = get_existing_records(event_id)  # Retrieve records from a database or cache
    if not existing_records:
        return True
    return False
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "event_id": "12345",
  "event_data": {
    "timestamp": 1672531200,
    "value": 9.5,
    "metric_type": "cpu_usage"
  }
}

Output:

JSON
Copy
{
  "log_message": "Processed event with id: 12345",
  "status": true
}
```

**Data:**

```json
{
  "event": {
    "id": "12345",
    "data": {
      "timestamp": 1672531200,
      "value": 9.5,
      "metric_type": "cpu_usage"
    }
  },
  "existing_records": [
    { "event_id": "12345", "processed": true },
    { "event_id": "67890", "processed": false }
  ]
}
```

**Flow:**

1. No
2. Event Received
3. Is event processed?
4. Process Event
5. C

