---
type: "practice"
difficulty: "Expert"
---

# How does write skew manifest in a distributed inventory system split across shards, and why can per-shard Serializable still permit the global anomaly? What's the architectural fix?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ From Serializable to Sagas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/From_Serializable_to_Sagas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Write skew is a common issue in distributed systems where writes to different sharded databases can introduce anomalies, even if each shard operates under its own Serializable isolation level. The root cause lies in the lack of global transactional consistency when updates are applied independently across shards. To mitigate this, an architectural fix involving coordination and distributed transactions becomes necessary.

#### Triaging the Question

The question is asking you to explain how write skew occurs in a sharded inventory system where each shard operates under Serializable isolation but still allows global anomalies. You need to understand why per-shard Serializable does not guarantee global consistency and propose a solution that addresses this issue.

Key Assumptions:
The system is sharded, meaning the inventory database is split across multiple shards.
Each shard uses Serializable isolation to ensure transactions are isolated from each other.
Write skew occurs when independent writes to different shards can result in an inconsistent global state.

#### First-Principles Solution
Understanding Write Skew

Write skew occurs when a transaction that updates one part of the database conflicts with another transaction updating a related but disjointed part. In a distributed system, this is exacerbated because each shard operates independently and may not communicate until after their transactions are committed. Even though each shard ensures its own data consistency through Serializable isolation (which allows transactions to run without conflicts within the same shard), the overall system can still suffer from global anomalies.

#### Example Scenario

#### Cons
ider an online store with an inventory split across two shards: one for men's clothing and another for women's clothing. If a transaction tries to update the stock count of a men's shirt, but simultaneously another transaction updates the stock of a women's dress, these changes will be applied independently within their respective shards. However, if both transactions commit without coordination, you end up with an inconsistent global state where one part of the system thinks the inventory has changed while the other does not.

Why Serializable Per Shard Fails

The Serializable isolation level ensures that no transaction can read “dirty” data—that is, data that has been written by a transaction but not yet committed. However, this isolation guarantee only holds within a single shard and doesn’t prevent anomalies when updates are made across different shards without coordination.

Architectural Fix

To address write skew in distributed systems like the one described, you need to implement global transactions or introduce some form of coordinated updates. Here’s how:

1. Global Transaction Approach

Implementing a global transaction manager that coordinates writes across all relevant shards can help ensure consistency. This approach involves:

Enforcing strict ordering of operations across all shards.
Ensuring that all related writes are part of the same transaction.
2. Distributed Locks

Using distributed locks to serialize access to critical sections of data can also mitigate write skew by ensuring that only one shard at a time can perform certain types of updates.

3. Eventual Consistency with Backfill

In scenarios where global transactions aren't feasible, you might opt for eventual consistency models and backfill mechanisms:

Eventual Consistency: Allow the system to converge on a consistent state over time.
Backfill: Periodically reconcile differences between shards to bring them into alignment.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Write skew is a common challenge in distributed systems, especially when data is sharded and operates under Serializable isolation at the shard level. While Serializable ensures local consistency within each shard, it fails to provide global transactional integrity. To address this, you must implement coordinated updates or use techniques like global transactions, distributed locks, or eventual consistency models.

#### Common mistakes
include:

Assuming that Serializable isolation at the shard level automatically results in a globally consistent system.
Neglecting to account for the potential for write skew and its consequences on user experience and data integrity.
Overlooking the importance of robust coordination mechanisms when operating across shards.

For follow-ups, you might explore how specific technologies like distributed databases (e.g., CockroachDB) handle these challenges or discuss trade-offs between different consistency models in real-world systems.

**Comparison:**

```
Shard	Transactions
men_clothing_shard	101:UPDATE_STOCK, 204:READ_STOCK
women_clothing_shard	307:UPDATE_STOCK, 502:READ_STOCK
Global Transaction Manager Actions
- Coordinate write operations
- Ensure global consistency
```

**Code:**

```python
# Example code to implement a simple global transaction
def perform_global_transaction(transactions):
    for txn in transactions:
        shard_id, operation = txn.split(":")
        if not coordinate_write(shard_id, operation):
            return False
    # All writes are coordinated; commit globally.
    commit_transactions(transactions)
    return True

def coordinate_write(shard_id, operation):
    # Implement logic to coordinate write across shards.
    pass

def commit_transactions(transactions):
    # Commit all transactions in a global manner.
    pass
```

**Example input / output:**

```text
Input:
- Men's clothing shard transaction: 101:UPDATE_STOCK
- Women's clothing shard transaction: 307:UPDATE_STOCK

Output:
- Transactions are not coordinated, leading to potential write skew.
- Global transaction manager is introduced and coordinates writes across shards.

Dry-run:
- Men's clothing shard: 101:UPDATE_STOCK -> COMMIT
- Women's clothing shard: 307:UPDATE_STOCK -> COMMIT
- All transactions are committed globally without conflicts.
```

**Data:**

```json
{
  "shards": {
    "men_clothing_shard": {
      "transactions": [
        "101:UPDATE_STOCK",
        "204:READ_STOCK"
      ]
    },
    "women_clothing_shard": {
      "transactions": [
        "307:UPDATE_STOCK",
        "502:READ_STOCK"
      ]
    }
  }
}
```

**Flow:**

1. Shard A
2. Transaction 1
3. Transaction 2
4. Shard B
5. Transaction 3
6. Transaction 4
7. Global Transaction Manager

