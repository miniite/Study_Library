---
type: "practice"
difficulty: "Expert"
---

# How would you evolve a monolith's single-transaction Serializable checkout into a saga across services while preserving the same no-oversell invariant? What replaces isolation?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ From Serializable to Sagas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/From_Serializable_to_Sagas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evolving a monolithic, single-transaction Serializable checkout process to a distributed saga architecture, the primary challenge is maintaining consistency and preventing over-selling of resources (e.g., stock). The goal is to ensure that each service transactionally manages its part of the checkout process while preserving the overall invariant. In a distributed system, we can no longer rely on database isolation levels like Serializable; instead, we need to use compensating transactions and saga patterns.

#### Triage

The question asks about transitioning from a single-transaction approach with strong consistency (Serializable) to a multi-service saga pattern that maintains the same invariant of preventing over-selling. The key is understanding how each service ensures its part of the transactional logic while coordinating with other services through compensating actions.

#### First Principles Solution
Decompose the Checkout Process: Break down the checkout process into discrete steps (e.g., check stock, apply discount, create order). Each step becomes a separate transaction in a saga.
Use Saga Patterns: Design each service to handle its part of the transaction and ensure it can roll back if any step fails. Compensating actions are used to reverse previous operations when a saga is terminated due to failure or timeout.
Ensure Idempotency: Make sure that each step in the saga is idempotent, meaning repeated execution does not change the state more than once. This helps handle retries and failures gracefully.
Atomicity via Compensating Actions: For steps that require strong consistency (like checking stock), use compensating actions to ensure they can be undone if something goes wrong.

#### Example Scenario

#### Cons
ider a simplified checkout process with three services:

StockService: Checks stock availability
DiscountService: Applies discounts based on conditions
OrderService: Creates an order and updates inventory
StockService: Checks if the product is in stock.
If out of stock, record a failure state and notify downstream services.
DiscountService: Applies any applicable discounts.
If there's no discount to apply, record a success state.
OrderService: Creates an order and updates inventory.

If any step fails (e.g., the product is sold out), all previous steps need to be rolled back:

StockService rolls back by marking the stock as available again.
DiscountService rolls back by removing any applied discount.
OrderService rolls back by canceling the order and updating the inventory.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is that in a distributed system, we must rely on compensating actions to ensure consistency. Each service needs to be designed to handle transactions locally but also account for potential failures. Compensating actions are crucial for maintaining the no-oversell invariant.

#### Common mistakes
include:

Not handling retries properly: Failing to implement idempotent operations can lead to duplicate or inconsistent states.
Lack of clear failure handling: Not having a defined rollback strategy means that failed transactions might leave the system in an inconsistent state.
Ignoring transactional boundaries: Services may inadvertently perform actions outside their intended scope, leading to unexpected side effects.

#### Follow-up
questions could include:

How do you ensure idempotency across services?
What are some strategies for managing timeouts and retries in a distributed saga?

**Comparison:**

```
Service	Step	Action
Stock	1	Check in stock
Discount	2	Apply discount
Order	3	Create order
Service	Rollback Action
Stock	Mark as available again
Discount	Remove applied discount
Order	Cancel order
```

**Code:**

```python
# Example of compensating action for StockService
class StockService:
    def check_stock(self, product_id):
        if not self.is_in_stock(product_id):
            # Log failure state
            logging.error(f"Stock failed for product {product_id}")
            raise OutOfStockError()

    def is_in_stock(self, product_id):
        return self.inventory.get(product_id) > 0

# Compensating action to mark stock as available again
def mark_stock_as_available(self, product_id):
    # Mark the stock record as available again
    logging.info(f"Marking stock as available for {product_id}")
    self.inventory[product_id] = True
```

**Example input / output:**

```text
Input:
- Check if product 12345 is in stock

Output:
- Product 12345 is available
- Mark product 12345 as out of stock after the transaction fails
```

**Data:**

```json
{
  "inventory": {
    "12345": true,
    "67890": false // Out of stock
  }
}
```

**Flow:**

1. Available?
2. No
3. Yes
4. Start
5. Check Stock
6. Apply Discount
7. Create Order
8. Mark as Out of Stock
9. Create Order
10. Success

