---
type: "practice"
difficulty: "Intermediate"
---

# What is the complexity of matrix multiplication for two n-by-n matrices, and why does it matter when multiplying weight matrices in a neural net?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Matrix Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with matrix multiplication for two (n \times n) matrices, the fundamental operation's complexity is crucial to understand, especially in the context of neural networks where large matrices are common. The core complexity of multiplying two such matrices is (O(n^3)). This is because each element in the resulting matrix requires a summation over all elements from one row and one column, leading to (n) multiplications per element times (n) elements (both rows and columns), hence (n \times n \times n = O(n^3)).

The complexity of this operation directly matters when working with large matrices in neural networks. Neural network weight matrices can be very large, often representing millions or billions of parameters. Therefore, the (O(n^3)) complexity can become a significant computational bottleneck.

To break it down from first principles:

Understanding Matrix Multiplication: When multiplying two matrices (A) (of size (n \times m)) and (B) (of size (m \times n)), each element in the resulting matrix (C) of size (n \times n) is computed as a dot product between a row from (A) and a column from (B). For an (n \times n) case, this means:

Each element in (C) requires (m) multiplications.
There are (n^2) elements in (C). Therefore, the total number of operations is (n \times m \times n = O(n^3)).

Why This Matters: In neural networks, weight matrices often have dimensions in the thousands or millions. For instance, a 1024x1024 matrix multiplication involves approximately (10^{9}) multiplications and additions. Even with modern hardware, this can be computationally expensive.

Efficiency Considerations: Many optimizations exist to reduce the complexity of matrix multiplication, such as using Strassen's algorithm or even more advanced techniques like the Coppersmith-Winograd algorithm. However, these are less practical for smaller matrices and often come at a cost in terms of increased memory usage or implementation complexity.

Real-World Impact: In practice, the (O(n^3)) complexity can be mitigated by:

Using more efficient algorithms.
Utilizing parallel processing techniques (e.g., GPUs).
Implementing matrix sparsity to reduce the number of operations.

In summary, while the basic complexity is (O(n^3)), there are ways to optimize this for practical use in large-scale neural networks. Understanding these complexities and potential optimizations is crucial for designing efficient machine learning models.

**Comparison:**

```
i	j	k	C[i][j] += A[i][k] * B[k][j]
0	0	0	C[0][0] += 1 * 5
0	0	1	C[0][0] += 2 * 7
0	1	0	C[0][1] += 1 * 6
0	1	1	C[0][1] += 2 * 8
1	0	0	C[1][0] += 3 * 5
1	0	1	C[1][0] += 4 * 7
1	1	0	C[1][1] += 3 * 6
1	1	1	C[1][1] += 4 * 8
```

**Code:**

```python
def matrix_multiply(A, B):
    n = len(A)
    C = [[0] * n for _ in range(n)]

    for i in range(n):
        for j in range(n):
            for k in range(n):
                C[i][j] += A[i][k] * B[k][j]
    return C
```

**Example input / output:**

```

Input:

A = [[1, 2], [3, 4]]
B = [[5, 6], [7, 8]]

Output:

PYTHON
Copy
[
  [19, 22],
  [43, 50]
]
```

**Data:**

```json
{
  "A": [
    [1, 2],
    [3, 4]
  ],
  "B": [
    [5, 6],
    [7, 8]
  ]
}
```

**Flow:**

1. A
2. A[i][k]*B[k][j]
3. B

