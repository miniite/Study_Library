---
type: "practice"
difficulty: "Intermediate"
---

# If constraints say n up to 20, what complexity class is the interviewer probably hinting you can afford?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Big-O vs Constant Factors](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Big-O_vs_Constant_Factors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an interviewer mentions that n (the input size) can go up to 20, they are often indicating that a more complex solution might be acceptable due to the limited range of inputs. In this case, we should consider algorithms and data structures that have better time or space complexity but may not be practical for larger values of n.

#### Triaging the Question

The interviewer is probing whether you can design an algorithm with a more optimized complexity class (e.g., O(n log n) instead of O(n^2)) without compromising performance. They are also assessing your understanding of trade-offs between different complexities and how they impact real-world scenarios.

#### First Principles Solution

Given that n can be up to 20, we have a relatively small input size. For such constraints, the following complexities are generally acceptable:

O(1) - Constant Time: This is the best possible complexity class as it means the algorithm runs in the same amount of time regardless of the input size.
O(log n): Algorithms that run faster than linearly but not as fast as linearithmically. Suitable for problems where you can leverage binary search or logarithmic growth rates.
O(n) - Linear Time: This is a common complexity class and usually acceptable, especially if the algorithm can be optimized to this level.

#### Example Scenario
s

Sorting:

If sorting n elements (with n <= 20), we might use a simpler O(n^2) sorting technique like bubble sort or insertion sort, but merge sort would still work and is O(n log n).

Search Operations:

For searching in an array of size up to 20, binary search (O(log n)) can be used efficiently.

Graph Traversal:

For small graphs (e.g., with n <= 20 nodes), algorithms like BFS or DFS might suffice and are O(V + E).

#### Closing Summary

In summary, for an input size of up to 20, you can afford more complex algorithms as long as the constant factors remain manageable. It's important to choose a complexity class that optimizes performance while ensuring maintainability and readability.

#### Common Mistakes
& Follow-Ups
Mistake: Failing to recognize the limited input size and choosing unnecessarily simpler algorithms.

#### Follow-Up
Discussing edge cases, such as handling n = 0 or other small inputs. Interviewers might ask about how your solution behaves with these constraints.

**Code:**

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# Example usage
arr = [1, 3, 5, 7, 9]
target = 5
index = binary_search(arr, target)
print(index)  # Output: 2
```

**Example input / output:**

```text
Input:
arr = [1, 3, 5, 7, 9]
target = 5

Output:
2
```

