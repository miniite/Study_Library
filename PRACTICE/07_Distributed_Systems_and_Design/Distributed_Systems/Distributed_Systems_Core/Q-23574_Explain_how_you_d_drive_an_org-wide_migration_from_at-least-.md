---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd drive an org-wide migration from at-least-once to effectively-exactly-once semantics without a coordinated flag-day. Which foundational primitives (idempotency keys, dedup stores, outbox) do you standardize first, and how do you migrate hundreds of consumers incrementally?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To drive an org-wide migration from at-least-once to effectively-exactly-once semantics without a coordinated flag-day, we need a phased approach that ensures minimal disruption while building foundational primitives such as idempotency keys and deduplication stores. This involves a combination of designing robust data models, implementing messaging patterns, and incrementally migrating consumers. Let's break down the process step-by-step.

Firstly, it’s crucial to understand what we're aiming for: effectively-exactly-once semantics mean that every message is processed exactly once and in order, without any duplicates or out-of-order processing. This requires a solid foundation of idempotent operations and deduplication mechanisms. The key primitives include:

Idempotency Keys: Ensures that the same input will produce the same output no matter how many times it's applied.
Dedup Stores: Manages duplicate messages to ensure they are not processed more than once.

Here’s a high-level approach:

Standardize Idempotency Keys: Introduce idempotency keys across the organization, ensuring that every unique operation is tracked and recorded.
Implement Deduplication Stores: Develop centralized or distributed stores (like Redis) to track which messages have been processed.
Introduce Outbox Pattern: Use an outbox pattern where unprocessed messages are stored before being sent to downstream systems.

To migrate incrementally, we follow these steps:

Identify Critical Services: Focus on services with the highest operational impact first (e.g., payments, order processing).
Start with Idempotency: Implement idempotency checks in new or updated consumer code.
Deploy Deduplication Stores: Gradually integrate deduplication into the system architecture to track processed messages.
Roll Out Outbox Pattern: Ensure that unprocessed messages are stored before sending, providing a buffer for retries.

This approach ensures minimal disruption while building the necessary infrastructure. The transition is managed through careful planning and incremental changes rather than a massive overhaul.

**Code:**

```python
# Example of an Idempotent Operation with an Idempotency Key

def update_order(order_id, new_status):
    # Check if order already exists in the database using the idempotency key
    existing_order = get_order_by_id(order_id)

    if not existing_order:
        raise ValueError("Order does not exist")

    # Perform the update
    existing_order.status = new_status
    save_order(existing_order)

# Example of an Outbox Pattern

def handle_message(message):
    outbox_entry = OutboxEntry(message=message, status='PENDING')
    outbox.save(outbox_entry)

    # Send message to downstream service using a retry mechanism if necessary
```

**Example input / output:**

```text
**Input:** Update an order with status `COMPLETED`
- Request to update the order ID `12345` to status `COMPLETED`.

**Output:** Order updated successfully.
- The system checks if the order exists using the idempotency key. If it does, the status is updated.
```

**Data:**

```json
{
  "order_id": "12345",
  "status": "COMPLETED"
}
```

