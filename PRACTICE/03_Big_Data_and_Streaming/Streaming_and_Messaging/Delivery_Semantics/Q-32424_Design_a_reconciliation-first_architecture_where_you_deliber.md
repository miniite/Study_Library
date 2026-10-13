---
type: "practice"
difficulty: "Expert"
---

# Design a reconciliation-first architecture where you deliberately accept at-least-once and duplicates, and rely on periodic reconciliation to achieve eventual correctness. When is this preferable to inline exactly-once, and what are its failure modes?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Exactly Once](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system with message delivery semantics, there's often a trade-off between achieving exactly-once delivery versus accepting at-least-once delivery with the benefit of eventual correctness through periodic reconciliation. This approach is preferable when exact order or duplicate messages are less critical than operational simplicity and resource efficiency.

#### Triage
Understanding the Question

The question seeks an architectural design where we tolerate duplicates and at-least-once delivery in exchange for leveraging periodic reconciliation to ensure eventual correctness. The interviewer is probing your understanding of message semantics, trade-offs between different approaches, and how to handle failure modes effectively.

Key Assumptions:
The system can tolerate occasional out-of-order or duplicate processing.
Periodic reconciliation can be performed without significant resource constraints.
The application logic allows for graceful handling of duplicates and reprocessing.

#### First-Principles Solution
Step 1: Understanding the Semantics

At-least-once delivery ensures that messages are delivered at least once, but it doesn't guarantee exactly one time. This can lead to potential issues like duplicate processing, which we will handle via reconciliation mechanisms.

Step 2: Designing for Periodic Reconciliation
Message Storage: Use a reliable storage system (e.g., database) to track message identifiers and ensure they are processed only once.
Dead Letter Queue (DLQ): Implement a DLQ to capture messages that could not be processed successfully in the first attempt. This helps in debugging and ensuring no data loss.
Step 3: Implementing Reconciliation Logic
Periodic Checkpoint: Schedule periodic tasks (e.g., hourly) to check for any unprocessed or duplicated messages.
Retrying Mechanism: For messages that were not processed due to transient errors, retry them with appropriate backoff strategies.
Duplicate Detection: Utilize hash-based checksums or unique identifiers to detect and discard duplicates.
Step 4: Handling Failure Modes
Message Processing Failures: Ensure robust error handling in message processors. Log errors and mark messages for reprocessing.
Resource Constraints: Monitor system resources (CPU, memory) during reconciliation tasks to avoid impacting regular operations.
Concurrency Control: Use appropriate locking mechanisms or distributed coordination services like Redis to prevent race conditions.
Step 5: Monitoring and Logging
Metrics: Track message processing metrics such as throughput, latency, and error rates.
Logging: Maintain logs for failed messages and reconciliation activities for troubleshooting.

#### Closing Summary

This architecture leverages the simplicity of at-least-once delivery while ensuring eventual correctness through periodic reconciliation. It is particularly suitable in scenarios where exact order is less critical but operational efficiency is paramount. The primary failure modes include resource constraints during reconciliation, undetected duplicates, and transient processing failures that can lead to non-idempotent actions.

#### Common Mistakes
and Follow-Ups
Overlooking Duplicate Processing: Failing to implement effective duplicate detection mechanisms.
Ignoring Concurrency Issues: Not using proper concurrency control leading to race conditions or data inconsistencies.
Neglecting Performance Considerations: Overlooking the impact of periodic reconciliation on system performance during peak loads.

**Comparison:**

```
Operation	At-Least-once	Exactly-once
Simplicity	High	Low
Cost	Low	High
Correctness	Eventually	Immediate
```

**Code:**

```python
# Example Python code for marking messages as processed in a database
def mark_message_as_processed(message_id):
    with db_connection.transaction():
        result = db.execute("UPDATE message_table SET is_processed=true WHERE id=?", (message_id,))
        if not result.rowcount:
            raise ValueError(f"Message {message_id} not found")
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
Message to process: {"id": 1, "data": "Hello"}

Output:

PLAINTEXT
Copy
Message processed successfully.
```

**Data:**

```json
{
  "messages": [
    {"id": 1, "data": "Hello"},
    {"id": 2, "data": "World"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Message Received
6. Is Message Processed?
7. Mark as Processed
8. Retry with Backoff
9. Is Retry Successful?
10. Log Error, DLQ Insertion
11. Periodic Checkpoint:Reconciliation Logic
12. Maintain Logs forTroubleshooting

