---
type: "practice"
difficulty: "Expert"
---

# Your sliding-window rate limiter stores a sorted set of timestamps per key and memory blew up for a hot key with millions of requests; how do you cap memory while keeping accuracy acceptable, and when is an approximate counter fine?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rate Limiting at the Gateway](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with rate limiting using a sliding-window approach in an API Gateway, the primary goal is to track request frequency per key accurately over a defined window of time. However, storing all timestamps for a hot key can lead to excessive memory usage. To manage this without compromising accuracy significantly, we need to introduce mechanisms that cap memory while maintaining acceptable levels of precision.

Firstly, let’s restate what the question is asking: We need to design an efficient system that limits the number of requests per key within a specified time window but also ensures that the memory footprint remains manageable. The key aspects are:

Memory Management: Limiting the storage of timestamps for each key.
Accuracy: Ensuring that rate limiting decisions remain accurate, even if some precision is lost.

To address this, we can use an approximate counting algorithm like the HyperLogLog (HLL) or a combination of sampling and periodic cleanup. Here’s a step-by-step solution:

Step 1: Understand the Problem
Hot Key Issue: A key with a high volume of requests causing excessive memory usage.
Rate Limiter Requirements: Track request frequency over a sliding window while managing memory.
Step 2: Approximate Counting Algorithms

Approximate counting algorithms are designed to provide an estimate of distinct elements in a stream. They offer a balance between accuracy and memory efficiency, which is ideal for our use case.

HyperLogLog (HLL)
Concept: HLL uses probabilistic data structures to approximate the number of unique items in a dataset.

#### Implementation
Store multiple HLL counters per key, each representing different time windows. As requests come in, increment the appropriate counter based on the current timestamp.
Step 3: Hybrid Approach

Combining an exact count for recent requests with an approximate count for older ones can provide both accuracy and memory efficiency:

Recent Requests: Maintain a small, exact count of recent requests.
Older Requests: Use HLL to approximate the number of unique requests over longer periods.
Step 4: Memory Management

To cap memory:

Partition Keys: Split keys into buckets or partitions based on their hotness (number of requests).
Sampling: For less frequent keys, use an exact count.
Periodic Cleanup: Periodically clean up old data to avoid excessive growth. Use a combination of time and frequency thresholds.
Step 5: Designing the Solution

Data Structure:

PYTHON
class RateLimiter:
    def __init__(self, window_size_seconds, max_entries):
        self.window_size_seconds = window_size_seconds
        self.max_entries = max_entries
        self.recent_requests = {}
        self.hll_counters = {}

Tracking Recent Requests:

PYTHON
from time import time

def track_request(key, current_time):
    recent_requests[key] = current_time
    if len(recent_requests) > max_entries:
        oldest_key = min(recent_requests, key=recent_requests.get)
        del recent_requests[oldest_key]

Using HLL for Older Requests:

PYTHON
from hll import HyperLogLog

def update_hll(key, current_time):
    timestamp = int(current_time // window_size_seconds) * window_size_seconds
    if key in hll_counters and timestamp not in hll_counters[key]:
        hll_counters[key][timestamp] = HyperLogLog()
    hll_counter = hll_counters[key][timestamp]
    # Increment the counter based on request features (e.g., URL path)
    hll_counter.add(request_features)

Rate Limit Decision:

PYTHON
def is_allowed(key, current_time):
    if key not in recent_requests and key not in hll_counters:
        return False

    if key in recent_requests:
        # Exact count check for recent requests
        if time() - recent_requests[key] < window_size_seconds:
            return True

    # Approximate count check using HLL
    timestamp = int(current_time // window_size_seconds) * window_size_seconds
    hll_counter = hll_counters[key][timestamp]
    estimated_unique_requests = len(hll_counter)
    if estimated_unique_requests <= max_allowed_requests:
        return True

    return False
Step 6: Closing Summary and Common Mistakes
Summary: The solution balances accuracy with memory usage by using exact counts for recent requests and approximate counts via HLL for older ones. Regular cleanup ensures that the system remains efficient.

#### Common Mistakes
Over-reliance on Exact Counting: This can lead to excessive memory usage, especially for hot keys.
Neglecting Memory Management: Without proper cleanup or partitioning strategies, the system may run out of memory.
Ignoring Approximation Accuracy: While approximations save memory, they must be accurate enough for rate limiting decisions.
Step 7: Follow-ups
How do you handle very cold keys that rarely generate requests?
What other techniques could be used to further optimize memory usage?

