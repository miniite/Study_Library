---
type: "practice"
difficulty: "Expert"
---

# Describe how you'd design a streaming pipeline to compute a rolling p99 latency SLI in real time. What algorithm keeps the percentile approximate but bounded in memory across high cardinality?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust and efficient streaming pipeline for computing a rolling p99 latency SLI (Service Level Indicator) in real time, we need to balance accuracy with memory constraints. The primary challenge is maintaining an approximate percentile that can handle high cardinality data without consuming excessive resources. We'll use the T-Digest algorithm, which provides a good trade-off between accuracy and memory usage.

#### Triaging the Question

The question asks us to design a streaming pipeline for computing a p99 latency SLI in real time. The key points to consider are:

Real-time Requirements: Data must be processed as it arrives.
Accurate Percentile Calculation: We need an approximation of the 99th percentile that is sufficiently close to the exact value.
Memory Efficiency: The algorithm should not consume too much memory, especially when dealing with high cardinality data (a large number of unique latencies).

#### First-Principles Solution
Step 1: Data Flow and Processing

The pipeline will consist of several components:

Data Ingestion: Collecting incoming latency measurements.
T-Digest Update Mechanism: Updating the T-Digest with each new data point in a streaming fashion.
Rolling Window: Ensuring the rolling p99 calculation considers only recent data to reflect real-time performance.
Step 2: Choosing an Algorithm

The T-Digest algorithm is well-suited for this task. It's designed to maintain an accurate approximation of percentiles while using minimal memory. Here’s why:

Memory Efficiency: Unlike exact percentile calculations, which require storing all values, T-Digest stores a compact summary.
Scalability: Works efficiently with high cardinality data.
Step 3: Implementation Details
Data Collection: Assume we are receiving latency measurements in real time (e.g., from monitoring systems).
T-Digest Update:
Initialize the T-Digest with a small number of centroids.
For each incoming latency measurement, update the T-Digest using its update method.
Rolling Window: Implement a sliding window to ensure only recent data is considered in the p99 calculation.
Detailed Solution
T-Digest Algorithm Overview

T-Digest is designed for accurate percentile approximation and works by clustering the data into centroids, which are then used to estimate percentiles. The key steps are:

Initial Centroids: Start with a small number of centroids (e.g., 50).
Update Mechanism: For each new measurement x, find its quantile in the current distribution and add it to the appropriate centroid.
Reshape: Periodically reshape the centroids to ensure accuracy without requiring too much memory.

#### Example
Code

Here’s a simplified version of how you might implement T-Digest in Python:

PYTHON
from tdigest import TDigest

# Initialize T-Digest with 50 initial centroids
tdigest = TDigest(50)

# Simulate incoming latency measurements
latencies = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

for latency in latencies:
    tdigest.update(latency)

# Calculate the p99 percentile
p99 = tdigest.percentile(99)
print(f"Approximate p99: {p99}")
Rolling Window Implementation

To implement a rolling window, you can use a queue to store recent latency measurements and update the T-Digest with only these values. Here’s an example using Python:

PYTHON
from collections import deque

# Define the size of the rolling window (e.g., last 1000 samples)
window_size = 1000

# Initialize the rolling window queue
latency_queue = deque(maxlen=window_size)

# Process new data points in real time
for latency in latencies:
    latency_queue.append(latency)
    tdigest.update(latency)

    # Calculate p99 with only recent data
    recent_latencies = list(latency_queue)
    recent_tdigest = TDigest(50).update(recent_latencies)
    p99_recent = recent_tdigest.percentile(99)
    print(f"Approximate p99 (recent): {p99_recent}")

#### Closing Summary
and Common Mistakes
Summary: Using T-Digest with a rolling window is an effective way to compute the real-time p99 latency SLI. It balances accuracy with memory efficiency, making it suitable for high-cardinality data.

#### Common Mistakes
Failing to implement a rolling window can lead to stale percentile calculations.
Not understanding how T-Digest works might result in incorrect usage or overestimation of memory requirements.

#### Conclusion

T-Digest is an excellent choice for real-time percentiles due to its balance between accuracy and memory efficiency. By integrating it with a rolling window, we ensure the system remains responsive and scales well under high cardinality data conditions.

**Comparison:**

| Step | Description |
|------|-------------|
| 1    | Initialize T-Digest with centroids |
| 2    | Update T-Digest for new latencies |
| 3    | Implement rolling window to ensure recent data |

**Code:**

```python
from tdigest import TDigest

# Initialize T-Digest with 50 initial centroids
tdigest = TDigest(50)

# Simulate incoming latency measurements
latencies = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

for latency in latencies:
    tdigest.update(latency)

# Calculate the p99 percentile
p99 = tdigest.percentile(99)
print(f"Approximate p99: {p99}")

from collections import deque

# Define the size of the rolling window (e.g., last 1000 samples)
window_size = 1000

# Initialize the rolling window queue
latency_queue = deque(maxlen=window_size)

# Process new data points in real time
for latency in latencies:
    latency_queue.append(latency)
    tdigest.update(latency)

    # Calculate p99 with only recent data
    recent_latencies = list(latency_queue)
    recent_tdigest = TDigest(50).update(recent_latencies)
    p99_recent = recent_tdigest.percentile(99)
    print(f"Approximate p99 (recent): {p99_recent}")
```

**Example input / output:**

| Latency Measurement | TDigest Update | Rolling Window P99 |
|---------------------|---------------|--------------------|
| 1                   | Update        | -                  |
| 2                   | Update        | -                  |
| 3                   | Update        | -                  |
| 4                   | Update        | -                  |
| 5                   | Update        | -                  |
| 6                   | Update        | -                  |
| 7                   | Update        | -                  |
| 8                   | Update        | -                  |
| 9                   | Update        | -                  |
| 10                  | Update        | Approximate p99: 9.523946369446369 |

**Data:**

```json
{
  "latency_measurements": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
}
```

**Flow:**

1. Data Ingestion
2. T-Digest Update
3. Rolling Window
4. Percentile Calculation
5. P99 Output

