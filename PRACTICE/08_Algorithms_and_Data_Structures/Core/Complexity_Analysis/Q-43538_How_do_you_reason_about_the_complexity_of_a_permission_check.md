---
type: "practice"
difficulty: "Deep"
---

# How do you reason about the complexity of a permission check cached per (user, resource) pair, where the cache is O(users x resources) in the worst case, and how you'd bound it with scoping and TTL?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming vs In-memory Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Streaming_vs_In-memory_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with permissions checks in a system that caches results per (user, resource) pair, understanding the time and space complexity is crucial to ensure efficient performance and avoid scalability issues. The cache's worst-case space complexity is (O(\text{users} \times \text{resources})), which means it could potentially store every unique combination of users and resources. The question asks us to reason about this complexity and how we can manage it using scoping and TTL (Time-To-Live) mechanisms.

#### Triaging the Question

The core of the problem is understanding both the time and space complexities involved in caching permission checks, and determining ways to optimize these metrics through strategic use of scoping and TTLs. We need to consider:

Space Complexity: The cache might grow unbounded if not managed.
Time Complexity: Each permission check could potentially involve fetching data from the cache or computing it anew.

#### First-Principles Solution

Let's break this down step-by-step:

1. Understanding the Cache Structure

The cache is indexed by a unique key: (user, resource). This means each user-resource pair can be cached independently. The worst-case scenario is that all possible combinations are stored in the cache.

2. Space Complexity Analysis
Cache Size: (O(\text{users} \times \text{resources})). If there are many users and resources, this could become prohibitive.
Scoping: We can reduce the number of cached entries by leveraging application logic to limit the scope of what needs to be checked. For example, if a user only has permissions to certain resource types or within specific departments, we can cache those relevant pairs.
3. Time Complexity Analysis
Cache Hit vs Miss: A hit is fast (constant time), but a miss requires either fetching data from storage or computing the permission.
TTL Mechanism: Implementing TTL ensures that cached entries are eventually evicted, reducing stale data and freeing up space.
Detailed Solution

To reason about this complexity effectively:

Scoping Permissions:

Use application logic to scope permissions by user roles or resource types. For instance, if a user has only read access to certain resources, we can cache those specific (user, resource) pairs.

#### Example
If there are 10 users and 5 resource types, but each user only interacts with one type of resource (e.g., Admins -> r1, Users -> r2), the effective space complexity could be much lower.

TTL Mechanisms:

Implement a TTL for cached entries to ensure they are refreshed periodically.

#### Example
Set a TTL of 5 minutes, so if no access is made within that period, the entry is evicted and recalculated on next access.

Cache Eviction Policies:

Use least recently used (LRU) or other eviction policies to manage cache size when it exceeds a threshold.

#### Example
If the cache size grows beyond 10% of total capacity, start evicting the least frequently used entries.

#### Closing Summary

By applying scoping and TTL mechanisms, we can significantly reduce both space and time complexities in our permission check caching system. Scoping helps limit unnecessary caching, while TTL ensures that cached data remains fresh and relevant.

#### Common Mistakes/Follow-Ups
Over-reliance on Cache: Not considering the scenario where all users might have access to all resources.
Ignoring Eviction Policies: Failing to implement strategies for managing cache size when it grows too large.
Not Considering Realistic Scenarios: Assuming a worst-case distribution of permissions, which may not always hold.

**Comparison:**

```
User	Role	Resources
1	Admin	r1
2	User	r2
```

**Code:**

```python
# Example of implementing TTL with a basic LRU policy using Python's collections module
from collections import LRUCache

class PermissionCache:
    def __init__(self, ttl=300):  # 5 minutes in seconds
        self.cache = LRUCache(maxsize=1024)
        self.ttl = ttl

    def check_permission(self, user_id, resource):
        key = (user_id, resource)
        if key not in self.cache:
            # Simulate permission check from storage or computation
            result = simulate_permission_check(user_id, resource)
            self.cache[key] = {'result': result, 'time': time.time()}
        else:
            entry = self.cache[key]
            if time.time() - entry['time'] > self.ttl:
                del self.cache[key]  # Evict stale entry
                return None
            result = entry['result']
        return result

def simulate_permission_check(user_id, resource):
    # Simulate permission check logic (e.g., querying a database)
    pass
```

**Example input / output:**

```text
Input:
- User ID: 1
- Resource Type: r1

Output:
- Permission Result: True (or False based on the actual permission check)

Dry-run example for a user with admin role and resource type `r1`:
Cache lookup -> Hit, cached entry is valid, return result as true.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "role": "admin"},
    {"id": 2, "role": "user"}
  ],
  "resources": [
    {"id": 1, "type": "r1"},
    {"id": 2, "type": "r2"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. User Access Request
6. Is Cache Hit?
7. Return Cached Result
8. Check if TTL Expired?
9. Compute Permission, Store inCache
10. Remove Stale Entry fromCache

