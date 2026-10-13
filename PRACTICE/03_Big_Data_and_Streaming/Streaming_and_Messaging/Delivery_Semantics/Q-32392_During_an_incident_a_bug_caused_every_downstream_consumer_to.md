---
type: "practice"
difficulty: "Expert"
---

# During an incident, a bug caused every downstream consumer to reprocess 12 hours of a topic, double-charging customers. Reconstruct the retro: what layered defenses (idempotency, dedup, reconciliation) should have contained the blast radius?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconstruct this incident and understand how to prevent such issues in the future, we need to focus on message delivery semantics, specifically ensuring idempotency and implementing deduplication mechanisms. The goal is to contain the blast radius of data reprocessing by leveraging retries, idempotent operations, and reconciliation strategies.

#### Triaging the Question

The core issue here involves a bug that caused every downstream consumer to process 12 hours of a topic multiple times, leading to double-charging customers. We need to analyze this from an architectural standpoint, considering layered defenses such as idempotency, deduplication, and reconciliation mechanisms.

#### Assumptions
and Context
The system uses message queues (e.g., Kafka) for processing.

#### Cons
umers are stateful, meaning they can maintain state between messages.
Each message has a unique identifier.
The bug affected the consumer logic, causing it to reprocess messages incorrectly or multiple times.

#### First-Principles Solution

Idempotency at the Consumer Level:

Ensure that each message operation is idempotent, meaning performing an operation multiple times on the same state should not lead to unexpected side effects.

#### Example
If a payment processing system sends a payment request, it should be able to retry this request without causing additional charges.

Deduplication Mechanism:

Implement a deduplication layer that tracks messages as they are processed. This can be achieved using message offsets or unique identifiers.

#### Example
Kafka consumers can maintain a set of seen offsets or IDs in memory or use a database to track processed messages.

Reconciliation Layer:

A reconciliation layer can help identify and correct discrepancies after an incident, ensuring that no data is lost or incorrectly processed.

#### Example
A periodic audit of the state against expected outcomes can be implemented.
Step-by-Step Solution

Idempotency Implementation:

PYTHON
def process_payment(payment_id):
    # Check if payment has already been processed
    if not Payment.objects.filter(id=payment_id).exists():
        # Process the payment
        # Save the state to ensure it's idempotent
        save_state(payment_id)
        return True
    return False

Deduplication Implementation:

PYTHON
def consume_message(message):
    message_id = message['id']
    if message_id not in processed_messages:
        # Process the message and mark it as seen
        process_and_store(message)
        processed_messages.add(message_id)
        return True
    return False

Reconciliation Implementation:

PYTHON
def reconcile_processed_data():
    expected = set(range(1, 10000))  # Example range of messages
    actual = set(Payment.objects.values_list('id', flat=True))
    missing = expected - actual
    for id in missing:
        # Re-process the missing data
        process_payment(id)

#### Closing Summary
and Common Mistakes

In summary, to mitigate such incidents, it's crucial to implement idempotency at the consumer level, ensure deduplication mechanisms are in place, and set up a reconciliation layer for post-incident audits. These layers would have contained the blast radius by ensuring that messages were processed only once and by identifying any discrepancies after an incident.

#### Common Mistakes

Failing to Make Operations Idempotent: Without ensuring idempotency, operations can be retried multiple times, leading to unintended side effects.
Lack of Deduplication Mechanisms: Messages can be processed multiple times if there's no tracking mechanism in place.
Insufficient Reconciliation: Post-incident audits are crucial but often overlooked, leading to undetected issues.

**Comparison:**

| Operation | Description |
|-----------|-------------|
| `process_payment` | Checks if a payment has already been processed to ensure idempotency. |
| `consume_message` | Tracks message processing using deduplication mechanisms. |
| `reconcile_processed_data` | Audits the state of processed data to correct any discrepancies post-incident. |

**Code:**

```python
def process_payment(payment_id):
    # Check if payment has already been processed
    if not Payment.objects.filter(id=payment_id).exists():
        # Process the payment
        # Save the state to ensure it's idempotent
        save_state(payment_id)
        return True
    return False

def consume_message(message):
    message_id = message['id']
    if message_id not in processed_messages:
        # Process the message and mark it as seen
        process_and_store(message)
        processed_messages.add(message_id)
        return True
    return False

def reconcile_processed_data():
    expected = set(range(1, 10000))  # Example range of messages
    actual = set(Payment.objects.values_list('id', flat=True))
    missing = expected - actual
    for id in missing:
        # Re-process the missing data
        process_payment(id)
```

**Example input / output:**

```text
Input:
- A bug causes all consumers to reprocess messages for a 12-hour period.

Output:
- Idempotent operations ensure that payment processing does not result in double-charging.
- Deduplication tracks processed messages, preventing redundant processing.
- Reconciliation audits identify and correct any discrepancies after an incident.
```

**Data:**

```json
{
  "payments": [
    {"id": 1, "amount": 50.0},
    {"id": 2, "amount": 75.0},
    {"id": 3, "amount": 100.0}
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Message Received
6. Is Message Processed?
7. Mark as Seen
8. Process Message
9. Is Payment Id Exists?
10. Skip Processing
11. Save State, Process Payment
12. Reconciliation Layer IdentifiesDiscrepancies
13. Correct Discrepancies ifNeeded

