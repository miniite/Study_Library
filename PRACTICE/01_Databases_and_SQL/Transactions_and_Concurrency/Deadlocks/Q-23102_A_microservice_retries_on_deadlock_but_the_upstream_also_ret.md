---
type: "practice"
difficulty: "Deep"
---

# A microservice retries on deadlock but the upstream also retries, causing a retry storm; how do you make the retry strategy safe end-to-end?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Retry Strategy](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Retry_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a common issue in distributed systems where retries can lead to a "retry storm," particularly when both the microservice and its upstream service are retrying on deadlocks. The goal is to ensure that the retry strategy remains safe end-to-end, avoiding infinite loops or excessive load on the system.

Triaging the Problem

The question is fundamentally about managing retries in a distributed environment where multiple services interact through transactions. We need to ensure that:

Deadlocks are detected and resolved.
Retries do not lead to an infinite loop (retry storm).
The retry strategy is consistent across all involved services.

#### Assumptions
Microservices communicate via APIs or message queues.
Deadlock Detection: We use a mechanism like Distributed Locking or Database Transactions.
Retry Mechanism: A predefined number of retries with exponential backoff.
Upstream Service: Also implements retry logic and can detect deadlocks.

#### First Principles Solution

Identify Deadlock Conditions:

In distributed systems, deadlock detection is more complex due to the lack of a centralized transaction manager.
One approach is to use distributed locks (e.g., using Redis) or database transactions with pessimistic locking strategies.

Implement Circuit Breaker Pattern:

To prevent retries from overwhelming the system, implement a circuit breaker pattern that temporarily fails requests and then recovers after a period of time if no deadlocks are detected.

Retry Logic with Backoff Strategy:

Define a retry strategy where each attempt is separated by an exponentially increasing interval.
Limit the number of retries to avoid infinite loops (e.g., 5-10 attempts).

Service Level Agreements (SLAs):

Agree on SLAs between microservices, ensuring that both services respect the retry limits and do not exceed them.
Step-by-Step Solution

Detect Deadlocks:

Yes
No
Yes
Deadlock Detected
No Deadlock
Transaction Starts
Is Lock Requested?
Acquire Lock
Lock Acquired?
Rollback Transaction
Continue Transaction
Check for Deadlock
Abort Transaction, NotifyUpstream
Complete Transaction

Retry Logic:

If a deadlock is detected, the microservice aborts its transaction and notifies the upstream service.
The upstream service then retries the request after a backoff interval.

Circuit Breaker Pattern:

Yes
No Deadlock
Deadlock Detected
No
Request
Service Up?
Process Request
Check for Deadlock
Return Response
Fail Fast, Notify Downstream
Circuit Breaker Open
Fail All Requests Temporarily

Exponential Backoff:

Implement a backoff strategy where the time between retries increases exponentially.
PYTHON
def exponential_backoff(attempt):
    return 2 ** attempt  # Example: Exponential backoff with base 2

for attempt in range(10):  # Max 10 attempts
    if not perform_request():
        sleep(exponential_backoff(attempt))

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these strategies, we can ensure that the retry mechanism is safe end-to-end. The key is to have a well-defined circuit breaker pattern combined with exponential backoff to prevent infinite loops. Common mistakes include not setting proper limits on retries or failing to detect deadlocks correctly.

For follow-ups:

Question: How would you measure the effectiveness of your deadlock detection?
Answer: Use monitoring tools and logging to track transaction outcomes, deadlocks, and retry counts. Implement automated tests that simulate deadlock conditions.
Question: What other strategies can be used to manage retries in distributed systems?
Answer: Consider using idempotent operations, message deduplication, and circuit breaker patterns across all microservices.

**Code:**

```python
def perform_request():
    # Simulate performing a request that may result in deadlock or success
    return True  # Example of a successful operation

def exponential_backoff(attempt):
    return 2 ** attempt

for attempt in range(10):  # Max 10 attempts
    if not perform_request():
        time.sleep(exponential_backoff(attempt))
```

**Example input / output:**

```text
Input:
- Service A requests a resource from Service B.

Output:
- If deadlock is detected, both services abort their transactions and retry after backoff intervals.
- Circuit breaker opens to prevent further retries for a period.
```

**Flow:**

1. Yes
2. No Deadlock
3. Deadlock Detected
4. No
5. Request
6. Service Up?
7. Process Request
8. Check for Deadlock
9. Return Response
10. Abort Transaction, NotifyUpstream
11. Circuit Breaker Open
12. Fail All Requests Temporarily

