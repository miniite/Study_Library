---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether an operation belongs in the synchronous request path or should be pushed to a queue? Give the concrete criteria you'd apply during design review.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lag-based Consumer Autoscaling](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding where an operation should reside—whether in the synchronous request path or within a message queue—the key is to balance real-time responsiveness with system scalability and fault tolerance. We need to assess several factors, including latency requirements, resource constraints, and operational complexity.

The primary criteria we'll consider are:

Latency Sensitivity: Operations that require immediate feedback should remain in the synchronous path.
Resource Utilization: Long-running tasks or those consuming significant resources (like database writes) may be better off queued to avoid blocking the request handler.
Scalability and Load Distribution: Queues can help distribute load across multiple services, improving system resilience and performance.
Fault Tolerance: Operations that are critical but do not require real-time responses should have retries or failover mechanisms built into a queue.

Let's walk through an example to illustrate this:

#### Example Scenario

#### Cons
ider an e-commerce application where a user places an order. The synchronous request path includes:

Checking stock availability.
Processing payment.
Logging the transaction in the database.

However, tasks like sending confirmation emails and updating inventory levels across multiple warehouses can be handled asynchronously via message queues.

Criteria Application
Latency Sensitivity:
Stock Check: This needs to be done immediately to ensure the user gets accurate feedback about whether their order can be processed.
Payment Processing: While this is critical, it generally takes a few seconds and doesn't require real-time confirmation.
Resource Utilization:
Database Writes: Operations like updating the database with transaction details should happen immediately to maintain data integrity. However, writing to multiple databases (e.g., inventory and transactions) can be resource-intensive; queuing these writes could help manage the load better.
Scalability and Load Distribution:
Inventory Update Across Warehouses: This task involves communicating with various microservices, which might not all be available or performant at the same time. Queuing these updates ensures that even if one service is down, others can still process orders without interruption.
Fault Tolerance:
Email Confirmation: If sending confirmation emails fails, it should ideally be retried until successful rather than breaking the user experience.
Detailed Design Review

During a design review for this system, we would ask ourselves:

Are there any critical operations that must complete before returning to the client?
Can certain operations be delayed without impacting key user experiences or business processes?
What is the potential impact of queuing on overall latency and throughput?

By systematically applying these criteria, we can ensure that our design is both performant and resilient.

#### Common Mistakes
Overusing Queues: Not all tasks benefit from being queued; overly complex systems with too many queues can introduce unnecessary complexity.
Underestimating Latency Sensitivity: Underestimating the importance of immediate responses can lead to poor user experiences, especially in real-time applications like live streaming or trading platforms.
Neglecting Fault Tolerance: Ignoring retries and fallback mechanisms can leave critical operations exposed to failure.

**Code:**

```python
# Example Python function for order processing logic

def process_order(order_id):
    # Synchronous path for real-time checks
    if not check_stock_availability(order_id):
        return "Insufficient stock"

    if not process_payment(order_id):
        return "Payment failure"

    # Asynchronous queue tasks
    update_inventory_queue.put(order_id)
    log_transaction_queue.put(order_id)

def check_stock_availability(order_id):
    # Simulate a real-time check
    time.sleep(1)  # Simulate network delay
    if random.random() < 0.9:  # 90% chance of success for demo purposes
        return True
    else:
        raise Exception("Stock out")

def process_payment(order_id):
    # Simulate a real-time payment processing
    time.sleep(2)  # Simulate network delay
    if random.random() < 0.8:  # 80% chance of success for demo purposes
        return True
    else:
        raise Exception("Payment rejected")
```

**Example input / output:**

| Input Order Processing | Output Status |
|------------------------|---------------|
| `process_order(order_id)` | "Payment failure" |

**Data:**

```json
{
  "order_id": 12345,
  "stock_status": true,
  "payment_status": false,
  "log_transaction": {
    "status": true,
    "message": "Transaction logged successfully"
  },
  "update_inventory": {
    "status": false,
    "message": "Inventory update failed due to network issues"
  }
}
```

