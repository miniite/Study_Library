---
type: "practice"
difficulty: "Deep"
---

# You cache model predictions for repeated inputs to cut GPU cost. Explain the cache-stampede risk when a popular input's cache entry expires under heavy load, and how request coalescing prevents 10k simultaneous identical inferences.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

Cache stampede is a critical issue that can arise when cached model predictions expire during high traffic, leading to a large number of requests hitting the backend simultaneously. Request coalescing is an effective strategy to mitigate this risk by ensuring that multiple identical requests from different clients are merged into a single request.

#### Triage

The question asks us to explain two key concepts:

Cache Stampede: The situation where cache entries expire during peak load, causing many identical requests to hit the backend simultaneously.
Request Coalescing: A strategy that prevents multiple requests from being sent to the backend when they are identical and can be handled by a single request.

We will assume that:

We have a caching system in place for model predictions.
The model predictions expire after some period, leading to cache misses during high traffic.
There are 10k simultaneous identical requests hitting the system due to expired caches.

#### First Principles Solution
Step-by-Step Analysis

Understanding Cache Stampede

When a popular input's cache entry expires, multiple clients that have cached predictions for this input might simultaneously fetch new predictions from the backend.
If these requests hit the backend at once, it can overwhelm the system, leading to high latency and potential failures due to resource constraints.

Cache Expiry and Concurrent Requests

Assume a model predicts 10k users' sentiments based on their input text. When the cache for this input expires, each of these 10k clients will make a request.
If all these requests hit the backend at once (e.g., due to high load), it can cause a significant bottleneck.

Impact on Backend

High traffic on the backend can lead to increased resource usage, potentially causing CPU and memory bottlenecks.
The backend might not be able to handle such a surge in requests, leading to delayed responses or even service downtime.

Request Coalescing Strategy

To mitigate this issue, we need to ensure that multiple identical requests are merged into a single request at the cache layer.
This approach reduces the number of requests hitting the backend and helps manage the load more efficiently.

#### Implementation
Details

Introduce a hash-based identifier for each unique request based on the input parameters.
Use this identifier to group incoming requests with identical inputs and send only one back to the backend.
Implement a mechanism to ensure that the first request that hits the system is processed, while subsequent identical requests are ignored or delayed.

Benefits of Request Coalescing

Reduces the number of backend requests by merging multiple identical requests.
Improves overall system performance and responsiveness during high load conditions.
Ensures that only one instance of a request is sent to the backend, reducing resource consumption.

#### Closing Summary

Cache stampede can significantly impact system performance when cache entries expire during peak traffic. To mitigate this risk, implementing request coalescing helps manage incoming requests more efficiently by merging identical requests into a single back-end call. This approach ensures that the system remains responsive and reduces the load on backend resources.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring cache stampede risks can lead to high backend load, causing service degradation or downtime.

#### Follow-Up
Discussing alternative strategies such as using distributed locks or implementing more sophisticated caching mechanisms like write-through caches.

**Comparison:**

```
Client	Input Parameters	Result
A	"Example text"	P1
B	"Example text"	P2
C	"Example text"	-
D	"Example text"	-
Client	Input Parameters	Result
E	"Different text"	P3
```

**Code:**

```python
# Example of a simple request coalescing mechanism in Python

from collections import defaultdict
import hashlib

cache = defaultdict(set)

def process_request(input_params):
    # Generate a unique identifier for the input parameters
    key = hashlib.md5(str(input_params).encode()).hexdigest()

    if key not in cache:
        # No existing entry, proceed with backend request and cache it
        result = get_prediction_from_backend(input_params)
        cache[key].add(result)
        return result

    # Check if the entry has been processed already
    for result in cache[key]:
        if result is None:  # Assuming we handle errors appropriately
            continue
        return result

    # If no valid result, process and cache it
    new_result = get_prediction_from_backend(input_params)
    cache[key].add(new_result)
    return new_result

def get_prediction_from_backend(params):
    # Simulated backend call to predict model output
    import time
    time.sleep(0.1)  # Simulating delay
    return "predicted_value"
```

**Example input / output:**

```text
Input:
- Client A requests prediction for "Example input text" at timestamp 1683459720.
- Client B requests prediction for the same "Example input text" at a slightly later time.

Output:
- Cache miss; both clients send request to backend. Backend processes and caches result.
- Subsequent identical client requests (C, D) are ignored or delayed until cache is processed.
```

**Data:**

```json
{
  "input_params": {
    "text": "Example input text",
    "timestamp": 1683459720
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client Request
6. Cache Hit?
7. Use Cache
8. Generate Unique Key
9. Cache Miss
10. Backend Call
11. Store Result in Cache
12. Store Result in Cache
13. Client Response
14. Client Response
15. Is Duplicate Request?
16. Use Existing Cache Entry
17. Proceed to Backend Call

