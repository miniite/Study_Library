---
type: "practice"
difficulty: "Deep"
---

# Design a CDN's origin-shield and cache-fill layer so a cache miss on a hot object doesn't stampede the origin — how do you coalesce fills tier the cache and handle cache-key normalization?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective CDN's origin-shield and cache-fill layer, we need to ensure that a cache miss on a hot object doesn't overwhelm the origin server. The goal is to coalesce fills, tier the cache effectively, and handle cache-key normalization correctly. Let’s break this down from first principles.

#### Triage
What Are We Trying To Achieve?

The core problem here is managing cache behavior in a way that prevents overwhelming the origin during peak times when a large number of users experience a cache miss for a popular content item. This involves coalescing cache fills, tiering the cache to ensure locality and performance, and ensuring consistent cache keys across different nodes.

#### Assumptions
The CDN is highly distributed with multiple edge locations.
There are varying levels of cache hits and misses depending on the object's popularity and geographical distribution.
We need to handle both content requests (GET) and update requests (PUT/POST).

#### First-Principles Solution

Coalescing Cache Fills:

When a cache miss occurs, rather than flooding multiple origin servers with multiple requests, we should coalesce these into a single request from the closest edge location.

Tiered Cache Structure:

Implement multiple layers of caching to handle different levels of content popularity and locality.
Use an LRU (Least Recently Used) eviction policy at each tier to manage cache size effectively.

Cache Key Normalization:

Ensure that the same URL or key is interpreted consistently across all edge locations to avoid redundant requests.
Step-by-Step Solution
1. Coalescing Cache Fills

When a cache miss occurs, the following steps should be taken:

Edge Node Detection: The edge node detects the cache miss and logs it.
Request Aggregation: A request aggregator is used to collect requests from other edge nodes in the vicinity that also experienced the same cache miss.
Single Origin Request: Once a sufficient number of such requests are aggregated, the system sends a single origin request with all aggregated URLs or keys.
2. Tiered Cache Structure
L1 Cache (Edge Cache): This is where the most frequently accessed content is stored. It's typically small but fast.
L2 Cache (Regional Cache): Higher capacity than L1, this layer stores more cached content and serves as a buffer between L1 and origin.
Origin Server: The backend server that holds all data.

Each tier should have its own eviction policy based on factors like access frequency, size, and importance.

3. Cache Key Normalization
Use consistent hashing or URL rewriting techniques to ensure the same URL is interpreted consistently across edge locations.
Implement a centralized key normalization service if necessary to handle variations in input.

#### Example
Workflow

User Request:

User requests content at https://example.com/hot-object.

Cache Miss Detection:

Edge node A detects a cache miss for /hot-object and logs it.

Request Aggregation:

Edge nodes B, C, D also experience the same cache miss. They send their requests to a central aggregator.

Single Origin Request:

The aggregating node sends a single request from edge A to the origin server with the URLs of all four requests: https://example.com/hot-object.

Origin Response:

Origin returns the content and updates its cache.

Cache Update:

All edge nodes (A, B, C, D) update their caches based on the response from the origin.

#### Closing Summary

By coalescing cache fills and tiering the cache, we ensure that a hot object's cache miss doesn't overwhelm the origin server. Normalizing cache keys ensures consistency in interpretation of URLs across edge locations. This design balances performance and reliability effectively.

#### Common Mistakes/Follow-Ups
Not Coalescing Requests Properly: Failing to aggregate requests can lead to redundant origin server hits.
Improper Cache Tiering: Not having a multi-tiered cache system may result in poor performance or missed updates.
Inconsistent Key Normalization: Variations in key interpretation across edge locations can lead to redundant requests.
Triaging: What the Interviewer is Probing

The interviewer will likely probe on how you handle edge cases, such as varying network conditions and ensuring consistency during cache fills. They may also ask about performance trade-offs between different caching strategies or about handling dynamic content updates.

**Comparison:**

```
Tier	Description	Size	Eviction Policy
L1	Edge Cache	256 MB	LRU
L2	Regional Cache	4 GB	LRU
Origin	Backend Server	N/A	Least Popular
```

**Code:**

```python
# Example coalescing logic in Python

def coalesce_cache_requests(cache_misses):
    aggregated_urls = set()
    for request in cache_misses:
        if should_aggregate(request):
            aggregated_urls.add(request.url)

    return list(aggregated_urls)

def should_aggregate(request):
    # Simple example: aggregate requests from the same user/IP within 1 second
    return is_from_same_user_and_ip(request) and time_since_last_request(request) < 1

# Example of a simple cache key normalization service
class KeyNormalizer:
    def normalize_key(self, key):
        # Use consistent hashing or URL rewriting
        normalized_key = self.hash_function(key)
        return normalized_key

normalized_key = key_normalizer.normalize_key('/hot-object')
```

**Example input / output:**

```text
Input:
- User requests content at https://example.com/hot-object from four different edge nodes.

Output:
- Aggregator sends a single request to the origin server with URLs from all nodes.
- Origin responds and caches content, then edge nodes update their caches based on the response.
```

**Data:**

```json
{
  "cache_misses": [
    { "url": "/hot-object", "timestamp": 1630458012, "user_id": 1 },
    { "url": "/hot-object", "timestamp": 1630458013, "user_id": 1 },
    { "url": "/hot-object", "timestamp": 1630458014, "user_id": 2 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Edge Node
5. Cache Hit?
6. Content Served Locally
7. Aggregator
8. Request Aggregated
9. Single Origin Request Sent
10. Origin Response
11. Edge Nodes Update Cache

