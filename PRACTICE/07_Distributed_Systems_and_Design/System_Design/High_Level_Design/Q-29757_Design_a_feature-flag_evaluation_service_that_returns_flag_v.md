---
type: "practice"
difficulty: "Deep"
---

# Design a feature-flag evaluation service that returns flag values in under 1ms server-side with gradual percentage rollouts and targeting rules — how do you distribute config and keep evaluation deterministic per user?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To design a high-level feature-flag evaluation service that returns flag values in under 1ms server-side with gradual percentage rollouts and targeting rules, we need to focus on distributed configuration management, deterministic user-based flag evaluations, and efficient cache strategies. The solution involves distributing the configuration data across multiple nodes, using consistent hashing for user assignment, implementing a combination of local caching and remote querying mechanisms, and ensuring idempotent operations.

Question Triage: The interviewer is probing your understanding of high-level design principles, distributed systems architecture, and optimization techniques to ensure low-latency feature flag evaluations. Key assumptions include the use of a scalable backend service with multiple instances, user identification for targeting rules, and consistent hashing to handle user distribution across nodes efficiently.

#### First-Principles Solution

Distributed Configuration Management:

Use a distributed configuration storage system like Etcd or Consul to store feature flag configurations.
Ensure the configuration data is version-controlled to manage rollouts and rollbacks effectively.

User Assignment with Deterministic Hashing:

Implement consistent hashing for assigning users to specific nodes based on their unique identifier (e.g., UUID, username).
This ensures that user assignments are predictable and do not change frequently, leading to stable caching behavior.

Caching Mechanism:

Use in-memory caches like Redis or Memcached to store the evaluated flag values for users.
Implement a read-through and write-behind cache strategy to ensure that cache updates are consistent with the distributed configuration storage.

Remote Query vs Local Cache:

For each request, first check the local cache (Redis/Memcached).
If not found in the cache, perform a remote query to the distributed configuration store.
Upon successful evaluation, update the local cache and return the result to the user.

Deterministic Evaluation with Targeting Rules:

Implement logic to apply targeting rules based on user attributes (e.g., role, location).
Ensure that the same user always gets the same flag value during a session by using a deterministic algorithm for evaluation.

Concurrency and Idempotency:

Handle concurrent requests from different nodes without race conditions.
Ensure idempotent operations to avoid conflicts when multiple nodes evaluate flags simultaneously.

Scalability and Fault Tolerance:

Design the system to handle increased load by adding more instances of the feature flag service.
Implement health checks and failover mechanisms to ensure high availability.

Monitoring and Logging:

Set up monitoring tools like Prometheus for latency metrics.
Use ELK stack or similar for logging to track failures and performance issues.

#### Closing Summary
The design ensures that each user gets consistent flag values within 1ms by leveraging distributed configuration management, deterministic hashing, caching strategies, and remote querying. This approach balances between reducing server load through efficient local caching while ensuring consistency with the centralized configuration store.

#### Common Mistakes/Follow-Ups

Mistake: Failing to consider user distribution strategies like consistent hashing.
Mistake: Not handling concurrency issues effectively, leading to race conditions or conflicts during evaluation.

#### Follow-Up
Discussing edge cases such as what happens when the distributed configuration store is down and how you ensure failover.

**Comparison:**

```
User ID	Flag Name	Value
1234567890abcdef	beta-feature	True
```

**Code:**

```python
from redis import Redis
import etcd3

# Initialize cache and client
cache = Redis(host='localhost', port=6379)
etcd_client = etcd3.client()

def get_flag_value(user_id, flag_name):
    # Check local cache first
    cached_value = cache.get(flag_name + user_id)
    if cached_value:
        return cached_value.decode('utf-8')

    # Perform remote query to configuration store
    value = etcd_client.get(f'/feature-flags/{flag_name}/{user_id}')
    if not value:
        return None

    # Update local cache
    cache.set(flag_name + user_id, value.value)
    return value.value.decode('utf-8')
```

**Example input / output:**

```text
**Input**: `get_flag_value(1234567890abcdef, 'beta-feature')`
**Output**: `True` (if the flag is active for that user based on targeting rules)
```

**Data:**

```yaml
user_id: 1234567890abcdef
flag_name: beta-feature
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Is value in cache?
7. Return cached value
8. Query configuration store
9. Value found?
10. Store result in cache andreturn
11. Flag not active, return None

