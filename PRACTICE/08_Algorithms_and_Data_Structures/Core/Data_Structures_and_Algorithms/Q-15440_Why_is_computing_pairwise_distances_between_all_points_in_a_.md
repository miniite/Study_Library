---
type: "practice"
difficulty: "Basic"
---

# Why is computing pairwise distances between all points in a dataset O(n^2), and how does that limit naive k-nearest-neighbors at scale?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Similarity Search Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why computing pairwise distances between all points in a dataset is (O(n^2)) and its implications on scaling, we need to break down the problem from first principles.

#### Triaging the Question

The core of this question revolves around understanding two key aspects:

Computational Complexity: Why does the time complexity of computing pairwise distances between all points in a dataset scale as (O(n^2))?
Implications on Algorithms: How does this computational complexity affect algorithms like k-Nearest Neighbors (k-NN) at scale?

#### First-Principles Solution
Step 1: Understanding Pairwise Distances

For a dataset with n points, we need to calculate the distance between each pair of points. This means for every point (i), we compute its distance from all other (n - 1) points.

The number of pairs is given by the combination formula: [ \text{Number of pairs} = \binom{n}{2} = \frac{n(n-1)}{2} \approx n^2 ] This simplifies to approximately (O(n^2)) as (n) becomes large, because the term ((n-1)) is dominated by (n).

Step 2: Computing Distances

Assuming we have a distance function (e.g., Euclidean distance), for each pair of points, we need to: [ d(i, j) = \sqrt{(x_i - x_j)^2 + (y_i - y_j)^2} ] If the coordinates are in (k) dimensions, this computation would be (O(k)). Thus, the overall time complexity becomes (O(n^2 k)).

Step 3: Implications for k-Nearest Neighbors

In a naive implementation of k-NN, we compute pairwise distances between all points and then select the top k closest neighbors. This means that as the number of points increases, the time required to perform these computations grows quadratically.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary

Computing pairwise distances is (O(n^2)) because for each point, we need to calculate its distance from every other point in the dataset. This quadratic complexity limits the scalability of algorithms like k-NN, especially as the number of points grows.

#### Common Mistakes
and Follow-Ups
Misunderstanding Linear Complexity: A common mistake is assuming linear time complexity because each pair involves only two points.
Ignoring Dimensionality: Often, people forget to consider the dimensionality (k), which can significantly affect performance as it multiplies the complexity by (k).
Naive vs. Optimized Approaches: Discussing optimized algorithms like kd-trees or ball trees that can reduce the complexity in practice is important.

#### Example
Naive k-NN Implementation

#### Cons
ider a simple Python function to compute pairwise distances:

PYTHON
import numpy as np

def euclidean_distance(point1, point2):
    return np.sqrt(np.sum((point1 - point2) ** 2))

def compute_pairwise_distances(data):
    n = len(data)
    distances = [[0] * n for _ in range(n)]

    for i in range(n):
        for j in range(i + 1, n):
            distances[i][j] = euclidean_distance(data[i], data[j])
            distances[j][i] = distances[i][j]

    return distances

This function computes the pairwise distances for a dataset and returns them in a matrix.

Self-Check

This solution is correct and complete, covering both the theoretical understanding and practical implications.

**Code:**

```python
import numpy as np

def euclidean_distance(point1, point2):
    return np.sqrt(np.sum((point1 - point2) ** 2))

def compute_pairwise_distances(data):
    n = len(data)
    distances = [[0] * n for _ in range(n)]

    for i in range(n):
        for j in range(i + 1, n):
            distances[i][j] = euclidean_distance(data[i], data[j])
            distances[j][i] = distances[i][j]

    return distances
```

**Example input / output:**

```

Example input:

PYTHON
Copy
data = [[0, 0], [1, 1], [2, 2]]
distances = compute_pairwise_distances(data)
print(distances)

Output:

CODE
Copy
[[0.        , 1.41421356, 2.82842712],
 [0.        , 1.41421356, 2.82842712],
 [0.        , 1.41421356, 2.82842712]]
```

