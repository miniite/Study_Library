---
type: "practice"
difficulty: "Deep"
---

# Design a system to safely process refunds where the same refund request may arrive multiple times and the gateway is eventually consistent. How do you guarantee at-most-one refund?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that safely processes refunds, especially in scenarios where the same refund request might arrive multiple times due to network issues or retry mechanisms, and ensuring eventual consistency with a gateway, we need to focus on preventing duplicate refunds while maintaining system robustness under various failure modes. This involves understanding the nature of eventual consistency, managing retries, and implementing reliable idempotent operations.

#### Triaging the Question

The core problem here is ensuring that each refund request results in at-most-one successful execution despite potential duplicates or retries. The key aspects to consider are:

Idempotence: Ensuring an operation can be safely repeated without changing the result.
Concurrency Control: Handling concurrent requests from multiple clients.
Gateway Consistency: Dealing with eventual consistency between the refund system and external services.

#### First Principles Solution
Idempotent Operation: Design the refund process to be idempotent, meaning that re-executing a request does not change its result beyond the first execution.
Unique Identifier: Use a unique identifier for each refund request to track its status.
Concurrency Control: Implement a mechanism to handle concurrent requests by ensuring they are processed only once.
Gateway Consistency: Ensure eventual consistency with the external gateway through retries and backoff strategies.
Step-by-Step Solution

Unique Request Identifier:

Generate a unique identifier (e.g., UUID) for each refund request.
Store this identifier in a database or cache to track its status.

Idempotent Refund Function:

PYTHON
class RefundService:
    def process_refund(self, order_id: str, amount: float) -> bool:
        unique_request_id = generate_unique_id()

        if self.is_request_existing(unique_request_id):
            return False  # Request already processed

        self.mark_request_as_processed(unique_request_id)

        # Process the refund
        result = self.gateway.refund(order_id, amount)
        if not result:
            self.revert_processing(unique_request_id)  # Rollback in case of failure

        return result

Concurrency Control:

Use a distributed lock or semaphore to ensure only one instance processes the same request.
PYTHON
from threading import Lock

class RefundService:
    def __init__(self):
        self.lock = Lock()

    def process_refund(self, order_id: str, amount: float) -> bool:
        with self.lock:
            # Check and process logic as above

Gateway Consistency:

Handle retries based on the gateway's eventual consistency model.
PYTHON
class RefundService:
    def refund(self, order_id: str, amount: float):
        for attempt in range(5):  # Example retry strategy
            try:
                result = self.gateway.refund(order_id, amount)
                if result:
                    return True
            except GatewayException as e:
                logging.error(f"Failed to process refund due to {e}")
                time.sleep(attempt * 2)  # Exponential backoff

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By ensuring each request is idempotent, tracking unique identifiers, handling concurrency with distributed locks, and implementing a robust retry strategy for the gateway, we can guarantee at-most-one refund.

#### Common Mistakes
Overlooking the need for idempotence in operations that might be retried.
Failing to implement proper concurrency control which could lead to duplicate processing.
Ignoring eventual consistency issues with external services.

#### Follow-Ups
How would you handle cases where the refund is partially processed due to network failures?
Can you discuss potential performance trade-offs in your solution?

**Comparison:**

```
Operation	Status
Refund	SUCCESS
Revert	ROLLBACK
Gateway	SUCCESS (with retries)
```

**Code:**

```python
class RefundService:
    def process_refund(self, order_id: str, amount: float) -> bool:
        unique_request_id = generate_unique_id()

        if self.is_request_existing(unique_request_id):
            return False  # Request already processed

        self.mark_request_as_processed(unique_request_id)

        result = self.gateway.refund(order_id, amount)
        if not result:
            self.revert_processing(unique_request_id)  # Rollback in case of failure

        return result

    def is_request_existing(self, unique_request_id: str) -> bool:
        return unique_request_id in self.request_tracker

    def mark_request_as_processed(self, unique_request_id: str):
        self.request_tracker.add(unique_request_id)

    def revert_processing(self, unique_request_id: str):
        # Logic to reverse the refund operation
        pass

class RefundService:
    def __init__(self):
        self.lock = Lock()

    def process_refund(self, order_id: str, amount: float) -> bool:
        with self.lock:
            result = self.gateway.refund(order_id, amount)
            if not result:
                self.revert_processing(unique_request_id)

        return result
```

**Example input / output:**

```

Input: RefundService.process_refund(order_id="123", amount=50.0)

Output: True if successful, otherwise False

Gateway Response Example:

JSON
Copy
{
  "status": "SUCCESS",
  "message": "Order refunded"
}
```

**Data:**

```json
{
  "request_tracker": {
    "uuid1": true,
    "uuid2": false
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Refund Request
6. Is request existing?
7. Mark as processed andprocess refund
8. Return True
9. Mark as processed, processrefund
10. Gateway Response OK?
11. Return True
12. Revert processing and returnFalse

