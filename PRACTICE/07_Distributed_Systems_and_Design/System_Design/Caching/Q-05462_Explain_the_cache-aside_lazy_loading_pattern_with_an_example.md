---
type: "practice"
difficulty: "Intermediate"
---

# Explain the cache-aside (lazy loading) pattern with an example.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache-aside and Read Replicas](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache-aside_and_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cache-aside, also known as lazy loading, is a design pattern used in distributed systems where data is cached at the application layer to reduce database load and improve performance. This approach ensures that cache updates are handled carefully without risking data inconsistencies. Here's how it works:

#### Overview
When a request comes in for some data, the system first checks if the data exists in the cache. If not (the "cache-aside" part), it retrieves the data from the database and stores it in the cache before returning it to the user. This ensures that while the data is being fetched, any changes made elsewhere are still reflected correctly.

#### Triage
The question asks you to explain the cache-aside pattern with an example. You should cover how this pattern handles both reads and writes, ensuring consistency between the database and the cache.

#### First-principles Solution

Read Operation: When a read request is made for data that exists in the cache, it's served directly from the cache.
Write Operation: Any write operation updates the database first, then invalidates or updates the relevant entries in the cache to reflect the change.

#### Example Scenario
Consider an e-commerce application where products are frequently accessed and updated. When a user requests information about a product (say, Product A), the system checks if it's already cached. If not, it queries the database for the latest data, stores it in the cache, and then returns the data to the user.

Key Considerations:

Cache Invalidation: When a write operation updates the database, you need mechanisms (like timestamps or version numbers) to ensure that all caches are invalidated or updated.

#### Cons
istency Models: Choose appropriate consistency models based on your application's requirements. For instance, strong consistency might be required in financial transactions but not as much in read-heavy applications like social media.

#### Closing Summary
and Common Mistakes: The cache-aside pattern is a powerful tool for managing data across distributed systems, ensuring that reads are fast while writes maintain database integrity. However, common mistakes include inadequate cache invalidation mechanisms leading to stale data and poor choice of consistency models causing performance bottlenecks or overcomplication.

**Comparison:**

```
Consistency Model	Description
Strong	Ensures all nodes see the same state. Useful in financial applications but can be costly in terms of performance.
Eventual	Data is consistent across all nodes eventually, typically used for read-heavy workloads where some staleness is acceptable.
```

**Code:**

```python
def get_product(product_id):
    # Check if product is in the cache
    cached_product = cache.get(f"product:{product_id}")
    if cached_product:
        return cached_product
    else:
        # Fetch from database and update cache
        db_product = db.query_product_by_id(product_id)
        cache.set(f"product:{product_id}", db_product, timeout=3600)  # Cache for an hour
        return db_product

def update_product(product_id, new_data):
    # Update the database first
    db.update_product(product_id, new_data)

    # Invalidate or update the cache
    cache.delete(f"product:{product_id}")  # Simple invalidation strategy
```

**Example input / output:**

```text
Input:
- GET /product/123

Output:
- Cached Product A details (if in cache)
- Database query executed and Product A details returned from database, then cached for an hour.

Input:
- PUT /product/123
  {
    "name": "Updated Product",
    "description": "New description"
  }

Output:
- Database updated with new product data.
- Cache entry for product:123 deleted to ensure next request fetches the latest data.
```

**Data:**

```json
{
  "cache": {
    "type": "redis",
    "options": {
      "host": "localhost",
      "port": 6379,
      "db": 0
    }
  },
  "db": {
    "type": "mysql",
    "options": {
      "host": "localhost",
      "user": "root",
      "password": "password",
      "database": "ecommerce"
    }
  }
}
```

**Flow:**

1. Yes
2. Data Served
3. No
4. Data Retrieved
5. Data Sent
6. Yes
7. Read Request
8. Cache Check
9. Cache Hit?
10. Database Not Accessed
11. Return Cache Data
12. Database Accessed
13. Update Cache
14. Return Data
15. Write Operation
16. Cache Invalidation
17. Delete Cache Entry

