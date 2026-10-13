---
type: "practice"
difficulty: "Deep"
---

# Design a distributed cache with a write-through vs write-around vs write-back policy for a hot product-inventory key. Which do you ship for correctness-critical stock, and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-tenant Migrations](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a distributed cache system, one critical decision is choosing between different write policies—write-through, write-around, and write-back. For a hot product-inventory key in a correctness-critical stock environment, we must prioritize ensuring that the main database remains up-to-date with minimal latency issues. In this scenario, write-through would be the optimal choice.

#### Triage
Restating the Question and Probing

The question is asking us to evaluate three write policies for a distributed cache system in an e-commerce context where product inventory data must remain correct and consistent across all nodes. The key aspects being probed are:

Write-through: Ensures that any write operation also updates the main database.
Write-around: Bypasses the main database, only updating the cache.
Write-back: Updates the cache first, then asynchronously updates the main database.
First Principles: Reasoning Step-by-Step

Correctness-Critical Nature:

In a correctness-critical environment like managing inventory for an e-commerce product, it is crucial that every write operation in the distributed system eventually reaches the main database to ensure data consistency.

Latency Considerations:

Write-through introduces additional latency due to the main database being updated simultaneously with the cache. However, this ensures no divergence between cache and database states.

Write-around and Write-back:

Write-around: This could lead to inconsistencies if a node fails or the network drops packets. It’s less reliable but can improve performance in some scenarios.
Write-back: While it can reduce write latency, it risks data loss during power failures or system crashes without proper synchronization mechanisms.
Full Solution: Write-Through Policy

Given these considerations, write-through is the best choice for a correctness-critical stock environment. This policy ensures that every write operation updates both the cache and the main database simultaneously. Here’s why:

Data Consistency: Ensures data consistency by always updating the main database, reducing the risk of divergence between the cache and the database.
Reliability: Minimizes risks associated with partial writes or inconsistencies in other policies.
Auditability: Easier to audit as all write operations are recorded in the main database.

#### Closing Summary
+ Common Mistakes/Follow-ups

Key Takeaways:

Write-through is essential for maintaining data consistency and reliability, making it suitable for correctness-critical applications like stock management.
While other policies can offer performance benefits, they introduce risks that make them unsuitable in environments where data integrity is paramount.

#### Common Mistakes

Failing to update the main database in write-through scenarios can lead to inconsistent states across nodes.
Overlooking the need for robust network and system reliability checks when choosing write-around or write-back policies.
Common Follow-ups
How do you handle network failures during a write operation?
What mechanisms would you implement to ensure eventual consistency in a distributed cache with write-through policy?
How does your solution scale with high traffic and write load?

**Comparison:**

```
Node	Action
Cache	SET operation for updating cache key
Main DB	UPDATE operation to update database
```

**Code:**

```python
# Example of a simplified write-through cache update logic
class DistributedCache:
    def __init__(self, main_db):
        self.main_db = main_db

    def update_inventory(self, product_id, new_quantity):
        # Simulate updating both the cache and the main database
        cache_key = f"inventory:{product_id}"

        try:
            # Update cache (this is just a placeholder)
            cache.set(cache_key, new_quantity)

            # Write to main DB
            self.main_db.update_inventory(product_id, new_quantity)

            return "Update successful"
        except Exception as e:
            return f"Failed: {str(e)}"
```

**Example input / output:**

```text
Input:
- main_db: Relational Database (MySQL)
- cache: Redis

Output:
1. Update request received for product_id: 456, new_quantity: 100.
2. Cache operation: SET "inventory:456" -> 100
3. DB Operation: UPDATE inventory_table SET quantity = 100 WHERE product_id = 456
4. Response: "Update successful"
```

**Data:**

```json
{
  "main_db": {
    "type": "RelationalDB",
    "host": "db.example.com",
    "port": 3306,
    "user": "root",
    "password": "securepass123"
  },
  "cache": {
    "backend": "Redis",
    "nodes": [
      {"host": "cache1.example.com", "port": 6379},
      {"host": "cache2.example.com", "port": 6379}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Retry
6. Fail
7. Receive Write Request
8. Is Main DB Available?
9. Update Main DB
10. Queue Write Request
11. Write-to-Cache
12. Send Response: 'UpdateSuccessful'
13. Write to Cache Only
14. Retry Policy
15. Log Error and Queue for LaterAttempt

