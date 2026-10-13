---
type: "practice"
difficulty: "Deep"
---

# Two POST requests for the same idempotency key arrive simultaneously (double-click). Walk through every interleaving and design storage so the second request returns the first's response without re-executing.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Concurrency issues can lead to race conditions in distributed systems, especially with idempotent requests like POST operations. To handle double-click scenarios effectively, we need to ensure that only the first request gets processed, while subsequent ones should return the result of the initial one immediately. This involves designing a robust storage mechanism and carefully managing request states.

#### Triaging the Question

The question is asking how to design a system to manage idempotent requests so that a second identical request does not trigger an unnecessary execution but instead returns the result from the first request. The key concepts here are concurrency patterns, race conditions, and idempotency.

#### Assumptions
Idempotency Key: Each POST request has a unique idempotency key.
Concurrency: Two requests arrive almost simultaneously with the same key.
Response Caching: The system needs to cache or store responses efficiently.

#### First-Principles Solution

To solve this problem, we'll design a two-step approach:

State Management: Use a distributed state management mechanism like a database or a cache to track whether the first request has been processed.
Response Caching: Cache the response from the first request so that subsequent requests can return it immediately.
Step 1: State Management

We need a reliable way to record and check the processing status of each idempotency key. This can be done using a database table or an in-memory cache with strong consistency guarantees (like Redis).

Table Schema:
SQL
CREATE TABLE RequestStates (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  idempotencyKey VARCHAR(255) NOT NULL UNIQUE,
  processed BOOLEAN DEFAULT FALSE,
  response JSON NULL,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Step 2: Response Caching

Store the responses from successfully executed requests in a cache. This ensures that future requests can quickly return cached results.

Cache Schema:
JSON
{
  "key": "<idempotencyKey>",
  "response": <JSON Response>,
  "timestamp": <Timestamp>
}

#### Implementation
Database Interaction for Processing State
Check Existing Request: Before processing, check if the request with a given idempotency key already exists and has been processed.
Update Status: If not processed, mark it as processed and store the response in both database and cache.
Yes
No
Request Received
Is Request Processed?
Return Cached Response
Process Request -> StoreResult in DB & Cache
Return Stored Response fromCache

#### Example
Code for Processing State
PYTHON
def handle_request(idempotency_key):
    # Check if request is already processed
    existing_request = db.get_request_by_idempotency_key(idempotency_key)

    if existing_request and not existing_request.processed:
        return existing_request.response

    # Process the new request
    response = process_request(...)  # Assume this function processes the request

    # Store in database
    db.update_request_status(idempotency_key, processed=True, response=response)

    # Cache response
    cache.set(idempotency_key, response)

    return response

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary

By ensuring that we track the processing state of each request using a database table or an in-memory cache, we can efficiently handle double-click scenarios. This approach prevents unnecessary reprocessing while returning cached results quickly.

#### Common Mistakes
and Follow-Ups
Race Conditions: Ensure strong consistency mechanisms (like distributed transactions) to avoid race conditions.
Cache Validity: Consider time-to-live (TTL) for cache entries to invalidate stale responses.
Scalability: Design the storage system to scale horizontally, especially in high-traffic scenarios.
Failures and Recovery: Implement mechanisms to handle failures and ensure data consistency.
Summary

The solution involves a combination of state management and response caching to efficiently handle double-click scenarios while maintaining correctness and performance.

**Comparison:**

|id | idempotencyKey | processed | response          | createdAt               |
|---|---------------|-----------|-------------------|------------------------|
| 1 | 12345         | true      | {"id": 100, ...}   | 2023-10-01T10:00:00Z    |

**Code:**

```python
def handle_request(idempotency_key):
    existing_request = db.get_request_by_idempotency_key(idempotency_key)

    if existing_request and not existing_request.processed:
        return existing_request.response

    response = process_request(...)  # Assume this function processes the request
    db.update_request_status(idempotency_key, processed=True, response=response)
    cache.set(idempotency_key, response)

    return response
```

**Example input / output:**

```

Input:

POST /api/resource
Headers: X-Idempotency-Key: 12345

Output (First Request):

JSON
Copy
{
  "status": "created",
  "data": {"id": 100, "name": "Resource A"}
}

Input (Second Request):

POST /api/resource
Headers: X-Idempotency-Key: 12345

Output (Second Request):

JSON
Copy
{
  "status": "already processed",
  "data": {"id": 100, "name": "Resource A"}
}
```

**Data:**

```sql
CREATE TABLE RequestStates (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    idempotencyKey VARCHAR(255) NOT NULL UNIQUE,
    processed BOOLEAN DEFAULT FALSE,
    response JSON NULL,
    createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. No
2. Yes
3. Request Received
4. Is Request Processed?
5. Process Request -> StoreResult in DB & Cache
6. Return Cached Response
7. Return Stored Response fromCache

