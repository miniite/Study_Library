---
type: "practice"
difficulty: "Expert"
---

# Design a saga for a multi-step order workflow spanning payment, inventory, and shipping services — orchestration vs choreography, and how do you handle compensating actions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary

A high-level design (HLD) for a multi-step order workflow involves orchestrating tasks across payment, inventory, and shipping services using a saga pattern. This approach ensures each step is atomic and idempotent, allowing for compensating actions if any part of the process fails. The key choices are between choreography and orchestration, with orchestrations offering better control over the sequence but requiring more coordination.

#### Triaging the Question

The question asks us to design a saga for handling an order workflow that spans multiple services: payment, inventory, and shipping. This involves understanding both orchestration and choreography models and deciding which is appropriate given our requirements. Additionally, we need to consider how compensating actions are handled in case of failures.

#### Assumptions
The system must be resilient against partial failures.
Each service (payment, inventory, shipping) operates independently but needs to interact with others during the order process.
We will use a saga pattern to ensure that each step is idempotent and can be compensated if needed.
The services are loosely coupled.
Step-by-Step Solution
1. Understanding Orchestration vs Choreography

Orchestration: In an orchestration model, a central service (orchestrator) coordinates the flow of the process. It manages the state and ensures that all services complete their tasks in the correct order.

Choreography: In a choreography model, each service defines its own interactions independently. Coordination is done via events or messages, with no centralized control.

2. Choosing Between Orchestration and Choreography

For this specific use case:

Orchestration: Since we need to ensure that the order process follows a specific sequence (payment -> inventory update -> shipping), orchestration provides better control over the flow.
Choreography: If our services are highly independent, choreography might be more suitable. However, for an order workflow, coordination is crucial.
3. Designing the Saga

A saga pattern involves breaking down the process into multiple steps and using compensating actions to revert any completed steps if a failure occurs.

Steps in the Order Workflow:

Payment: Capture payment details and create an order.
Inventory Update: Check inventory availability and update it.
Shipping: Schedule shipping and send confirmation to customer.

Each step should be idempotent, meaning it can be safely retried without causing issues.

Compensating Actions:

If the payment fails in Step 1, rollback any changes made.
If the inventory check or update fails in Step 2, revert the order state.
If shipping fails in Step 3, cancel the order and notify the customer.
4. Implementing the Saga
Failure
Failure
Failure
Order Created
Payment Capture
Check Inventory Availability
Update Inventory
Schedule Shipping
Ship Order
Confirm Shipment toCustomer
Rollback Payment and RevertOrder
Revert Inventory Changes
CANCEL ORDER, NotifyCustomer
5. Compensating Actions
Payment Capture Failure: Roll back the transaction.
Inventory Check/Update Failure: Restore inventory to previous state.
Shipping Failure: Cancel order and notify customer.

#### Closing Summary

In conclusion, for a multi-step order workflow, orchestrating the process provides better control over sequence but requires more coordination. The saga pattern ensures idempotent steps and handles compensating actions effectively in case of failures.

#### Common Mistakes/Follow-Ups
Ignoring Compensating Actions: Not all steps can be rolled back; this needs careful planning.
Overcomplicating the Saga: Keeping it simple reduces complexity but ensure all necessary compensations are covered.
Failing to Monitor and Test: Regularly testing and monitoring the saga ensures reliability.

**Comparison:**

```
Step	Task	Outcome	Compensating Action
1	Payment Capture	Payment ID	Rollback transaction
2	Inventory Check	Inventory Status	Restore inventory
3	Shipping	Shipment Confirmed	Cancel order and notify customer
```

**Code:**

```python
# Example of a simplified saga implementation in Python

class OrderSaga:
    def __init__(self):
        self.payment_id = None
        self.inventory_status = None

    def start_order(self, order_details):
        # Capture payment and create an order
        # self.payment_id = capture_payment(order_details)
        pass

    def check_inventory(self):
        # Check inventory availability
        self.inventory_status = check_inventory()
        return self.inventory_status

    def update_inventory(self):
        # Update inventory if available
        if self.inventory_status:
            update_inventory(self.payment_id)
            return True
        else:
            return False

    def schedule_shipping(self):
        # Schedule shipping and send confirmation to customer
        ship_order(self.payment_id)
        notify_customer()
```

**Example input / output:**

```text
Input:
- Order Details: {customer_id: 123, payment_method: 'credit_card', items: [{'item_id': 456, 'quantity': 1}]}
- Inventory Status: true

Output:
- Payment Captured and Order Created (payment_id = 100)
- Inventory Updated (inventory_status = true)
- Shipping Scheduled (shipping_id = 789)

Compensating Actions:
- If payment fails: Rollback transaction
- If inventory update fails: Restore inventory
- If shipping fails: Cancel order and notify customer
```

**Data:**

```json
{
  "order_details": {
    "customer_id": 123,
    "payment_method": "credit_card",
    "items": [
      {"item_id": 456, "quantity": 1}
    ]
  },
  "inventory_status": true,
  "shipping_details": {
    "shipping_id": 789
  }
}
```

**Flow:**

1. Failure
2. Failure
3. Failure
4. Order Created
5. Payment Capture
6. Check Inventory Availability
7. Update Inventory
8. Schedule Shipping
9. Ship Order
10. Confirm Shipment toCustomer
11. Rollback Payment and RevertOrder
12. Revert Inventory Changes
13. CANCEL ORDER, NotifyCustomer

