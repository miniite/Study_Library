---
type: "practice"
difficulty: "Intermediate"
---

# When choosing a datastore for user sessions or a shopping cart, why might you deliberately choose ephemerality and a key-value store over durable relational rows? Where does that bite you?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When selecting a database for managing user sessions or a shopping cart, using an ephemeral, key-value (KV) store like Redis can be advantageous due to its speed and ease of use, especially in scenarios where data needs to persist only as long as the session is active. However, this choice comes with trade-offs, such as loss of data upon failure or restart.

#### Triaging the Question

The question asks us to explain why we might choose a key-value store for user sessions or shopping carts despite its ephemeral nature and potential drawbacks. The interviewer likely wants to understand our reasoning behind non-durability decisions, especially in mission-critical applications like e-commerce.

#### First Principles Solution
1. Context of Use:
User Sessions: These are temporary data structures that hold information about a user's interaction with a web application.
Shopping Carts: Store items selected by users for purchase before checkout, often requiring quick and frequent updates.
2. Why Key-Value Stores?
Speed and Performance: KV stores like Redis offer extremely low latency due to their in-memory nature, making them ideal for real-time applications.
Simplicity and Flexibility: No need to define a schema or worry about complex queries; simply store key-value pairs.
In-Memory Persistence: Data is stored in RAM, which allows very fast read/write operations.
3. Ephemerality Consideration:
Temporary Data: User sessions and shopping carts are temporary data that do not need to be persisted long-term. They are relevant only as long as the user session exists.
Scalability: By using ephemeral storage, we can easily scale the application without worrying about the database's durability.
4. Trade-offs and Risks:
Data Loss: If the system fails or restarts, any data in the KV store is lost. This could be a significant issue if session state or cart contents are critical.
Backup Considerations: Since the data is not durable, we need to ensure that user sessions and carts can be reconstructed from other sources like logs or re-computation if necessary.
5. Example Scenario:

Suppose an e-commerce site has a high volume of short-lived shopping cart entries. By using Redis as the KV store for these carts, the application can achieve near-instantaneous updates to cart contents without the overhead of a traditional RDBMS. However, in case of a system failure, all pending cart entries would be lost unless they are periodically backed up or re-computed.

6. Mitigation Strategies:
Periodic Snapshots: Take periodic snapshots of key data and store them in a durable storage solution.
Fault Tolerance: Implement mechanisms to handle failures gracefully, such as using Redis Cluster or Sentinel for high availability.
Re-computation Logic: Develop logic to recompute user sessions or cart contents based on logs or other means if the KV store fails.

#### Closing Summary

Choosing a key-value store like Redis for managing ephemeral data such as user sessions or shopping carts can provide significant performance benefits and simplification. However, this decision must be carefully weighed against the risk of losing critical data upon system failure. Proper mitigation strategies should be in place to ensure data integrity and availability.

#### Common Mistakes/Follow-Ups
Overlooking Data Loss: Not considering the implications of data loss on user experience or business operations.
Ignoring Backup Mechanisms: Failing to implement backup mechanisms, which can lead to significant downtime recovery issues.
Scaling Considerations: Overlooking the need for scalable solutions that can handle high volumes of ephemeral data.

**Comparison:**

```
Key	Value
cart:12345:item_id:A001	quantity:1
cart:12345:item_id:B002	quantity:2
```

**Code:**

```python
# Example code snippet using Redis for a shopping cart session

from redis import StrictRedis

redis_client = StrictRedis(host='localhost', port=6379, db=0)

def add_item_to_cart(user_id, item_id):
    # Add an item to the user's cart in Redis
    redis_client.hset(f'cart:{user_id}', mapping={'item_id': item_id})

def get_cart_items(user_id):
    # Retrieve all items in a user's cart from Redis
    return redis_client.hgetall(f'cart:{user_id}')
```

**Example input / output:**

```text
Input:
- Add item A001 to cart of user 12345

Output:
- Redis key 'cart:12345' is updated with {'item_id': b'A001', 'quantity': b'1'}

Input:
- Get all items in the cart for user 12345

Output:
- Redis key 'cart:12345' returns {'item_id': b'A001', 'quantity': b'1'}, {'item_id': b'B002', 'quantity': b'2'}
```

**Data:**

```json
{
  "cart": {
    "12345": [
      {"item_id": "A001", "quantity": 1},
      {"item_id": "B002", "quantity": 2}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. User Places Order
4. Is Cart Data Critical?
5. Use RDBMS to Persist CartData
6. Use KV Store for EphemeralCart Data
7. Periodically Backup Cart Data

