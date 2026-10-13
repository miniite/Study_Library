---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design cache invalidation for a read-heavy .NET service using Redis — cache-aside vs write-through, stampede protection, and TTL vs event-driven invalidation trade-offs.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cache invalidation is a critical aspect of designing efficient and scalable systems, especially in read-heavy environments like those built with .NET services using Redis as a caching layer. In this scenario, we need to carefully consider several strategies including cache-aside vs write-through, stampede protection, and the trade-offs between Time-To-Live (TTL) and event-driven invalidation.

To provide a deep understanding of how to design effective cache invalidation in such an environment, let's break down each component:

#### Triaging the Question

The question is essentially asking for an in-depth analysis of different strategies for managing cached data. Specifically, we need to cover:

Cache-Aside vs Write-Through: Whether updates are pushed directly to the database or if a write-through cache mechanism is used.
Stampede Protection: Measures taken to prevent a sudden spike in requests from overwhelming the system.
TTL vs Event-Driven Invalidation Trade-offs: Balancing between manually setting expiration times and dynamically invalidating data based on events.

#### Assumptions
We are designing for a read-heavy .NET application using Redis as our caching layer.
The service must handle both GET (read) and SET (write) operations efficiently.
There is a need to balance between performance, cost, and complexity.
Cache-Aside vs Write-Through

Cache-Aside: In this approach, writes are always directed directly to the database, while reads first check the cache for up-to-date data. This ensures that the database state is always accurate but may result in stale reads if the cache isn’t invalidated promptly. For a read-heavy service like ours, Cache-Aside can be more efficient because it reduces the number of direct writes.

Write-Through: Here, updates are pushed both to the cache and the database simultaneously. This ensures that any update is immediately reflected in the cache, but at the cost of additional write operations on both the cache and the database. While this provides strong consistency, it can be less scalable due to increased load on the database.

Stampede Protection

Stampede protection involves mechanisms to handle a sudden surge in requests without causing a cascade failure or overwhelming the system resources. Key strategies include:

Rate Limiting: Limiting the number of concurrent requests.
Circuit Breaker: Failing fast if the service becomes unresponsive, avoiding further requests until stability is restored.
Load Shedding: Temporarily redirecting traffic away from a failing component.
TTL vs Event-Driven Invalidation

Time-To-Live (TTL): Setting an expiration time on cached items ensures that outdated data is automatically invalidated. This approach simplifies the system by reducing dependencies on external events but can lead to more frequent cache misses and increased server load as expired entries are reloaded from the database.

Event-Driven Invalidation: Dynamically invalidating cache entries in response to specific events (e.g., database updates, transaction commits) ensures that only relevant data is refreshed. This approach reduces unnecessary reloads but requires additional complexity for detecting and propagating such events.

#### First-Principles Solution

For a read-heavy .NET service using Redis, a hybrid approach combining Cache-Aside with event-driven invalidation can be most effective:

Cache-Aside: Use Redis as the primary cache layer to handle reads.
Event-Driven Invalidation:
Implement a pub/sub system where write operations (e.g., database updates) publish events.
Subscribe to these events on the server-side, triggering cache invalidation for affected keys.

#### Implementation
Details
CSHARP
// Example C# code snippet for event-driven invalidation
public class CacheInvalidator
{
    private readonly IConnectionMultiplexer _redis;
    private readonly IPubSubConnection _pubSub;

    public CacheInvalidator(IConnectionMultiplexer redis)
    {
        _redis = redis;
        _pubSub = (IPubSubConnection)_redis.GetSubscriber();
    }

    public void InvalidateCache(string key)
    {
        _pubSub.Publish("invalidate_cache", key);
    }
}

// Example event handler
public class EventHandler : IDispatcherSubscriber<string>
{
    private readonly IRedisCache _cache;

    public EventHandler(IRedisCache cache)
    {
        _cache = cache;
    }

    public void Handle(string message, CancellationToken cancellationToken)
    {
        // Invalidate the key in Redis
        _cache.Invalidate(message);
    }
}

#### Closing Summary

By combining Cache-Aside with event-driven invalidation, we can achieve a balance between performance and consistency. Event-driven invalidation ensures that cached data remains up-to-date without overly frequent reloads from the database. Stampede protection mechanisms help manage unexpected surges in traffic, ensuring robustness.

#### Common Mistakes/Follow-Ups
Over-reliance on TTL: Not considering event-driven invalidation can lead to stale reads.
Ignoring stampede protection: Failing to implement proper rate limiting or circuit breakers can result in system overload during surges.
Manual vs Event-Driven: Choosing a manual approach over event-driven mechanisms may not fully leverage Redis’s capabilities and could introduce unnecessary complexity.

**Comparison:**

```
Key	Value
TTL	60 seconds
Event	"invalidate_cache"
```

**Code:**

```csharp
public class CacheInvalidator
{
    private readonly IConnectionMultiplexer _redis;
    private readonly IPubSubConnection _pubSub;

    public CacheInvalidator(IConnectionMultiplexer redis)
    {
        _redis = redis;
        _pubSub = (IPubSubConnection)_redis.GetSubscriber();
    }

    public void InvalidateCache(string key)
    {
        _pubSub.Publish("invalidate_cache", key);
    }
}

public class EventHandler : IDispatcherSubscriber<string>
{
    private readonly IRedisCache _cache;

    public EventHandler(IRedisCache cache)
    {
        _cache = cache;
    }

    public void Handle(string message, CancellationToken cancellationToken)
    {
        // Invalidate the key in Redis
        _cache.Invalidate(message);
    }
}
```

**Example input / output:**

```text
- **Input**: `InvalidateCache("product:123")`
- **Output**: Cache invalidated for key `"product:123"` in Redis
```

**Data:**

```json
{
  "key": "product:123",
  "value": "Product A"
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. User Request
6. Is Cache Available?
7. Retrieve from Cache
8. Fetch from Database
9. Cache Miss?
10. Invalidate Cache Key
11. Store in Cache and Return

