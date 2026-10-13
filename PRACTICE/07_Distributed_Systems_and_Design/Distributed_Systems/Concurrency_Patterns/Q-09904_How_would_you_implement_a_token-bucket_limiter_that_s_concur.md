---
type: "practice"
difficulty: "Deep"
---

# How would you implement a token-bucket limiter that's concurrency-safe in-process without a global lock per request? Discuss atomic refill math and the race between refill and consume.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a concurrency-safe token bucket limiter within an in-process system, avoiding a global lock per request, we need to ensure that multiple concurrent operations can safely update the token bucket state without race conditions. This involves using atomic operations and careful synchronization logic. The key aspects include understanding the refill algorithm, ensuring thread safety during both refills and consumes, and managing the race condition between these two operations.

