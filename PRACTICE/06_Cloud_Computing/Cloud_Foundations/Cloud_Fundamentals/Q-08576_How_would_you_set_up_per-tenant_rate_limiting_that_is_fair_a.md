---
type: "practice"
difficulty: "Deep"
---

# How would you set up per-tenant rate limiting that is fair across a fleet of stateless API instances, and where would you store the counters?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Tenant_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up per-tenant rate limiting in a scalable and stateless architecture, we need to ensure fairness among tenants while managing distributed counter states efficiently. This involves setting up a consistent hashing mechanism to distribute tenant requests across API instances and using a distributed storage system to track the rate limits.

The core idea is that each tenant's request will be hashed to determine which API instance handles it. The state of the rate limit (i.e., the number of allowed requests per time window) for each tenant must be stored in a way that can be accessed by any API instance, ensuring fairness and consistency. We will use a distributed database like DynamoDB or Redis with a strong consistency model to store these counters.

#### Triage

The question is probing how you would design a system that ensures fair rate limiting per-tenant across multiple stateless instances. This involves understanding the trade-offs between performance, scalability, and consistency in distributed systems. Key assumptions include:

Each tenant has a unique identifier.
Requests are made to the API by tenants.
The rate limits need to be enforced based on tenant identity.

#### First-Principles Solution

Request Handling: When a request comes in, we hash the tenant ID using a consistent hashing function (like MurmurHash or CRC32) to determine which API instance should handle it. This ensures that requests from the same tenant are always handled by the same API instance, maintaining state consistency.

Rate Limit Storage: We use a distributed storage system like Amazon DynamoDB or Redis with a consistent data model (e.g., hash tables). Each tenant has their own rate limit counter stored in this system. The counter is incremented for each request and reset periodically to enforce the time window constraint.

Read/Write Operations:

Increment Counter: When an API instance receives a request, it checks if the corresponding tenant's rate limit counter exists. If not, it initializes it.
Check Limit: Before processing the request, the API instance fetches the current value of the rate limit counter and decrements it by one.
Persist Change: The new value is then persisted back to the storage system.

Concurrency Control: To handle concurrent access and prevent race conditions, we use atomic operations provided by the distributed database. For example, in Redis, we can use GET-SET commands or Lua scripts for safe counter updates.

Backup Strategies: In case of failures, a backup strategy like leader election or replication can be implemented to ensure data integrity and availability.

#### Closing Summary

In summary, setting up per-tenant rate limiting involves consistent hashing to route requests to the correct API instances and using distributed databases with strong consistency guarantees for storing and managing tenant-specific rate limits. The key is ensuring that all operations are atomic and that state across instances remains coherent.

#### Common Mistakes/Follow-Ups
Not Using Consistent Hashing: Not properly distributing tenants' requests can lead to an uneven load distribution.
Lack of Concurrency Control: Failing to use atomic operations can result in race conditions, leading to incorrect rate limit enforcement.
Choosing the Wrong Data Store: Selecting a distributed database without considering its consistency model and scalability might lead to performance bottlenecks or data integrity issues.

For follow-ups, you might be asked about specific technologies, trade-offs between consistency and availability, or how this solution scales with increasing tenant numbers and request rates.

**Comparison:**

```
Tenant ID	Current Rate Limit
tenant-123	8
tenant-456	7
Instance ID	Rate Limits Counted
api-instance-0	1
api-instance-1	2
api-instance-2	3
```

**Code:**

```python
import redis

# Initialize Redis client
client = redis.StrictRedis(host='localhost', port=6379, db=0)

def get_instance_for_tenant(tenant_id):
    # Consistent hashing function (example using CRC32)
    hash_value = int.from_bytes(crc32(tenant_id.encode()), 'big') % num_instances
    return f"api-instance-{hash_value}"

def handle_request(tenant_id, request_type):
    instance = get_instance_for_tenant(tenant_id)

    with client.pipeline() as pipe:
        # Check and decrement the rate limit counter
        key = f"rate-limit:{tenant_id}"
        current_count = pipe.get(key)

        if current_count is None or int(current_count) <= 0:
            return "Rate limit exceeded"

        new_count = int(current_count) - 1
        pipe.decr(key, amount=1)
        pipe.set(key, new_count)

        # Process the request (for demonstration purposes)
        response = f"Processed {request_type} by tenant: {tenant_id}"

    # Execute pipeline to ensure atomicity
    pipe.execute()

    return response
```

**Example input / output:**

```

Input:

Request type: GET /api/v1/data
Tenant ID: tenant-123

Output:

PLAINTEXT
Copy
Processed GET /api/v1/data by tenant: tenant-123
rate-limits: {
  "tenant-123": 8,
  "tenant-456": 7
}
```

**Data:**

```json
{
  "rate-limits": {
    "tenant-123": 9,
    "tenant-456": 7
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Request
5. Is tenant ID consistent?
6. Hash tenant ID
7. Get API instance from hash
8. Fetch rate limit counter fortenant
9. Decrement and persist counter
10. Process request
11. Return response
12. Check if tenant has anyrequests
13. Initialize rate limit counter
14. Fallback to default handler orlog error

