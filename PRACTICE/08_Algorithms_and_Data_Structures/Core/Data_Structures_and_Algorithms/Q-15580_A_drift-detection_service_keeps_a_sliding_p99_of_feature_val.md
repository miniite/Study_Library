---
type: "practice"
difficulty: "Deep"
---

# A drift-detection service keeps a sliding p99 of feature values cheaply; why is exact percentile over a window expensive and what sliding-window sketch or bucketed histogram bounds error and memory?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the problem of keeping a sliding p99 of feature values efficiently, we need to understand why maintaining an exact percentile over a window is expensive and explore more practical sketching techniques that balance accuracy and resource utilization.

#### Triaging the Question

The core issue here is understanding the trade-offs between precision and computational cost. Maintaining an exact p99 (99th percentile) over a sliding window of data involves processing every new value as it comes in, which can be computationally expensive for large datasets. Instead, we aim to use more efficient sketching techniques that provide approximate percentiles with bounded errors.

#### First Principles Solution
Why Exact Percentile Calculation is Expensive
Cost of Processing Each Value: Maintaining an exact percentile requires sorting or maintaining a sorted list of the top 1% values as new data arrives. Sorting operations are O(n log n) in time complexity, which can be prohibitive for large windows.
Memory Usage: Storing all historical feature values to maintain exact percentiles can consume significant memory, especially with high-frequency data.
Sliding Window Sketch Techniques

To address these challenges, we use sketching techniques that allow us to approximate percentiles efficiently. Two popular methods are the t-digest and HyperLogLog, but for our purposes, a bucketed histogram is particularly useful due to its simplicity and ease of implementation.

Bucketed Histograms

A bucketed histogram divides the feature value range into discrete buckets (bins) and counts how many values fall into each bin. By carefully designing the number and size of these bins, we can bound both error and memory usage effectively.

Detailed Solution
Define Bins: Divide the range of possible feature values into a fixed or variable number of bins. The choice of bin boundaries is crucial.
Count Frequencies: As new data arrives, increment the count for each bin that the value falls into.
Estimate Percentile: To estimate the p99, find the smallest bin whose cumulative frequency exceeds 99% of the total counts.
Error Bounds

The error in this method depends on how finely you partition the bins and the distribution of your data:

Bin Size vs. Accuracy: Smaller bins increase accuracy but require more memory and computation.
Empirical Error Analysis: Theoretical analysis can provide bounds, but empirical testing is also important to ensure desired performance.
Memory Efficiency

Bucketed histograms use much less memory than storing every value. For example, if you have 1000 buckets and each bucket stores a count, the total memory usage is proportional to the number of unique values in your data stream.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Overestimating bin size can lead to high accuracy but excessive memory use; underestimating it can result in poor approximation.

#### Follow-Ups
Discussing how this solution scales with varying data sizes and arrival rates, considering different update strategies (e.g., incremental vs. batch), and exploring advanced techniques like t-digest for more precise approximations.

By using a bucketed histogram, we achieve a balance between accuracy and resource efficiency, making it suitable for large-scale drift detection services.

**Comparison:**

| Bin Index | Count |
|-----------|-------|
| 0         | 1     |
| 1         | 2     |
| 2         | 3     |
| ...       | ...   |

**Code:**

```python
# Example implementation of a simple bucketed histogram in Python

class BucketedHistogram:
    def __init__(self, num_buckets):
        self.buckets = [0] * num_buckets  # Initialize buckets with zero counts

    def update(self, value):
        bin_index = hash(value) % len(self.buckets)  # Simple hashing to determine bucket index
        self.buckets[bin_index] += 1  # Increment count for the assigned bucket

    def get_p99_estimate(self):
        total_count = sum(self.buckets)
        cumulative_freq = 0
        p99_bucket = -1

        for i, count in enumerate(self.buckets):
            cumulative_freq += count
            if cumulative_freq / total_count >= 0.99:
                p99_bucket = i
                break

        return p99_bucket

# Example usage
histogram = BucketedHistogram(100)
for value in [23, 45, 67, 89, ...]:  # Simulate data stream
    histogram.update(value)
p99_estimate = histogram.get_p99_estimate()
print(f"Estimated p99: {p99_estimate}")
```

**Example input / output:**

```text
Input: Data stream [23, 45, 67, 89, ...]
Output: Estimated p99 value and its index in the bucketed histogram
Request: Update the histogram with a new value
Response: Updated counts for each bin
```

**Data:**

```json
{
  "buckets": [0, 0, 1, 2, 3, 4, ...],  # Example bucket counts
  "total_count": 1000  // Total number of data points processed
}
```

**Flow:**

1. Receive New Data
2. Update Bucket Count
3. Determine P99 Bin
4. Estimate P99 Value
5. Output Estimated Percentile

