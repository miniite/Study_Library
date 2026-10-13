---
type: "practice"
difficulty: "Intermediate"
---

# You must enforce a per-tenant ingest quota (events/sec) across 20 collector gateways with a hard global cap. Compare a shared Redis token bucket against local buckets with periodic reconciliation, and what consistency you sacrifice under a Redis blip.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce per-tenant ingest quotas across multiple collector gateways while maintaining a global hard cap, we can compare two approaches: using a shared Redis token bucket versus using local buckets with periodic reconciliation. The key difference lies in how consistency is handled during a potential Redis outage.

#### Triaging the Question

The question is essentially asking us to evaluate two strategies for managing rate limits and ensuring that no tenant exceeds their allocated quota, while also respecting a global cap on the total number of events per second. The trade-offs we need to consider include complexity, performance, and consistency in the face of Redis outages.

#### Assumptions
Tenant Quotas: Each tenant has an individual ingest quota.
Global Cap: There is a hard limit on the total number of events that can be ingested across all tenants.
Redis: Used as a backend for token bucket management and state synchronization.
Collector Gateways: These are responsible for enforcing rate limits before sending data to storage.

#### First-Principles Solution
Shared Redis Token Bucket

A shared Redis token bucket approach involves maintaining the token bucket in one central Redis instance, which all collector gateways read from and write to. This ensures that each gateway can accurately enforce tenant quotas globally.

Steps:

Initialization: Each gateway initializes its local state by fetching current tokens and remaining quota information from Redis.
Token Request: When a new event arrives, the gateway requests a token from Redis.
Token Granting: Redis grants or denies the request based on the available tokens and tenant quotas.
Periodic Reconciliation: Periodically, all gateways reconcile their local states with Redis to ensure consistency.

Advantages:

Ensures global consistency in quota enforcement.
Simplifies rate limit management as it is handled by a single entity (Redis).

Disadvantages:

High latency due to network communication between gateways and Redis.
Single point of failure if the central Redis instance goes down, leading to potential data inconsistencies.
Local Buckets with Periodic Reconciliation

In this approach, each gateway maintains its own token bucket locally. At regular intervals, these buckets are reconciled with a shared Redis instance for synchronization purposes.

Steps:

Initialization: Each gateway initializes its local state based on initial values from Redis.
Token Request: When a new event arrives, the gateway requests a token from its local bucket.
Local Granting: The gateway decides whether to grant or deny the request based on its local state.
Periodic Reconciliation: Periodically, gateways reconcile their local states with Redis to ensure consistency.

Advantages:

Reduced latency as tokens are granted locally without network communication.
Higher resilience in case of temporary network outages or issues with Redis.

Disadvantages:

Potential for inconsistent state across gateways if a gateway fails to sync due to an outage.
Increased complexity in managing local states and ensuring global consistency during synchronization.

#### Cons
istency Under a Redis Blip

When Redis is down, the shared token bucket approach will have all gateways waiting indefinitely, leading to potential loss of events. In contrast, the local buckets with periodic reconciliation strategy can continue operating as long as at least one gateway manages to sync its state with Redis before the next synchronization interval.

#### Closing Summary
and Common Mistakes
Shared Token Bucket: Best for scenarios where global consistency is critical but network latency is a concern.
Local Buckets with Periodic Reconciliation: More robust against temporary outages, but requires careful handling of local states to maintain overall consistency.

#### Common mistakes
include not adequately considering the impact of Redis failures or underestimating the complexity of state synchronization. Always ensure that your solution can handle edge cases and provide graceful degradation during disruptions.

**Comparison:**

```
Tenant	Quota (tokens/sec)
tenant1	50
tenant2	30
State Synchronization Interval (s)	Frequency of Reconciliation
60	Every minute
```

**Code:**

```python
# Example code for shared token bucket approach

import redis

class TokenBucket:
    def __init__(self, redis_client, tenant_id, quota):
        self.redis = redis_client
        self.tenant_id = tenant_id
        self.quota = quota

    def get_token(self):
        # Request a token from Redis
        result = self.redis.zrangebyscore(f'token_bucket:{self.tenant_id}', 0, float('inf'))
        if not result:
            return False  # No tokens available
        else:
            self.redis.zrem(f'token_bucket:{self.tenant_id}', result[0])  # Remove the token
            return True

    def update_state(self):
        # Periodic reconciliation with Redis
        current_state = self.redis.hgetall(f'state:{self.tenant_id}')
        if current_state:
            self.quota = int(current_state.get('quota', 0))

# Example usage
redis_client = redis.Redis(host='localhost', port=6379, db=0)
bucket = TokenBucket(redis_client, 'tenant1', 50)  # Assuming a quota of 50 tokens/sec

event_received = bucket.get_token()
if event_received:
    print("Event processed")
else:
    print("Quota exceeded")
```

**Example input / output:**

```text
# Example dry-run

redis_client = redis.Redis(host='localhost', port=6379, db=0)

# Initialize state for tenant1
bucket_tenant1 = TokenBucket(redis_client, 'tenant1', 50)
bucket_tenant1.update_state()

print(bucket_tenant1.get_token())  # True if a token is available

redis_client.flushall()  # Simulate Redis outage

# Reinitialize after a delay (simulating sync)
bucket_tenant1 = TokenBucket(redis_client, 'tenant1', 50)
bucket_tenant1.update_state()

print(bucket_tenant1.get_token())  # False if no tokens are available
```

**Data:**

```json
{
  "token_bucket": {
    "tenant1": {
      "quota": 50,
      "tokens": [1628430000, 1628430001, 1628430002]
    },
    "tenant2": {
      "quota": 30,
      "tokens": []
    }
  },
  "state": {
    "tenant1": {
      "quota": 50
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Collector Gateway
4. Request Token
5. Token Available?
6. Process Event
7. Wait for Token
8. Check Redis State
9. Sync with Redis
10. Re-evaluate Token Request

