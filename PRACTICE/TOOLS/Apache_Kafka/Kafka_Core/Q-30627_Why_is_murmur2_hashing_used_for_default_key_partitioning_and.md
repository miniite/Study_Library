---
type: "practice"
difficulty: "Deep"
---

# Why is murmur2 hashing used for default key partitioning, and what breaks if a client uses a different hash?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Murmur2 hashing is widely used as the default key partitioning algorithm in Apache Kafka due to its high speed, good distribution properties, and stability. However, using a different hash function can lead to suboptimal partition assignment and potential hotspots.

#### Triage

The core of this question revolves around understanding why Murmur2 hashing is preferred over other hashing algorithms for key partitioning in Kafka and the implications of using a different hash function. The interviewer is probing your knowledge of Kafka's internal mechanisms, hashing techniques, and performance considerations.

#### Assumptions

You are familiar with Kafka's architecture.
You understand the importance of efficient and fair data distribution across partitions.

#### First Principles Solution
Why Murmur2 Hashing?

Speed: Murmur2 is one of the fastest hash functions available today. It can process keys very quickly, which is crucial for high-throughput systems like Kafka where processing speed can significantly impact performance.

Distribution Properties: Murmur2 aims to distribute key values evenly across the range of possible hash values. This property ensures that data is spread uniformly across partitions, reducing the likelihood of hotspots and load imbalances.

Stability: The algorithm is designed to be consistent across different implementations and environments, ensuring predictable behavior.

What Breaks If a Client Uses a Different Hash?

Using a different hash function can lead to several issues:

Non-Uniform Distribution: A poorly designed or less robust hash function might not distribute keys evenly. This could result in some partitions receiving significantly more data than others, leading to imbalanced load distribution.

Hotspots and Performance Degradation: If certain partitions receive an abnormally high amount of data due to a suboptimal hash function, it can lead to hotspots where these partitions become bottlenecks. This can degrade overall performance and affect the scalability of your Kafka cluster.

Incompatibility with Kafka's Internal Mechanisms: Kafka relies on the stability and predictability of its default hashing mechanism. Using a different hash function might break compatibility with internal optimizations and assumptions, leading to unexpected behavior or errors.

Increased Complexity: Switching from the default Murmur2 hash to another algorithm can introduce additional complexity in maintaining consistency across partitions. This may require custom logic to ensure data is evenly distributed, which can be error-prone and resource-intensive.

#### Closing Summary

In summary, Murmur2 hashing is chosen for its speed, distribution properties, and stability, making it an optimal choice for key partitioning in Kafka. Using a different hash function can lead to suboptimal performance due to non-uniform data distribution, potential hotspots, and compatibility issues with Kafka's internal mechanisms.

#### Common Mistakes/Follow-Ups
Ignoring Distribution Properties: Failing to consider the importance of even distribution across partitions.
Not Understanding Load Imbalance: Overlooking the impact of imbalanced load on overall system performance.
Falsely Assuming Any Hash Function Works: Believing that any hash function can be used without considering its implications on Kafka's architecture.

**Comparison:**

| Partition ID | Keys           |
|--------------|----------------|
| 0            |                |
| 1            | key1, key2     |
| 2            | key3          |

**Code:**

```python
# Example of generating a Murmur2 hash in Python using khashlib
from khashlib import murmur_hash

key = "example_key"
partition_id = murmur_hash(key, seed=0) % num_partitions
```

**Example input / output:**

```text
| Key | Murmur2 Hash (seed=0) % 10 |
|-----|----------------------------|
| key1 | 4                          |
| key2 | 5                          |
| key3 | 6                          |

### Example of Inconsistent Distribution

| Key   | Murmur2 Hash (seed=0) % 10 | New Hash Function (seed=1) % 10 |
|-------|----------------------------|--------------------------------|
| key1  | 4                          | 7                              |
| key2  | 5                          | 8                              |
| key3  | 6                          | 9                              |

In the example above, using a different hash function results in a more uneven distribution of keys across partitions.
```

**Data:**

```json
{
  "num_partitions": 10,
  "keys": ["key1", "key2", "key3"],
  "expected_partitions": [4, 5, 6]
}
```

**Flow:**

1. Yes
2. No
3. Client Sends Key
4. Is Hash Function Consistent?
5. Murmur2 Hashed Key
6. Kafka Assigns Partition
7. Inconsistent Data Distribution
8. Hotspots and Load Imbalance
9. Performance Degradation

