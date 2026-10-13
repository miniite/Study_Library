---
type: "practice"
difficulty: "Deep"
---

# Explain the search and write thread pools and their bounded queues. When you see EsRejectedExecutionException, what does it actually mean and how do you respond correctly rather than just raising the queue size?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Index Internals and Mappings](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing Elasticsearch's search and write thread pools, we're delving into the architecture of its internal processing layers. The core idea is to ensure that Elasticsearch can handle a high volume of requests efficiently by distributing them across multiple threads. Thread pools are essential in managing concurrent operations while maintaining system stability.

#### Triage

The question probes your understanding of how Elasticsearch manages and processes search and write operations, particularly focusing on the thread pools and their associated bounded queues. The EsRejectedExecutionException is a critical exception that occurs when the queue bound for a specific thread pool is full, indicating potential bottlenecks or misconfigurations.

#### First-Principles Solution

Elasticsearch uses two main thread pools: one for search operations (search) and another for write operations (write). These thread pools are designed to handle different types of workloads efficiently. Each thread pool has a specified number of threads, a queue size (bounded by a certain limit), and a maximum thread count.

Search Thread Pool:

Role: Handles search queries.
Configuration: Typically configured with fewer threads but higher queue sizes to allow for backpressure in the event of heavy querying.
Bounded Queue: A fixed-size queue that holds search requests until they can be processed by a thread.

Write Thread Pool:

Role: Manages document indexing and other write operations.
Configuration: Often configured with more threads to handle frequent updates, but with lower queue sizes compared to the search thread pool due to its nature of handling fewer but often blocking operations.
Bounded Queue: A fixed-size queue that holds write requests until they can be processed.

When EsRejectedExecutionException is thrown, it means a request couldn't fit into the bounded queue and was thus rejected by the thread pool. This can happen due to various reasons, such as too many simultaneous search or write operations overwhelming the system's capacity.

Response Strategy

To address EsRejectedExecutionException, we need to understand its root cause and take corrective actions:

Reduce Concurrent Requests: Temporarily reduce the number of concurrent requests if possible.
Increase Thread Pool Size: Consider increasing the thread pool size, especially for write operations which often have lower queue limits.
Adjust Queue Sizes: Increase the bounded queue sizes to accommodate more simultaneous requests.
Optimize Operations: Analyze and optimize slow or blocking operations that might be causing excessive load.
Monitor System Health: Continuously monitor system metrics like CPU, memory usage, and request latency to identify bottlenecks.

#### Closing Summary

Understanding how Elasticsearch manages its search and write thread pools is crucial for ensuring a stable and performant system. By recognizing the implications of EsRejectedExecutionException, we can proactively manage and optimize our Elasticsearch instances to handle high loads effectively.

#### Common Mistakes/Follow-Ups
Ignoring the Exception: Simply ignoring the exception without investigating its cause can lead to persistent performance issues.
Over-Reacting by Increasing All Settings: Improperly increasing all thread pool settings indiscriminately may not resolve the issue and can introduce other problems.
Lack of Monitoring: Failing to set up proper monitoring can make it difficult to identify when adjustments are necessary.

**Comparison:**

| Thread Pool | Type    | Threads | Queue Size |
|-------------|---------|---------|------------|
| Search      | Fixed   | 50      | 200        |
| Write       | Fixed   | 100     | 50         |

**Code:**

```java
// Example configuration snippet for search thread pool
threadPool {
  search {
    type: fixed
    threads: 50
    queue_size: 200
  }
}

// Example configuration snippet for write thread pool
write {
  type: fixed
  threads: 100
  queue_size: 50
}
```

**Example input / output:**

```text
Input:
- Search request rate: 600 qps, Write request rate: 200 w/s

Output:
- `EsRejectedExecutionException` is thrown for write requests due to queue overflow.
- Response: Increase write thread pool size and queue to handle higher load.

Dry Run:
- Increase write thread pool threads to 150 with a queue size of 75.
```

**Data:**

```json
{
  "thread_pools": {
    "search": {
      "type": "fixed",
      "threads": 50,
      "queue_size": 200
    },
    "write": {
      "type": "fixed",
      "threads": 100,
      "queue_size": 50
    }
  }
}
```

**Flow:**

1. Monitor System Metrics
2. Identify Bottlenecks
3. Adjust Thread Pool Settings
4. Optimize Operations
5. Increase Queue Sizes
6. Reduce Concurrent RequestsTemporarily
7. Monitor System HealthContinuously

