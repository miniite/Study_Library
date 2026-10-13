---
type: "practice"
difficulty: "Expert"
---

# You must increment a per-user counter millions of times per second across many threads. Compare a single atomic, striped/sharded counters (LongAdder-style), and thread-local aggregation. What breaks first at scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the concurrency pattern challenge in incrementing a per-user counter with high throughput, we need to compare two strategies: using an atomic striped/sharded counter (such as LongAdder), and employing thread-local aggregation. The key is to understand how these patterns handle concurrent updates and which one might break first under heavy load.

The core of the question revolves around concurrency control and understanding race conditions in multithreaded environments. We need to evaluate both approaches based on their performance, scalability, and reliability under heavy loads. Let's break it down step by step from first principles.

First, let’s define our terms:

Atomic Striped/Sharded Counter (e.g., LongAdder): This is a specialized concurrent counter that uses an array of atomic long variables to distribute the updates across multiple threads, reducing contention.
Thread-Local Aggregation: Each thread maintains its own local counter, and periodically aggregates these counts into a global total.

Now let's analyze each approach in detail:

Atomic Striped/Sharded Counter (e.g., LongAdder)

#### Implementation
Details:

LongAdder uses an array of atomic long variables to distribute the load across multiple threads.
Each thread updates a specific slot in this array, reducing contention compared to a single global lock.

Advantages:

Better scalability due to reduced contention on a single lock.
Efficient memory usage as it only allocates slots for active users.

Disadvantages:

Still has some inherent overhead from managing the array and atomic updates.
May introduce false sharing if threads update adjacent slots in the array.
Thread-Local Aggregation

#### Implementation
Details:

Each thread maintains its own local counter.
Periodically, these local counters are aggregated into a global total to ensure accuracy.

Advantages:

Simplifies implementation as there is no need for complex atomic operations.
Easier to reason about as each thread operates independently.

Disadvantages:

Higher overhead from frequent aggregation and synchronization.
May introduce latency due to the periodic updates and aggregations.
Comparative Analysis
Scalability: LongAdder is generally more scalable because it distributes the updates across multiple threads, reducing contention on a single point. Thread-local aggregation might become a bottleneck if too many threads are updating their local counters simultaneously.
Latency: LongAdder offers lower latency as increments are performed immediately without waiting for an aggregation phase. In contrast, thread-local aggregation can introduce significant latency due to the periodic aggregation steps.
Memory Efficiency: LongAdder is more memory-efficient by only allocating slots for active users, whereas thread-local aggregation requires local storage in each thread.

#### Common Mistakes
and Follow-Ups
Ignoring False Sharing: While LongAdder reduces contention, it can still suffer from false sharing if threads update adjacent slots.
Overlooking Periodic Aggregation Overhead: Thread-local aggregation might break first due to the overhead of periodic aggregation steps, which can introduce significant latency and reduce overall throughput.
Not Considering Long-Term Stability: LongAdder provides a more stable long-term solution by maintaining accuracy without frequent aggregations.

In conclusion, for high-throughput scenarios where immediate updates are critical, LongAdder is generally the better choice due to its reduced contention and lower latency. However, in environments with less stringent real-time requirements, thread-local aggregation might be simpler to implement but could break first under heavy load due to increased overhead from periodic aggregations.

