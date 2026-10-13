---
type: "practice"
difficulty: "Deep"
---

# How would you choose a Cosmos DB consistency level (strong to eventual) for a shopping cart versus an order ledger, and what's the cost of each choice?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Functions and Cosmos DB Tuning](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When choosing a consistency level for Azure Cosmos DB in a scenario involving both a shopping cart and an order ledger, it is crucial to balance the requirements of data integrity with performance and cost considerations. The key difference between strong and eventual consistency impacts how transactions are handled, and thus influences the choice.

#### Triage

The question requires understanding the trade-offs between strong and eventual consistency in Cosmos DB for specific use cases: a shopping cart (which needs high availability and low latency) versus an order ledger (where eventual consistency can be more acceptable due to higher transaction volumes). The interviewer is probing your knowledge of consistency models, their implications on performance, and cost.

#### First-Principles Solution

Understanding the Use Cases:

Shopping Cart: This system requires high availability to avoid losing user sessions or incomplete transactions.
Order Ledger: This system can tolerate a slight delay in recording updates as long as it eventually reflects all changes accurately.

#### Cons
istency Levels:

Strong Consistency: Ensures that any read operation will return the most recent write, and this consistency is maintained across partitions. Suitable for scenarios where data integrity is paramount.
Eventual Consistency: Data may be temporarily inconsistent but will eventually stabilize to the latest state. This can provide better performance and lower costs.

Shopping Cart (High Availability Requirement):

Choose Strong Consistency or Bounded Staleness for this use case. Strong consistency ensures that every read operation returns the most recent write, which is critical for a shopping cart where user sessions need to be uninterrupted.
Bounded staleness allows writes to be delayed by a configurable duration (e.g., 10 seconds) before being visible globally. This can provide some flexibility in cost while maintaining acceptable levels of consistency.

Order Ledger (Higher Volume and Tolerable Latency):

Choose Session Consistency or Eventual Consistency for this use case. Session consistency ensures that the same session sees the same data, which is useful within a single user transaction but not across multiple sessions.
Eventual consistency can be used if the system can tolerate some latency in reflecting changes to the order ledger. This choice will reduce costs significantly.

#### Closing Summary
and Common Mistakes

In summary, the shopping cart should use strong or bounded staleness consistency to ensure high availability and data integrity, while the order ledger can leverage session consistency or eventual consistency for better performance and cost efficiency. Misunderstanding these trade-offs can lead to suboptimal system design, either in terms of functionality (e.g., inconsistent data) or economics (higher costs than necessary).

Cost Considerations
Strong Consistency: Generally more expensive due to the overhead required to maintain high availability across partitions.
Eventual Consistency: Offers lower costs but may involve delays and potential inconsistencies.

**Comparison:**

| Database | Container | Partition Key Path | Throughput (RU/s) | Consistency Level |
|----------|-----------|--------------------|------------------|-------------------|
| ShoppingCartDB | ShoppingCartItems | /id | 1000 | BoundedStaleness |
| OrderLedgerDB | Orders | /id | 400 | Eventual |

**Code:**

```python
# Example configuration for Cosmos DB using Azure SDK

from azure.cosmos import CosmosClient, PartitionKey

# Initialize client with connection string and endpoint
client = CosmosClient('<your-endpoint>', '<your-key>')

database_name = 'ShoppingCartDB'
container_name = 'ShoppingCartItems'

# Strong Consistency for Shopping Cart
shopping_cart_db = client.create_database_if_not_exists(id=database_name)
shopping_cart_container = shopping_cart_db.create_container_if_not_exists(
    id=container_name,
    partition_key=PartitionKey(path="/id"),
    throughput=1000,
    consistency_level="BoundedStaleness"  # or "Strong"
)

# Eventual Consistency for Order Ledger
order_ledger_db = client.create_database_if_not_exists(id='OrderLedgerDB')
order_ledger_container = order_ledger_db.create_container_if_not_exists(
    id='Orders',
    partition_key=PartitionKey(path="/id"),
    throughput=400,
    consistency_level="Session"  # or "Eventual"
)
```

**Example input / output:**

```text
Input (Shopping Cart Setup):
- Database Name: ShoppingCartDB
- Container Name: ShoppingCartItems
- Partition Key Path: /id
- Throughput: 1000 RU/s
- Consistency Level: BoundedStaleness

Output:
Database ShoppingCartDB created or found.
Container ShoppingCartItems created with BoundedStaleness consistency and 1000 RU/s throughput.

Input (Order Ledger Setup):
- Database Name: OrderLedgerDB
- Container Name: Orders
- Partition Key Path: /id
- Throughput: 400 RU/s
- Consistency Level: Eventual

Output:
Database OrderLedgerDB created or found.
Container Orders created with Eventual consistency and 400 RU/s throughput.
```

**Data:**

```json
{
  "shoppingCartDB": {
    "name": "ShoppingCartDB",
    "containers": [
      {
        "name": "ShoppingCartItems",
        "partitionKeyPath": "/id",
        "throughput": 1000,
        "consistencyLevel": "BoundedStaleness"
      }
    ]
  },
  "orderLedgerDB": {
    "name": "OrderLedgerDB",
    "containers": [
      {
        "name": "Orders",
        "partitionKeyPath": "/id",
        "throughput": 400,
        "consistencyLevel": "Eventual"
      }
    ]
  }
}
```

**Flow:**

1. BoundedStaleness
2. Strong
3. Eventual
4. Session
5. Shopping Cart Setup
6. Consistency Level
7. High Availability, Low Latency
8. High Availability, No Delay
9. Order Ledger Setup
10. Consistency Level
11. Lower Costs, Tolerable Latency
12. Local Consistency withinSession

