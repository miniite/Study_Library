---
type: "practice"
difficulty: "Deep"
---

# Your FastAPI service intermittently double-processes a webhook because the provider retries before your ack; how do you make handling idempotent and how do you dedupe across replicas?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Long-running Jobs and Async Patterns](<../../../../TOOLS/Python/FastAPI/Long-running_Jobs_and_Async_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this scenario, we need to handle an issue where a FastAPI service intermittently processes webhooks twice due to retries from the webhook provider before acknowledgment. To address this, we'll implement idempotency and deduplication mechanisms that ensure each webhook is processed exactly once, even if it's retried.

#### Triage
The question is probing our understanding of handling edge cases in asynchronous processing systems, particularly dealing with retries and ensuring idempotence. We need to assume the FastAPI service might have multiple replicas running concurrently and discuss strategies to prevent double-processing.

#### First Principles Solution

Idempotency: Ensure that a webhook can be processed multiple times without changing its effect.
Deduplication Across Replicas: Implement mechanisms to track which webhooks have already been processed, ensuring no duplicate processing by any replica.

To achieve this:

Use a database or distributed cache to keep a record of processed webhooks.
Apply unique identifiers (e.g., webhook IDs) in the request payload for tracking.
Implement retries and backoff strategies on the client side.

Let's break it down step-by-step:

Database Tracking:

Store a list of processed webhooks in a database or distributed cache like Redis.
Each time a webhook is received, check if its ID is already in this list before processing.

Idempotent Processing:

Design the service to handle repeated requests gracefully without additional side effects.
Use HTTP methods and status codes properly (e.g., PUT for idempotent operations).

Replica Coordination:

Ensure all replicas use the same tracking mechanism, which could be a shared database or distributed cache.

Retry Handling:

Implement exponential backoff on the client side to avoid overwhelming the service with repeated requests.
Use HTTP headers (e.g., X-Retry-Count, If-Match) to control retries and deduplication from the client's perspective.

#### Closing Summary
By implementing a robust tracking system using a database or distributed cache, ensuring idempotency in the processing logic, and coordinating across replicas, we can effectively handle webhook retries without causing double-processing. Common mistakes include neglecting coordination between replicas and not properly handling edge cases such as partial failures during processing.

#### Common Mistakes/Follow-Ups

Forgetting to use a distributed cache like Redis for tracking processed webhooks.
Not implementing idempotent processing logic, leading to side effects on repeated requests.
Failing to coordinate across replicas, resulting in double-processing issues.
Not properly handling retries and backoff strategies from the client's perspective.

**Comparison:**

```
Webhook ID	Status
12345	Processed
67890	Not Found
11111	Processed
22222	Not Found
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
import redis

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0)

@app.post("/webhook")
async def process_webhook(webhook_id: str):
    if await redis_client.exists(webhook_id):  # Check if webhook has already been processed
        raise HTTPException(status_code=409, detail="Webhook already processed")

    # Process the webhook here (e.g., update database or perform action)
    print(f"Processing webhook {webhook_id}")

    await redis_client.setex(webhook_id, 3600, "processed")  # Store in cache for 1 hour
```

**Example input / output:**

```text
**Input:** A POST request to `/webhook` with a JSON payload containing `{"id": "12345"}`.
- First attempt: Webhook ID is not in Redis, processed and stored.
- Subsequent attempts within an hour (due to TTL setting): HTTP 409 returned.

**Output:**
- If the webhook has already been processed, a 409 error indicating "Webhook already processed".
- Otherwise, the request is processed, and success status code is returned.
```

**Data:**

```json
{
  "webhook": {
    "id": "12345",
    "data": {"key": "value"}
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Webhook Request
4. Is Webhook ID in Cache?
5. Return HTTP 409
6. Process Webhook
7. Store Webhook ID in Cache

