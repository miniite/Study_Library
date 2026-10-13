---
type: "practice"
difficulty: "Deep"
---

# You must guarantee each inference request from an at-least-once queue is scored exactly once (idempotent); what dedup key structure, storage, and TTL window prevent double-scoring without unbounded memory?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that each inference request from an at-least-once queue is scored exactly once (idempotently), we need to design a system that maintains uniqueness of requests while efficiently managing storage and preventing unbounded memory usage. The core idea here is to use a combination of deduplication keys, storage mechanisms with time-to-live (TTL) windows, and efficient data structures.

The question revolves around identifying an appropriate dedup key structure, the underlying storage mechanism, and implementing a TTL window to manage requests effectively without consuming excessive memory or unbounded resources. This solution must balance performance, reliability, and resource management to ensure scalability and maintainability over time.

Let’s break down the problem step-by-step:

Identify Dedup Key Structure: The dedup key should be unique for each request while being compact enough to fit in memory efficiently.
Choose Storage Mechanism with TTL Window: Use a storage mechanism that can manage in-flight requests and expire outdated entries.
Implement Efficient Data Structures: Ensure the data structures used support quick insertions, deletions, and lookups.

#### Triaging the Question

The interviewer is probing your understanding of:

Idempotency guarantees
Deduplication techniques
Storage strategies with TTL windows

#### Trade-offs
between performance and resource management

#### Implementation
details of key data structures

Key assumptions include:

The system handles a large volume of inference requests.
We need to ensure each request is processed exactly once, avoiding duplicate scoring.

#### First Principles Solution
Dedup Key Structure

A dedup key should be unique for each request. For ML inferences, we can use the combination of the request ID and some contextual information (e.g., timestamps) as the dedup key. This ensures that even if requests have identical payloads, they are uniquely identified based on their submission time.

Storage Mechanism with TTL Window

We need a storage mechanism that supports both fast insertions and lookups while allowing us to manage memory usage efficiently through TTL windows. A combination of in-memory data structures and persistent storage can achieve this balance.

In-Memory Set: Use an in-memory set (e.g., a hash set) for quick lookups to check if the request has already been processed.
Persistent Storage with TTL: Store the dedup keys persistently in a database or key-value store that supports TTLs. This ensures requests are not lost due to crashes, while expired entries can be safely removed.
Efficient Data Structures
In-Memory Set (HashSet): For fast O(1) lookups.
Persistent Storage with TTL: Use a database like Redis or an in-memory key-value store that supports TTLs. This allows us to set an expiration time for each entry, ensuring old entries are automatically removed.

#### Implementation
Steps

Receive Request:

Extract the request ID and any contextual information.
Combine them to form the dedup key.

Check In-Memory Set:

Check if the dedup key is already in the set.
If yes, discard the request as it has been processed before.
If no, proceed with processing the request and add the dedup key to both the in-memory set and persistent storage.

Process Request:

Perform inference scoring on the request.
Update any necessary state or store results securely.

Persist Dedup Key with TTL:

Store the dedup key in a persistent database or key-value store with a short TTL (e.g., 1 hour).
This ensures that requests are not processed multiple times, but also allows for occasional cleanup of old entries.

#### Closing Summary

To ensure each inference request is scored exactly once, we use a combination of an in-memory set and persistent storage with TTLs. The in-memory set provides quick lookups, while the persistent storage manages the state durably and ensures memory efficiency through TTLs. This approach balances performance, reliability, and resource management effectively.

#### Common Mistakes/Follow-Ups
Mistake: Using only a large in-memory structure without any persistence leads to data loss upon restart.

#### Follow-Up
Discussing how to handle edge cases where the system crashes before the TTL expires (e.g., using transactional semantics or manual cleanup).
Mistake: Ignoring the need for context information in the dedup key, leading to duplicate processing of requests with identical payloads but different contexts.

#### Follow-Up
Explaining the importance of including contextual information and how it can help in distinguishing between similar requests.

**Comparison:**

```
Data Structure	Purpose	Operations
In-Memory Set	Quick lookups for duplicates	sadd, srem
Redis Store	Persistent storage with TTLs	setex, sadd, expire
```

**Code:**

```python
# Example code snippet for handling request deduplication

import time
from redis import Redis, ExpiredValueError

# Initialize Redis client with a small TTL (e.g., 1 hour)
redis_client = Redis(host='localhost', port=6379, db=0)

def handle_inference_request(request_id: str):
    # Dedup key based on request ID and timestamp
    dedup_key = f"{request_id}:{time.time()}"

    try:
        # Check in-memory set for the request
        if redis_client.sismember('inference_requests', dedup_key):
            print(f"Request {request_id} already processed.")
            return

        # Insert into in-memory set and persistent storage with TTL
        redis_client.sadd('inference_requests', dedup_key)
        try:
            # Perform inference scoring (pseudo-code for illustration)
            score = perform_inference_scoring(request_id, ...)
            print(f"Processed request {request_id} with score: {score}")
        except ExpiredValueError as e:
            print("Error processing expired request:", e)

    except Exception as e:
        print("Error handling request:", e)
```

**Example input / output:**

```
Input: request_id = 'req_12345'
Output:
PLAINTEXT
Copy
Request req_12345 already processed.
Dry-run:
Insert request into in-memory set and persistent storage.
Check if the same request is inserted again, and ensure it’s not processed twice.
```

**Data:**

```json
{
  "inference_requests": {
    "type": "Set",
    "ttl": "3600" // 1 hour
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is Dedup Key in Set?
5. Discard Request
6. Process Request
7. Persist Dedup Key with TTL
8. End

