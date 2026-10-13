---
type: "practice"
difficulty: "Deep"
---

# Your gateway does response caching for a read-heavy endpoint and a cache stampede on invalidation floods the origin; how do you implement stale-while-revalidate and request coalescing at the edge to smooth the reload?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our API Gateway can handle high read traffic efficiently while mitigating cache stampedes when the underlying data changes. To achieve this, we will implement two key strategies: stale-while-revalidate and request coalescing.

First, let's clarify what these terms mean:

Stale-while-Revalidate (SWR) allows clients to continue using a cached response even after it has expired, as long as they are also requesting the latest version from the origin server.
Request Coalescing consolidates multiple requests into one to reduce the load on both the cache and the origin server.

#### Triaging the Question

The question is essentially about implementing robust caching strategies in an API Gateway to handle high read traffic effectively while managing the risk of cache stampedes during invalidation. We need to ensure that cached responses are valid for a reasonable period even after they have expired, and we should minimize redundant requests by grouping them.

#### Assumptions
The gateway is using HTTP caching mechanisms.
The origin server handles both read and write operations.
There is a mechanism in place to invalidate the cache when data changes.
Step-by-Step Solution
1. Implement Stale-while-Revalidate (SWR)

To implement SWR, we need to configure our API Gateway such that it serves stale responses even after they have expired, as long as requests are still being made for updated versions of the content.

Configuration Example:

YAML
http:
  caches:
    my_cache:
      default_ttl: 30s # Cache data for 30 seconds
      max_stale: 1m   # Allow serving stale responses for 1 minute after expiration

paths:
  /api/data:
    get:
      cache_control: public, max-age=30, s-maxage=60, stale-while-revalidate=240s

Here, the stale-while-revalidate directive is set to 240 seconds. This means that clients can continue using cached responses for up to 4 minutes after they have expired.

2. Implement Request Coalescing

Request coalescing involves grouping multiple requests into a single request before forwarding them to the origin server. This reduces the load on both the cache and the origin by minimizing the number of requests that need to be processed.

#### Implementation
Strategy:

Use a rate-limiter or burst buffer at the gateway level.
Batch incoming requests for a specific endpoint within a short time window (e.g., 1 second).
Forward the batched request to the origin server once it reaches a certain size or after a timeout period.

#### Example
Flow:

Yes
Yes
No
No
Request arrives
Is coalescing enabled?
Batch requests
Timeout reached?
Send batched request to origin
Directly send request to origin

#### Closing Summary

By implementing stale-while-revalidate and request coalescing, we can effectively manage the cache behavior in our API Gateway. Stale-while-revalidate ensures that clients continue using cached responses even after they have expired, while request coalescing reduces redundant requests and minimizes the load on both the cache and the origin server.

#### Common Mistakes/Follow-Ups
Misconfiguration of Cache TTLs: Ensure that cache times are set appropriately to balance between freshness and resource usage.
Over-reliance on SWR: While SWR is useful, it can lead to stale data being served for too long if not managed correctly.
Improper Coalescing Logic: The coalescing logic should be carefully designed to avoid delaying legitimate requests unnecessarily.

**Code:**

```yaml
http:
  caches:
    my_cache:
      default_ttl: 30s # Cache data for 30 seconds
      max_stale: 1m   # Allow serving stale responses for 1 minute after expiration

paths:
  /api/data:
    get:
      cache_control: public, max-age=30, s-maxage=60, stale-while-revalidate=240s
```

**Data:**

```json
{
  "cache_control": {
    "public": true,
    "max-age": 30,
    "s-maxage": 60,
    "stale-while-revalidate": 240
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Request arrives
6. Is coalescing enabled?
7. Batch requests
8. Timeout reached?
9. Send batched request to origin
10. Directly send request to origin

