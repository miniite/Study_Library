---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between Redis and Memcached and when would you pick each?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Redis Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Redis_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Redis and Memcached are both popular in-memory data stores, often used to implement caching mechanisms in applications. However, they differ significantly in terms of features, performance characteristics, and use cases. To understand which to pick, it's crucial to consider the requirements of your application.

#### Triaging the Question

The question is asking for a comparison between Redis and Memcached, focusing on their differences and appropriate use scenarios. The interviewer might be probing your understanding of both systems' features, performance, and suitability in various contexts. Assumptions include familiarity with basic caching concepts and an understanding of when to choose one system over the other.

#### First-Principles Solution

Redis vs. Memcached: Key Differences

Data Types: Redis supports a wide range of data types like strings, hashes, lists, sets, sorted sets, and more complex structures such as hyperloglogs and streams. In contrast, Memcached primarily stores simple key-value pairs.

Persistence: Redis offers both in-memory storage and optional disk-based persistence for durability. Memcached is purely in-memory with no built-in persistence.

Expiry Mechanism: Both systems support time-to-live (TTL) expiration but differ in how they handle it. Redis provides more advanced expiry features, such as setting a maximum idle time or using the expireat command to set an absolute timestamp. Memcached only supports simple TTLs.

Concurrency and Locking: Redis includes powerful concurrency primitives like transactions, pub/sub messaging, and distributed locks. Memcached does not offer these features, making it less suitable for scenarios requiring complex coordination between clients.

Performance and Memory Usage: Both are designed to be fast, but Redis can leverage its data structures to achieve higher performance in certain use cases. For example, using hashes or sets can lead to more efficient storage and retrieval operations compared to simple key-value pairs in Memcached.

Use Cases: Use Redis when you need a robust cache with advanced features like persistence, complex data types, and pub/sub capabilities. Choose Memcached when simplicity, low memory footprint, and high performance are paramount, but you do not require the additional functionality provided by Redis.

#### Closing Summary

In summary, while both Redis and Memcached excel in caching scenarios, their differences make them suitable for different use cases. Selecting between them depends on your application's specific requirements regarding data types, persistence needs, concurrency support, and performance trade-offs.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that all caching scenarios should be handled by Memcached due to its simplicity.

#### Follow-Up
What are the implications of choosing Redis over Memcached for a system with stringent data consistency requirements? How would you handle eviction policies in both systems?

**Comparison:**

```
Feature	Redis	Memcached
Data Types	String, Hash, List, Set, Sorted Set, Hyperloglog, Stream	Key-Value Pair
Persistence	In-memory + Optional Disk-Based	None
Expiry Mechanism	Time-to-Live, Absolute Expiration	Simple Time-to-Live
Concurrency	Transactions, Pub/Sub, Locks	None
Performance	Variable Based on Data Type	Fast but Limited
```

**Code:**

```python
# Example Python code snippets for setting TTLs in both Redis and Memcached

# Setting TTL in Redis
import redis
client = redis.Redis()
client.set('key', 'value', ex=3600)  # Expiry after 1 hour

# Setting TTL in Memcached (requires a client library like pylibmc)
from pylibmc import Client, MEMCACHE_NO_FLAGS, MEMCACHE_ETAG_NONE, binary
memc = Client(['localhost'])
memc.set(b'key', b'value', time=3600)  # Expiry after 1 hour
```

**Example input / output:**

```text
Input (Redis):
client.set('user:123', json.dumps({'name': 'John Doe'}), ex=60)
client.get('user:123')  # Response: {'name': 'John Doe'}

Input (Memcached):
memc.set(b'user:123', b'{"name": "John Doe"}', time=60)
memc.get(b'user:123')  # Response: b'{"name": "John Doe"}'
```

**Data:**

```json
{
  "redis": {
    "expiryTypes": ["time-to-live", "absolute-expiration"],
    "dataStructures": ["string", "hash", "list", "set", "sorted-set", "hyperloglog", "stream"]
  },
  "memcached": {
    "expiryTypes": ["time-to-live"],
    "dataStructures": ["key-value-pair"]
  }
}
```

**Flow:**

1. String::Hash::List::Set::SortedSet::Hyperloglog::Stream
2. In-memory::Disk-Based
3. Time-to-Live::AbsoluteExpiration
4. Transactions::Pub/Sub::Locks
5. Key-Value Pair
6. None
7. Time-to-Live
8. None
9. Redis
10. Data Types
11. Advanced
12. Persistence
13. Persistent Redis
14. Expiry
15. G
16. Concurrency
17. Redis Features
18. Memcached
19. Data Types
20. Simple Caching
21. Persistence
22. N
23. Expiry
24. P
25. Concurrency
26. Basic Caching

