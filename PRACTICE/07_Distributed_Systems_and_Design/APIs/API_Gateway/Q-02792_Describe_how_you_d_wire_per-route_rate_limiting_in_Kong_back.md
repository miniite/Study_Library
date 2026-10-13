---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you'd wire per-route rate limiting in Kong backed by Redis across three replicas; what happens to limits when a Kong node loses Redis, and how do you decide the local-vs-cluster policy?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limiting at the Gateway](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To wire per-route rate limiting in Kong using Redis as the backend, we'll leverage Kong's rate-limiting plugins along with Redis to enforce limits on a per-route basis. This setup ensures that traffic is controlled at the API level and distributed across multiple replicas of the Kong Gateway. Let's break down how this can be achieved and what happens when a node loses its connection to Redis.

#### Triaging the Question

The question asks about implementing rate limiting with an API gateway (specifically, Kong) using Redis as the backend. It also seeks insights on how the system behaves if a Kong instance loses connectivity to Redis and how we should decide between local vs. cluster policies for rate limits.

Key Assumptions:
Kong Nodes: We have three replicas of the Kong Gateway.
Redis Cluster: A Redis cluster is set up to provide high availability and performance.
Rate-Limiting Plugin: The kong-plugin-rate-limit will be used for rate limiting.

#### First-Principles Solution
Setting Up Rate Limiting with Kong and Redis

Install the Rate-Limiting Plugin:

Use the rate-limit plugin in Kong to manage per-route rate limits.
BASH
kong plugins install rate-limit

Configure the Plugin:

Define routes and their respective rate limits using the Kong Admin API or UI.
JSON
{
  "name": "route1",
  "paths": ["/api/v1/endpoint1"],
  "rate_limiting": true,
  "per second": 50,
  "max limit": 200
}

Connect to Redis:

Configure the rate-limit plugin with a connection string for your Redis cluster.
YAML
upstream {
    rate_limit_backend = "redis"
    redis_host = "192.168.1.1, 192.168.1.2, 192.168.1.3"
    redis_password = "your-redis-password"
}

Replication and Load Balancing:

Ensure that Kong nodes are configured to communicate with the Redis cluster for rate limiting.
Use a load balancer (e.g., HAProxy) to distribute traffic among the Kong replicas.
Handling Node Failures

If a Kong node loses its connection to Redis, the local rate limiters will continue to operate but may become out of sync. This can lead to inconsistent enforcement of rate limits across nodes.

Local Policy:

When using the local policy, each Kong instance stores and enforces rate limits locally.

#### Pros
Minimal impact on performance; no network latency for rate limit checks.

#### Cons
Synchronization issues if a node loses Redis connection temporarily or permanently.

Cluster Policy:

The cluster policy ensures that rate limits are consistent across all nodes as they communicate with the shared Redis backend.

#### Pros
Ensures accurate enforcement of rate limits; no data loss upon temporary disconnections.

#### Cons
Higher network overhead due to frequent communication and potential performance impact during high traffic.
Decision Factors for Local vs. Cluster Policies

#### Cons
istency Requirements: If absolute consistency is critical, the cluster policy should be used.

#### Performance Considerations
For high-throughput applications, the local policy can offer better performance but may introduce synchronization issues.
Network Reliability: A stable network with low latency favors the cluster policy; otherwise, the local policy might be more robust.

#### Closing Summary

By configuring Kong to use Redis for rate limiting and considering the trade-offs between local vs. cluster policies, we ensure that our API gateway can handle high traffic while maintaining rate limits effectively. The key is to understand the implications of network failures and choose the appropriate strategy based on your specific requirements.

#### Common Mistakes
and Follow-Ups
Ignoring Network Latency: Failing to account for the potential impact of network latency when choosing a policy.
Overlooking Synchronization Issues: Not addressing how rate limiters will behave if they are not in sync due to Redis disconnections.
Insufficient Testing: Not thoroughly testing the rate limiting setup under various failure scenarios.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
  title Rate Limiting Strategies with Redis in Kong
  dateFormat YYYY-MM-DD
  section Configuration
    clusterPolicy: fromNow, 6d, Local Policy
    localPolicy: after clusterPolicy, 5d, Cluster Policy
```

**Code:**

```yaml
# Example configuration for Kong with local policy
upstream {
    rate_limit_backend = "local"
}

# Example configuration for Kong with cluster policy
upstream {
    rate_limit_backend = "redis"
    redis_host = "192.168.1.1, 192.168.1.2, 192.168.1.3"
    redis_password = "your-redis-password"
}
```

**Example input / output:**

```text
- **Input**: POST request to `/api/v1/endpoint1` with rate-limiting enabled.
- **Output**: Response status code (e.g., 429 Too Many Requests) and body if the rate limit is exceeded.

| Input | Output |
|-------|--------|
| POST /api/v1/endpoint1 | {"status": "429", "message": "Rate limit exceeded"} |
```

**Data:**

```json
{
  "name": "route1",
  "paths": ["/api/v1/endpoint1"],
  "rate_limiting": true,
  "per second": 50,
  "max limit": 200
}
```

**Flow:**

1. Yes
2. No
3. Rate Limiting Setup
4. Is Redis Connection Available?
5. Local Rate Limiter
6. Cluster Rate Limiter
7. Enforce Rate Limits Locally
8. Fetch and Enforce Rate Limitsfrom Redis

