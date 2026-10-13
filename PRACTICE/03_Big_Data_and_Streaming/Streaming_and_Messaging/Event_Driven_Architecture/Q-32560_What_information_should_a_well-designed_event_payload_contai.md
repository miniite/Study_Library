---
type: "practice"
difficulty: "Deep"
---

# What information should a well-designed event payload contain, and should it carry full data or just an ID?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fat vs Thin Events](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an event-driven architecture with message queues, the payload of an event is crucial as it carries essential information that drives the processing logic in downstream systems. A well-designed event payload should contain minimal yet sufficient data to enable efficient and context-rich processing. It's important to consider whether the payload should carry full data or just a unique identifier (ID). Let’s delve into the key aspects of this decision.

**Comparison:**

```
Event Type	Payload Content	Reasoning
Order Placed	order_id, customer_id	Minimal essential data for immediate processing and validation.
Product Updated	product_id, new_price, stock_count	Sufficient to update the system with new information, but avoids unnecessary details.
```

**Code:**

```python
# Example Event Payload Classes
class OrderPlacedEvent:
    def __init__(self, order_id, customer_id, product_ids):
        self.order_id = order_id  # Unique identifier for the event
        self.customer_id = customer_id  # ID of the customer who placed the order
        self.product_ids = product_ids  # List of product IDs in the order

class ProductUpdatedEvent:
    def __init__(self, product_id, new_price, stock_count):
        self.product_id = product_id  # Unique identifier for the event
        self.new_price = new_price  # New price of the product
        self.stock_count = stock_count  # Updated stock count
```

**Example input / output:**

```text
# Example Input-Output Scenarios

def processOrderPlaced(event):
    # Retrieve full order details from the database using event.orderId
    order_details = fetch_order_by_id(event.order_id)
    # Process the order (e.g., send to payment system, update inventory)
    print(f"Processing order {event.order_id} for customer {event.customer_id}")

def processProductUpdated(event):
    # Update product details in the database using event.productId
    update_product(event.product_id, event.new_price, event.stock_count)
    # Log the change
    log_event("product_updated", {"productId": event.product_id})

# Sample Inputs
input_order_placed = OrderPlacedEvent("123456789", "ABCD1234", ["P001", "P002"])
input_product_update = ProductUpdatedEvent("P001", 9.99, 50)

# Outputs
processOrderPlaced(input_order_placed)
processProductUpdated(input_product_update)
```

**Data:**

```json
{
    "orderPlacedEvent": {
        "orderId": "123456789",
        "customerId": "ABCD1234",
        "productIds": ["P001", "P002"]
    },
    "productUpdatedEvent": {
        "productId": "P001",
        "newPrice": 9.99,
        "stockCount": 50
    }
}
```

**Flow:**

1. No
2. Yes
3. Event Occurs
4. Is Payload Full Data Needed?
5. Use Identifier Only
6. Retrieve Full Data fromDatabase
7. Process Event with Full Details
8. Process Event with MinimalData

