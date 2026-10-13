---
type: "practice"
difficulty: "Deep"
---

# How would you design an idempotency layer that deduplicates retried payment requests, and what data structure and TTL policy prevents both double-charges and unbounded memory growth?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an idempotent layer for deduplicating retried payment requests, we need to ensure that any request made by a user is processed exactly once despite retries or failures. This requires a robust data structure and an effective Time-To-Live (TTL) policy to prevent both double-charges and unbounded memory growth.

#### Triaging the Question

The question is essentially asking us to design a system for handling idempotent requests, which are unique identifiers generated for each request that can be used to determine if a request has been processed before. The key aspects are:

Uniqueness of Requests: Ensure that no two identical payment requests are processed twice.
Handling Retries: Allow the same client to retry their payment without reprocessing it unnecessarily.
Memory Management: Prevent memory leaks by removing old or unnecessary entries from the system.

#### Assumptions
The input is a unique request identifier (e.g., UUID).
The output should be either "Processed" or "Already Processed."
We will use an in-memory data structure for simplicity, but we'll also consider edge cases and potential failures.
Each request has associated metadata like the amount, user ID, etc.
Step-by-Step Solution
Data Structure Selection: A hash table (or dictionary) is ideal because it provides constant-time average complexity for insertions and lookups.
TTL Policy Implementation: We need to set a TTL on entries to prevent them from occupying memory indefinitely if the system fails.
Designing the Idempotency Layer
Request Identifier Storage:
Use a hash table where each key is the request identifier (e.g., UUID).
The value can be a struct containing metadata about the request.
Handling Requests:
When a payment request comes in, check if it exists in the hash table.
If it does not exist, insert it with its metadata and return "Processed."
If it already exists, return "Already Processed."

#### Example
Implementation
PYTHON
class IdempotencyLayer:
    def __init__(self):
        self.requests = {}
        self.ttl = 3600  # TTL in seconds

    def handle_request(self, request_id: str) -> str:
        if request_id not in self.requests:
            self.requests[request_id] = {"timestamp": time.time()}
            return "Processed"
        else:
            return "Already Processed"

# Example usage
layer = IdempotencyLayer()
print(layer.handle_request("12345"))  # Returns "Processed" and stores the request.
print(layer.handle_request("12345"))  # Returns "Already Processed."
TTL Policy

To manage memory growth, we need to remove entries that are older than the defined TTL. This can be done in two ways:

Periodic Cleanup: Run a background task periodically to clean up old entries.
Graceful Removal on Insertion: When inserting a new entry, check if it is older than the TTL and remove it before insertion.

#### Example
with Periodic Cleanup
PYTHON
class IdempotencyLayerWithTTL:
    def __init__(self):
        self.requests = {}
        self.ttl = 3600  # TTL in seconds

    def handle_request(self, request_id: str) -> str:
        if request_id not in self.requests or time.time() - self.requests[request_id]["timestamp"] > self.ttl:
            self.requests.pop(request_id, None)
            self.requests[request_id] = {"timestamp": time.time()}
            return "Processed"
        else:
            return "Already Processed"

    def cleanup(self):
        current_time = time.time()
        keys_to_remove = [k for k, v in self.requests.items() if current_time - v["timestamp"] > self.ttl]
        for key in keys_to_remove:
            del self.requests[key]

# Example usage
layer_with_ttl = IdempotencyLayerWithTTL()
print(layer_with_ttl.handle_request("12345"))  # Returns "Processed" and stores the request.
time.sleep(3601)  # Wait for TTL to expire.
print(layer_with_ttl.handle_request("12345"))  # Returns "Already Processed."

#### Closing Summary

The design of an idempotent layer involves using a hash table to store unique request identifiers and their metadata. A TTL policy ensures that old entries are removed, preventing memory leaks while ensuring that no duplicate requests are processed.

#### Common Mistakes
and Follow-Ups
Forgetting TTL: Failing to implement or correctly set the TTL can lead to unbounded memory growth.
Incorrect Cleanup Logic: Not properly removing old entries during insertion or cleanup can result in data inconsistencies.
Handling Failures: Ensuring that requests are processed reliably even if the system fails is crucial. Consider implementing a durable storage layer for cases where the memory-based solution might fail.

**Code:**

```python
class IdempotencyLayer:
    def __init__(self):
        self.requests = {}
        self.ttl = 3600

    def handle_request(self, request_id: str) -> str:
        if request_id not in self.requests:
            self.requests[request_id] = {"timestamp": time.time()}
            return "Processed"
        else:
            return "Already Processed"

class IdempotencyLayerWithTTL:
    def __init__(self):
        self.requests = {}
        self.ttl = 3600

    def handle_request(self, request_id: str) -> str:
        if request_id not in self.requests or time.time() - self.requests[request_id]["timestamp"] > self.ttl:
            self.requests.pop(request_id, None)
            self.requests[request_id] = {"timestamp": time.time()}
            return "Processed"
        else:
            return "Already Processed"

    def cleanup(self):
        current_time = time.time()
        keys_to_remove = [k for k, v in self.requests.items() if current_time - v["timestamp"] > self.ttl]
        for key in keys_to_remove:
            del self.requests[key]
```

**Example input / output:**

| Input       | Output    |
|-------------|-----------|
| handle_request("12345") | Processed |
| handle_request("12345") | Already Processed |
| cleanup()   | -         |
| Input       | Output    |
|-------------|-----------|
| handle_request("12345") | Processed |
| time.sleep(3601)  |          |
| handle_request("12345") | Already Processed |

**Data:**

```json
{
  "request_id": "12345",
  "timestamp": 1672531200.0
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Receive Payment Request] --> B{Is request_id in requests?};
4. B -->|Yes| C[Return "Already Processed"];
5. B -->|No| D{Is current_time - timestamp > TTL?};
6. D -->|Yes| E[Remove request_id from requests];
7. E --> F[Insert new entry with request_id, set timestamp to current_time];
8. F --> G[Return "Processed"];
9. D -->|No| C;

