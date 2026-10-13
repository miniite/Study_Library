---
type: "practice"
difficulty: "Expert"
---

# A gateway plugin calls an external policy service (OPA) synchronously per request and adds 15ms; how do you cache decisions, batch, or push policy to the gateway to cut this without letting stale policy authorize revoked access?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Scaling for Traffic Spikes](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/Gateway_Scaling_for_Traffic_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of reducing latency from calling an external policy service (OPA) while ensuring that authorization decisions remain up-to-date, we can implement a combination of caching and batch processing strategies. The goal is to minimize the 15ms overhead by leveraging local caching at the API Gateway level without risking stale data or security breaches.

#### Triaging the Question

The question asks how to cache OPA decisions, batch requests, or push policy updates to the gateway to reduce latency while maintaining accuracy and security. This involves understanding the trade-offs between performance and consistency and ensuring that cached policies are refreshed as necessary.

Key Assumptions
The API Gateway has a reasonable amount of memory for caching.
Policies can be updated frequently but not too often, reducing the risk of stale data.
OPA decisions take 15ms to compute synchronously.
We aim to reduce this latency without significantly compromising on the freshness of policies.

#### First-Principles Solution
Caching Decisions

Caching is a key strategy here. We can cache the results from OPA for a certain period, reducing the number of direct calls to OPA per request. However, we need to ensure that cached data does not become stale and lead to unauthorized access.

Cache Configuration: Configure the API Gateway to cache decisions from OPA for a specific duration (e.g., 60 seconds). This can be done using Redis or an in-memory store.
Freshness Handling: Implement a freshness check mechanism where cached responses are periodically validated against current policies. If a policy is found to have changed, the corresponding cached entries are invalidated and refreshed.
Batch Processing

Batching requests can further reduce latency by minimizing the number of OPA calls per group of similar requests.

Request Aggregation: Group incoming requests based on common parameters or patterns that share the same policy requirements.
Batch Execution: Execute a single batch request to OPA and cache the results for all grouped requests, reducing the overall number of API Gateway-to-OPA calls.
Policy Push

Pushing policies directly to the gateway can eliminate the need for constant querying but requires careful management to ensure that policies are only pushed when necessary.

Policy Versioning: Use versioning for OPA policies and push new versions to the gateway as they become available.
Delta Updates: Implement a mechanism where only changes in policies are sent to the gateway, reducing unnecessary updates and minimizing impact on performance.
Full Solution

Combining caching, batching, and policy push can significantly reduce latency while maintaining security:

Caching Decisions:

Cache OPA decisions for 60 seconds.
Validate cached entries every minute against current policies.
Invalidate and refresh entries if a policy change is detected.

Batch Processing:

Group similar requests based on common parameters.
Send batch requests to OPA and cache the results for all grouped requests.
Use a single call per group of requests, reducing the overall number of API Gateway-to-OPA calls.

Policy Push:

Version policies in OPA.
Push only updated versions to the gateway using a delta update strategy.
Ensure that policy changes are communicated effectively and processed efficiently on the gateway side.

#### Closing Summary

By implementing caching, batching, and policy push strategies, we can significantly reduce the 15ms latency from calling OPA. Caching ensures freshness while minimizing redundant queries, batching reduces the number of API Gateway-to-OPA calls, and pushing policies ensures that only necessary changes are communicated, optimizing performance.

#### Common Mistakes/Follow-Ups
Over-caching: Caching too aggressively can lead to stale data if not properly managed.
Insufficient Batching: Not aggregating requests effectively can result in many separate OPA calls.
Policy Management: Overlooking the need for versioning and delta updates can lead to unnecessary policy retransmissions.

Would you like to explore any of these strategies further or consider other optimization techniques?

**Comparison:**

```
Strategy	Description	Benefits
Caching	Cache OPA decisions for 60 seconds.	Reduces redundant API calls.
Batching	Group similar requests and batch.	Minimizes the number of API calls.
Policy Push	Push only updated policies.	Optimizes policy changes communication.
```

**Code:**

```python
# Example Cache Configuration (using Redis)
import redis

cache = redis.Redis(host='localhost', port=6379, db=0)

def get_policy_decision(request_id):
    cached_response = cache.get(request_id)
    if cached_response:
        return eval(cached_response)  # For simplicity; use proper deserialization in practice
    else:
        response = opa_client.query_policy(request_id)  # Assume OPA client is pre-configured
        cache.set(request_id, str(response), ex=60)  # Cache for 60 seconds
        return response

def validate_cache():
    for key in cache.scan_iter("*"):
        if should_validate(key):  # Implement logic to check policy changes
            update_cache(key)
```

**Example input / output:**

| Input (Request ID) | Output (Policy Decision) |
|--------------------|--------------------------|
| 12345              | {"allowed": true, ...}    |

**Data:**

```json
{
  "request_id": "12345",
  "policy_version": "v1.0"
}
```

**Flow:**

1. Yes
2. No
3. Request Received
4. Check Cache
5. Cache Hit?
6. Retrieve from Cache
7. Call OPA
8. Store in Cache
9. Return Policy Decision

