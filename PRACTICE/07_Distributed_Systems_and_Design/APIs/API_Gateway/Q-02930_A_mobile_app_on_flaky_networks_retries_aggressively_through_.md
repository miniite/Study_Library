---
type: "practice"
difficulty: "Deep"
---

# A mobile app on flaky networks retries aggressively through the gateway; how do you design gateway behavior (idempotency, dedupe, backpressure) to be resilient to unreliable clients?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Resilience to Dependencies](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Resilience_to_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API Gateway to handle a mobile app that retries aggressively due to flaky network conditions, the key is to ensure idempotency, deduplication, and backpressure mechanisms are in place to maintain system resilience and prevent overwhelming the backend services. Let's break this down from first principles.

Firstly, idempotency ensures that any API call can be safely retried without causing side effects or altering state improperly. For our gateway, we need to ensure that if a request fails (e.g., due to network issues), retrying the same request will not result in duplicate actions being performed on the backend.

Secondly, deduplication is crucial to prevent the gateway from unnecessarily forwarding multiple identical requests to the backend, which can lead to excessive load and unnecessary processing. This mechanism should be robust enough to handle retries without causing duplicated work.

Lastly, backpressure is necessary to manage the rate of incoming requests to the gateway, ensuring that it does not overwhelm the backend services when clients (like mobile apps) retry aggressively due to network issues or other factors.

Now, let's design these mechanisms step by step:

Idempotency Mechanism:

Introduce a unique identifier for each request in the API Gateway.
Store this identifier along with the request payload temporarily.
If a request is received with an identical identifier and payload, treat it as a retry and do not forward it to the backend.

Deduplication Mechanism:

Use a cache (e.g., Redis) to store unique identifiers and their state.
Before forwarding any request, check if the identifier already exists in the cache with an "in progress" or "success" state.
If it does, treat it as a duplicate and do not forward; mark it as processed.

Backpressure Mechanism:

Implement rate limiting on the gateway to control the number of requests forwarded to the backend per second.
Use exponential backoff with jitter for retries, where each retry is delayed by an increasing amount of time (with random jitter) to avoid overwhelming the system during bursts.

Here's a simplified flowchart using mermaid:

Yes
No
In Progress | Duplicate
Not Duplicate
Yes
No
Receive Request
Is Idempotent?
Forward to Backend
Store as Retry
Process Request
Check Cache
Mark as Processed, Do NotForward
Forward to Backend
Rate Limit Exceeded?
Backoff with Jitter

**Comparison:**

| Request ID | State         | Timestamp       |
|------------|---------------|-----------------|
| request1234567890 | in_progress   | 2023-10-01T12:00:00Z |
| requestABCDEF1234 | success       | 2023-10-01T12:05:00Z |

**Code:**

```python
# Example of idempotency check and handling in a gateway API
def handle_request(request, cache):
    # Generate unique ID for the request
    request_id = generate_unique_id(request)

    if is_idempotent(cache.get(request_id)):
        store_request_as_retry(cache, request_id)
    else:
        forward_to_backend(request, cache)

def is_idempotent(entry):
    return entry and entry['state'] in ['in_progress', 'success']

def store_request_as_retry(cache, request_id):
    # Store the request with a retry state
    cache.set(request_id, {'state': 'retry'})

def forward_to_backend(request, cache):
    # Forward to backend if not a duplicate
    if not is_duplicate(cache.get(request_id)):
        send_request_to_backend(request)
```

**Example input / output:**

```text
Input:
{
  "method": "GET",
  "path": "/api/v1/data",
  "headers": {"Content-Type": "application/json"},
  "body": {}
}

Cache Before Processing:
{
  "request1234567890": {
    "state": "in_progress",
    "timestamp": "2023-10-01T12:00:00Z"
  }
}

Cache After Processing (No Duplication):
{
  "request1234567890": {
    "state": "success",
    "timestamp": "2023-10-01T12:00:00Z"
  }
}
```

**Data:**

```json
{
  "cache": {
    "request1234567890": {
      "state": "in_progress",
      "timestamp": "2023-10-01T12:00:00Z"
    },
    "requestABCDEF1234": {
      "state": "success",
      "timestamp": "2023-10-01T12:05:00Z"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. In Progress | Duplicate
4. Not Duplicate
5. Yes
6. Receive Request
7. Is Idempotent?
8. Forward to Backend
9. Store as Retry
10. Process Request
11. Check Cache
12. Mark as Processed, Do NotForward
13. Forward to Backend
14. Rate Limit Exceeded?
15. Backoff with Jitter

