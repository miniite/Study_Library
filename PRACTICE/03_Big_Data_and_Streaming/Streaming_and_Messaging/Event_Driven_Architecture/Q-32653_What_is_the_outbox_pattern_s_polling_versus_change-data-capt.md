---
type: "practice"
difficulty: "Expert"
---

# What is the outbox pattern's polling versus change-data-capture (CDC) approach, and what are the trade-offs?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fat vs Thin Events](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The outbox pattern and change-data-capture (CDC) are both approaches used to ensure eventual consistency in distributed systems, particularly in message queues and event-driven architectures. The outbox pattern uses polling to check for new events that need to be processed, while CDC captures changes directly from the database as they occur. Let's dive into how each approach works and their trade-offs.

#### Triage
What is the Question Really Asking?

The question asks us to compare the polling-based outbox pattern with the CDC approach in terms of their implementation details and the key trade-offs between them. The interviewer might be probing your understanding of these architectural patterns, your ability to explain their differences, and how you would choose one over the other based on specific requirements.

Step-by-Step Solution From First Principles
Outbox Pattern

Definition:

The outbox pattern involves storing events in a dedicated queue (outbox) after they are processed by the application but before they have been acknowledged as successfully sent to the downstream system.

Polling Mechanism:

This approach uses polling to check for new messages in the outbox at regular intervals. The application periodically queries this queue and processes any new events.

#### Pros

Reliability: Ensures that if a message fails to be delivered, it will eventually be retried.
Isolation from downstream failures: The application is decoupled from the state of the downstream system, which might go down or misbehave.

#### Cons

Polling can lead to unnecessary load on the database if new events are added frequently but not processed quickly enough.
Potential for delayed processing if the polling interval is too long.
Change-Data-Capture (CDC)

Definition:

CDC captures changes directly from the database, which means it monitors the database transactions and streams them to the application or event queue in real-time.

#### Implementation

Tools likeDebeziumorKafka Connect can be used for CDC, capturing the changes as they happen.

#### Pros

Real-time processing: Events are processed as soon as they occur, reducing latency.
No polling overhead: The application does not need to periodically check a queue; it is notified of events in real-time.

#### Cons

Complexity: Setting up CDC requires careful integration with the database and understanding its internal schema and transactional semantics.
Potential for data inconsistencies if the database state changes between the time of capture and processing.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, both outbox pattern polling and CDC have their strengths. The outbox pattern is simpler to implement but may introduce delays in event handling due to polling intervals. On the other hand, CDC offers real-time processing with lower latency but comes with higher complexity.

When choosing between these approaches, consider factors such as:

Latency Requirements: If immediate updates are crucial, CDC might be preferable.
Complexity of Integration: If setting up and maintaining CDC is not feasible within your team's expertise or resources, the outbox pattern might be a better choice.
Reliability Needs: The outbox pattern ensures that events are retried if they fail to be processed immediately.

#### Common Mistakes
Overlooking Polling Overhead: Failing to consider the load introduced by frequent polling can lead to performance issues.
Misunderstanding CDC Complexity: Underestimating the setup and maintenance of CDC systems, which often require a deep understanding of database internals.

#### Follow-Ups
How would you handle conflicts in event processing?
What about scenarios where the downstream system is unavailable for extended periods?

**Comparison:**

```
Polling Interval	Outbox Pattern Pros	CDC Pros
Ensures reliability	Real-time processing	No polling overhead
Isolation from downstream issues	Lower latency	Higher complexity
```

**Code:**

```python
# Example of polling-based outbox pattern
def process_outbox():
    while True:
        # Polling interval (e.g., 1 second)
        time.sleep(1)
        new_events = get_new_events_from_queue()
        if new_events:
            for event in new_events:
                process_event(event)

# Example CDC implementation using Debezium
DebeziumConsumer().start_consuming()  # This would be a hypothetical method
```

**Example input / output:**

```text
Input (CDC):
Database transaction occurs at timestamp "2023-10-01T15:00:00Z"

Output:
Debezium captures the change and sends it to an event queue.
```

**Data:**

```json
{
    "outbox": [
        {"id": 1, "event_type": "OrderCreated", "timestamp": "2023-10-01T12:00:00Z"},
        {"id": 2, "event_type": "PaymentReceived", "timestamp": "2023-10-01T14:00:00Z"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Database Transaction
4. Is CDC Enabled?
5. CDC Streams Change to EventQueue
6. Outbox Stores New Events
7. Periodically Poll Outbox forNew Events

