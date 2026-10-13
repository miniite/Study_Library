---
type: "practice"
difficulty: "Deep"
---

# You're setting SLOs for a system whose correctness depends on serialized processing. How do you express "zero double-processing" as an error budget, decide the acceptable violation rate, and design the guardrails so the org doesn't trade correctness for velocity unknowingly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our system adheres to zero double-processing, we need to express "zero double-processing" as an error budget, determine the acceptable violation rate, and design guardrails that prevent any such violations without compromising velocity. This involves understanding the concurrency patterns at play, identifying potential race conditions, and setting up robust error handling mechanisms.

#### Triage
Understanding the Question

The core of this question revolves around ensuring that a system processes each request exactly once. "Zero double-processing" means no two threads or processes should handle the same input simultaneously, which can lead to data inconsistencies, duplicate work, or even failures. The interviewer is probing your ability to:

Identify Concurrency Issues: Recognize scenarios where race conditions might arise.
Define Error Budgets: Establish a framework for measuring and managing these errors.
Design Guardrails: Implement mechanisms that enforce single-threaded processing without impacting the system's velocity.

#### First-Principles Solution
Step 1: Identify Concurrency Patterns

First, we need to understand the concurrency patterns of our system. For instance, consider a scenario where multiple requests can arrive simultaneously, and each request must be processed in a serialized manner. A common pattern here is the Reader-Writer Lock or a simpler Semaphore, which ensures that only one thread processes a given resource at any time.

Step 2: Define the Error Budget

To define the error budget for "zero double-processing," we need to calculate how much failure in this regard can be tolerated without risking the integrity of our system. For example, if our SLO (Service Level Objective) is that no more than 0.1% of requests should experience double-processing:

Violation Rate: We set a maximum violation rate.
Error Budget: This is the cumulative amount of time or number of requests during which we can tolerate these violations.
Step 3: Design Guardrails

To enforce serialized processing, we can implement several guardrails:

Unique Request IDs: Assign unique identifiers to each request and use a database (e.g., Redis) to track which requests are currently being processed.
Mutex Locks: Use mutex locks or atomic operations to ensure only one thread can process a given request at any time.
Dead Letter Queue (DLQ): Implement a DLQ where any failed or improperly processed messages can be reprocessed later.

#### Closing Summary
and Common Mistakes/Follow-Ups

In conclusion, ensuring "zero double-processing" involves understanding the concurrency patterns in your system, setting error budgets based on acceptable violation rates, and designing robust guardrails to enforce single-threaded processing. Common mistakes include:

Ignoring Edge Cases: Failing to consider edge cases like request timeouts or network failures.
Overly Complicated Solutions: Implementing overly complex solutions that might introduce their own issues without significantly improving the situation.

For follow-ups, you could discuss:

How different concurrency patterns (like Read/Write locks vs. Semaphores) affect error budgets.
Strategies for managing false positives and ensuring high availability.

#### Trade-offs
between performance and correctness in distributed systems.

#### Common Mistakes/Follow-Ups
Overlooking Edge Cases: Ensure to account for scenarios like network partitions or request timeouts, which can lead to missed updates.
Underestimating Complexity: Serializing requests can introduce additional latency; make sure you're not trading correctness for performance unknowingly.
Lack of Monitoring and Feedback: Implement logging and monitoring to detect and address violations early.

**Comparison:**

| Request ID | Timestamp          |
|------------|--------------------|
| 1          | 2023-10-05T14:30:00Z |
| 2          | 2023-10-05T14:31:00Z |

**Code:**

```python
import threading
from datetime import datetime

# Example using a threading lock to ensure serialized processing
lock = threading.Lock()

def process_request(request_id):
    with lock:
        print(f"Processing {request_id} at {datetime.now()}")
        # Process the request
```

**Example input / output:**

```text
Input: [{"id": 1}, {"id": 2}]
Output:
Processing 1 at 2023-10-05 14:30:00.123
Processing 2 at 2023-10-05 14:31:00.123
```

**Data:**

```json
{
  "requests": [
    {"id": 1, "timestamp": "2023-10-05T14:30:00Z"},
    {"id": 2, "timestamp": "2023-10-05T14:31:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is Request ID in Cache?
5. Process Request
6. Update Cache
7. Store Request ID in Cache
8. Process Request

