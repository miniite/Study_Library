---
type: "practice"
difficulty: "Deep"
---

# How would you cache a value that a handful of admin users can change instantly but millions of end users read, so admins see writes immediately and users within seconds?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Caching Expensive Queries](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this caching challenge, we need to design a system where a small number of administrators can make changes that are visible almost instantaneously to them, while ensuring that millions of end-users see these changes with minimal latency (seconds). The solution involves a combination of local and distributed caches, consistent hashing for key distribution, and a robust invalidation strategy. Here’s how we can approach it:

TL;DR: We will use a two-tier caching architecture: a Redis cluster for real-time updates and a memcached or similar cache for broader read access. Admins write directly to the Redis cluster, which is then propagated via a reliable messaging system (like Kafka) to other instances of the same application, ensuring eventual consistency across all nodes.

#### Triage
The core challenge here is achieving sub-second visibility for admins while maintaining low-latency reads for end-users. We need a mechanism that guarantees immediate writes for admins and fast, consistent reads for users, with some degree of tolerance to staleness in the user cache due to eventual consistency.

#### First-Principles Solution

Local Cache (Redis): Admins can write directly to Redis, which is designed for real-time operations.
Distributed Cache (memcached): User requests will hit this layer first. If the data is not available here, it will fetch from Redis and invalidate the user cache entry.
Messaging System (Kafka): To ensure eventual consistency across all nodes, updates are pushed to Kafka topics which are then consumed by other instances of the application.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The solution balances between performance and consistency, prioritizing admin visibility with immediate writes but accepting a small window where end-users might see stale data.

#### Common mistakes
include failing to properly handle cache invalidation or not considering the trade-offs in choosing between different caching strategies. It’s crucial to understand the impact of eventual consistency on user experience.

