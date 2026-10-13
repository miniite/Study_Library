---
type: "practice"
difficulty: "Expert"
---

# Design the idempotency semantics for a mutation API behind the message bus where clients supply the key. How do you handle key reuse across different payloads, and what's your response contract for a retried-but-different request?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In designing idempotency semantics for a mutation API behind a message bus, we need to ensure that each key is processed exactly once and handle key reuse carefully by tracking states in a distributed manner. The response contract should inform the client whether their request was processed successfully or if it needs to be retried.

#### Triage
The question requires us to design an idempotent mutation API that uses a unique key for each operation, ensuring at-least-once and exactly-once semantics across a message bus. Key reuse must be handled by maintaining state tracking, and the response contract should specify whether the request was processed or needs to be retried.

#### First-Principles Solution

Message Bus Integration: We will integrate our mutation API with a reliable message bus like Apache Kafka or RabbitMQ.
Key Tracking: Maintain a distributed state store (like Redis) to track keys and their processing status.
Exactly-Once Semantics: Implement exactly-once semantics by leveraging the message bus's unique ID and ensuring idempotent operations at the API level.
Response Contract: Design a response contract that informs clients about the outcome of their request, enabling them to retry if necessary.
Step-by-Step Solution:

Message Bus Setup:

Clients send mutation requests through the message bus with a unique key.
The message bus ensures at-least-once delivery by acknowledging receipt and processing messages once.

Key Tracking in Distributed State Store:

Use Redis to store keys and their status (processed/unprocessed).
On receiving a request, check if the key exists and has not been processed yet.
If the key does not exist or is unprocessed, proceed with processing the mutation.

Handling Key Reuse:

When a client retries a request with an existing key, check if it was already processed.
If the key was processed before but with different payload, ensure idempotency by checking if the operation can be safely repeated (e.g., incrementing a counter).
Maintain a state indicating that the key has been processed and is now in a retryable state.

Exactly-Once Semantics:

Use transactional semantics or compensating actions to ensure idempotency.
For example, in SQL databases, use MERGE statements for upsert operations.
In NoSQL databases, use conditional checks like UPSERT with IF NOT EXISTS.

Response Contract:

Design a RESTful API response that includes:
Success status and result if the mutation was processed.
Retry information if the request needs to be retried (e.g., key is in retry state).

#### Example
JSON Response:
JSON
{
  "status": "success",
  "result": {
    // details of the mutation if successful
  }
}
or
JSON
{
  "status": "retry",
  "key": "1234567890",
  "retry_info": {
    "reason": "Key was reused with different payload",
    "next_action": "Resend the request"
  }
}

#### Closing Summary
and Common Mistakes:

#### Common Mistakes

Ignoring key reuse without proper state tracking.
Failing to implement idempotent operations at the API level, leading to duplicate mutations.
Overlooking exactly-once semantics, causing retries that overwrite previous successful states.

#### Follow-Ups

How would you handle partial failures in distributed systems?
What are some best practices for maintaining state across different nodes in a distributed system?

**Comparison:**

```
Key	Status	Retry Info
1234567890	processed	None
0987654321	retry	{"reason": "Duplicate payload", "next_action": "Resend"}
```

**Code:**

```python
# Example code snippet to check key existence and process mutation
import redis

client = redis.Redis(host='localhost', port=6379, db=0)
key = "1234567890"

if client.exists(key):
    # Key exists, check if it needs to be retried or processed
    retry_info = client.hgetall(f"retry:{key}")
    if retry_info:
        print("Key is in retry state: ", retry_info)
    else:
        print("Key has been processed before.")
else:
    # Key does not exist, proceed with processing mutation
    result = process_mutation(key)
    client.set(key, "processed")
    client.hset(f"retry:{key}", mapping={"reason": "Duplicate payload", "next_action": "Resend"})
```

**Example input / output:**

```text
Input:
POST /api/mutation?key=1234567890&payload={...}
Headers: {"Authorization": "Bearer token"}

Output:
{
  "status": "success",
  "result": {
    // mutation result details
  }
}

Input (retried request):
POST /api/mutation?key=1234567890&payload={...}
Headers: {"Authorization": "Bearer token"}

Output (retry response):
{
  "status": "retry",
  "key": "1234567890",
  "retry_info": {
    "reason": "Duplicate payload",
    "next_action": "Resend the request"
  }
}
```

**Data:**

```yaml
# Sample Redis Data for Key Tracking
keys:
  - key: "1234567890"
    status: processed
  - key: "0987654321"
    retry_info: {"reason": "Duplicate payload", "next_action": "Resend"}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Client Sends Request
6. Message Bus AcknowledgesReceipt
7. Key Tracking in Redis
8. Key Exists?
9. Check for Retry Info
10. Retry Reason Present?
11. Inform Client to Retry
12. Process Mutation & Mark asProcessed
13. Process Mutation & Mark asProcessed

