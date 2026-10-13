---
type: "practice"
difficulty: "Deep"
---

# How would you compute quantiles for feature binning on a very large column without sorting all of it, and what approximate structure helps?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently compute quantiles for feature binning in a very large dataset without sorting the entire column, we can leverage an efficient data structure called T-Digest. This approach provides an accurate approximation of quantiles with minimal memory usage and computational complexity.

#### Triaging the Question

The question is asking about how to approximate quantile computation on a massive dataset efficiently. The interviewer is probing your understanding of:

Efficient Algorithms: How to avoid sorting the entire column, which would be computationally expensive.
Data Structures: What data structures can help in approximating quantiles without full sorting.
Approximation Techniques: Understanding the trade-offs between accuracy and resource usage.
Key Assumptions
The dataset is too large to fit into memory all at once.
The goal is to compute a specific set of quantiles (e.g., 0.25, 0.5, 0.75).
We need an approximate solution that balances accuracy and computational efficiency.

#### First Principles Solution
Step-by-Step Explanation

Understanding Quantiles: A quantile is the value below which a given percentage of observations in a group of observations fall. For example, the median (50th percentile) splits the data into two halves.

T-Digest Algorithm:

Purpose: T-Digest is designed to provide an accurate approximation of quantiles without requiring full sorting or storing all elements.
Data Structure: It uses a collection of "blobs" or "clusters" that represent the distribution of values in the dataset.
Insertion: Each value in the dataset is inserted into the digest incrementally, and it adjusts the clusters to maintain their size and position.

Algorithm Workflow:

Initialize an empty T-Digest.
Insert each element from the large column one by one.
For each insertion, adjust the existing clusters or create new ones if necessary.
After processing all elements, query the digest for any desired quantile using a simple lookup.
Detailed Implementation

The following is a simplified pseudocode to illustrate how T-Digest can be used:

PYTHON
from tdigest import TDigest

def compute_quantiles(column):
    # Initialize an empty T-Digest with a target number of clusters (e.g., 1024)
    digest = TDigest(1024)

    # Iterate through the column, one element at a time
    for value in column:
        # Insert the value into the T-Digest
        digest.update(value)

    # Query the T-Digest to get desired quantiles
    quantiles = [digest.value_at_percentile(p) for p in [25, 50, 75]]

    return quantiles

# Example usage:
column = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
quantiles = compute_quantiles(column)
print(quantiles)  # Output: [2.75, 5.5, 8.25]

#### Closing Summary

In summary, T-Digest is a powerful tool for approximating quantiles in large datasets without sorting them entirely. It strikes a balance between accuracy and efficiency by maintaining a compact representation of the data distribution.

#### Common Mistakes
and Follow-Ups
Mistake: Using simple histogram-based approaches or even full sort for such tasks.

#### Follow-Up
Discussing how T-Digest handles edge cases like ties, negative values, and extremely large datasets.

By leveraging T-Digest, you can efficiently compute quantiles in a way that is both accurate and scalable.

**Code:**

```python
from tdigest import TDigest

def compute_quantiles(column):
    digest = TDigest(1024)
    for value in column:
        digest.update(value)
    quantiles = [digest.value_at_percentile(p) for p in [25, 50, 75]]
    return quantiles
```

**Example input / output:**

```text
column = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
quantiles = compute_quantiles(column)
print(quantiles)  # Output: [2.75, 5.5, 8.25]
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. Initialize T-Digest
6. Is there a value to insert?
7. Insert value into digest
8. Check if new quantiles areneeded
9. Next value or done?
10. Return computed quantiles

