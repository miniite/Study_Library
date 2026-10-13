---
type: "practice"
difficulty: "Intermediate"
---

# You compute cosine similarity between one query embedding and a million item embeddings on CPU and it's too slow; how does batching into a single matrix multiply change the complexity constant, and where do BLAS libraries help?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Similarity Search Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Similarity_Search_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of computing cosine similarity between one query embedding and a million item embeddings on CPU, we can significantly improve performance by batching the operations into a single matrix multiply operation. This change reduces the complexity constant from O(m) to O(1), where ( m ) is the number of dimensions in the embeddings.

#### Triage

The question asks how batching changes the complexity and highlights the role of BLAS libraries in optimizing the computation. The core idea here is understanding that matrix multiplication can be much faster than repeated scalar multiplications due to vectorization and optimization provided by hardware-level instruction sets like SIMD (Single Instruction, Multiple Data).

#### First-Principles Solution

Step 1: Understanding Cosine Similarity Calculation Cosine similarity between a query embedding ( \mathbf{q} ) of dimensionality ( d ) and an item embedding matrix ( \mathbf{I} ) with shape ( (d, m) ), where ( m ) is the number of items, is given by: [ \text{similarity}(i) = \frac{\mathbf{q} \cdot \mathbf{i}_j}{|\mathbf{q}|_2 |\mathbf{i}_j|_2} ] For a single query and ( m ) items, this means performing ( m ) dot products.

Step 2: Batched Matrix Multiplication By stacking the embeddings into a matrix: [ \mathbf{I} = \begin{bmatrix} \mathbf{i}_1 \ \mathbf{i}_2 \ \vdots \ \mathbf{i}_m \end{bmatrix} ] and performing the matrix multiplication with ( \mathbf{q} ), we get: [ \mathbf{M} = \mathbf{I}^T \cdot \mathbf{q} = \begin{bmatrix} \mathbf{i}_1 & \mathbf{i}_2 & \cdots & \mathbf{i}_m \end{bmatrix}^T \cdot \mathbf{q} = \begin{bmatrix} \mathbf{i}_1 \cdot \mathbf{q} \ \mathbf{i}_2 \cdot \mathbf{q} \ \vdots \ \mathbf{i}_m \cdot \mathbf{q} \end{bmatrix} = \mathbf{I}^T \cdot \mathbf{q} ] This operation computes ( m ) dot products in a single pass, reducing the complexity to O(1) per item.

Step 3: Normalization We then normalize each result by dividing it by the norms of the query and corresponding items: [ \text{similarity}(i_j) = \frac{\mathbf{i}_j \cdot \mathbf{q}}{|\mathbf{q}|_2 |\mathbf{i}_j|_2} ]

#### Closing Summary

By batching all computations into a single matrix multiply, we reduce the number of scalar multiplications from ( O(m) ) to ( O(1) ), significantly improving performance. BLAS (Basic Linear Algebra Subprograms) libraries like OpenBLAS or Intel MKL provide optimized implementations of these operations that leverage hardware acceleration for further speedup.

#### Common Mistakes
and Follow-Ups
Mistake: Not understanding the difference between scalar multiplications and matrix operations.

#### Follow-Up
Discussing the impact on memory usage, especially with large datasets. How can out-of-memory issues be mitigated? Are there other optimization techniques like incremental updates or approximate methods?

