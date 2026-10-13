---
type: "practice"
difficulty: "Intermediate"
---

# Roughly how much faster is reading from RAM versus a spinning disk versus a network round-trip? Why does that motivate caching?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ What to Cache](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/What_to_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Caching is essential in backend development because it significantly reduces the latency of data retrieval by storing frequently accessed data closer to the point of use, typically in RAM. In this context, we need to understand how much faster reading from different storage mediums can be.

To answer the question: reading from RAM is orders of magnitude faster than a spinning disk, and both are vastly faster than a network round-trip. This difference motivates caching because it allows us to minimize the time spent fetching data by storing it in a location that offers the fastest access times.

Let's break this down step-by-step:

Understanding the Storage Mediums:

RAM (Random Access Memory): This is volatile memory, typically built with DRAM chips, and provides access times on the order of nanoseconds.
Spinning Disk: Traditional hard drives have platters that spin at speeds like 7200 RPM or higher. They provide random access but with slower seek times (milliseconds) compared to RAM.
Network Round-Trip: Data retrieval over a network involves multiple steps: sending packets, traversing through routers and switches, and receiving the data back. Each of these steps introduces latency, often measured in milliseconds.

Access Times:

RAM Access Time: On average, a RAM read operation can complete within a few nanoseconds (ns).
Spinning Disk Access Time: A typical disk access time is around 8-10 ms.
Network Round-Trip: For a local network, the round-trip delay might be in the range of 2-5 ms; for remote networks, it could increase to tens or even hundreds of milliseconds.

Comparative Analysis:

If we consider a scenario where data is read from RAM versus a spinning disk: accessing the same data in RAM would be millions of times faster (8-10 million times) compared to a single access to a disk.
Reading from a network introduces additional latency on top of the slower storage medium. For example, if we have to fetch data over a local network and then from a spinning disk, the combined time can still be significantly higher than just accessing RAM.

Why Caching Motivates:

By storing frequently accessed data in faster memory like RAM (or caching it closer to where it's needed), applications can reduce the number of slower operations and thus improve overall performance.
For example, if a web application fetches user data from a database on every request, introducing an in-memory cache can drastically reduce the load time for subsequent requests.

Edge Cases:

Cache invalidation: When the underlying data changes, ensuring that the cached copy is updated can be challenging but crucial.
Capacity and Eviction Policies: Caches need to manage their size; strategies like least recently used (LRU) or least frequently used (LFU) help in deciding which entries to evict when the cache reaches its limit.

#### Closing Summary
Caching is motivated by the significant difference in access times between RAM, spinning disks, and network round-trips. By storing data closer to where it's needed, caching can dramatically reduce latency and improve performance. However, managing caches effectively requires dealing with issues like invalidation, eviction policies, and limited storage capacity.

#### Common Mistakes

Overestimating the impact of network delays.
Neglecting the overhead of cache management.
Not considering the trade-offs between cache size and hit rate.

**Comparison:**

```
Storage Medium	Access Time
RAM	<10 ns
Spinning Disk	8-10 ms
Network	2-5 ms (local), 30+ ms (remote)
```

**Code:**

```python
# Example code snippet to demonstrate caching (pseudo-code)
def get_data_from_disk(key):
    return read_from_spinning_disk(key)

def cache_data(key, value):
    # Simulating storing in memory cache
    pass

def fetch_data_with_cache(key):
    if key in local_cache:
        return local_cache[key]
    else:
        data = get_data_from_disk(key)
        cache_data(key, data)
        return data
```

**Example input / output:**

```text
Input: Fetch user data for key 'user123'
Output:
  - Direct from Disk: User data loaded (8-10 ms)
  - From Cache: User data returned directly (ns)

Request to fetch user data over network:
  - Round-Trip Time with Network + Disk Access: ~7-15 ms
```

**Data:**

```json
{
  "disk_access_time": "8-10 ms",
  "network_round_trip_time": "2-5 ms"
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[User Request] --> B(Check Cache);
4. B --> C(Fetch from Disk)[Fetch Data];
5. C --> D(Return Data)[Return Data];
6. B --> E[Hit in Cache];
7. E --> F(Return Cached Data)[Return Cached Data];

