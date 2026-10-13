---
type: "practice"
difficulty: "Deep"
---

# Explain the transactional-outbox pattern for a service that must both update its DB and emit an event to your data pipeline. Why is dual-writing to DB and Kafka unsafe and how does the CDC relay avoid losing or duplicating events?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Dead Letter Queues and Outbox](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Dead_Letter_Queues_and_Outbox.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The transactional-outbox pattern is a critical technique in distributed systems, especially when dealing with services that need to update their database and emit an event to a data pipeline like Kafka simultaneously. In this context, a "safe" operation means ensuring that either both the database update and the event emission succeed or neither does. Dual-writing directly to the database and Kafka is inherently unsafe because it can lead to inconsistencies where one of these operations might fail independently.

The CDC (Change Data Capture) relay avoids these issues by using a decoupled approach. Instead of writing directly to both systems, it writes changes to an outbox store that then asynchronously emits events to the data pipeline. This ensures consistency and reliability by leveraging a reliable write mechanism before propagating the event.

Let's break down why dual-writing is unsafe and how CDC relay avoids these pitfalls:

**Comparison:**

```
Approach	Consistency Guarantee
Dual-Writing	No explicit guarantee; may lead to inconsistent states
CDC Relay (Outbox Pattern)	Guaranteed consistency and durability
```

**Code:**

```python
# Example of unsafe dual-writing approach (simplified)
def update_db_and_emit_event(user_id, new_status):
    try:
        db.update_user_status(user_id, new_status)  # Database write
        kafka_producer.send("user_events", {"user_id": user_id, "status": new_status})  # Kafka emit
    except Exception as e:
        print(f"Error: {e}")

This approach can fail in two ways: either the database update succeeds but the Kafka send fails (or vice versa), or both operations fail. The outbox pattern addresses these risks.
```

**Example input / output:**

```plaintext
Input:
- Database update request to change user status
Output:
- Outbox entry written to persistent storage

Relay Process:
1. Write outbox entry -> Persistent Storage (e.g., journal or queue)
2. Poll for outbox entries and emit them to Kafka asynchronously

This ensures that the event is recorded in a durable manner before being sent, avoiding race conditions.
```

**Data:**

```json
{
  "outbox_entry": {
    "id": "12345",
    "type": "user_event",
    "data": {"user_id": "67890", "status": "active"},
    "timestamp": "2023-10-01T12:00:00Z"
  }
}

The outbox entry is a record of the intended event, stored persistently.
```

**Flow:**

1. Database Update Request
2. Write Outbox Entry
3. Persist in Storage
4. Periodic Polling for OutboxEntries
5. Emit to Kafka
6. This flowchart illustrates the process of writing changes to an outbox and then emitting them asynchronously.

