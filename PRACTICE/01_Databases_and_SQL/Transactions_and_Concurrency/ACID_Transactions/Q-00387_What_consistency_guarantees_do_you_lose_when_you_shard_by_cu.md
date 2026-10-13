---
type: "practice"
difficulty: "Deep"
---

# What consistency guarantees do you lose when you shard by customer and then need a cross-customer transaction?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ ACID Durability and Consistency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sharding databases by customer can significantly improve scalability, but it comes with challenges, particularly in maintaining ACID (Atomicity, Consistency, Isolation, Durability) properties across customer boundaries. Specifically, when you need to perform a cross-customer transaction, several consistency guarantees are lost due to the distributed nature of sharded data.

In this scenario, we're primarily concerned with how sharding affects the isolation and sometimes even the atomicity of transactions. Let's break down these issues step-by-step from first principles.

**Comparison:**

```
Transaction Type	ACID Properties Maintained
Within-Shard	Yes
Cross-Shard	No
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Sharded Database] --> B{Customer 1 Shards}
    B --> C[Customer 2 Shards]
    A --> D{Cross-Customer Transaction}
    D --> E[Transaction Spanning Multiple Nodes]
```

**Example input / output:**

```text
Input:
- Customer 1's data stored in Shard 1
- Customer 2's data stored in Shard 2

Output:
- Cross-customer transaction attempting to update both shards simultaneously.
```

