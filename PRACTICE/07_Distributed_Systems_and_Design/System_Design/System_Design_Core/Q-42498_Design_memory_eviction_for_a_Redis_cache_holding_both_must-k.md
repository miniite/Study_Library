---
type: "practice"
difficulty: "Intermediate"
---

# Design memory eviction for a Redis cache holding both must-keep session data and evictable cache entries in the same instance. How do you stop maxmemory eviction from dropping sessions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read-through and Write-behind Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design memory eviction for a Redis cache that holds both "must-keep" session data and evictable cache entries, we need to ensure that critical session data is not evicted due to maxmemory constraints. This involves understanding the nature of session data, configuring Redis properly, and employing strategies to manage memory usage effectively.

#### Triage

The question asks how to prevent maxmemory eviction from dropping sessions in a Redis instance that stores both session data and cache entries. The key points being probed are:

Understanding Session Data: What kind of data qualifies as must-keep session information.
Redis Configuration: How to configure Redis for different memory management strategies.
Cache Management Strategies: Techniques to ensure critical sessions remain in the cache while other cache entries can be evicted.

#### First Principles Solution
Identify Session Data: Define which data needs absolute retention, such as active user sessions or user authentication tokens.
Configure Redis for Memory Management:
Use maxmemory and maxmemory-policy.
Set up volatile-lru, allkeys-lru, etc., to evict entries based on least recently used (LRU) principles.
Prioritize Session Data: Ensure that session data is not evicted by marking it as volatile or setting specific eviction policies.
Detailed Solution
Session Data Identification:
Session data, such as active user sessions or tokens, must be retained at all costs to avoid user experience issues and security risks.
Redis Configuration:
Set a maxmemory limit: This is the total memory that Redis will use for caching.
Use maxmemory-policy: Choose an appropriate policy like volatile-lru, which evicts entries with the least recent access, but keeps session data marked as volatile.
BASH
maxmemory 100mb
maxmemory-policy volatile-lru

Marking Session Data:

Use Redis commands to mark specific keys as volatile (e.g., EXPIRE or PEXPIRE).
BASH
EXPIRE session:123 60

Eviction Strategy:

The volatile-lru policy ensures that non-session data is evicted first, while volatile keys like sessions are kept.

Monitoring and Alerts:

Set up monitoring to alert on approaching maxmemory limits so you can scale resources if needed.

#### Closing Summary

By configuring Redis with appropriate eviction policies and marking session data as volatile, we ensure that critical user sessions remain in memory even when the cache approaches its maximum size. This approach balances performance with the need to retain essential data.

#### Common Mistakes/Follow-Ups
Mistake: Not identifying which keys are must-keep.

#### Follow-Up
Carefully analyze session data and mark it as volatile using EXPIRE or PEXPIRE.
Mistake: Overlooking the need for monitoring.

#### Follow-Up
Set up alerts to notify when memory usage approaches critical levels.

**Code:**

```bash
# Example Redis configuration snippet
maxmemory 100mb
maxmemory-policy volatile-lru

# Mark a key as volatile
EXPIRE session:123 60
```

**Flow:**

1. Redis Instance
2. Identify Session Keys
3. Configure maxmemory andPolicies
4. Mark Session Keys Volatile
5. Monitor Memory Usage

