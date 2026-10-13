---
type: "practice"
difficulty: "Deep"
---

# Your service retries on deadlock but the retry storm makes contention worse. How would you tune backoff and batch size to break the cycle?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Row Locks Two-phase Locking and Lock Footprint](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a retry storm caused by deadlocks in your database service, we need to carefully tune the retry strategy parameters—specifically, the backoff and batch size. The goal is to break the deadlock cycle while minimizing additional contention. This requires understanding how these parameters interact with the underlying locking mechanism and the concurrency control strategies.

#### Triaging the Question

The question probes our ability to understand and optimize retry mechanisms in a complex distributed system where deadlocks can occur frequently. Deadlocks are situations where two or more processes are blocked, each waiting for the other to release a resource. When retries happen too quickly (resulting in a retry storm), they exacerbate the deadlock situation by repeatedly attempting the same transactions.

#### Assumptions
and First Principles
Deadlock Detection: Detect deadlocks using a mechanism like the Banker’s algorithm or checking for cycles in the wait-for graph.
Retry Logic: The service retries operations that encounter deadlocks, which can lead to a retry storm if not managed properly.
Concurrency Control: Use appropriate locking mechanisms (e.g., optimistic vs. pessimistic) and isolation levels.
Tuning Backoff and Batch Size

To break the cycle of contention and improve system performance, we need to adjust both the backoff strategy and batch size.

1. Backoff Strategy

Backoff strategies help reduce the frequency of retries by increasing the delay between successive retry attempts. This can be implemented using exponential backoff or a combination with jitter (randomized delays).

Exponential Backoff: Start with a short initial delay, then increase it exponentially (e.g., 1s, 2s, 4s, 8s).
Jitter: Add some randomization to the backoff times to avoid synchronization of retry attempts from different services.

#### Example

PYTHON
def exponential_backoff(retries, max_delay):
    delay = min(2**retries, max_delay)
    return random.uniform(delay - (delay / 10), delay + (delay / 10))
2. Batch Size

Batch size determines how many transactions are retried together in a single attempt. A smaller batch size can reduce the impact of deadlocks on individual operations.

Small Batch Size: Process fewer transactions per retry to minimize resource contention.
Selective Retries: Only retry specific types or critical operations that are likely causing deadlocks, rather than all retries.

#### Example

PYTHON
def process_transactions(transactions, max_retries=3):
    for _ in range(max_retries):
        random.shuffle(transactions)  # Randomize order to avoid patterned conflicts
        for transaction in transactions:
            try:
                execute_transaction(transaction)
                return True
            except DeadlockError:
                continue
        sleep(exponential_backoff(_ + 1, 32))  # Adjust max_delay as needed
    return False

#### Closing Summary
and Common Mistakes

By carefully tuning the backoff strategy and batch size, we can mitigate the impact of retry storms caused by deadlocks. The key is to balance between reducing contention and avoiding infinite retries.

#### Common Mistakes

Over-reliance on Exponential Backoff: Without a maximum delay, exponential backoff can lead to uncontrolled retries.
Insufficient Jitter: Synchronous retries can exacerbate the deadlock situation by creating a rhythm that other services follow.
Large Batch Sizes: Retrying too many operations together increases overall contention and delays.

#### Follow-Ups
How would you implement deadlocking detection in your service?
What are the trade-offs between optimistic and pessimistic locking in this context?
How do you ensure that your solution is scalable with increasing traffic?

**Code:**

```python
def exponential_backoff(retries, max_delay):
    delay = min(2**retries, max_delay)
    return random.uniform(delay - (delay / 10), delay + (delay / 10))

def process_transactions(transactions, max_retries=3):
    for _ in range(max_retries):
        random.shuffle(transactions)  # Randomize order to avoid patterned conflicts
        for transaction in transactions:
            try:
                execute_transaction(transaction)
                return True
            except DeadlockError:
                continue
        sleep(exponential_backoff(_ + 1, 32))  # Adjust max_delay as needed
    return False
```

