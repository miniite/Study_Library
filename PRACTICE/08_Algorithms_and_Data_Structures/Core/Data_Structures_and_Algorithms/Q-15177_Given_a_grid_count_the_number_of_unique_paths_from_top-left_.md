---
type: "practice"
difficulty: "Intermediate"
---

# Given a grid, count the number of unique paths from top-left to bottom-right moving only right or down.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Grid Path Counting](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Grid_Path_Counting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to determine the number of unique paths from the top-left corner (0, 0) to the bottom-right corner (m-1, n-1) in a grid, moving only right or down. This is a classic combinatorial problem that can be solved using dynamic programming and combinatorics.

#### Triage

The question asks for the number of unique paths given specific movement constraints (only right or down). The interviewer is probing your understanding of dynamic programming techniques and combinatorial mathematics. We need to assume that each cell in the grid can be reached from either its left neighbor or top neighbor, and we aim to find all possible paths.

#### First-Principles Solution

Grid Representation: Consider a grid of size m x n.

Path Constraints: From any cell (i, j), you can only move to (i+1, j) or (i, j+1).

Base Cases:

The starting point (0, 0) has exactly one path to itself.
Any cell on the top row (except for the start) can only be reached by moving right from the previous column.
Any cell on the leftmost column (except for the start) can only be reached by moving down from the previous row.

Dynamic Programming Approach:

Initialize a 2D array dp where dp[i][j] represents the number of unique paths to reach cell (i, j).
For each cell in the grid, calculate dp[i][j] = dp[i-1][j] + dp[i][j-1], which means the path count is the sum of the path counts from the top and left cells.

Final Answer: The value at dp[m-1][n-1] will give us the total number of unique paths to reach the bottom-right corner.

#### Closing Summary

The problem can be efficiently solved using dynamic programming, leveraging the properties of combinatorial mathematics. By initializing a grid and filling it based on path counts from adjacent cells, we can find the number of unique paths in an elegant manner. Common mistakes include incorrect initialization or not properly handling edge cases like the first row and column.

Code Implementation

Here’s how you could implement this solution:

PYTHON
def unique_paths(m: int, n: int) -> int:
    # Initialize a 2D array with zeros
    dp = [[0 for _ in range(n)] for _ in range(m)]

    # Base case initialization
    for i in range(m):
        dp[i][0] = 1
    for j in range(n):
        dp[0][j] = 1

    # Fill the DP table
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]

    return dp[m-1][n-1]

# Example usage:
print(unique_paths(3, 7))  # Output: 28

#### Common Mistakes
Forgetting to initialize the first row and column.
Incorrectly calculating path counts in the DP table.
Failing to correctly handle edge cases where m or n is small.

