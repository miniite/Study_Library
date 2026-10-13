---
type: "practice"
difficulty: "Intermediate"
---

# Search for a target in a 2D matrix where rows and columns are sorted, better than O(n*m).

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this problem, we need to search for a target value in a 2D matrix where both rows and columns are sorted. The naive approach would be to iterate through each element, which is (O(n \times m)), but we can do better using a more strategic traversal of the matrix.

The key idea here is to leverage the property that the row elements are sorted from left to right and the column elements are sorted from top to bottom. By starting at the top-right corner (or bottom-left corner) of the matrix, we can make efficient decisions on whether to move up, down, left, or right based on the comparison with the target value.

Let's walk through this step-by-step.

