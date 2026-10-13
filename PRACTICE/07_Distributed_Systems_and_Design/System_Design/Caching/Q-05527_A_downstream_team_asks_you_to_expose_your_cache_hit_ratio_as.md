---
type: "practice"
difficulty: "Intermediate"
---

# A downstream team asks you to expose your cache hit ratio as an SLI. What exactly do you define as a hit, how do you avoid gaming the metric, and what SLO would you commit to?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Hit Ratio](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Hit_Ratio.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To measure cache performance accurately using the Cache Hit Ratio (CHR) as an SLI (Service Level Indicator), we need a clear definition of a cache hit, implement robust metrics that prevent gaming, and set meaningful SLOs (Service Level Objectives). For this role, I'd define a hit, outline best practices to avoid manipulation, and propose an SLO based on industry norms.

#### Triaging the Question

The interviewer is probing your understanding of caching mechanisms, how you can measure their effectiveness reliably, and the importance of setting realistic yet ambitious performance goals. The key assumptions include that we have a functioning cache system, downstream requests are consistent, and there's no external interference in the data serving process.

#### First-Principles Solution
Defining a Cache Hit

A cache hit occurs when a requested item is found directly from the cache without needing to fetch it from the original source. This definition must exclude cases like stale or invalid entries that might falsely inflate the hit ratio.

Avoiding Gaming the Metric

To prevent gaming, we need to ensure:

Cache Warm-Up: Ensure the initial data loaded into the cache is representative of real-world usage.

#### Cons
istency Check: Regularly validate cached content against its original source.
Stale Data Handling: Implement mechanisms to handle stale or invalid data appropriately without misleading the hit ratio.
Setting SLOs

For a backend system, typical CHR values range from 80% to 95%. A common SLO for an intermediate experience level would be around 90% with a target of 95%. This reflects industry standards while allowing room for improvement and ensuring meaningful metrics.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, accurately measuring the cache hit ratio involves defining clear criteria for hits, implementing checks to avoid gaming the metric, and setting realistic SLOs. A key mistake is underestimating the importance of validation mechanisms or over-relying on a single metric without considering other factors like cache eviction policies.

Common follow-ups might include:

How you measure consistency between cache and original source?
What happens if the cache is frequently invalidated due to frequent updates in the backend system?
Closing

Understanding these nuances ensures that we are not just measuring performance but doing so in a way that truly reflects the effectiveness of our caching strategy.

**Comparison:**

```
Metric	Suggested Value	Reasoning
Cache Hit Ratio (CHR)	90-95%	Industry standard, provides room for improvement while being ambitious
```

**Code:**

```python
# Example function to calculate CHR
def calculate_cache_hit_ratio(cached_requests, total_requests):
    hits = sum(1 for req in cached_requests if req['fromCache'])
    return (hits / total_requests) * 100

# Sample data
cached_requests = [
    {'fromCache': True},
    {'fromCache': False},
    {'fromCache': True},
    # ... more requests
]
total_requests = len(cached_requests)

chr = calculate_cache_hit_ratio(cached_requests, total_requests)
print(f"Cache Hit Ratio: {chr}%")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| [{'fromCache': true}, {'fromCache': false}, {'fromCache': true}] | Cache Hit Ratio: 66.67% |

**Data:**

```json
{
  "cached_requests": [
    {"fromCache": true},
    {"fromCache": false},
    {"fromCache": true}
  ],
  "total_requests": 3,
  "cache_hit_ratio": 66.67%
}
```

**Flow:**

1. Measure Cached Requests
2. Calculate CHR
3. Determine Consistency
4. Set SLOs

