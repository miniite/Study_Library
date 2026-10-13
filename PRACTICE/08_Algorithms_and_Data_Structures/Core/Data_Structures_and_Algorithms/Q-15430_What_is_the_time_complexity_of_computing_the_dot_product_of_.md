---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of computing the dot product of two dense vectors of length n, and how does it change for sparse vectors?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the time complexity of computing the dot product of two vectors, let's first briefly summarize what a dot product is. The dot product of two vectors (\mathbf{a}) and (\mathbf{b}) is calculated as:

[ \text{dot product} = \sum_{i=1}^n a_i \cdot b_i ]

where (a_i) and (b_i) are the elements of the respective vectors, and (n) is the length of the vectors.

Now, let's dive into the time complexity for both dense and sparse vectors.

