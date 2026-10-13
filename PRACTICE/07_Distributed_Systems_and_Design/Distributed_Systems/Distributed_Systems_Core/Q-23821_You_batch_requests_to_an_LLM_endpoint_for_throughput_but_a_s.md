---
type: "practice"
difficulty: "Deep"
---

# You batch requests to an LLM endpoint for throughput, but a single very long generation in the batch holds up all the shorter ones. Explain this head-of-line blocking in static batching and how continuous batching mitigates it, and what scheduling problem the router now owns.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadline Propagation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, static batching can lead to significant performance bottlenecks due to head-of-line (HOL) blocking. When a single request in a batch takes an exceptionally long time, it blocks all other requests in that batch from being processed immediately. This is problematic because shorter, simpler requests could be resolved much faster if they weren't waiting behind the longer one.

Continuous batching, on the other hand, avoids this issue by processing requests as soon as they are ready, thereby reducing HOL blocking and improving overall throughput. However, continuous batching introduces its own scheduling challenges that need to be managed carefully. The router now has a more complex task: it must efficiently manage incoming requests while ensuring that no single request dominates the system.

Let's break this down from first principles:

Understanding Head-of-Line Blocking in Static Batching

In static batching, all requests are grouped together at once and processed as a batch. If one request in the batch takes a long time to generate a response, it will hold up the entire batch. This is because the system waits for all requests in the batch to be ready before starting any processing.

For example:

Request A: 1 second
Request B: 2 seconds
Request C: 50 seconds

In static batching, the batch would wait until Request C is completed, which could take up to 50 seconds. During this time, Requests A and B are idle and contribute no value.

Continuous Batching: Mitigating HOL Blocking

Continuous batching addresses this issue by processing requests as soon as they are ready, rather than waiting for a batch of requests to accumulate. This approach ensures that shorter requests can be processed quickly, even if longer requests are still pending.

In continuous batching:

Request A (1 second) is processed immediately after it arrives.
Request B (2 seconds) starts processing once Request A completes, and so on.

This way, the system can maintain high throughput by minimizing idle time. The key challenge now becomes managing a potentially never-ending stream of requests while ensuring fairness and efficient use of resources.

Scheduling Problem for the Router

The router must now handle the following scheduling challenges:

Load Balancing: Efficiently distributing incoming requests across available processing units to avoid overloading any single resource.
Response Time Management: Ensuring that shorter, time-sensitive requests are processed quickly without getting blocked by longer-running ones.
Resource Utilization: Maximizing CPU and memory usage while minimizing idle time.

Here’s a simple example to illustrate the difference:

Batch Processing (Static)	Continuous Processing
[A(1s), B(2s), C(50s)]	A(1s)
	B(2s)
	C(50s)

In the static batch, Request C dominates and delays processing of Requests A and B. In continuous batching, each request is processed immediately upon completion.

#### Common Mistakes
and Follow-Ups
Ignoring Shorter Requests: Failing to prioritize shorter requests can lead to underutilization of resources.
Over-Processing Large Requests: Allowing a few large requests to dominate the system can cause other valuable requests to be delayed or dropped.
Scheduling Algorithms: Choosing an appropriate scheduling algorithm is crucial for balancing performance and resource allocation.

To mitigate these issues, advanced techniques like priority-based scheduling, adaptive batch sizes, and queuing strategies can be employed.

#### Closing Summary

In summary, static batching leads to head-of-line blocking, where a single long request delays all shorter ones. Continuous batching mitigates this by processing requests as soon as they are ready, but it introduces scheduling challenges that the router must address through careful load balancing, response time management, and resource utilization strategies.

**Comparison:**

```
Request ID	Processing Time (s)
1	1
2	2
3	50
```

**Code:**

```python
# Example of continuous batching in Python

def process_requests(requests):
    for request in requests:
        start_time = time.time()
        # Simulate processing time
        print(f"Processing {request} - took {time.time() - start_time:.2f}s")

# Sample requests with different processing times
requests = [1, 2, 50]

process_requests(requests)
```

**Example input / output:**

```text
| Input Requests | Output Processing Times |
|----------------|------------------------|
| [1s, 2s, 50s]   | 1.00s -> 3.00s -> 53.00s |

This table illustrates the difference between static and continuous batching.
```

**Data:**

```json
{
    "requests": [
        {"id": 1, "processing_time": 1},
        {"id": 2, "processing_time": 2},
        {"id": 3, "processing_time": 50}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Request A
5. Process?
6. Process Request A
7. D
8. Wait for Request C toComplete
9. Process?
10. Idle
11. Process Request B

