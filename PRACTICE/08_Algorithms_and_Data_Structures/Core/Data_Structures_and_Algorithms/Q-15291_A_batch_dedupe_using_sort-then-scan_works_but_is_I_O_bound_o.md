---
type: "practice"
difficulty: "Deep"
---

# A batch dedupe using sort-then-scan works but is I/O bound on huge inputs; when would you switch to hash-based partitioning, and how do you handle partitions that don't fit in memory?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address batch deduplication efficiently, especially with large datasets, we often start by considering a sort-then-scan approach. However, as data sizes increase, this method can become I/O bound due to frequent disk accesses and limited memory availability. In such scenarios, switching to a hash-based partitioning strategy is a more scalable solution.

The core idea here is to leverage hashing to divide the dataset into smaller chunks that fit in memory, process each chunk independently, and then merge results. This method reduces the number of I/O operations while ensuring that we do not exceed available memory constraints. Let's break this down from first principles:

#### Triage
Understanding the Problem
Problem Statement: We need to deduplicate a large dataset efficiently.
Current Approach (Sort-then-Scan): It works but is I/O bound on huge inputs.
Goal: Find an alternative that scales better with data size and memory constraints.

#### First Principles Solution
Step 1: Hashing for Partitioning

We can use a hash function to partition the dataset into smaller, manageable chunks. Each chunk will be small enough to fit in memory but large enough to provide effective parallel processing benefits.

Step 2: In-Memory Deduplication within Partitions

For each chunk, we perform an in-memory deduplication operation. This is feasible because each chunk fits in memory. We can use a data structure like a set or a hash map for this purpose.

Step 3: Merge Results Efficiently

After processing all chunks, we need to merge the results back into a single stream. This merging step should be optimized to minimize additional I/O operations and ensure that duplicate removal is consistent across partitions.

Handling Partitions That Don't Fit in Memory

If a partition does not fit entirely in memory, we can use a hybrid approach:

Streaming Deduplication: Use an external merge sort or a similar streaming algorithm to process the data as it streams into memory chunks.
Hashing and Filtering: For each chunk that doesn't fully fit in memory, apply a hash function to filter out duplicates before processing.

#### Example
Workflow

Let's consider a concrete example where we have a dataset of 10 million unique elements, and our system can handle up to 1 million elements in memory at once:

PLAINTEXT
Original Dataset: [element1, element2, ..., elementN]
Partitioning:
Hash Function: hash(element)
Define partitions based on hash values (e.g., using a modulo operation).
PYTHON
partitions = defaultdict(list)
for element in dataset:
    partition_id = hash(element) % 10  # Example with 10 partitions
    partitions[partition_id].append(element)
In-Memory Deduplication:

For each partition, use a set to deduplicate elements.

PYTHON
deduplicated_elements = []
for partition_id in range(10):
    seen_elements = set()
    for element in partitions[partition_id]:
        if element not in seen_elements:
            seen_elements.add(element)
            deduplicated_elements.append(element)
Merging Results:

Merge the results back into a single stream while ensuring no duplicates are introduced.

PYTHON
merged_deduplicated_elements = []
last_seen_element = None
for element in sorted(deduplicated_elements):
    if element != last_seen_element:
        merged_deduplicated_elements.append(element)
        last_seen_element = element

#### Closing Summary
and Common Mistakes/Follow-Ups

Switching to hash-based partitioning is a strategic decision that balances memory usage with I/O efficiency. Key takeaways include:

Partition Size: Ensure partitions are small enough to fit in memory but large enough to provide parallel processing benefits.
Hash Function Choice: Opt for a fast, low-collision hashing algorithm suitable for your dataset.
Merging Strategy: Efficiently merge results without introducing duplicates.

#### Common mistakes
include:

Over-partitioning: Making partitions too small can increase overhead from frequent I/O operations.
Under-partitioning: Making partitions too large can lead to memory overflow issues.

#### Follow-ups
could explore more advanced techniques like distributed hash-based deduplication or integrating this approach with other algorithms for better performance.

**Comparison:**

```
Partition	Elements
0	[0, 1, 2]
1	[3, 4, 5]
2	[6, 7, 8]
...	...
9	[9999997, 9999998, 9999999]
```

**Code:**

```python
from collections import defaultdict

# Example dataset and hashing function
dataset = [i for i in range(10_000_000)]  # Simulated large dataset
hash_function = lambda x: hash(x) % 10  # Simple modulo operation for partitioning

partitions = defaultdict(list)
for element in dataset:
    partitions[hash_function(element)].append(element)

# In-memory deduplication within each partition
deduplicated_elements = []
seen_elements = set()
for partition_id in range(10):
    seen_partition_elements = set()
    for element in partitions[partition_id]:
        if element not in seen_partition_elements:
            seen_partition_elements.add(element)
            deduplicated_elements.append((partition_id, element))

# Merging results
merged_deduplicated_elements = []
last_seen_element = None
for _, element in sorted(deduplicated_elements):
    if element != last_seen_element:
        merged_deduplicated_elements.append(element)
        last_seen_element = element

print(merged_deduplicated_elements[:10])  # Show first 10 deduplicated elements
```

**Example input / output:**

```

Input:

dataset: A list of integers from 0 to 9,999,999.

Output:

PYTHON
Copy
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

This example demonstrates the deduplication process.
```

**Data:**

```json
{
  "dataset": [
    {"id": i} for i in range(10_000_000)
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Dataset Large?
5. Partition Data
6. Deduplicate Partitions
7. Merge Results
8. End

