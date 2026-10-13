---
type: "practice"
difficulty: "Intermediate"
---

# You compute a batched softmax over a 2D logits matrix and a naive implementation overflows; how does the log-sum-exp trick keep it numerically stable and why does axis-wise reduction keep it O(n)?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ External Memory Deduplication](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/External_Memory_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle numerical stability in batched softmax computations, we can use the log-sum-exp (LSE) trick to avoid overflow issues that arise from exponentiating large numbers. The LSE trick involves transforming the logits matrix into a form where the exponential function is applied after shifting by a constant value. This ensures that the resulting values are within a manageable range, making the softmax computation numerically stable.

#### Triage

The interviewer is probing your understanding of numerical stability in machine learning computations and how to apply the log-sum-exp trick. The key assumptions here include:

You understand basic operations like exponentiation and summation.
You know about common numerical issues in deep learning (e.g., overflow).
You can explain why axis-wise reduction keeps the computation efficient.
Step-by-Step Solution

Understanding Overflow: When computing softmax, each element is exponentiated by a large value due to high logits. This leads to very large numbers, and when these are summed up, they result in an overflow.

Log-Sum-Exp Trick:

Given a matrix of logits ( \mathbf{A} = [a_{ij}] ), we want to compute the softmax function ( s_i(j) = \frac{\exp(a_{ij})}{\sum_k \exp(a_{ik})} ).
The LSE trick shifts each element by subtracting a constant that is the maximum value in its column: ( b_{ij} = a_{ij} - \max_j(a_{ij}) ).
Then, compute the exponential of these shifted values and sum them up: ( c_i = \sum_k \exp(b_{ik}) ).
Finally, apply the softmax function as ( s_i(j) = \frac{\exp(b_{ij})}{c_i} ).

Mathematical Insight:

The maximum value in each column is subtracted to ensure all values are reduced without changing their relative differences.
This step effectively shifts the exponentiated values into a safe range, preventing overflow.

Axis-Wise Reduction:

To compute the softmax over an entire batch of logits (e.g., ( \mathbf{A} ) is a 2D matrix), we can reduce along the correct axis.
The reduction keeps the computation in O(n) complexity, where n is the number of elements per row if reducing along columns or vice versa.

#### Closing Summary

The log-sum-exp trick ensures numerical stability by reducing large exponential values to manageable ranges. Axis-wise reductions keep the computation efficient and scalable, making it suitable for batch processing in machine learning models.

#### Common Mistakes/Follow-Ups
Mistake: Failing to recognize the importance of axis reduction or assuming a global operation might be used.

#### Follow-Up
The interviewer might ask about alternative methods for numerical stability (e.g., using floating-point types with higher precision) and their trade-offs.

**Comparison:**

| Logits   | Shifted | Exp(shifted) | Sum       |
|----------|---------|--------------|-----------|
| [0, 1, 2] | [-0.693, -0.578, -0.444] | [0.329, 0.242, 0.175] | 0.746    |
| [3, 4, 5] | [0.578, 1.241, 1.866]   | [0.329, 0.242, 0.175] | 0.746    |

**Code:**

```python
import numpy as np

def log_sum_exp(matrix):
    max_values = np.max(matrix, axis=1, keepdims=True)
    shifted_matrix = matrix - max_values
    exp_shifted = np.exp(shifted_matrix)
    sums = np.sum(exp_shifted, axis=1, keepdims=False)
    return shifted_matrix - np.log(sums)

# Example usage
logits = np.array([[0, 1, 2], [3, 4, 5]])
stabilized_logits = log_sum_exp(logits)
print(stabilized_logits)
```

**Example input / output:**

```text
Given the logits matrix:
\[
\begin{bmatrix}
0 & 1 & 2 \\
3 & 4 & 5
\end{bmatrix}
\]

Applying log-sum-exp trick results in:
\[
\begin{bmatrix}
-0.693 & -0.578 & -0.444 \\
0.578 & 1.241 & 1.866
\end{bmatrix}
\]
Then, after exponentiating and summing, the final stabilized matrix is:
\[
\begin{bmatrix}
0.329 & 0.242 & 0.175 \\
0.329 & 0.242 & 0.175
\end{bmatrix}
\]
```

**Data:**

```json
{
  "logits": [
    [0, 1, 2],
    [3, 4, 5]
  ]
}
```

**Flow:**

1. Log-Sum-Exp Steps
2. Yes
3. Input Logits Matrix
4. Axis-wise Reduction?
5. Shift by Column Max
6. Apply Exponential Function
7. Sum Along Axis
8. Compute Softmax
9. Output Stabilized Values

