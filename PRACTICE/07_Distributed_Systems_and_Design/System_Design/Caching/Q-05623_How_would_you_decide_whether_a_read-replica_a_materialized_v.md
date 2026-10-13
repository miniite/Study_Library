---
type: "practice"
difficulty: "Deep"
---

# How would you decide whether a read-replica, a materialized view, or a cache best serves a read-heavy analytical query, and when would you combine two of them?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache-aside and Read Replicas](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache-aside_and_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide which technology—read-replica, materialized view, or cache—to use for a read-heavy analytical query, we need to consider several factors including performance, cost, storage needs, and data freshness requirements. This decision often involves evaluating multiple aspects of the system architecture and operational constraints.

First, let's understand what each option entails:

Read-replica: A replica database that mirrors the primary database but is lagged behind to ensure consistency.
Materialized view: A precomputed query result stored as a table in the database. It can significantly reduce read latency by avoiding complex joins and aggregations.
Cache: Typically a high-speed, volatile memory layer used for caching frequently accessed data.

Now, let's break down the decision-making process:

Data Freshness:

Read-replica: Useful when near-real-time data is acceptable but with some latency.
Materialized view: Provides more up-to-date results compared to a read-replica because it can be updated more frequently.
Cache: Offers the freshest data, especially if updates are infrequent.

Read Performance:

Read-replica: Lower performance due to replication lag but still faster than querying the primary database.
Materialized view: High performance as it avoids complex query execution.
Cache: Highest performance with almost instant reads, but data staleness could be an issue.

Cost and Storage:

Read-replica: Additional cost for maintaining a replica, but less storage compared to materializing views or caching everything.
Materialized view: Requires additional storage space proportional to the size of the views.
Cache: Low storage overhead but potentially higher operational costs.

Complexity:

Read-replica: Simple to set up and maintain for basic scenarios.
Materialized view: More complex setup, especially with dynamic queries or frequent schema changes.
Cache: Requires more engineering effort for data invalidation strategies but is generally simpler than materializing views.

Combination Scenarios:

Read-replica + Cache: Use a read-replica to reduce load on the primary database and serve most reads from cache, ensuring lower latency.
Materialized view + Cache: Precompute frequently accessed queries as materialized views and cache them for ultra-fast access.

#### Example Scenario

Suppose we have an e-commerce platform with frequent analytical queries like "top-selling products by category" or "customer purchase history". These queries are read-heavy but can tolerate some staleness. Here’s how we might combine technologies:

Read-replica:
Use a lagged read-replica to handle most reads, reducing load on the primary database.
Materialized views:
Precompute and materialize frequently accessed query results (e.g., top-selling products by category) for even faster access.
Cache:
Cache intermediate query results in memory to serve requests that bypass read-replica and materialized views.

#### Closing Summary

Deciding between a read-replica, materialized view, or cache involves balancing data freshness, performance, cost, and complexity. Combining multiple technologies can optimize the overall system by leveraging each method’s strengths where appropriate.

#### Common Mistakes

Overusing caching for complex queries that benefit more from precomputation.
Ignoring data freshness requirements when choosing a solution.
Not considering operational costs in terms of storage and maintenance overhead.

**Comparison:**

```
Technology	Data Freshness	Read Performance	Cost and Storage	Complexity
Read-replica	Low to Medium	Moderate	Low to High	Low-Medium
Materialized view	Medium-High	High	High	High
Cache	Very High	High	High	High
```

**Code:**

```python
# Example Python code to initialize read-replica connection
import psycopg2

def setup_read_replica():
    replica_conn = psycopg2.connect(
        dbname="replica_db",
        user="user",
        password="password",
        host="replica_host",
        port=5432
    )
    return replica_conn

# Example code to fetch data from a materialized view
def get_top_selling_products():
    cursor = replica_conn.cursor()
    cursor.execute("SELECT * FROM top_selling_products")
    return cursor.fetchall()

# Example caching mechanism using Redis
import redis

cache_client = redis.Redis(host='localhost', port=6379, db=0)

def fetch_from_cache(key):
    cached_data = cache_client.get(key)
    if cached_data:
        print("Data fetched from cache.")
        return cached_data.decode('utf-8')
    else:
        # Fetch data and update the cache
        data = get_top_selling_products()
        cache_client.set(key, data)
        return data
```

**Example input / output:**

```text
Input:
- Read from cache or materialized view if available.
Output:
- Fetched data (e.g., top-selling products) with minimal latency.

Example Dry-run:
Request:
GET /top-products

Response:
[
  {"product_id": 1, "name": "Product A", "quantity_sold": 1000},
  ...
]
```

**Data:**

```json
{
  "read_replica_connection_details": {
    "dbname": "replica_db",
    "user": "user",
    "password": "password",
    "host": "replica_host",
    "port": 5432
  },
  "materialized_view": "top_selling_products"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Cache Miss
8. Yes
9. No
10. Read-Heavy Query
11. Check Read-replica Availability
12. Fetch from Read-replica
13. Check Cache Availability
14. Fetch from Cache
15. Check Materialized ViewAvailability
16. Fetch from Materialized View
17. Execute Query on Primary DB
18. Fetch from Read-replica
19. Check Cache Availability
20. Fetch from Cache and UpdateReplica
21. Execute Query on Primary DB

