---
type: "practice"
difficulty: "Deep"
---

# Your order state machine started clean but PMs keep adding states (returned, partially-shipped, disputed) and the transition table is unmanageable; how do you keep the design maintainable as states proliferate?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the growing complexity of an order state machine, we need to ensure that it remains manageable and maintainable as new states are added. The key is to design a flexible yet structured system that can handle these changes efficiently. This involves using pattern-based approaches, modular design principles, and robust testing practices.

The core challenge here is to manage the transitions between different states effectively while keeping the codebase clean and understandable. We need to ensure that each state transition adheres to business rules and that any new state additions do not disrupt existing functionality.

Let's break this down step by step:

Identify the States and Transitions: Start by clearly defining all possible states (e.g., returned, partially-shipped, disputed) and transitions between them. For example:

PLAINTEXT
Order -> Returned -> Closed
Order -> Partially-Shipped -> Shipped
Order -> Disputed -> Resolved / Closed

Use a State Machine Library: Leverage a state machine library like stateflow in Node.js, statecharts in Python, or even a dedicated state management tool like Redux in React applications. These libraries provide abstractions that help manage state transitions more declaratively.

Define the Business Rules: Ensure that each transition is governed by clear business rules. For example:

An order can only be marked as 'Returned' if it has been picked up and verified.
An order can be 'Partially-Shipped' only after some items have been dispatched but not all.

Implement Transition Guards: Use guards to validate the conditions under which a transition is allowed. For example:

JAVASCRIPT
function isOrderReturnable(order) {
  return order.status === 'placed' && order.items.length > 0;
}

function canPartiallyShip(order) {
  return order.status === 'awaiting-approval';
}

Keep the Code Modular: Organize your state machine into smaller, reusable components or modules. For instance:

JAVASCRIPT

#### cons
t states = {
  order: { states },
  transitions: { guards }
};

Test Thoroughly: Develop a comprehensive test suite to cover all possible transitions and edge cases. Use unit tests for individual state transitions and integration tests for complex sequences.

Document the State Machine: Maintain clear documentation that explains the current state machine design, including how new states are added and what changes require review.

By following these steps, we can ensure that the order state machine remains maintainable even as more states and transitions are introduced over time.

**Comparison:**

```
Guard Function	Description
isOrderReturnable	Checks if order status is 'placed' and has items.
canPartiallyShip	Ensures order is in the correct state for partial shipment.
```

**Code:**

```javascript
// Example of a simple state transition guard in JavaScript
function isOrderReturnable(order) {
  return order.status === 'placed' && order.items.length > 0;
}

function canPartiallyShip(order) {
  return order.status === 'awaiting-approval';
}
```

**Example input / output:**

```text
- **Input**: Order with status 'placed' and items.
- **Output**: Guard returns true for `isOrderReturnable`, false for `canPartiallyShip`.

| Guard          | Order Status     | Items Present? | Result |
|----------------|------------------|----------------|--------|
| isOrderReturnable | placed          | yes            | true   |
| canPartiallyShip  | awaiting-approval | no             | false  |
```

**Data:**

```json
{
  "order": {
    "id": 123,
    "status": "placed",
    "items": [{ "sku": "A", "quantity": 5 }]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Order placed
6. Has Items?
7. Returnable
8. Not returnable
9. Order status remains 'placed'
10. Mark as returned
11. Update order status to'returned'
12. Awaiting approval
13. Can partially ship?
14. Partial shipment allowed
15. Partial shipment not allowed
16. Dispatch some items
17. Order status remains'awaiting-approval'

