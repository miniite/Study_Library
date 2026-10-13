---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set a per-tenant inference cost ceiling so one customer's abusive prompt pattern (huge contexts, high call rate) can't silently erode your platform's margin. Where do you enforce the ceiling — API gateway, model router, token-budget middleware — and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shadow Mode for Cost Policies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_Mode_for_Cost_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set a per-tenant inference cost ceiling in a cloud-based ML platform, the core idea is to ensure that individual customers are charged fairly based on their usage patterns without affecting others' service. The key challenge here is to enforce this cost ceiling as early and efficiently as possible in the request lifecycle.

The question probes understanding of:

Cost Management: How different components of a cloud-based system contribute to overall costs.
Governance and Policy Enforcement: Where and how to implement rules that govern usage.
Scalability and Performance: Balancing cost control with maintaining performance for all tenants.

Key assumptions include:

The platform uses a multi-tenant architecture where multiple customers (tenants) share the same infrastructure but need distinct billing policies.
Each tenant has a unique set of inference patterns, some potentially abusive in terms of resource consumption.

We can enforce the ceiling at various points: API Gateway, model router, or token-budget middleware. The choice depends on factors like performance impact and ease of implementation.

Step-by-Step Solution

API Gateway:

Enforce: At this layer, we can inspect requests before they reach any internal services.
Advantages: Minimal performance overhead since it’s the first point of interaction with clients. Can easily track API calls and apply rate limits.
Disadvantages: Might not capture all model inference costs if models are invoked directly.

Model Router:

Enforce: This layer routes requests to appropriate models based on tenant-specific configurations.
Advantages: Direct control over the path of each request, allowing for precise cost allocation per model or even within a single model.
Disadvantages: Performance overhead due to additional decision-making logic. Requires careful implementation to avoid bottlenecks.

Token-Budget Middleware:

Enforce: This middleware runs before any inference operations and manages token budgets based on predefined policies.
Advantages: Fine-grained control over model execution, can be integrated deeply into the inference pipeline without altering existing models.
Disadvantages: Higher complexity in implementation due to needing to integrate with each model’s interface.
Detailed Implementation

API Gateway:

Implement a rate limiter that tracks API calls per tenant.
Use metadata tagging on requests to identify tenants and apply specific limits.

#### Example
RateLimit middleware using a token bucket algorithm.

Model Router:

Introduce a cost-aware router that inspects each request and decides whether it should proceed based on the remaining budget for that tenant.
Implement a decision engine that considers both historical usage and real-time metrics to dynamically adjust limits.

Token-Budget Middleware:

Design a middleware component that intercepts model calls, checks the token budget, and either allows or denies execution.
Use context propagation to pass tenant-specific information throughout the request lifecycle.

#### Example
Middleware that integrates with tracing frameworks like Jaeger for detailed logging.

#### Closing Summary

By enforcing cost ceilings at different points in the request lifecycle—API Gateway, model router, or token-budget middleware—we can effectively manage per-tenant inference costs. The choice depends on performance requirements and complexity trade-offs. API Gateway offers simplicity but may miss certain model costs; model router provides fine-grained control with potential overhead; token-budget middleware ensures precise cost management at the cost of integration effort.

#### Common Mistakes/Follow-Ups
Overlooking Performance Impact: Ignoring the impact on request latency and throughput by overly aggressive enforcement.
Inconsistent Policy Enforcement: Failing to apply consistent policies across different layers, leading to uneven billing experiences for tenants.
Ignoring Edge Cases: Not considering scenarios like burst usage or long-running inference tasks that could violate budget constraints.

**Comparison:**

```
Layer	Advantages	Disadvantages
API Gateway	Minimal overhead, easy implementation	May not catch all costs if models are invoked directly
Model Router	Fine-grained control over model execution	Higher performance overhead, complex integration
Token-Budget Middleware	Precise cost management	High complexity in implementation, may require changing existing models
```

**Code:**

```python
# Example of a RateLimit Middleware in Flask

from flask import request
import time

class RateLimiter:
    def __init__(self, max_tokens, refill_rate):
        self.max_tokens = max_tokens
        self.refill_rate = refill_rate
        self.tokens = max_tokens
        self.last_time = time.time()

    def _refill(self):
        now = time.time()
        elapsed_time = now - self.last_time
        new_tokens = elapsed_time * self.refill_rate
        self.tokens += min(new_tokens, self.max_tokens - self.tokens)
        self.tokens = min(self.tokens, self.max_tokens)
        self.last_time = now

    def allow_request(self):
        self._refill()
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        else:
            return False

rate_limiter = RateLimiter(max_tokens=1000, refill_rate=1)
```

**Example input / output:**

```text
Input Request:
- API Gateway receives a request with tenant ID: 12345

Enforcement Example at API Gateway:
API Gateway checks rate limit for tenant 12345, allowing the request if tokens are available.
If allowed, it passes the request to the model router.

Output Response:
- If budget is exceeded, response includes an error message indicating insufficient resources.
```

**Data:**

```json
{
    "tenant_id": "12345",
    "max_inference_budget": 50,
    "current_inference_usage": 30
}
```

**Flow:**

1. Yes
2. No
3. API Gateway
4. Allow Request?
5. Pass to Model Router
6. Deny Request
7. Model Invocation

