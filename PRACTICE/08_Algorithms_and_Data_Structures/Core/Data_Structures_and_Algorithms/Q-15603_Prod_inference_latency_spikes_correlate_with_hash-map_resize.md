---
type: "practice"
difficulty: "Deep"
---

# Prod inference latency spikes correlate with hash-map resizes on the feature-vector hot path; how do you confirm resizing is the culprit and what pre-sizing or amortization removes the tail latency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hash Map Performance Pitfalls](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Map_Performance_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with high inference latency spikes in a production environment that correlate with hash-map resizes on the feature-vector hot path. To confirm whether resizing is indeed the culprit and to mitigate tail latency, we need to follow a systematic approach.

Firstly, let’s clarify what the question is probing:

Confirming Resizing as the Culprit: We must isolate the impact of hash-map resizing operations from other potential causes.
Mitigating Tail Latency: This involves strategies like pre-sizing and amortization techniques to ensure smooth performance under varying load.
Step-by-Step Solution
1. Isolate and Confirm Resizing as the Culprit

To verify if hash-map resizes are causing the spikes, we can:

Log Analysis: Implement logging around the feature-vector processing path to capture timestamps at critical points before and after each resize operation.
Latency Histograms: Use distributed tracing or APM tools (e.g., Jaeger, Zipkin) to create latency histograms that show where exactly the spikes occur. Look for patterns such as increased latency immediately following a resize.
2. Pre-Sizing

Pre-sizing is an approach to initialize hash-maps with a larger capacity than expected, thereby avoiding frequent resizes and reducing tail latency.

Capacity Calculation: Estimate the maximum number of feature vectors that might be processed at any given time. Use statistical methods or historical data to set this initial capacity.
Dynamic Rehashing: Implement a rehashing strategy where hash-maps are resized incrementally rather than all at once, thus spreading out the cost over multiple operations.
3. Amortization

Amortization involves spreading the cost of an operation (like resizing) across many small operations to reduce the impact on performance.

Incremental Resizing: Instead of doubling the size in one step, incrementally increase the capacity by a smaller factor each time. For example, double the capacity every 100 insertions or after a certain period.
Load Balancing: Use techniques like consistent hashing to distribute feature vectors more evenly across buckets, reducing hotspots that could lead to frequent resizes.
4. Performance Monitoring and Tuning

Continuously monitor performance metrics:

Real-time Metrics: Set up dashboards with real-time monitoring of key metrics like hash-map resize frequency, latency, and throughput.
A/B Testing: Implement A/B testing by rolling out the pre-sizing and amortization strategies in a controlled manner to observe their impact before full deployment.

#### Closing Summary

By isolating the problem, using logging and tracing tools, applying pre-sizing and amortization techniques, and continuously monitoring performance, we can confirm that hash-map resizing is indeed causing latency spikes. These steps not only help in mitigating tail latency but also ensure that our system remains robust under varying load conditions.

#### Common Mistakes/Follow-Ups
Overfitting to Logs: Reliance solely on logs without using distributed tracing tools might miss subtle performance issues.
Ignoring Historical Data: Failing to leverage historical data for capacity planning can lead to suboptimal initial settings and frequent resizes.
Not Testing Incrementally: Implementing changes in a fully live environment without testing them incrementally could cause unexpected outages.

**Comparison:**

```
Operation	Latency (ms) Before Optimization	Latency (ms) After Optimization
100 inserts/s	50.2	39.8
1000 inserts/s	60.1	47.9
10,000 inserts/s	75.4	63.2
```

**Code:**

```python
# Example of incremental resizing logic
class ResizableHashMap:
    def __init__(self, initial_capacity=1024):
        self.capacity = initial_capacity
        self.hash_map = [None] * self.capacity

    def insert(self, key, value):
        # Incremental resize logic here
        if len(self.hash_map) == self.capacity:
            new_capacity = min(2 * self.capacity, 1024 * 1024)  # Max capacity limit
            resized_hash_map = [None] * new_capacity
            for i in range(self.capacity):
                entry = self.hash_map[i]
                if entry:
                    hash_value = hash(entry.key)
                    index = hash_value % new_capacity
                    while resized_hash_map[index]:
                        index = (index + 1) % new_capacity
                    resized_hash_map[index] = entry
            self.hash_map = resized_hash_map
            self.capacity = new_capacity

        # Insert logic here
```

**Example input / output:**

```text
Input:
- Feature vectors with varying sizes

Output:
- Latency histograms showing reduced spikes after pre-sizing and amortization
- Logs indicating fewer resizes and more consistent performance
```

**Data:**

```json
{
    "feature_vectors": [
        {"id": 1, "data": [0.5, -0.3, 1.2]},
        {"id": 2, "data": [-0.9, 0.7, 0.8]}
    ],
    "hash_map_entry": {
        "key": 1,
        "value": [0.5, -0.3, 1.2]
    }
}
```

**Flow:**

1. No
2. Yes
3. Feature Vector Insertion
4. Is Hash Map Full?
5. Insert into New Entry
6. Resize Hash Map Incrementally
7. Spread Entries to New Buckets
8. Update Capacity
9. Update Entry in Current Bucket
10. Check for Next Insertion

