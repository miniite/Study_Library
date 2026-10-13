---
type: "practice"
difficulty: "Expert"
---

# You must choose whether ordering/serialization guarantees live in the messaging layer, the datastore, or the application, for the org's default architecture. Argue how placing them in each layer changes coupling, portability across cloud vendors, and the ability to change the concurrency model later.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address whether ordering/serialization guarantees should be placed in the messaging layer, the datastore, or the application, we need to carefully consider several factors including coupling, portability across cloud vendors, and the flexibility of changing the concurrency model later.

#### Triage

The question is fundamentally about choosing an architectural decision that balances performance, consistency, and future-proofing. Placing guarantees in different layers impacts how components interact and can affect system behavior under high load or failures. Let’s break down each layer:

Messaging Layer: Ensures messages are processed in a specific order before being persisted.
Datastore: Guarantees that data is written to the database in an ordered manner, typically via transactions.
Application: Enforces ordering at the application level, potentially affecting its architecture and complexity.

#### First Principles Solution
Messaging Layer

Guarantees: Ensures messages are processed in order before being persisted. This can help with idempotency but may introduce complexities for ensuring eventual consistency across distributed systems.

Coupling: High coupling between the messaging system and application components, as both must coordinate on ordering semantics.
Portability: Limited portability since different messaging brokers might support varying levels of guaranteed delivery and order.
Flexibility: Challenges in changing concurrency models later because the entire flow depends on this layer.
Datastore

Guarantees: Ensures transactions are atomic, ensuring ordered writes to the database. This is a common approach for maintaining consistency.

Coupling: Lower coupling between application components and the database, allowing more independent scaling of services.
Portability: More portable across cloud vendors since most offer transactional capabilities (e.g., ACID properties).
Flexibility: Easier to change concurrency models later as the ordering is abstracted from the application logic.
Application

Guarantees: Ensures data integrity and order at the application level, often through custom logic or middleware.

Coupling: High coupling with the application architecture, making it harder to scale independently of database operations.
Portability: Potentially low portability since custom solutions may need rework when changing cloud vendors.
Flexibility: Greatest flexibility in terms of implementing different concurrency models as needed.

#### Closing Summary

Placing ordering/serialization guarantees at the datastore level offers a good balance between coupling, portability, and future-proofing. It allows for more independent scaling of application components while maintaining transactional integrity across distributed systems.

#### Common Mistakes/Follow-Ups

Over-Engineering Messaging Layer: Assuming all applications will benefit from guaranteed message processing without considering the complexity.
Neglecting Application-Level Solutions: Ignoring application-level solutions, which might be simpler and more maintainable in many cases.
Underestimating Portability Issues: Failing to account for vendor-specific differences that can impact the ease of migration.

**Comparison:**

```
Layer	Coupling	Portability	Flexibility
Messaging Layer	High	Low	Low to Medium
Datastore	Medium	High	Medium to High
Application	High	Low	High
```

**Code:**

```python
# Example code to handle transactions in a datastore
from sqlalchemy import create_engine, text

engine = create_engine('sqlite:///example.db')
with engine.connect() as connection:
    with connection.begin():
        result = connection.execute(text("INSERT INTO orders (user_id, item) VALUES (:user_id, :item)", {"user_id": 1, "item": "book"}))
```

**Example input / output:**

```

Input: A user places an order with item="book".

Output: The order is processed and saved in the database with a timestamp.

JSON
Copy
{
  "user_id": 1,
  "item": "book",
  "order_timestamp": "2023-10-05T14:30:00Z"
}
```

**Data:**

```json
{
  "user_id": 1,
  "item": "book",
  "order_timestamp": "2023-10-05T14:30:00Z"
}
```

**Flow:**

1. Datastore
2. Transaction 1
3. Order Saved
4. Transaction 2
5. Order Confirmed
6. Order Timestamped
7. Application
8. Datastore
9. Messaging Layer
10. B

