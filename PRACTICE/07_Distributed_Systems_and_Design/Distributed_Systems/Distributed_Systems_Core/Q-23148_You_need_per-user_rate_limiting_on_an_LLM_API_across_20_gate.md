---
type: "practice"
difficulty: "Intermediate"
---

# You need per-user rate limiting on an LLM API across 20 gateway pods with a hard tokens-per-minute quota. Compare a shared Redis token bucket against local buckets with periodic reconciliation, and what consistency you sacrifice with each under a Redis blip.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage per-user rate limiting across 20 gateway pods in an LLM API, we need to compare two strategies: using a shared Redis token bucket versus having local buckets that periodically reconcile with a central Redis instance. The key trade-offs are consistency and fault tolerance.

#### Triage

The question is essentially asking us to evaluate the performance and reliability of these two approaches under potential issues like a Redis blip (i.e., a temporary failure). We'll need to consider how each approach handles token management, consistency requirements, and the impact of failures on overall system behavior.

Shared Redis Token Bucket

In this setup, all 20 gateway pods would share the same Redis instance for managing tokens. Each pod would update the token bucket when a user makes a request, ensuring that no more than the allowed number of tokens are used per minute. The main advantage is that it's simpler to implement and maintain since there’s only one source of truth.

Advantages:
Simplicity: Easier to manage as there’s less state.

#### Cons
istency: Tokens are managed centrally, ensuring uniform rate limiting across all pods.
Disadvantages:
Single Point of Failure (SPOF): If Redis goes down, the entire system stops working until it recovers.
Latency Sensitivity: Network latency between pods and Redis can introduce delays in token updates and limits the number of concurrent requests handled by each pod.
Local Buckets with Periodic Reconciliation

In this approach, each gateway pod maintains its own local token bucket. These buckets are periodically reconciled with a shared Redis instance to ensure consistency. This design is more resilient as it doesn’t rely on a single Redis instance and can tolerate temporary failures better.

Advantages:
Resilience: Can handle Redis outages without halting the service completely.
Latency: Lower network latency since each pod only talks to the central Redis when reconciliation happens.
Disadvantages:
Complexity: More complex to implement and maintain due to the need for periodic reconciliation logic.

#### Cons
istency Trade-Offs: During a blip, there might be temporary inconsistencies in token counts until the next reconciliation cycle.
Failure Modes

Let's consider what happens during a Redis blip:

Shared Redis Token Bucket:

If Redis fails, all 20 pods will stop updating tokens.
This can lead to over-requesting of API calls once Redis recovers and resumes its operations.

Local Buckets with Reconciliation:

Pods continue working locally during the blip.
Once the blip ends and reconciliation occurs, any discrepancies are resolved.
However, users might experience temporary rate limiting issues depending on when and how often reconciliation happens.

#### Closing Summary

The shared Redis token bucket is simpler but less resilient to outages. Local buckets with periodic reconciliation offer better resilience at the cost of added complexity. The choice between these depends on the specific requirements for consistency and fault tolerance in your system.

#### Common Mistakes/Follow-Ups
Forgetting to consider the impact of network latency on performance.
Not accounting for potential discrepancies during reconciliation.
Ignoring the need for a proper fallback strategy if Redis becomes unavailable.

**Comparison:**

```
Approach	Advantages	Disadvantages
Shared Redis	- Simplicity	- Single point of failure
Local Buckets	- Resilience to outages	- Complex implementation and maintenance
```

**Code:**

```python
# Example local bucket implementation with periodic reconciliation

import time
from threading import Lock, Thread

class LocalBucket:
    def __init__(self, max_tokens):
        self.max_tokens = max_tokens
        self.tokens = 0
        self.lock = Lock()

    def update_tokens(self):
        # Simulate token refill after a minute
        with self.lock:
            current_time = time.time()
            elapsed_minutes = (current_time - start_time) // 60
            if elapsed_minutes > 1:
                self.tokens = min(max_tokens, self.tokens + 1)
                start_time = time.time()

    def consume_token(self):
        with self.lock:
            if self.tokens > 0:
                self.tokens -= 1
                return True
            else:
                return False

# Example periodic reconciliation function
def reconcile_buckets(buckets, redis_client):
    for bucket in buckets:
        # Fetch latest token count from Redis and update local state
        pass
```

**Example input / output:**

```text
Input (Example):
- Request from Pod 1 to consume a token for user A
Output (Shared Redis Token Bucket):
- Success if tokens are available, otherwise fail with rate limit exceeded error

Input (Example):
- Request from Pod 2 to consume a token for user B
Output (Local Buckets with Reconciliation):
- Success if tokens are available locally, and reconciliation resolves any inconsistencies later.
```

**Data:**

```json
{
  "gateway_pods": [
    {"id": 1},
    {"id": 2},
    // ... up to 20 pods
  ],
  "local_buckets": [
    {
      "pod_id": 1,
      "max_tokens": 50,
      "tokens": 0,
      "lock": {}
    },
    // ... other local buckets
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Token Available?
5. Allow Request
6. Reject Request
7. Rate Limit Exceeded

