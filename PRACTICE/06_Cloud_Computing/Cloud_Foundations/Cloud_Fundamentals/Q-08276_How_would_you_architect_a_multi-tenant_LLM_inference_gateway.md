---
type: "practice"
difficulty: "Deep"
---

# How would you architect a multi-tenant LLM inference gateway that enforces per-tenant token budgets, rate limits, and priority so one customer's burst cannot exhaust the shared GPU pool for everyone?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Quota Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a multi-tenant Large Language Model (LLM) inference gateway that enforces per-tenant token budgets, rate limits, and priorities, we need to design a scalable system that can efficiently manage resource allocation while ensuring fair usage across multiple tenants. At its core, the solution involves several key components: load balancing, quota enforcement, rate limiting, and dynamic priority management.

Firstly, let's break down what the question is really asking:

The system must handle multiple tenants with varying resource needs.
Each tenant should have a token budget that limits their usage to avoid exceeding their allocated resources.
Rate limiting ensures each tenant’s requests do not overwhelm the system, maintaining overall performance and stability.
Prioritization allows different levels of service for different tenants based on factors such as contract terms or business value.
Step-by-Step Solution
Load Balancing: Use a load balancer to distribute incoming inference requests across multiple GPU instances. This ensures that no single tenant monopolizes the entire resource pool, maintaining fairness and efficiency.
Token Budget Management: Implement a per-tenant token quota system using a database or key-value store (e.g., Redis) to track each tenant's remaining tokens.
Rate Limiting Mechanism: Utilize rate limiting techniques such as Leaky Bucket or Token Bucket algorithms to control the number of requests from each tenant within a given time window.
Priority Queue with Custom Logic: Develop a priority queue that considers both the token budget and pre-defined priorities for different tenants. This ensures that high-priority tenants get preferential treatment, even if they have fewer tokens left.

#### Implementation
Details

Load Balancer

Use an HTTP load balancer like NGINX or AWS ALB to distribute requests.
Configure the load balancer to route traffic based on health checks and availability of GPU instances.

Token Budget Management (Using Redis)

PYTHON
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def get_tenant_tokens(tenant_id):
    return int(r.get(f"tenant:{tenant_id}:tokens"))

def decrement_tenant_tokens(tenant_id):
    current_tokens = get_tenant_tokens(tenant_id)
    if current_tokens > 0:
        r.set(f"tenant:{tenant_id}:tokens", current_tokens - 1)
    else:
        raise Exception("Tenant token limit exceeded")

Rate Limiting (Token Bucket Algorithm)

PYTHON
import time

class TokenBucket:
    def __init__(self, capacity: int, refill_rate: float):
        self.capacity = capacity
        self.tokens = 0
        self.refill_rate = refill_rate
        self.last_refill_time = time.time()

    def get_tokens(self) -> int:
        now = time.time()
        if now > self.last_refill_time + 1.0 / self.refill_rate:
            self.tokens = self.capacity
            self.last_refill_time = now
        else:
            delta_time = min(now - self.last_refill_time, 1.0 / self.refill_rate)
            self.tokens += delta_time * self.refill_rate
        return int(min(self.tokens, self.capacity))

    def consume_tokens(self, num_tokens: int) -> bool:
        if self.get_tokens() >= num_tokens:
            self.tokens -= num_tokens
            return True
        else:
            return False

# Example usage
bucket = TokenBucket(capacity=100, refill_rate=5)
if bucket.consume_tokens(40):
    print("Tokens consumed successfully")

Priority Queue

Design a custom priority queue that processes requests based on both token availability and pre-defined priorities.
PYTHON
from queue import PriorityQueue

class TenantRequest:
    def __init__(self, tenant_id: int, num_tokens: int):
        self.tenant_id = tenant_id
        self.num_tokens = num_tokens

    def __lt__(self, other):
        return (self.tenant_id, -self.num_tokens) < (other.tenant_id, -other.num_tokens)

requests_queue = PriorityQueue()

# Example request processing
while not requests_queue.empty():
    request = requests_queue.get()
    if decrement_tenant_tokens(request.tenant_id):
        bucket = TokenBucket(capacity=100, refill_rate=5)
        if bucket.consume_tokens(request.num_tokens):
            process_request(request)
        else:
            print("Rate limit exceeded for this tenant")

#### Closing Summary
and Common Mistakes
Correctness: Ensure the system accurately enforces token budgets, rate limits, and priorities without overwhelming any single tenant.
Completeness: The solution should cover all aspects of resource management, including fallback mechanisms in case of failures or unexpected behavior.

#### Common mistakes
include:

Not considering edge cases such as rapid bursts from high-priority tenants.
Failing to implement robust error handling and fallback strategies.
Overlooking the importance of monitoring and logging for early detection of issues.

**Comparison:**

```
Tenant ID	Tokens Remaining	Requested Tokens	Outcome
1	50	20	Token limit exceeded
2	40	30	Processed successfully
```

**Code:**

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def get_tenant_tokens(tenant_id):
    return int(r.get(f"tenant:{tenant_id}:tokens"))

def decrement_tenant_tokens(tenant_id):
    current_tokens = get_tenant_tokens(tenant_id)
    if current_tokens > 0:
        r.set(f"tenant:{tenant_id}:tokens", current_tokens - 1)
    else:
        raise Exception("Tenant token limit exceeded")

class TokenBucket:
    def __init__(self, capacity: int, refill_rate: float):
        self.capacity = capacity
        self.tokens = 0
        self.refill_rate = refill_rate
        self.last_refill_time = time.time()

    def get_tokens(self) -> int:
        now = time.time()
        if now > self.last_refill_time + 1.0 / self.refill_rate:
            self.tokens = self.capacity
            self.last_refill_time = now
        else:
            delta_time = min(now - self.last_refill_time, 1.0 / self.refill_rate)
            self.tokens += delta_time * self.refill_rate
        return int(min(self.tokens, self.capacity))

    def consume_tokens(self, num_tokens: int) -> bool:
        if self.get_tokens() >= num_tokens:
            self.tokens -= num_tokens
            return True
        else:
            return False

class TenantRequest:
    def __init__(self, tenant_id: int, num_tokens: int):
        self.tenant_id = tenant_id
        self.num_tokens = num_tokens

    def __lt__(self, other):
        return (self.tenant_id, -self.num_tokens) < (other.tenant_id, -other.num_tokens)

requests_queue = PriorityQueue()
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `TenantRequest(1, 30)` | Processed successfully if tenant has sufficient tokens and rate limit is not exceeded. |

**Data:**

```json
{
  "tenant": {
    "id": 1,
    "tokens": 50
  }
}
```

**Flow:**

1. Load Balancer
2. Token Bucket Algorithm
3. Priority Queue
4. Process Request
5. Monitor and Log

