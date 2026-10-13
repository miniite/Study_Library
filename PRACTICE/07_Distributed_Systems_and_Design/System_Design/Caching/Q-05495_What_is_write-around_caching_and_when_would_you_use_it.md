---
type: "practice"
difficulty: "Deep"
---

# What is write-around caching and when would you use it?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Write-through and Write-back Consistency](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Write-through_and_Write-back_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Write-around caching, also known as stale-while-revalidate (SWR) caching, is a strategy used in distributed systems to manage cache invalidation without causing staleness issues during the transition phase. Essentially, it involves serving a stale copy of data from the cache and simultaneously fetching fresh data from the origin source, ensuring that there's no interruption in service for the user while updates are being made.

This concept arises when we need a balance between minimizing cache misses (i.e., reducing the load on the origin) and preventing the propagation of outdated information. It is particularly useful in scenarios where frequent updates are expected but the application can tolerate some level of staleness during these updates.

**Comparison:**

```
Operation	Cache Behavior
Cache Hit	Return stale data and start fetching fresh data.
Cache Miss	Fetch fresh data from origin, update cache, return fresh data.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Cache Hit] --> B{Is Cache Valid?}
    B -->|Yes| C[Return Cached Data]
    B -->|No| D[Fetch Fresh Data from DB]
    D --> E[Update Cache with New Data]
    C --> F[Respond to Client with Stale Data]
    E --> F
```

**Example input / output:**

```
Input: A request for user data with cache key user_123.
Output (Cache Hit):
JSON
Copy
{
  "name": "Alice",
  "age": 30,
  "lastUpdated": "2023-09-05T14:30:00Z"
}
Input: A request for user data with cache key user_123 after a database update.
Output (Cache Miss + Fresh Data):
JSON
Copy
{
  "name": "Alice (updated)",
  "age": 31,
  "lastUpdated": "2023-09-06T15:30:00Z"
}
And the cache is updated to reflect this new data.
```

**Data:**

```json
{
  "cacheKey": "user_123",
  "cacheValue": {
    "name": "Alice",
    "age": 30,
    "lastUpdated": "2023-09-05T14:30:00Z"
  },
  "freshDataFromDB": {
    "name": "Alice (updated)",
    "age": 31,
    "lastUpdated": "2023-09-06T15:30:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Request
4. Cache Hit?
5. Return Cached Data
6. Start Fetching Fresh Data
7. Fresh Data from DB
8. Update Cache with New Data
9. Respond to Client with FreshData

