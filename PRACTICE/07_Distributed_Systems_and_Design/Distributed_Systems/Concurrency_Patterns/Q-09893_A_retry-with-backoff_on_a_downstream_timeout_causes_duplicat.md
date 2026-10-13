---
type: "practice"
difficulty: "Expert"
---

# A retry-with-backoff on a downstream timeout causes duplicate side effects (double charge). Explain the interaction between retries, timeouts, and idempotency, and design an idempotency-key store that's itself race-safe.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When implementing a retry-with-backoff strategy to handle downstream service timeouts, it’s crucial to ensure that operations are idempotent and that an idempotency key store is itself race-safe. This prevents duplicate side effects such as double charging by ensuring that each request with the same idempotency key is processed only once.

#### Triaging the Question

The core of this question revolves around understanding how retries, timeouts, and idempotency interact, especially in scenarios where a retry-with-backoff strategy might lead to race conditions. The interviewer is probing your knowledge of concurrency patterns, race condition prevention, and design principles that ensure reliability and correctness.

#### Assumptions
& First Principles:
Retry-With-Backoff Strategy: A mechanism to automatically resend requests after a timeout with increasing delays between retries.
Timeouts: Timeouts can occur due to various reasons like network issues or service unavailability at the downstream.
Idempotency: An operation that has the same effect no matter how many times it is executed, as long as the input remains unchanged. For example, a payment processing API should ensure that charging an amount twice results in the exact same outcome as doing so once.

#### Solution
Step 1: Understanding Idempotence

For idempotent operations, we must design our system to handle retries gracefully without causing unintended side effects. A common approach is to use idempotency keys—unique identifiers associated with each operation that can be used to ensure the request's uniqueness.

Step 2: Designing an Idempotency Key Store

An idempotency key store should be race-safe, meaning it must prevent concurrent operations from occurring in parallel. A race condition occurs when multiple retries simultaneously attempt to process the same request, leading to duplicate side effects.

To achieve this, we can use a distributed lock mechanism combined with an idempotency key store:

Generate Idempotency Key: When a request is made to your service, generate a unique idempotency key.
Store in Idempotency Key Store: Check if the generated key already exists in the idempotency key store:
If it doesn’t exist, proceed with processing the request and save the key in the store.
If it does exist, return a 409 Conflict status indicating that the operation was already processed.
Step 3: Implementing Retries

When implementing retries, ensure each retry uses the same idempotency key. This ensures that even if multiple retries occur due to timeouts, they are treated as one operation:

Initial Request: Generate an idempotency key and send the request.
Timeout Occurs: If a timeout occurs on the downstream service, automatically retry with the generated idempotency key after applying backoff logic (e.g., exponential backoff).
Check Key Store: Before executing any side effects, check if the operation has already been processed by querying the idempotency key store.
Process or Return Early: If the key is present and indicates the request was previously processed, return early without performing additional operations.
Step 4: Handling Concurrency

To ensure the idempotency key store itself is race-safe, use a distributed locking mechanism such as Redis's SETNX (Set if Not Exists) command or a library like Consul or Etcd for distributed locks:

Lock Before Processing: Acquire a lock before processing the request and storing the key in the idempotency key store.
Unlock After Completion: Ensure that the lock is released after the operation completes to allow other retries to proceed.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By implementing an idempotent strategy with careful handling of retries and a race-safe idempotency key store, we can prevent duplicate side effects such as double charging. Key takeaways include ensuring that each request is uniquely identifiable, using distributed locking to manage concurrency safely, and carefully handling the retry logic.

#### Common mistakes
include:

Forgetting to check for existing keys before processing.
Not properly releasing locks after processing retries.
Ignoring race conditions in idempotency key stores.

#### Follow-ups
could involve discussing how to handle transient failures, managing long-lived locks, or exploring alternative concurrency patterns like circuit breakers and bulkheads.

**Comparison:**

```
Step	Action	Outcome
1	Generate Idempotency Key	Unique key generated
2	Check Store	Key not found, proceed
3	Process Request	Side effects applied
4	Save Key in Store	Key saved
5	Subsequent Retries	Check for existing key first
```

**Code:**

```python
import time

def process_request(request):
    # Generate an idempotency key
    idempotency_key = generate_idempotency_key()

    # Check if the key already exists in the store
    with idempotency_key_store.get_lock(idempotency_key):
        if not idempotency_key_store.exists(idempotency_key):
            # Process the request and save the key in the store
            process_and_save(request, idempotency_key)
        else:
            # Return 409 Conflict status indicating the operation was already processed
            return "Already processed", 409

def generate_idempotency_key():
    # Implement a method to generate unique keys
    pass

def process_and_save(request, key):
    # Process the request and save the key in the store
    pass
```

**Example input / output:**

```text
**Input**: `process_request({"amount": 100, "userId": 123})`
- **Initial Request**: Generate key and check store.
- **Timeout Occurs**: Retried with the same key.

**Output**:
- **First Retry**: Processed request and saved key.
- **Subsequent Retries**: Returned 409 Conflict status.
```

**Data:**

```json
{
  "idempotencyKeyStore": {
    "type": "redis",
    "connectionString": "redis://localhost:6379"
  }
}
```

**Flow:**

1. Generate Idempotency Key
2. Key Exists?
3. Process and Save
4. Return 409 Conflict
5. Acquire Lock
6. Process Request
7. Save Key in Store

