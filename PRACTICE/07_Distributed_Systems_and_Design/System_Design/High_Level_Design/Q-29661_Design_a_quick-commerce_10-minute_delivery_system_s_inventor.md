---
type: "practice"
difficulty: "Deep"
---

# Design a quick-commerce (10-minute delivery) system's inventory and dark-store routing — how do you decide which store fulfills an order and reserve stock in real time?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a high-level inventory and routing system for a quick-commerce platform, our primary goal is to ensure rapid fulfillment of orders while maintaining operational efficiency. This involves making decisions on which store should service an order and how to reserve stock in real time. Let's break down the key considerations and design a solution step-by-step.

#### Triage
Understanding the Question

The core question revolves around selecting the most appropriate store for fulfilling an order based on proximity, availability, and operational status. The system must also handle inventory reservation in real-time to prevent overselling or stockouts. We need to consider various factors such as distance from the customer, current stock levels, store capacity, and potential delivery times.

#### Assumptions
Stores are geographically dispersed: Customers can be served by multiple stores.
Real-time data access: The system has up-to-date information on each store's inventory and operational status.
Multiple order types: Different products may have different delivery times, impacting routing decisions.

#### First-Principles Solution
Data Model: Define the necessary entities and their relationships to support real-time decision-making.
Routing Algorithm: Implement an algorithm that considers multiple factors to determine the best store for fulfilling the order.
Inventory Reservation System: Ensure seamless reservation of stock without double booking.
Detailed Solution
Data Model

The system will require several core data entities:

Order: Contains details such as customer, delivery address, and items ordered.
Store: Stores inventory information, operational status, and geolocation.
Item: Represents individual products with inventory levels.
Reservation: Tracks reserved stock for each order.
Routing Algorithm

The algorithm will evaluate stores based on the following criteria:

Proximity: Calculate distance from the customer to each store using a geographical distance formula (e.g., Haversine).
Stock Availability: Check if the required items are in sufficient quantity at each store.
Operational Status: Ensure that selected stores are operational and can handle current orders.

Here’s a simplified pseudocode for the algorithm:

PYTHON
def select_store(order, stores):
    # Calculate distance to each store
    distances = {store.id: calculate_distance(store.location, order.customer_location) for store in stores}

    eligible_stores = [store for store in stores if are_items_available(order.items, store.inventory)]

    if not eligible_stores:
        return None  # No suitable store found

    # Sort by proximity and availability
    ordered_stores = sorted(eligible_stores, key=lambda s: (distances[s.id], -sum([item.quantity for item in order.items])))

    selected_store = ordered_stores[0]  # Select the closest eligible store with sufficient stock

    return selected_store
Inventory Reservation System

To reserve stock without double booking:

Lock Stock: Before fulfilling an order, lock the required quantity of each item.
Commit or Rollback: Upon successful fulfillment (e.g., delivery), commit the reservation; otherwise, rollback.
PYTHON
def reserve_stock(order_items, store):
    for item in order_items:
        if not store.inventory[item.id].quantity >= item.quantity:
            return None  # Not enough stock

        # Reserve the stock
        store.inventory[item.id].quantity -= item.quantity

    return True

def commit_reservation(store, order_items):
    # Commit the reservation after successful delivery
    for item in order_items:
        store.inventory[item.id].quantity += item.quantity

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, our high-level design focuses on a combination of geographical proximity, real-time inventory checks, and operational status to efficiently fulfill orders. The key is ensuring that the routing algorithm considers all relevant factors while minimizing delays.

#### Common Mistakes
Ignoring Operational Status: Failing to check if a store is operational can lead to failed deliveries.
Overselling Inventory: Not locking stock before fulfilling an order can result in stockouts and customer dissatisfaction.

#### Follow-Ups
How would you handle situations where multiple stores are equally close?
What mechanisms would you implement for real-time updates on inventory changes?
How do you ensure the system scales with increasing numbers of orders and stores?

**Comparison:**

```
Item	Quantity in Order	Quantity in Store
1	2	5
2	1	3
Store	Selected?	Reasoning
1	Yes	Closest and sufficient stock for both items.
2	No	Sufficient stock but not the closest.
```

**Code:**

```python
def calculate_distance(loc1, loc2):
    # Haversine formula implementation here

def are_items_available(order_items, store_inventory):
    return all([store_inventory[item.id].quantity >= item.quantity for item in order_items])

def reserve_stock(order_items, store):
    for item in order_items:
        if not store.inventory[item.id].quantity >= item.quantity:
            return None
        store.inventory[item.id].quantity -= item.quantity
    return True

def commit_reservation(store, order_items):
    for item in order_items:
        store.inventory[item.id].quantity += item.quantity
```

**Example input / output:**

```text
Input:
Order:
- item_id: 1, quantity: 2
- item_id: 2, quantity: 1

Store Inventory:
- item_id: 1, quantity: 5
- item_id: 2, quantity: 3

Output:
Selected Store ID: 1
Reservation Success: True
```

**Data:**

```json
{
  "Order": {
    "id": 1,
    "customer_id": 1001,
    "items": [
      {"item_id": 1, "quantity": 2},
      {"item_id": 2, "quantity": 1}
    ]
  },
  "Store": {
    "id": 1,
    "location": [40.7128, -74.0060],
    "inventory": [
      {"item_id": 1, "quantity": 5},
      {"item_id": 2, "quantity": 3}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Store Operational?
5. Select Store
6. Reserve Stock
7. Commit Reservation
8. Choose Next Closest Store
9. Fulfill Order
10. Log Transaction

