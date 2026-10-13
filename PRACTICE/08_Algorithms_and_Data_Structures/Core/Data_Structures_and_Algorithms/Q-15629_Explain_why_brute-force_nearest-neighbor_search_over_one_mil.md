---
type: "practice"
difficulty: "Intermediate"
---

# Explain why brute-force nearest-neighbor search over one million embeddings is O(n*d) per query and why that motivates approximate vector indexes.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Similarity Search Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a large dataset of one million embeddings, performing a brute-force nearest-neighbor search can quickly become computationally expensive. The time complexity of such an approach is O(n*d) per query, where ( n ) is the number of embeddings and ( d ) is the dimensionality of each embedding vector. Let's break down why this is the case and how it motivates the use of approximate vector indexes.

TL;DR

A brute-force nearest-neighbor search involves calculating the distance between a query point and every other point in the dataset, leading to a time complexity of O(n*d) per query. This high cost encourages the use of approximate index structures that can provide faster but slightly less accurate results.

#### Triage
Understanding the Question and What is Being Probed

The question asks us to explain the time complexity of brute-force nearest-neighbor search for one million embeddings, which involves understanding the underlying algorithmic principles and performance implications. The interviewer likely wants to assess your grasp of data structures, algorithms, and their practical applications in real-world scenarios.

Step-by-Step Solution from First Principles

Understanding Brute-Force Search:

In a brute-force nearest-neighbor search, we need to compare the query point with every other point in the dataset.
For each comparison, we calculate the distance between the query point and the current data point.

Time Complexity Breakdown:

Suppose we have ( n ) embeddings, each of dimensionality ( d ).
For each embedding, we need to compute its distance from the query point.
The computation of a single distance involves iterating over all dimensions (or features) in the vector.

Computation Cost:

To find the nearest neighbor for one query point, you would have to compute the distances for ( n-1 ) other points.
For each comparison, you need to iterate through all ( d ) dimensions.
Therefore, the total number of distance computations is ( (n-1) \times d ).

Simplifying the Analysis:

In big-O notation, we typically drop constants and focus on the dominant term.
Thus, the time complexity simplifies to ( O(n*d) ).

Impact for Large Datasets:

With one million embeddings (( n = 10^6 )) and each vector having a dimensionality of ( d ), the number of computations can become very large.
For example, if ( d = 256 ) (a common embedding size in NLP tasks), we would need to perform around ( 10^{8} ) distance calculations.

#### Closing Summary
and Common Mistakes

Brute-force nearest-neighbor search is straightforward but computationally intensive for large datasets. The time complexity of ( O(n*d) ) makes it impractical when dealing with millions of vectors, especially in high-dimensional spaces. This inefficiency motivates the use of approximate vector indexes like ANNOY, Faiss, or HNSW, which can provide faster search times at the cost of slightly less accurate results.

#### Common Mistakes
Mistaking Complexity: Sometimes candidates might confuse the complexity with ( O(n^2) ), which would be incorrect for a single query in brute-force search.
Ignoring Dimensionality: Candidates may overlook the impact of vector dimensionality on performance, focusing solely on the number of points.

**Comparison:**

```
Index	Value
Query Point	[0.1, 0.2, ..., 0.9]
Embeddings Count	1,000,000
Dimensionality (d)	256
```

**Code:**

```python
# Example Python code to illustrate distance calculation (simplified)
import numpy as np

def euclidean_distance(v1, v2):
    return np.sqrt(np.sum((v1 - v2) ** 2))

query_point = np.random.rand(256)
embeddings = np.random.rand(1000000, 256)

# Brute-force nearest-neighbor search
nearest_neighbor = None
min_distance = float('inf')
for embedding in embeddings:
    distance = euclidean_distance(query_point, embedding)
    if distance < min_distance:
        min_distance = distance
        nearest_neighbor = embedding

print(f"Nearest neighbor found with distance: {min_distance}")
```

**Example input / output:**

```

Given the query point and a subset of embeddings:

Query Point: [0.1, 0.2, ..., 0.9]
Embeddings: A dataset of one million vectors

Example output for nearest neighbor search (simplified):

PYTHON
Copy
Nearest neighbor found with distance: 0.5436
```

**Data:**

```json
{
  "query_point": [0.1, 0.2, 0.3, ..., 0.9],
  "embeddings": [
    [0.5, 0.6, 0.7, ... , 1.0],
    [0.4, 0.5, 0.6, ... , 0.9],
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the dataset large?
5. Calculate distance for eachembedding
6. Find minimum distance andnearest neighbor
7. Use index to speed up search
8. End
9. High computation cost

