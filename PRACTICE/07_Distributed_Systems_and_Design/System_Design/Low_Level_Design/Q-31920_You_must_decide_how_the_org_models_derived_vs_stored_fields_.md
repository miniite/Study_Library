---
type: "practice"
difficulty: "Deep"
---

# You must decide how the org models 'derived vs stored' fields (an order total: recompute on read or store and keep in sync); walk through the consistency, performance, and audit trade-offs and set a guideline for when each is correct at scale.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to derive an "order total" field on-the-fly (recompute on read) or store it and keep in sync, we need to carefully balance consistency, performance, and auditability trade-offs at scale. This decision will significantly impact the architecture of our system. Let’s break down each aspect:

#### Cons
istency: Storing the order total ensures that it is always consistent with the underlying data, as any changes are immediately reflected in the stored field. On-the-fly calculation could lead to temporary inconsistencies if there are delays or errors during computation.
Performance: Deriving the total on read can reduce storage overhead and potentially simplify the database schema. However, it incurs a computational cost every time the order total is accessed. In contrast, storing the total reduces the computational load but requires additional storage space.
Auditability: Stored totals are easier to audit as they provide a direct record of the final value. Derived values may require more complex tracking mechanisms to ensure accuracy and traceability.

#### Triage

The question is probing our understanding of low-level design decisions in distributed systems, specifically focusing on trade-offs between consistency, performance, and auditability. We need to make an informed decision based on these considerations, especially at scale where these factors can amplify their impact significantly.

#### First-Principles Solution

Let’s start by considering a scenario with millions of orders being processed per day.

#### Cons
istency:

Storing the Order Total: Ensures that the order total is always up-to-date and consistent.
Derived on Read: May introduce latency in read operations, especially if there are frequent updates to the order details.

Performance:

Storing the Order Total: Requires more storage space but reduces the computational cost during reads.
Derived on Read: Reduces storage overhead and may be faster for read-heavy workloads, though it incurs a computation cost per access.

Auditability:

Storing the Order Total: Directly stores the computed value, making audits straightforward.
Derived on Read: Auditing requires tracking the state of all intermediate values used in the calculation and ensuring their correctness over time.
Guideline for Decision

Given these trade-offs, here’s a guideline to follow:

Use Storing the Order Total when:

The system has high read traffic.
The order total is critical for audit purposes.
Storage overhead is not a significant concern.

Derive the Order Total on Read when:

Computational cost during reads is significantly lower than storage costs.
Real-time consistency is less critical and acceptable to be slightly delayed.
The system has low read traffic or the derived value can be cached effectively.

#### Closing Summary

In summary, the decision between storing the order total and deriving it on read depends heavily on the specific requirements of your application. At scale, where performance and auditability are critical, storing the total is often a better choice despite higher storage costs. Conversely, for systems with lower read traffic or those prioritizing real-time consistency over auditability, deriving the total on read can be more efficient.

#### Common Mistakes
and Follow-Ups
Ignoring Audit Requirements: Failing to consider the need for accurate auditing may lead to issues in compliance or data integrity.
Overlooking Caching Strategies: Not implementing effective caching strategies for derived values could negate performance gains from derivation.
Inadequate Scalability Planning: Assuming that storage costs will remain constant as the system scales can lead to unexpected expenses.

**Comparison:**

```
Decision	Consistency	Performance	Auditability
Storing	High	Medium	High
Derived	Low	High	Low
```

**Code:**

```python
# Example of a simple function to derive order total on read
def get_order_total(order):
    total = sum(item['price'] * item['quantity'] for item in order['items'])
    return total

# Example of a function to store the derived value (simplified)
def save_order_total(order_id, total):
    # Store logic here
    pass
```

**Example input / output:**

```text
Input:
- Order with items and no precomputed total

Output:
- Derived order total if stored is not a concern (real-time consistency)
- Stored order total if space is expensive but crucial for auditing
```

**Data:**

```json
{
  "order": {
    "id": 123,
    "items": [
      {"name": "Product A", "price": 50.00, "quantity": 2},
      {"name": "Product B", "price": 75.00, "quantity": 1}
    ],
    "total": null
  }
}
```

**Flow:**

1. Derived on Read
2. Read Order
3. Compute Total
4. Return Total
5. Storing in DB
6. Audit Trace

