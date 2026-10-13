---
type: "practice"
difficulty: "Expert"
---

# A single hot key is saturating one Redis shard's CPU. Walk me through detection, immediate mitigation, and a permanent fix.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Failover and Resilience](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Cache_Failover_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When a single hot key is saturating a Redis shard's CPU, we need to detect the issue, mitigate its impact immediately, and address it permanently by optimizing our caching strategy or improving the key distribution.

Triaging the Problem

The question revolves around detecting an overload on a specific Redis shard due to excessive traffic on a single cache key. The interviewer is probing your ability to diagnose problems under heavy load, implement temporary fixes to stabilize performance, and develop long-term solutions to prevent future issues.

#### Assumptions
We are dealing with Redis 6.x or later, which supports more advanced metrics.
The system has access to Redis monitoring tools like redis-cli, Prometheus, Grafana, or any similar telemetry solution.
There is a logging mechanism in place to capture and analyze the incoming requests.
Detection

Monitor CPU Usage:

Utilize Redis's built-in commands such as INFO REPLICATION and MONITOR to check real-time data.
Use external monitoring tools like Prometheus, Grafana, or custom scripts to periodically poll Redis metrics via INFO REPLICATION, INFO CPU, and INFO MEMORY.

Analyze Logs:

Check application logs for patterns indicating the hot key(s) causing issues.
Look for repeated access times or high-frequency operations on a single key.

Performance Indicators:

Monitor the overall health of Redis using redis-cli:
SH
redis-cli INFO REPLICATION
Check the CPU usage specifically:
SH
redis-cli INFO REPLICATION | grep used_cpu_sys
Immediate Mitigation

Circuit Breaker:

Implement a circuit breaker pattern to temporarily block or throttle requests for the hot key.

#### Example
using a library like Hystrix in Java:
JAVA
CircuitBreaker circuitBreaker = circuitBreakerRegistry.getCircuitBreaker("hotKeybreaker");

if (circuitBreaker.isOpen()) {
    return "Service Temporarily Unavailable";
}

// Proceed with normal request handling.

Throttling:

Use rate limiting or token bucket algorithms to limit the number of requests per second for the hot key.
Implement this using Redis's ACL (Access Control List) for finer-grained control:
SH
redis-cli ACL SETUSER myuser on +@all -get hotKey

Fallback Mechanism:

Define a fallback strategy where requests to the hot key are redirected to a slower but more reliable service or database.

#### Example
in Python:
PYTHON
def get_hot_key():
    # Normal cache retrieval logic
    if some_condition:
        return fallback_get()

def fallback_get():
    # Fallback to slower but stable service
    pass
Permanent Fix

Optimize Caching Strategy:

Analyze the caching strategy and ensure it is aligned with actual use cases.
Use Redis's eviction policies like LRU (Least Recently Used) or LFU (Least Frequently Used) to avoid holding onto frequently accessed keys.

Key Distribution:

Re-evaluate key distribution across shards.
Ensure that the hot key is spread evenly across multiple shards using consistent hashing or random distribution techniques.

#### Example
of consistent hash ring implementation:
PYTHON
class ConsistentHashRing:
    def __init__(self, nodes):
        self.nodes = sorted(nodes)
        self.ring = {hash(node): node for node in self.nodes}

    def get_node(self, key):
        hash_key = hash(key) % (2 ** 32)
        for i in range(len(self.nodes)):
            if hash_key < list(self.ring.keys())[i]:
                return list(self.ring.values())[i]

ring = ConsistentHashRing(['node1', 'node2', 'node3'])
node = ring.get_node('hotKey')

Load Balancing:

Implement a load balancer to distribute requests more evenly across Redis instances.
Use HAProxy or Nginx as reverse proxies to manage traffic distribution.

Scaling Strategy:

#### Cons
ider scaling the Redis cluster by adding more nodes.
Evaluate if a distributed cache solution like Memcached or an external caching service could be beneficial for higher load scenarios.

#### Closing Summary
and Common Mistakes

By detecting, mitigating, and fixing the issue, we ensure that the system remains responsive under high loads. Key mistakes to avoid include over-reliance on caching without proper monitoring and inadequate fallback strategies. Always validate solutions with real-world data and stress test them before implementing in production.

Common Follow-Ups
How do you handle cache invalidation?
What are your thoughts on using a hybrid approach with both Redis and Memcached?
Can you discuss the trade-offs between different caching eviction policies?

**Comparison:**

```
Key	Value
Hot Key	/api/user/profile
Frequency	100 requests/s
Fallback URL	http://backupdb.example.com/api/user/profile
```

**Code:**

```

// Example Circuit Breaker Implementation CircuitBreaker circuitBreaker = circuitBreakerRegistry.getCircuitBreaker("hotKeybreaker"); if (circuitBreaker.isOpen()) { return "Service Temporarily Unavailable"; }

CODE
Copy

```python
# Token Bucket Rate Limiter
import time

class TokenBucket:
    def __init__(self, capacity=1000):
        self.capacity = capacity
        self.tokens = 0
        self.rate = 50  # Tokens per second
        self.last_time = time.time()

    def consume(self, tokens_needed):
        now = time.time()
        elapsed_seconds = now - self.last_time
        self.tokens += min(elapsed_seconds * self.rate, self.capacity)
        if tokens_needed <= self.tokens:
            self.tokens -= tokens_needed
            return True
        else:
            return False

# Example usage
bucket = TokenBucket(capacity=1000)
if not bucket.consume(50):
    # Handle rate limit exceeded
```

**Example input / output:**

```text
# Example Redis INFO output
INFO REPLICATION
used_cpu_sys:85

# Monitoring System Alert
Alert: Hot key /api/user/profile causing high CPU usage, current used_cpu_sys is 85%
```

**Data:**

```json
{
  "hotKey": "/api/user/profile",
  "frequency": {
    "requestsPerSecond": 100,
    "lastRequestTime": "2023-10-01T14:00:00Z"
  },
  "fallbackService": "http://backupdb.example.com/api/user/profile"
}
```

**Flow:**

1. Yes
2. No
3. Monitor Redis CPU
4. CPU > Threshold?
5. Circuit Breaker Open
6. Fallback Mechanism
7. Check Logs for Patterns
8. Distribute Load Evenly

