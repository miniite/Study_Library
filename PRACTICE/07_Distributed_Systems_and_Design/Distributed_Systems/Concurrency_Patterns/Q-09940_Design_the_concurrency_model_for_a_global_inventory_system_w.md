---
type: "practice"
difficulty: "Expert"
---

# Design the concurrency model for a global inventory system where oversell has real dollar cost but strong global serialization would kill throughput at flash-sale scale. How do you partition, reserve, and reconcile so correctness holds while each region runs largely independently?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-service Invalidation Races](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR

In designing a concurrency model for a global inventory system that needs to balance between strong consistency (to avoid overselling) and high throughput during flash sales, we need to carefully partition the system into regions while ensuring correctness through robust reservation mechanisms. The key is to use a hybrid approach that balances distributed transactions with local optimizations.

#### Triaging the Question

The interviewer is probing our understanding of concurrency patterns, race conditions, and how to implement them in a global, distributed system context. We need to consider both the business logic (avoiding overselling) and technical constraints (high throughput during flash sales). The assumptions here include that inventory is partitioned by regions, each region has local autonomy, and we aim for correctness through careful design.

#### First-Principles Solution
1. Partition Inventory by Regions

We will divide the global inventory across multiple regions to ensure that each region can operate independently while still maintaining a cohesive view of the overall inventory. Each region will handle its own local transactions and updates, which helps in achieving high throughput during flash sales.

2. Reservation Mechanism

To prevent overselling, we implement a reservation mechanism where a client must reserve an item before it is allowed to purchase it. This ensures that each transaction is atomic and consistent across regions. The reservation can be implemented using either distributed locks or a more complex consensus algorithm like Raft or Paxos.

3. Conflict Resolution

For cases where multiple regions try to sell the same item simultaneously, we need a conflict resolution strategy. A common approach is to use a global timestamp or versioning mechanism to decide which transaction wins. This can be implemented using a distributed coordination service that maintains a globally consistent view of transactions.

4. Local Optimizations

To maximize throughput during flash sales, each region should have its own database and processing pipeline. However, we need to ensure that these local optimizations do not compromise the global integrity of the inventory system. This can be achieved by periodically reconciling inventories between regions to maintain consistency.

#### Implementation
Details
1. Region-Based Partitioning

Assume our global inventory is partitioned into ( N ) regions, each responsible for a subset of items. Each region has its own database and processing pipeline.

Global Inventory
Partition by Regions
N regions
Region 1
Region 2
Region N
2. Reservation Mechanism Using Distributed Locks

We use distributed locks to ensure that only one transaction can reserve an item at a time, even when multiple transactions are running concurrently in different regions.

True
False
Client Request
Check Inventory
Lock Item
Purchase Item
Wait for Lock
3. Conflict Resolution with Timestamps

During a flash sale, if multiple regions try to sell the same item simultaneously, we use timestamps to resolve conflicts. The transaction that started first (based on the timestamp) gets the item.

Item Sold
Item Not Sold
Flash Sale
Multiple Reservations
Select Winner by Timestamp
Purchase Confirmed
Conflict Resolution Failed

#### Closing Summary
and Common Mistakes

In summary, the key to designing a concurrency model for this global inventory system is to balance strong consistency with high throughput. By partitioning inventory by regions and implementing robust reservation mechanisms, we can achieve both goals. The main challenges are ensuring correctness during conflicts and maintaining performance under heavy load.

#### Common mistakes
include over-reliance on global serialization, which can lead to low throughput; or ignoring the need for local optimizations, leading to unnecessary complexity in reconciliation processes.

Self-Check

This solution addresses the core requirements while considering edge cases and potential failure modes. The design ensures both correctness and performance during flash sales.

**Comparison:**

| Region | Items  | Reservations |
|--------|--------|--------------|
| R1     | A(5) B(3) C(4) D(2) | 1, 1, 0, 0   |
| R2     | E(6) F(7) G(8) H(9) | 0, 0, 1, 0   |

**Code:**

```
Example of a distributed lock implementation using Redis

import redis

client = redis.Redis(host='localhost', port=6379, db=0)

def reserve_item(item_id): with client.lock(f'item:{item_id}:lock'): # Check if item is available if not client.get(f'item:{item_id}'): # Reserve the item and update inventory client.set(f'item:{item_id}', 1) return True else: return False

def purchase_item(item_id): if reserve_item(item_id): # Process purchase pass

CODE
Copy

```python
# Conflict resolution with timestamps
import time

def resolve_conflict(timestamps, item_id):
    max_timestamp = max(timestamps, key=lambda x: x[1])[0]
    return max_timestamp == item_id
```

**Example input / output:**

| Client Request | Response           |
|----------------|--------------------|
| Reserve item_a | Reserving item_a   |
| Purchase item_a| Item reserved      |
|               | Purchased item_a   |
| Reserve item_b | Conflict with client_2 |
| Purchase item_b| Conflict resolution failed |

**Data:**

```json
{
  "inventory": {
    "region_1": {"item_a": 10, "item_b": 5},
    "region_2": {"item_c": 7, "item_d": 3}
  },
  "reservations": {
    "client_1": {"item_a": [time.time(), "region_1"]},
    "client_2": {"item_b": [time.time(), "region_1"]}
  }
}
```

**Flow:**

1. True
2. False
3. Client Request
4. Check Inventory
5. Purchase Item
6. Wait for Lock
7. Acquire Lock
8. Purchase Item

