---
type: "practice"
difficulty: "Deep"
---

# You're designing a bulk notification fan-out to millions of users; how do you reason about the O(recipients) work, chunking/sharding it, and the memory of holding recipient batches in flight?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system to handle bulk notifications for millions of users, we need to ensure that both time complexity (O(n)) and space complexity are managed effectively. The core challenge here is to distribute the workload efficiently while minimizing memory usage and ensuring low latency. This involves chunking recipients into manageable batches, sharding the task across multiple worker nodes, and managing the state and memory of these batches.

#### Triage

The question asks us to reason about how we handle (O(n)) work for a large number of notifications (where (n) is the number of recipients). Specifically, we need to consider:

Chunking/Chunk Size: How do we divide the list of recipients into smaller chunks to process them in parallel?
Sharding Strategy: How many worker nodes should we use and how do they interact with each other?
Memory Management: What is the optimal way to hold these batches of recipients in memory without causing a memory bottleneck?

#### First-Principles Solution

Chunking/Chunk Size:

We need to determine an appropriate chunk size that balances parallelism and communication overhead.
A common heuristic is to start with a small chunk size (e.g., 100 recipients) and adjust based on the system's performance.

Sharding Strategy:

Use a distributed sharding strategy where each worker node processes a subset of chunks.

#### Cons
ider using a consistent hashing algorithm to distribute users across nodes, ensuring that the load is balanced.
Implement a coordination service (e.g., a message broker) to manage task assignments and result aggregation.

Memory Management:

Use in-memory data structures like sets or maps to hold batches of recipients.
Optimize memory usage by reusing batch objects and minimizing overhead.
Monitor memory consumption closely, especially during peak load times.

#### Example
Workflow
Task Initialization: The master node initializes the task with a list of (n) recipients.
Chunking: The master splits the list into smaller chunks (e.g., 500-1000 recipients per chunk).
Sharding: Each worker node is assigned one or more chunks to process.
Notification Processing: Workers send notifications in parallel, maintaining state for ongoing batches.
Result Aggregation: Once all batches are processed, the results are aggregated back to the master node.

#### Closing Summary

The key takeaway here is that by carefully chunking and sharding the workload, we can achieve a scalable system capable of handling millions of recipients efficiently. The choice of chunk size and sharding strategy directly impact performance, so it’s crucial to test and fine-tune these parameters in real-world scenarios.

#### Common Mistakes/Follow-Ups
Mistake: Overly aggressive chunking that leads to excessive communication overhead.

#### Follow-Up
Discuss edge cases such as varying recipient update frequencies or handling new subscribers during the batch processing period.

**Comparison:**

```
Chunk Size	Pros	Cons
Small	Low communication overhead	More workers needed
Large	Fewer workers needed	Higher communication overhead
```

**Code:**

```python
# Example code for sharding strategy using consistent hashing

def get_shard_key(user_id):
    return user_id % num_shards

def assign_tasks(workers, recipients):
    task_queue = []
    for recipient in recipients:
        shard_index = get_shard_key(recipient['id'])
        worker = workers[shard_index]
        task_queue.append((worker, recipient))
    return task_queue
```

**Example input / output:**

| Input: List of recipients | Output: Task queue for workers |
|--------------------------|-------------------------------|
| [123, 456, 789]           | [(worker0, 123), (worker1, 456)] |

**Data:**

```json
{
  "users": [
    {"id": 123, "email": "user1@example.com"},
    {"id": 456, "email": "user2@example.com"},
    // ... millions of users
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initialize task
4. Is chunk size optimal?
5. Split list into chunks
6. Tune chunk size and retry
7. Assign tasks to workers
8. Process notifications in parallel
9. Aggregate results
10. Complete task

