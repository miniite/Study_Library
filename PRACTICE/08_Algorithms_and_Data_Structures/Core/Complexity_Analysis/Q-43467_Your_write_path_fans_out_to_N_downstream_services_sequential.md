---
type: "practice"
difficulty: "Deep"
---

# Your write path fans out to N downstream services sequentially, so p99 grows linearly with N; how do you model this as O(N) latency and decide where parallel fan-out or a saga changes the complexity?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we have a write path that fans out to (N) downstream services sequentially, the p99 latency indeed grows linearly with (N). To model this as (O(N)) and determine where parallel fan-out or a saga changes the complexity, we need to understand the trade-offs between these approaches. This discussion will cover the basics of time and space complexity, then delve into specific scenarios.

First, let's clarify what we mean by "p99 latency" and "complexity." The p99 latency refers to the 99th percentile response time, which is a common metric for measuring system performance. Complexity here describes how resource usage (time or space) scales with input size (N).

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Time and Space Complexity: How the performance metrics scale.
Sequential vs Parallel Execution: The trade-offs between these two execution strategies.
Sagas: A pattern used to handle complex transactions across multiple services.

#### Assumptions
Each downstream service takes a constant time (t) to process.
There is no communication overhead or other factors affecting the processing time of individual services.

#### First-Principles Solution
Sequential Execution

In sequential execution, each downstream call is made one after another. The total p99 latency for this path would be the sum of the p99 latencies of all (N) services, i.e., (O(N \cdot t)). Since we are considering p99 latency, we assume that the processing time (t) itself might have some variability. However, the dominant term in complexity analysis is often the number of steps.

Parallel Execution

In parallel execution, you can fan out to multiple services simultaneously. This reduces the overall waiting time if there is no synchronization overhead. The total p99 latency here would be dominated by the slowest service among all (N) services, which is still (O(t)), but the number of steps (fan-out operations) becomes a concern.

Sagas

Sagas involve compensating actions in case one or more downstream calls fail. They are useful when you need to ensure atomicity across multiple services even if some calls take longer than others. The complexity here is higher because each call and its potential rollback might introduce additional overhead, making it (O(N \cdot t')) where (t') includes both the processing time and rollback time.

#### Closing Summary

To model this write path as (O(N)), you would need to carefully choose between sequential and parallel execution based on the specific use case. Sequential execution is simpler but can lead to longer p99 latencies, while parallel execution reduces waiting times but may introduce additional complexity in managing fan-out operations.

#### Common Mistakes

Overlooking the communication overhead when considering parallelism.
Ignoring potential failures and compensating actions which increase complexity with sagas.

#### Follow-ups
might include discussing how to optimize individual service performance (t), or exploring advanced techniques like circuit breakers, retries, and backoff strategies to manage failure modes effectively.

**Code:**

```python
# Example of sequential execution in Python
def sequential_write_path(data):
    for service_id in range(N):
        response = send_request_to_service(service_id, data)
        if not response:
            return False  # Early termination on failure
    return True

# Example of parallel execution using threads (simplified)
import threading

def parallel_write_path(data):
    threads = []
    results = [False] * N

    def handle_service(i):
        nonlocal results
        response = send_request_to_service(i, data)
        results[i] = response  # Store result in shared list

    for i in range(N):
        thread = threading.Thread(target=handle_service, args=(i,))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    return all(results)  # Check if all services succeeded
```

**Example input / output:**

```text
# Example input and output for sequential execution
data = {"key": "value"}
response = sequential_write_path(data)
print("Sequential success:", response)  # Should print True or False based on service responses

# Example input and output for parallel execution
data = {"key": "value"}
response = parallel_write_path(data)
print("Parallel success:", response)  # Should print True if all services succeeded, otherwise False
```

**Data:**

```json
{
  "N": 10,
  "service_ids": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
  "data": {"key": "value"}
}
```

