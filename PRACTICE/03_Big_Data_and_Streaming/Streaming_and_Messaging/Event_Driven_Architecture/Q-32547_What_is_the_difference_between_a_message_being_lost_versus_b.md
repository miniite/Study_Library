---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a message being lost versus being duplicated, and which is worse for your use case?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Commit Modes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Offset_Commit_Modes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the differences between a message being lost versus being duplicated in an event-driven architecture with message queues, let’s start by summarizing the core concepts. In such systems, messages are typically processed asynchronously, ensuring that no single point of failure can disrupt the entire system. However, losing or duplicating messages can lead to different consequences depending on your application's requirements.

#### Triaging the Question

The key aspects we need to consider include:

Message Loss: This occurs when a message is not delivered to its intended consumer.
Message Duplication: This happens when the same message is processed more than once.
The relative impact of each on your use case: whether losing messages could lead to data loss or inconsistent state, and how duplicates might create redundant processing or erroneous states.

#### First-Principles Solution
Message Loss

Impact: Losing a message can result in partial or complete failure of the intended operation. This is particularly critical if the message represents a transactional or critical task.

#### Example
In an e-commerce system, if a purchase confirmation email is not sent due to message loss, it could lead to discrepancies between the database state and user expectations.
Mitigation: Implement retries with backoff strategies, use dead-letter queues (DLQs) to handle undeliverable messages, and ensure critical operations are idempotent or have compensating transactions.
Message Duplication

Impact: Duplicates can lead to redundant processing, which might be inefficient but generally does not cause catastrophic failure. However, in some cases, it could lead to incorrect state changes.

#### Example
In a social media platform, if the "like" count is incremented twice due to duplicate messages, it results in an inaccurate representation of user activity.
Mitigation: Use message deduplication techniques like sequence numbers, timestamps, or unique identifiers. Ensure that your processing logic can handle idempotent operations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the severity of losing a message versus duplicating one depends heavily on the nature and importance of the operation being performed.

Losing Messages is generally more critical as it can lead to state inconsistency or loss.
Duplicating Messages might be less impactful but still requires careful handling to avoid performance degradation.

#### Common mistakes
include:

Overlooking Message Loss: Assuming that message queues automatically ensure reliability without implementing additional checks.
Neglecting Deduplication Logic: Failing to implement mechanisms to prevent duplicate processing, which can lead to inefficiencies or erroneous states.

#### Follow-ups
could involve discussing specific scenarios where these issues might arise and how you would design the system to handle them effectively.

**Comparison:**

```
Scenario	Consequence	Mitigation Strategy
Message Loss	Inconsistent State	Dead-letter Queues, Retries with Backoff
Message Duplication	Redundant Processing	Sequence Numbers, Timestamps
```

**Code:**

```python
# Example of a simple deduplication mechanism using a set to track seen messages
seen_messages = set()

def process_message(message_id, payload):
    if message_id in seen_messages:
        print(f"Duplicate message detected for {message_id}")
        return

    # Process the message safely here...
    seen_messages.add(message_id)
```

**Example input / output:**

```text
Input:
- Message ID: 1234567890
- Payload: {"purchase_confirmation_email": {...}}

Output:
- Processed Successfully

Input:
- Message ID: 1234567890 (Duplicate)
- Payload: {"purchase_confirmation_email": {...}}

Output:
- Duplicate message detected for 1234567890
```

**Data:**

```json
{
  "purchase_confirmation_email": {
    "sender": "store@example.com",
    "recipient": "user@example.com",
    "subject": "Your Order has been Confirmed",
    "body": "Thank you for your purchase..."
  }
}
```

**Flow:**

1. Yes
2. No
3. Message Received
4. Is the message seen before?
5. Duplicate Detected
6. Process Message
7. Log Duplicate and Discard
8. Update State

