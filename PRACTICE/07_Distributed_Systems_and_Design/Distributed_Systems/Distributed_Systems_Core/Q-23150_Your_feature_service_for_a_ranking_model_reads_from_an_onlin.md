---
type: "practice"
difficulty: "Intermediate"
---

# Your feature service for a ranking model reads from an online store at p99 of 20ms but the model needs p99 of 50ms end-to-end including generation. Walk me through how you'd budget the deadline across hops and what you'd cut or parallelize.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deadline Propagation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, our feature service ranks items based on a model that requires data from an online store with a p99 latency of 20ms but needs to meet a stricter end-to-end deadline of 50ms, including the time for generating the ranking. The goal is to ensure the entire pipeline meets the required latency while optimizing resource usage and performance. Let's break down how we would budget this deadline across the different hops in the system.

Firstly, it’s crucial to understand that both p99 latencies are given as 20ms, but the total end-to-end requirement is 50ms. This means there's an additional 30ms of latency that must be managed within our service or any subsequent processes. We need to determine where this extra time can be allocated and whether it’s feasible to achieve the goal through parallelization or optimization.

#### Triaging the Question

The interviewer is probing how well you understand distributed systems, specifically in managing latency across multiple components. The key aspects are:

Understanding p99 latency.
Determining how much additional time (30ms) can be allocated.

#### Cons
idering parallel processing and optimizing resource usage.

#### Assumptions
and Reasoning
We assume the online store is a reliable source with consistent performance at 20ms p99.
The feature service has some flexibility in terms of how it processes data, but we need to ensure the end-to-end requirement is met without violating the service's SLA.

#### First Principles Solution

Assess Current Latency:

We start by confirming that the online store provides consistent 20ms p99 latency.
The feature service needs an additional 30ms to meet the end-to-end requirement of 50ms.

Budgeting Across Hops:

First Hop (Online Store): Since this is a well-understood and reliable component, we can budget up to 20ms for data retrieval.
Second Hop (Data Processing): This hop should ideally account for the remaining 30ms. However, we need to consider if there are ways to reduce or parallelize tasks.

Parallelization:

If possible, we can parallelize tasks such as feature extraction, caching, and model scoring to overlap processing times.
For example, we might use an asynchronous approach where data is fetched in parallel with some initial preprocessing steps starting immediately after fetching.

Optimization Techniques:

Implementing caching strategies for frequently accessed data can significantly reduce latency.
Using preloading or speculative loading techniques can help mitigate the delay by ensuring that required resources are ready before they are needed.
Optimizing network I/O operations and minimizing overhead in processing pipelines can also contribute to reducing overall latency.

Monitoring and Feedback:

Continuous monitoring of each component’s performance is essential to ensure no bottlenecks arise during runtime.
Implementing a feedback loop where the system dynamically adjusts its behavior based on real-time performance metrics can help maintain consistent service levels.

Failure Modes:

Ensure robust error handling and retries are in place to avoid cascading failures that could impact overall latency.
Design the system to handle potential spikes in load gracefully, possibly by throttling or queuing requests during high-traffic periods.

#### Closing Summary

In conclusion, we need to budget 20ms for data retrieval from the online store and utilize a combination of parallelization, optimization techniques, and monitoring mechanisms to achieve the remaining 30ms required for processing. This approach ensures that the feature service meets its end-to-end latency requirements while maintaining efficient resource usage.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the online store's consistent performance and assuming all additional time must be managed within the feature service.

#### Follow-Up
Discussing scenarios where the online store’s performance might degrade, and how to handle such situations without violating SLAs.

**Comparison:**

```
Component	Latency (p99)
Online Store	20ms
Data Processing	30ms
```

**Code:**

```python
# Example of parallel processing in Python using threading or asyncio

import asyncio

async def fetch_data(url):
    # Simulate fetching data with 20ms p99 latency
    await asyncio.sleep(0.02)

async def process_data(data):
    # Simulate data processing, potentially taking up to 30ms p99
    await asyncio.sleep(0.04)  # Assuming this includes some initial async tasks

async def main():
    url = "http://online-store.com/data"
    data = await fetch_data(url)
    await process_data(data)

# Run the main function using asyncio
import threading
threading.Thread(target=asyncio.run, args=(main(),)).start()
```

**Example input / output:**

```text
Input:
- URL: http://online-store.com/data

Output:
- Retrieved data from online store with p99 latency of 20ms.
- Processed the retrieved data using parallelized tasks, ensuring end-to-end processing time remains within 50ms p99.

Dry Run:
1. Fetch Data (20ms)
2. Parallel Process Data (4ms + asynchronous tasks overlap)
3. Total End-to-End Time: 24ms
```

**Data:**

```json
{
  "url": "http://online-store.com/data",
  "data": {
    "items": [
      {"id": 1, "features": [0.1, 0.2, 0.3]},
      {"id": 2, "features": [0.4, 0.5, 0.6]}
    ]
  }
}
```

**Flow:**

1. Online Store
2. Fetch Data
3. Data Processing
4. Parallelize Tasks
5. End-to-End Latency <= 50ms

