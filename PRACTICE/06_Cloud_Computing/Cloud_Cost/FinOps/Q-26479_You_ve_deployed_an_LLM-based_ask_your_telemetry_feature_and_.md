---
type: "practice"
difficulty: "Deep"
---

# You've deployed an LLM-based 'ask your telemetry' feature and a single vague natural-language query can fan out into expensive full-corpus scans plus a large-context prompt. Architect the cost-governance — query cost preview, result caching, per-user token budget — so the convenience feature can't 50x a team's AIOps bill.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ RAG and Vector Costs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/RAG_and_Vector_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To govern costs effectively in an LLM-based 'ask your telemetry' feature, we need to implement a combination of query cost preview, result caching, and per-user token budgeting mechanisms. By doing so, we can ensure that the convenience of natural-language queries does not come at the expense of the team's AIOps budget.

#### Triaging the Question

The core of this problem is about managing costs in an LLM system where a single query can trigger expensive operations. The key aspects are:

Query Cost Preview: Before executing a query, providing an estimate of the cost.
Result Caching: To avoid redundant and costly scans by caching results.
Per-User Token Budgets: Ensuring users do not exceed their allocated budget.

#### First Principles Solution
1. Query Cost Estimation

To accurately predict costs, we need to model the resource usage of each query. Let's break it down:

Context Tokens: Number of tokens in the prompt and any context.
Completion Tokens: Estimated number of tokens for the completion.
Resource Intensity: Different operations (e.g., full-corpus scans vs. indexed lookups) have different resource costs.

We can create a cost model using historical data:

PYTHON
def estimate_cost(prompt_tokens, completion_tokens, resource_intensity):
    return prompt_tokens * 0.1 + completion_tokens * 0.2 + resource_intensity * 1.5
2. Result Caching

To cache results effectively, we need to design a caching mechanism that can handle both small and large contexts efficiently.

Key Generation: Use hash functions or indexing schemes to generate unique keys for each query.
Cache Storage: Utilize in-memory caches like Redis or distributed caches like Hazelcast.
Eviction Policies: Implement policies such as Least Recently Used (LRU) or Time-To-Live (TTL).

#### Example
cache entry:

PYTHON
def cache_results(query_key, result):
    redis_client.setex(query_key, 3600, json.dumps(result))
3. Per-User Token Budgets

Implementing token budgets requires setting limits and monitoring usage.

Token Allocation: Allocate a fixed number of tokens per user or per session.
Real-time Monitoring: Use metrics like Prometheus to track token usage in real-time.
Notification Mechanisms: Send alerts when the budget is approaching or exceeded.

#### Example
budget management:

PYTHON
class TokenManager:
    def __init__(self, max_tokens):
        self.max_tokens = max_tokens
        self.used_tokens = 0

    def use_token(self, token_cost):
        if self.used_tokens + token_cost <= self.max_tokens:
            self.used_tokens += token_cost
            return True
        else:
            return False

    def reset_usage(self):
        self.used_tokens = 0

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a combination of query cost estimation, result caching, and per-user token budgets, we can ensure that the 'ask your telemetry' feature remains both convenient and cost-effective. However, it's important to consider edge cases:

Overfitting on Historical Data: Ensure the cost model accurately reflects current usage patterns.
Cache Hit vs. Miss Ratio: Monitor cache performance to optimize storage and retrieval efficiency.
Token Budget Exhaustion: Implement soft limits that allow some overage with alerts, but not hard limits that could disrupt workflow.

#### Follow-ups
might include:

How do you handle multi-tenant environments where different teams have varying budget constraints?
What are your thoughts on using machine learning to further optimize the cost model based on usage patterns?

**Comparison:**

```
Token	Cost (USD)
Prompt	0.1
Completion	0.2
Resource Intensity	1.5
```

**Code:**

```python
# Cost Estimation Model
def estimate_cost(prompt_tokens, completion_tokens, resource_intensity):
    return prompt_tokens * 0.1 + completion_tokens * 0.2 + resource_intensity * 1.5

# Caching Mechanism
from redis import Redis

redis_client = Redis(host='localhost', port=6379, db=0)

def cache_results(query_key, result):
    redis_client.setex(query_key, 3600, json.dumps(result))

# Token Manager
class TokenManager:
    def __init__(self, max_tokens):
        self.max_tokens = max_tokens
        self.used_tokens = 0

    def use_token(self, token_cost):
        if self.used_tokens + token_cost <= self.max_tokens:
            self.used_tokens += token_cost
            return True
        else:
            return False

    def reset_usage(self):
        self.used_tokens = 0
```

**Example input / output:**

```text
query_key = "telemetry_query_abc"
result = {
    "data": [/* large telemetry data */],
    "metadata": {"timestamp": "2023-10-01T14:00:00Z"}
}

# Cache Usage
cache_results(query_key, result)

# Cost Estimation
estimated_cost = estimate_cost(150, 200, 3)
print(f"Estimated cost: {estimated_cost}")
```

**Data:**

```json
{
    "prompt_tokens": 150,
    "completion_tokens": 200,
    "resource_intensity": 3
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Queries
6. Query Cost Estimation
7. Cached Result Check
8. Cached?
9. Retrieve Cached Data
10. Execute Query, GenerateResults
11. Store in Cache
12. Token Usage Check
13. Budget Exceeded?
14. Notify User and Limit Access
15. Return Results to User

