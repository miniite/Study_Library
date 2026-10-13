---
type: "practice"
difficulty: "Deep"
---

# You inherited a synchronous checkout flow that calls payment, inventory, and email inline. Prod times out under load. How would you redesign it to be async and idempotent while keeping the user-facing latency low?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to refactor the synchronous checkout flow into an asynchronous one that ensures idempotency while maintaining acceptable user-facing latency. The goal is to decouple the components (payment, inventory, and email) so they can operate independently and asynchronously.

#### Triage

The core of the question is about redesigning a synchronous process to be asynchronous and idempotent without significantly increasing user wait time. This involves understanding the current flow, identifying bottlenecks, and planning for retries while ensuring that no updates are lost or duplicated. The key assumptions include:

The system currently has issues with timeouts during high load.
User-facing latency is a critical metric to maintain.

#### First-Principles Solution
Decoupling: Separate the payment, inventory, and email services into independent components that can be called asynchronously.
Idempotency: Ensure each service call can be retried without causing unintended side effects.
Retrying Mechanism: Implement a retry strategy to handle transient failures while avoiding infinite loops.
User Experience: Minimize the perceived latency by batching requests and using optimistic UI updates.
Step-by-Step Solution

Identify Components:

Payment Service: Processes payment transactions.
Inventory Service: Manages inventory levels.
Email Service: Sends confirmation emails to users.

Asynchronous Flow Design:

The frontend will initiate the checkout process but not block on any service calls.
Each component (payment, inventory, email) will be called asynchronously and independently.
Use a message broker like RabbitMQ or AWS SQS to handle the asynchronous communication between services.

Idempotency:

Implement idempotent operations for each service call by ensuring that the same request results in the same outcome regardless of how many times it is invoked.
For example, use UUIDs as transaction IDs to uniquely identify each checkout process and check if a specific operation has already been performed.

Retry Mechanism:

Implement exponential backoff with jitter for retries to avoid overwhelming the system during transient failures.
Use circuit breakers in the frontend to handle service outages gracefully, ensuring that only valid requests are retried.

User Experience:

Provide immediate feedback to users through an optimistic UI update (e.g., showing a "processing" state).
Update the UI with real-time status messages from the backend as each component completes its task.
Ensure that the checkout process is idempotent so that even if the user refreshes or navigates away, they do not lose their progress.

#### Closing Summary

By redesigning the system to be asynchronous and idempotent, we can improve resilience under load while maintaining a good user experience. The key steps involve decoupling services, ensuring idempotency, implementing robust retry mechanisms, and providing real-time feedback to users.

#### Common Mistakes/Follow-Ups
Mistake: Overcomplicating the solution by making each service call synchronous.

#### Follow-Up
Discussing how to handle retries without causing infinite loops (e.g., using a timeout mechanism).
Mistake: Ignoring user experience, leading to frustration and potentially lost sales.

#### Follow-Up
Explaining the importance of maintaining real-time updates and how to implement optimistic UI states.

**Comparison:**

```
Task	Service Call	Retry Mechanism
Payment	handle_payment	Exponential Backoff with Jitter
Inventory	handle_inventory	Circuit Breaker + Timeout
Email	handle_email	Circuit Breaker + Timeout
```

**Code:**

```python
# Example code snippet for initiating an asynchronous checkout flow
import uuid

def initiate_checkout(order_id):
    # Generate a unique transaction ID
    transaction_id = str(uuid.uuid4())

    # Asynchronously call payment service
    asyncio.create_task(payment_service.call(transaction_id, order_id))

    # Asynchronously call inventory service
    asyncio.create_task(inventory_service.call(transaction_id, order_id))

    # Asynchronously call email service
    asyncio.create_task(email_service.call(transaction_id, order_id))

async def handle_payment(transaction_id, order_id):
    # Payment logic here
    pass

async def handle_inventory(transaction_id, order_id):
    # Inventory logic here
    pass

async def handle_email(transaction_id, order_id):
    # Email logic here
    pass
```

**Example input / output:**

```text
Input:
Initiate Checkout(order_id="123456")

Output:
Asynchronous tasks started for payment, inventory, and email services.
Status updates will be sent back to the frontend as each service completes its task.
```

**Data:**

```json
{
  "order_id": "123456",
  "transaction_id": "abcd-efgh-ijkl-mnop"
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Initiate Checkout] --> B{Is Payment Ready?};
4. B -- Yes --> C[Update UI with "Payment Processing"]
5. B -- No --> D[Retry Payment Call]
6. D --> E{Is Retry Limit Reached?}
7. E -- No --> D
8. E -- Yes --> F[Log Failure and Notify User]
9. F --> G{Is Inventory Ready?};
10. G -- Yes --> H[Update UI with "Inventory Processing"]
11. G -- No --> I[Retry Inventory Call]
12. I --> J{Is Retry Limit Reached?}
13. J -- No --> I
14. J -- Yes --> K[Log Failure and Notify User]
15. K --> L{Is Email Ready?};
16. L -- Yes --> M[Update UI with "Email Sent"]
17. L -- No --> N[Retry Email Call]
18. N --> O{Is Retry Limit Reached?}
19. O -- No --> N
20. O -- Yes --> P[Log Failure and Notify User]

