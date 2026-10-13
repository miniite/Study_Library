---
type: "practice"
difficulty: "Expert"
---

# Design delivery semantics for a webhook-delivery product (Stripe-style) where you promise at-least-once to customers. How do you communicate the duplicate contract, sign for replay-safety, and cap retry storms against a slow customer endpoint?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Dedup Contracts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When designing delivery semantics for a webhook-delivery product that promises at-least-once delivery (i.e., ensuring messages are delivered to the endpoint at least once), we must consider several critical aspects:

Duplicate Contract: Ensuring each message is processed only once.
Replay-Safety: Protecting against duplicate processing due to retries or failures.
Retry Storms: Managing excessive retries to avoid overwhelming slow endpoints.

#### Triaging the Question

The core of this question revolves around delivering messages reliably while adhering to the at-least-once semantics without introducing duplicates, managing retries responsibly, and ensuring system stability even when endpoints are slow. The interviewer is probing your understanding of:

Delivery Semantics: At-least-once vs. exactly-once.
Concurrency Control: Managing duplicate message processing.
Retry Mechanisms: Handling failures gracefully to avoid overloading the endpoint.
System Design Trade-offs: Balancing reliability with performance and resource constraints.

#### First-Principles Solution
1. Duplicate Contract: Ensuring Each Message is Processed Only Once

To ensure that each message is processed only once, we can use a pattern called IDempotent Processing:

Message ID Generation: Generate a unique identifier (e.g., UUID) for each webhook.
Distributed Locking: Use distributed locks or leases to prevent duplicate processing. For example, store the message ID in a database with an expiration time. If a lock is acquired successfully, process the message and release the lock.
2. Replay-Safety: Protecting Against Duplicate Processing Due to Retries

To protect against duplicate processing due to retries:

Message Expiry: Set a timeout for message delivery. If a message expires before being acknowledged by the endpoint, it should be retried with a unique identifier.
Sequence Numbers or Stamping: Assign sequence numbers or timestamps to messages to detect and skip duplicates during replay.
3. Managing Retry Storms Against Slow Endpoints

To manage retry storms against slow endpoints:

Backoff Strategies: Implement exponential backoff (e.g., doubling the delay between retries). This helps in spreading out retry attempts, reducing the load on the endpoint.
Rate Limiting: Introduce rate limiting at various layers of your system to prevent overwhelming the endpoint. For example, limit the number of messages per second or batch processing requests.
4. System Design Trade-offs
Message Queueing Systems: Use a message queue like Kafka, RabbitMQ, or Amazon SQS to decouple producers and consumers. Queues can handle retries internally.
Database for State Management: Maintain state in a database to manage locks, retries, and sequence numbers.
Error Handling and Monitoring: Implement robust error handling and monitoring to detect and address issues early.

#### Closing Summary

In summary, designing an at-least-once delivery semantics system involves ensuring each message is processed only once, protecting against duplicates during replay, managing retry storms, and making strategic trade-offs in system design. Key techniques include using distributed locks, sequence numbers, exponential backoff, rate limiting, and leveraging message queues and databases for state management.

#### Common Mistakes/Follow-Ups
Not Using Exponential Backoff: Failing to implement proper backoff strategies can lead to excessive retries, causing performance issues.
Lack of Rate Limiting: Not implementing rate limiting can overwhelm slow endpoints and lead to system instability.
Ignoring Durable State Management: Forgetting to maintain durable state (locks, sequence numbers) in a database can result in data inconsistencies.

#### Follow-up
questions could include:

How would you handle scenarios where the endpoint is down for an extended period?
What measures would you take if messages are lost or not delivered due to network issues?

**Comparison:**

| Message ID | Expiration Time |
|------------|----------------|
| 0001-5e9d-4613-ba4c-2b8f2909d7eb | 2023-09-01T10:00:00Z |

**Code:**

```python
# Example of IDempotent Processing using a database with distributed locks
from sqlalchemy import create_engine, Table, MetaData
from sqlalchemy.orm import sessionmaker
import uuid

engine = create_engine('sqlite:///webhook.db')
Session = sessionmaker(bind=engine)
metadata = MetaData()

lock_table = Table('locks', metadata,
                   Column('message_id', String, primary_key=True),
                   Column('expiration_time', DateTime))

def acquire_lock(message_id: str) -> bool:
    """Acquire a lock for the message."""
    session = Session()
    try:
        lock = session.query(lock_table).filter_by(message_id=message_id).first()
        if not lock or datetime.now() > lock.expiration_time:
            new_lock = lock_table.insert().values(
                message_id=message_id,
                expiration_time=datetime.now() + timedelta(seconds=60)
            )
            session.execute(new_lock)
            return True
    except Exception as e:
        print(f"Failed to acquire lock: {e}")
    finally:
        session.close()
    return False

def process_webhook(message):
    """Process the webhook message."""
    if not acquire_lock(message.id):
        # Handle duplicate processing here
        return
    try:
        # Process the message
        pass
    finally:
        # Release the lock after successful processing
        release_lock(message.id)
```

**Example input / output:**

```text
Input:
- Webhook message with ID: 0001-5e9d-4613-ba4c-2b8f2909d7eb

Output:
- Message processed and lock released after successful processing.
- If the lock is already held or expired, a duplicate processing handler is invoked.

Dry-run:
- A message with ID 0001-5e9d-4613-ba4c-2b8f2909d7eb is sent at 2023-09-01T09:59:00Z.
- The system tries to acquire the lock and succeeds, starts processing the message.
- After successful processing, the lock is released.
```

**Data:**

```json
{
  "locks": [
    {"message_id": "0001-5e9d-4613-ba4c-2b8f2909d7eb", "expiration_time": "2023-09-01T10:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Webhook
4. Is Lock Acquired?
5. Process Message
6. Release Lock
7. Handle Duplicate Processing
8. Retry or Log

