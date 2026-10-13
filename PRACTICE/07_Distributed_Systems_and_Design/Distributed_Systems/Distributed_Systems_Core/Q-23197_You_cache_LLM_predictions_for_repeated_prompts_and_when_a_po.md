---
type: "practice"
difficulty: "Intermediate"
---

# You cache LLM predictions for repeated prompts, and when a popular prompt's cache entry expires under load, 10k identical inferences fire at once. Explain the cache-stampede risk and how request coalescing prevents the herd.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In a distributed system where LLM (Language Model) predictions are cached, a "cache stampede" can occur when a popular prompt's cache entry expires under load and triggers 10k inferences simultaneously. Request coalescing is a strategy to mitigate this risk by ensuring that requests for the same prompt are consolidated into fewer, more efficient batches.

#### Triaging the Question

The question asks us to explain two key concepts: cache stampede and request coalescing. We need to understand how these mechanisms interact in the context of a distributed system with LLMs (Large Language Models) that use caching for repeated prompts. Specifically, we must address why cache entries expiring under load can lead to performance issues and how request coalescing helps mitigate those issues.

#### Assumptions
Cache Expiration: The cache entry for a popular prompt expires.
Load Conditions: Multiple requests are made for the same prompt at once.
Inference Cost: Each inference is resource-intensive, which can cause performance degradation if not managed properly.

#### First-Principles Solution
Cache Stampede

Cache Expiry Scenario:

Assume a frequently requested prompt's cache entry expires under heavy load.
This means that multiple clients will simultaneously attempt to fetch the response from the system, leading to a spike in requests.

Simultaneous Requests:

In a distributed system, these 10k requests may hit different instances of the service or even bypass caching mechanisms due to cache expiry.
As a result, all these requests will trigger separate inferences, causing a significant load on the backend.

Performance Impact:

This sudden increase in inference requests can overwhelm the system, leading to increased latency and potentially even downtime if the server cannot handle the spike in load.
The repeated execution of identical inferences also leads to unnecessary resource consumption and inefficiencies.
Request Coalescing

#### Cons
olidation Strategy:

To mitigate this issue, request coalescing is employed. This strategy ensures that multiple incoming requests for the same prompt are combined into a single or fewer batches.

Batch Processing:

The system detects when many clients are requesting the same prompt and groups these requests together.
Instead of processing each request individually, it processes a batch containing all such requests in one go.

Efficiency Gains:

By reducing the number of inference operations required, the system significantly reduces the load on backend resources.
This approach minimizes latency for individual users and ensures that the system remains responsive under high traffic conditions.

#### Implementation
Example:

A simple implementation might use a queue to buffer incoming requests and then process batches from this queue in parallel or sequentially.
Advanced systems could employ sophisticated algorithms to dynamically adjust batch sizes based on real-time load metrics.

#### Closing Summary
+ Common Mistakes
Cache Stampede Risk: If not managed, cache expirations under heavy load can lead to a sudden spike in inference requests, causing performance issues and potential downtime.
Request Coalescing Benefits: By consolidating similar requests into batches, the system can efficiently handle high traffic while minimizing resource usage and maintaining responsiveness.

#### Common Mistakes

Failing to recognize the importance of cache expiration management.
Not understanding that simultaneous requests for the same prompt can lead to redundant computations.
Overlooking the need for dynamic batch sizing in request coalescing strategies.
Final Thoughts

Understanding both cache stampede risks and how request coalescing mitigates these issues is crucial for designing robust, scalable distributed systems. Effective management of cache expiration events and efficient handling of simultaneous requests are key to ensuring smooth operation under varying loads.

**Code:**

```python
# Example of a simple request coalescing mechanism using a queue

import time
from collections import defaultdict

class RequestCoalescer:
    def __init__(self, batch_size=10):
        self.queue = defaultdict(list)
        self.batch_size = batch_size

    def add_request(self, prompt):
        self.queue[prompt].append(time.time())

    def process_requests(self):
        # Process batches from the queue
        for prompt, requests in list(self.queue.items()):
            if len(requests) >= self.batch_size:
                self.process_batch(prompt)
                del self.queue[prompt]

    def process_batch(self, prompt):
        # Simulate batch processing (e.g., fetch predictions)
        print(f"Processing batch for {prompt}")
```

