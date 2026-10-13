---
type: "practice"
difficulty: "Deep"
---

# You're the reviewer on a design proposing eventual consistency for a shopping-cart-to-order handoff — probe the specific double-order and lost-item failures and decide where you'd insist on a stronger guarantee and what it costs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of eventual consistency in a shopping cart to order handoff, we need to critically evaluate potential failure modes such as double orders and lost items. The specific focus is on determining where strong guarantees should be enforced, given that we are working with an architecture that prioritizes eventual consistency.

#### Triage
What We're Probing

The question centers around the trade-offs between eventual consistency and stronger guarantees in a critical system component—handing off a shopping cart to an order. The core of this discussion revolves around identifying failure scenarios where strong consistency is essential, understanding their implications, and deciding on the necessary safeguards.

#### Assumptions
System Scope: We are dealing with a distributed e-commerce platform involving multiple components such as frontend (shopping cart), backend services (inventory, payment), and database.
Eventual Consistency Model: The system uses eventual consistency to balance between performance and reliability under certain constraints.
Failure Modes: Key concerns include double ordering of items and losing track of items in the shopping cart.

#### First-Principles Solution
Double Order Failure Mode

Scenario: A user adds an item to their shopping cart, the system acknowledges this addition, but before the order is placed, a network error or some other failure causes the initial state change to be lost. The user then refreshes the page and sees the same item again, potentially adding it to the cart again.

Impact:

Increased inventory usage (double ordering).
Customer dissatisfaction due to unexpected charges.
Revenue loss for the business.

To mitigate this risk:

Strong Consistency Guarantees: Ensure that the shopping cart state is fully updated before any acknowledgment of a successful add operation is sent back to the client. This can be achieved by implementing optimistic concurrency controls or locking mechanisms in the database.
Atomic Operations: Use atomic operations at the database level to ensure that adding an item to the cart and updating the inventory are treated as a single transaction.

Cost Implications:

Increased latency due to transactional support.
Potential increase in database complexity and resource usage.
Risk of deadlocks or other concurrency issues if not managed correctly.
Lost Item Failure Mode

Scenario: During the transition from shopping cart to order, an item is lost. This can occur due to network failures, inconsistent state updates between components, or system crashes before all relevant services are updated.

Impact:

Customer receives less than expected items in their order.
Potential revenue loss for the business.
Poor customer experience leading to potential churn.

To mitigate this risk:

#### Cons
istent State Management: Ensure that every component involved in the handoff (shopping cart, inventory, payment) maintains a consistent view of the state. This can be achieved through event sourcing or distributed transactions.
Redundancy and Monitoring: Implement redundancy in critical paths and robust monitoring to detect and recover from failures quickly.

Cost Implications:

Increased complexity in system design and implementation.
Potential increase in operational overhead for maintaining consistent states.
Additional costs associated with high availability setups (e.g., multiple replicas, failover mechanisms).

#### Closing Summary

In this scenario, while eventual consistency is a valuable trade-off to achieve performance and scalability, it must be carefully balanced against the critical failure modes of double ordering and lost items. For such scenarios, we should enforce strong guarantees through transactional support, optimistic concurrency controls, and consistent state management.

#### Common Mistakes/Follow-Ups
Overlooking the importance of strong consistency in critical operations.
Ignoring potential revenue and customer satisfaction impacts due to inconsistencies.
Underestimating the complexity and cost of implementing robust guarantees.

Would you like a more detailed analysis or any specific aspects clarified?

**Comparison:**

| id | items |
|----|-------|
| 123 | [456] |
| 789 | [2]   |
| id | items |
|----|-------|
| 987 | [456, 789] |

**Code:**

```
MERMAID
Copy
graph TD;
    A[User Adds Item to Cart] --> B[Shopping Cart Service];
    B --> C[Database Update];
    C --> D[Order Handoff Initiated];
    D --> E[Inventory Service];
    D --> F[Payment Service];
    E --> G[Inventory Updated];
    F --> H[Payment Confirmed];
    H --> I[Order Created];
    J[Network Error] --> K[Lost Item in Order];
```

**Example input / output:**

| Input | Action                 | Output                        |
|-------|------------------------|-------------------------------|
| User adds an item to the cart | Database update and UI refresh | Cart state updated, UI reflects change |
| Order handoff initiated      | Inventory service notified    | Inventory service updates its state |
| Payment confirmed            | Order created                | New order added to database, payment processed |

**Data:**

```json
{
  "shoppingCart": {
    "id": 123,
    "items": [
      { "itemId": 456, "quantity": 1 },
      { "itemId": 789, "quantity": 2 }
    ]
  },
  "order": {
    "id": 987,
    "items": [
      { "itemId": 456, "quantity": 1 },
      { "itemId": 789, "quantity": 2 }
    ]
  }
}
```

**Flow:**

1. User Adds Item to Cart
2. Database Update
3. Order Handoff Initiated
4. Inventory Service Notification
5. Payment Service Confirmation
6. Order Created

