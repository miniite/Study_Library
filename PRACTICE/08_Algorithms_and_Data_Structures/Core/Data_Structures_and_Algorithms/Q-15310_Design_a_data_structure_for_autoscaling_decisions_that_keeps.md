---
type: "practice"
difficulty: "Expert"
---

# Design a data structure for autoscaling decisions that keeps a sliding p95 of request latency cheaply; why is exact percentile over a window expensive and what approximation would you accept?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To design a data structure for autoscaling decisions that efficiently maintains a sliding p95 of request latency, we need to focus on balancing accuracy with computational efficiency. The exact percentile over a window is inherently expensive due to its high computational complexity and memory requirements. Instead, we can use an approximate approach by leveraging quantile summaries like the Greenwald-Khanna algorithm or maintaining a small sample set that approximates the true p95.

#### Triaging the Question

The core of this question revolves around designing a data structure for autoscaling decisions in a cost-effective manner while ensuring that it maintains accurate enough information about request latency to support scaling policies. The key challenge is to handle large volumes of incoming requests and compute the 95th percentile (p95) efficiently.

#### First-Principles Solution
Why Exact Percentile Over a Window Is Expensive:

High Computational Complexity: Calculating the exact p95 for a large dataset requires sorting all data points, which is (O(n \log n)). For real-time applications with high request rates, this can become prohibitive.

Memory Requirements: Storing and managing every single request's latency would require substantial memory usage, especially as the number of requests grows over time.

Latency Sensitivity: In scenarios where immediate decisions are critical, the delay introduced by sorting operations or maintaining a large dataset could be unacceptable.

Approximation Approaches:

Greenwald-Khanna Algorithm:

This algorithm is designed to compute quantiles in a single pass and with limited memory usage.
It works by maintaining a summary of data points that approximates the true distribution, allowing for efficient updates and queries.

Small Sample Set:

Maintain a fixed-size sample set of recent latency values.
Periodically update this set to include new samples while discarding old ones.
This approach balances accuracy with memory constraints but may introduce some level of noise due to the limited sample size.
Detailed Solution:

Greenwald-Khanna Algorithm:

The algorithm maintains a summary of data points using two sets: one for "heavy hitters" and another for "light weights".
Heavy hitters are elements with frequencies above a certain threshold, while light weights represent the remaining elements.
By dynamically adjusting these sets based on incoming data, the algorithm can approximate the p95 efficiently.

#### Implementation

PYTHON
class SlidingWindowP95:
    def __init__(self, window_size):
        self.window = []
        self.quantiles = [0.95]

    def update(self, latency):
        if len(self.window) < self.window_size:
            self.window.append(latency)
        else:
            p95_index = int(len(self.window) * 0.95)
            for i in range(p95_index + 1):
                self.window[i] = min(self.window[i], latency)

    def get_p95(self):
        return max(self.window)

# Example Usage
swp95 = SlidingWindowP95(window_size=1000)
for latency in [10, 20, 30, 40, 50]:
    swp95.update(latency)
print(swp95.get_p95())

#### Closing Summary

By using the Greenwald-Khanna algorithm or maintaining a small sample set, we can efficiently approximate the p95 of request latency. These methods offer a balance between accuracy and computational efficiency, making them suitable for real-time autoscaling decisions in high-traffic applications.

#### Common Mistakes/Follow-Ups
Ignoring Memory Constraints: Failing to consider the memory requirements of storing all data points can lead to performance degradation.
Over-relying on Approximations: Relying too heavily on approximations without validating their accuracy against actual data could result in suboptimal scaling decisions.
Not Considering Edge Cases: Not handling edge cases such as very small or very large windows, or bursts of high latency requests, can lead to inaccurate results.

**Code:**

```python
class SlidingWindowP95:
    def __init__(self, window_size):
        self.window = []
        self.quantiles = [0.95]

    def update(self, latency):
        if len(self.window) < self.window_size:
            self.window.append(latency)
        else:
            p95_index = int(len(self.window) * 0.95)
            for i in range(p95_index + 1):
                self.window[i] = min(self.window[i], latency)

    def get_p95(self):
        return max(self.window)

# Example Usage
swp95 = SlidingWindowP95(window_size=1000)
for latency in [10, 20, 30, 40, 50]:
    swp95.update(latency)
print(swp95.get_p95())
```

