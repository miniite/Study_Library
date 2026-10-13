---
type: "practice"
difficulty: "Basic"
---

# What is cache hit ratio and how would you measure it?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Hit Ratio](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Hit_Ratio.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cache hit ratio is a key performance metric that measures the effectiveness of caching in reducing the number of requests to an origin or backend system by utilizing cached data stored locally. In this context, we are measuring how often the cache successfully serves requested data without needing to fetch it from a slower source.

#### Triaging the Question

The question is essentially asking for an understanding of what the cache hit ratio is and how to calculate it. The interviewer might be probing your knowledge of caching mechanisms and their importance in optimizing system performance, as well as your ability to quantitatively measure the effectiveness of such optimizations.

#### First-Principles Solution

To calculate the cache hit ratio, you need two pieces of information:

Number of Cache Hits: This is the number of times a request was served from the cache.
Total Number of Requests: This includes both hits and misses (requests that did not have cached data).

The formula for calculating the cache hit ratio is:

[ \text{Cache Hit Ratio} = \left( \frac{\text{Number of Cache Hits}}{\text{Total Number of Requests}} \right) \times 100% ]

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the cache hit ratio helps you understand how well your caching mechanism is performing. A higher cache hit ratio means that more requests are being served from the cache rather than fetching data from a slower source.

#### Common mistakes
to avoid include:

Forgetting to account for both hits and misses when calculating total requests.
Not considering edge cases such as invalidation policies or what happens during cache eviction.
Additional Considerations

When discussing this in an interview, it's also worth mentioning that the cache hit ratio should be part of a broader performance optimization strategy. You might want to discuss factors like:

Cache Size: Larger caches can improve hit ratios but come with increased storage costs.
Eviction Policies: How you manage cache entries when the cache size exceeds capacity.
TTL (Time-to-Live): Setting appropriate expiration times for cached data.

#### Example
Calculation

Let's say a website has 10,000 requests in a day:

Out of these, 9,500 requests are served from the cache (cache hits).
The remaining 500 requests require fetching data from the backend (cache misses).

The cache hit ratio would be:

[ \text{Cache Hit Ratio} = \left( \frac{9,500}{10,000} \right) \times 100% = 95% ]

This indicates that 95% of the requests are being served from the cache.

**Code:**

```python
def calculate_cache_hit_ratio(hits, total_requests):
    return (hits / total_requests) * 100

# Example usage
cache_hits = 9500
total_requests = 10000
cache_hit_ratio = calculate_cache_hit_ratio(cache_hits, total_requests)
print(f"Cache Hit Ratio: {cache_hit_ratio}%")
```

**Example input / output:**

```text
Example input:
- `hits`: 9500
- `total_requests`: 10000

Output:
- Cache Hit Ratio: 95%
```

