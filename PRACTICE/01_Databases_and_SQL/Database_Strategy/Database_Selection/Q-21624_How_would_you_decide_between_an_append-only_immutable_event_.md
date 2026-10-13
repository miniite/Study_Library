---
type: "practice"
difficulty: "Deep"
---

# How would you decide between an append-only immutable event store and a mutable relational model for a system that needs full audit history and temporal queries?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When choosing between an append-only immutable event store and a mutable relational model for systems requiring full audit history and temporal queries, the decision hinges on several factors including data modeling complexity, query performance, transactional consistency, and maintainability. This answer will walk through these considerations from first principles to arrive at a strategic solution.

**Code:**

```text
# Example of a simple SQL table schema for mutable RDBMS
CREATE TABLE Orders (
    id INT PRIMARY KEY,
    order_id VARCHAR(255) NOT NULL,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(100)
);

# Example of an event store schema
CREATE TABLE OrderEvents (
    id SERIAL PRIMARY KEY,
    order_id VARCHAR(255) NOT NULL,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    action VARCHAR(100),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(100)
);
```

