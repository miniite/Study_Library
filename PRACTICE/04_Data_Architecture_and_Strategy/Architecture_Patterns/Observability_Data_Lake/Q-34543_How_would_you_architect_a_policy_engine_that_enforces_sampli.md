---
type: "practice"
difficulty: "Deep"
---

# How would you architect a policy engine that enforces sampling and cardinality budgets at the collector edge so cost control happens before data ever hits the lake?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to design a robust policy engine that ensures cost-effective data collection by enforcing sampling and cardinality budgets right at the source—the collector edge. This approach helps in managing resource usage efficiently without overwhelming the data lake with unnecessary or excessive data.

#### Triage

The key here is to understand that the policy engine must operate at the edge, meaning it needs to make decisions based on local metrics before any data is sent to the centralized data lake. The goal is to minimize the volume of data transferred while ensuring that critical information is not lost. We'll consider the types of sampling techniques and cardinality controls necessary to achieve this.

#### First-Principles Solution
Define Sampling Strategies: Implement various sampling methods such as uniform, reservoir, or adaptive sampling based on data characteristics.
Cardinality Management: Use techniques like count-min sketch or hyperLogLog for efficient cardinality estimation.
Edge Enforcement: Ensure the policy engine enforces these strategies directly at the source to control data volume before it reaches the lake.
Detailed Implementation

Sampling Strategies:

Uniform Sampling: Randomly select a subset of records with a predefined ratio.
Reservoir Sampling: Maintain a "reservoir" of samples, replacing older samples with new ones based on a specific probability.
Adaptive Sampling: Adjust sampling rates dynamically based on anomaly detection or historical data patterns.

Cardinality Management:

Count-Min Sketch: A probabilistic data structure that provides an estimate of the number of distinct elements in a multiset, useful for cardinality estimation without storing all data.
HyperLogLog: An even more efficient algorithm for estimating cardinalities using fewer bits.

Edge Enforcement:

Implement these strategies in a lightweight, high-performance manner to ensure minimal overhead at the collector edge.
Use streaming algorithms that can operate in real-time with low latency requirements.
Code Example

Here’s an example of how reservoir sampling might be implemented in Python:

PYTHON
import random

def reservoir_sampling(data, sample_size):
    samples = []
    for i, item in enumerate(data):
        if len(samples) < sample_size:
            samples.append(item)
        else:
            r = random.randint(0, i)
            if r < sample_size:
                samples[r] = item
    return samples

# Example usage
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sample_size = 3
samples = reservoir_sampling(data, sample_size)
print(samples)
Data Lake Integration

The sampled data and cardinality estimates would then be sent to the central data lake. The data lake can use these estimates to optimize storage and processing resources.

Mermaid Diagram
Collector Edge
Sampling & CardinalityEstimation
Data Lake

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that the data lake is not burdened with excessive volumes of data, leading to cost savings and efficient resource utilization. Key mistakes to avoid include overcomplicating the sampling logic or failing to consider the dynamic nature of anomaly detection in adaptive sampling.

#### Follow-ups
might involve discussing how to handle edge cases like high-cardinality events, optimizing for real-time vs batch processing scenarios, or integrating machine learning models to refine sampling strategies dynamically.

**Comparison:**

```
Sampled Data	Original Data
2	1
4	2
9	3
```

**Code:**

```python
import random

def reservoir_sampling(data, sample_size):
    samples = []
    for i, item in enumerate(data):
        if len(samples) < sample_size:
            samples.append(item)
        else:
            r = random.randint(0, i)
            if r < sample_size:
                samples[r] = item
    return samples

# Example usage
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sample_size = 3
samples = reservoir_sampling(data, sample_size)
print(samples)
```

**Example input / output:**

```

Input:

PYTHON
Copy
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sample_size = 3

Output:

PYTHON
Copy
[2, 4, 9]
```

**Data:**

```json
{
    "data": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    "sample_size": 3,
    "samples": []
}
```

**Flow:**

1. Collector Edge
2. Sample & Estimate Cardinality
3. Send to Data Lake
4. Log Errors/Warnings

