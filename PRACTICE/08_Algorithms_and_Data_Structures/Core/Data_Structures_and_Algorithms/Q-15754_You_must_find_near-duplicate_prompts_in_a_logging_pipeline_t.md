---
type: "practice"
difficulty: "Deep"
---

# You must find near-duplicate prompts in a logging pipeline to cluster them for eval; why is all-pairs cosine O(n^2) and how does an ANN index or LSH bring clustering down to a tractable cost?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Similarity Search Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of finding near-duplicate prompts in a logging pipeline, we need to compare each prompt against every other one to ensure comprehensive coverage. However, this brute-force approach—often referred to as all-pairs cosine similarity—has an O(n^2) time complexity, which quickly becomes intractable for large datasets. In contrast, approximate nearest neighbor (ANN) indexing and locality-sensitive hashing (LSH) significantly reduce the computational cost by allowing efficient clustering of near-duplicate prompts.

Let's break down why all-pairs cosine is O(n^2), how ANN and LSH work, and see a high-level comparison in detail.

TL;DR
All-pairs Cosine: O(n^2) due to pairwise comparisons.
ANN Indexing & LSH: Reduce the complexity by leveraging spatial indexing and hash functions for approximate similarity searches.

#### Triaging the Question

The question is probing your understanding of:

The computational complexity of comparing pairs in a dataset (all-pairs cosine).
Techniques for efficiently finding approximate nearest neighbors.
How these techniques can scale to large datasets.

We need to assume that we have a set of prompts, each represented as high-dimensional vectors (e.g., TF-IDF features), and the goal is to find similar prompts within this set.

#### First-Principles Solution
1. All-Pairs Cosine Similarity

Cosine similarity measures the cosine of the angle between two vectors. To find all pairs, we must compute the dot product for every pair of vectors:

PYTHON
def all_pairs_cosine_similarity(vecs):
    n = len(vecs)
    similarities = [[0 for _ in range(n)] for _ in range(n)]

    for i in range(n):
        for j in range(i+1, n):
            v1 = vecs[i]
            v2 = vecs[j]
            dot_product = np.dot(v1, v2)
            norm_v1 = np.linalg.norm(v1)
            norm_v2 = np.linalg.norm(v2)
            similarity = dot_product / (norm_v1 * norm_v2)
            similarities[i][j] = similarities[j][i] = similarity

    return similarities

This approach is O(n^2) because we have nested loops, each running n times.

2. Approximate Nearest Neighbor (ANN) Indexing

ANN indexing uses data structures like BK-trees, Annoy, or HNSW to approximate nearest neighbors with a predefined error margin ε. These indexes allow us to quickly find candidates for similarity without having to compare every vector:

PYTHON
# Using Annoy as an example
import annoy

def create_annoy_index(vecs):
    index = annoy.AnnoyIndex(len(vecs[0]), 'angular')
    for i, vec in enumerate(vecs):
        index.add_item(i, vec)
    index.build(10)  # Build the index with 10 trees
    return index

def find_nearest_neighbors(index, query_vector, k=5):
    neighbors = index.get_nns_by_vector(query_vector, k, include_distances=True)
    return neighbors

ANN indexing significantly reduces the number of pairwise comparisons needed.

3. Locality-Sensitive Hashing (LSH)

LSH hashes similar items into the same buckets with a high probability. For cosine similarity, hyperplane LSH can be used:

PYTHON
from sklearn.neighbors import NearestNeighbors

def lsh_hash(vecs):
    model = NearestNeighbors(n_neighbors=5, algorithm='brute', metric='cosine').fit(vecs)
    distances, indices = model.kneighbors(vecs)
    return indices  # Buckets of similar vectors

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary:

All-pairs cosine similarity is O(n^2) because it requires comparing each vector with every other vector.
ANN indexing reduces the complexity by leveraging indexes that approximate nearest neighbors efficiently.
LSH further optimizes by hashing similar vectors into the same buckets, making search operations faster.

#### Common mistakes
to avoid:

Overlooking the scalability issues of O(n^2) algorithms on large datasets.
Failing to understand the trade-offs between exact and approximate methods in terms of accuracy vs. performance.
Not considering the specific characteristics of the data (e.g., dimensionality) when choosing an indexing method.
Triangular Inequality and Failure Modes

Triangular inequality can help mitigate some issues:

If two vectors are close, they must be closer to each other than any vector in between them.
However, this is approximate and does not guarantee exact matches without additional checks.

#### Example
Data and Inputs
PYTHON
import numpy as np

prompts = [
    "The quick brown fox jumps over the lazy dog.",
    "A quick brown fox jumps over a lazy dog.",
    "The swift brown dog leaps over the sleeping cat.",
    "The clever dog leapt over the sleepy cat."
]
vecs = [np.random.rand(10) for _ in prompts]  # Randomized vectors for illustration

index = create_annoy_index(vecs)
nearest_neighbors = find_nearest_neighbors(index, vecs[0])
Self-Check

The solution is correct and complete.

**Code:**

```python
def all_pairs_cosine_similarity(vecs):
    n = len(vecs)
    similarities = [[0 for _ in range(n)] for _ in range(n)]

    for i in range(n):
        for j in range(i+1, n):
            v1 = vecs[i]
            v2 = vecs[j]
            dot_product = np.dot(v1, v2)
            norm_v1 = np.linalg.norm(v1)
            norm_v2 = np.linalg.norm(v2)
            similarity = dot_product / (norm_v1 * norm_v2)
            similarities[i][j] = similarities[j][i] = similarity

    return similarities

import annoy

def create_annoy_index(vecs):
    index = annoy.AnnoyIndex(len(vecs[0]), 'angular')
    for i, vec in enumerate(vecs):
        index.add_item(i, vec)
    index.build(10)  # Build the index with 10 trees
    return index

def find_nearest_neighbors(index, query_vector, k=5):
    neighbors = index.get_nns_by_vector(query_vector, k, include_distances=True)
    return neighbors

from sklearn.neighbors import NearestNeighbors

def lsh_hash(vecs):
    model = NearestNeighbors(n_neighbors=5, algorithm='brute', metric='cosine').fit(vecs)
    distances, indices = model.kneighbors(vecs)
    return indices  # Buckets of similar vectors
```

**Example input / output:**

```text
index = create_annoy_index(vecs)
nearest_neighbors = find_nearest_neighbors(index, vecs[0])
print(nearest_neighbors)
# Output: [(1, 0.89), (3, 0.75), ...] - Nearest neighbors and their distances
```

**Data:**

```python
prompts = [
    "The quick brown fox jumps over the lazy dog.",
    "A quick brown fox jumps over a lazy dog.",
    "The swift brown dog leaps over the sleeping cat.",
    "The clever dog leapt over the sleepy cat."
]
vecs = [np.random.rand(10) for _ in prompts]  # Randomized vectors for illustration
```

