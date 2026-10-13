---
type: "practice"
difficulty: "Deep"
---

# Probe: your DP solution uses O(n*m) memory; how would you reduce it to O(m)?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Algorithms](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sorting_Algorithms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to understand that the original dynamic programming (DP) solution has a space complexity of (O(n \times m)), where (n) is the length of one dimension and (m) is the length of another. The question asks us to reduce this memory usage to just (O(m)). This reduction can be achieved by leveraging the properties of DP solutions, particularly those that are based on tabulation.

#### Triaging the Question

The core of the question is about optimizing space complexity in a DP solution. We need to recognize that many DP problems involve state transitions where only the current and previous states are required at any given step. This allows us to reduce memory usage significantly by reusing space instead of storing all intermediate results.

#### Assumptions
The problem is based on a typical 2D DP table, which requires (O(n \times m)) space.
We need to find an alternative way to store the state transitions that only uses (O(m)) space.

#### First-Principles Solution

Let’s consider a common scenario where we have a 2D DP array dp of size (n \times m). In many DP problems, the current cell's value depends on the previous cells in some way. For instance, if the problem is to find the minimum path sum, each cell might depend on the values from the left and top neighbors.

To reduce space complexity, we can observe that only the current row and possibly one or two rows (or columns) of the DP table are needed at any given time. Thus, instead of storing an entire 2D array, we can use a single array to represent the current row and update it as we move through the problem.

Step-by-Step Solution
Initialization: Start with initializing a 1D array dp of size (m). This array will store the values for the first row (or column) of the original DP table.
First Row/Column Handling: Handle the first row/column separately as they are usually initialized differently.
Transition Update: For each cell in the remaining rows, update the dp array based on the previous row’s state and the current cell's value. Specifically:
If we are updating from top to bottom (for a row-major traversal), use the updated values of the dp array to compute the next row.
If using column-major traversal, you might need two arrays: one for the current column and another for the previous column.

Here’s an example in pseudocode:

PYTHON
def optimize_dp(n, m):
    # Initialize dp with the first row values or column values depending on the problem
    dp = [0] * m

    # Handle the first column/row separately if necessary
    for i in range(1, n):
        # Update current cell based on previous states
        for j in range(m):
            if (i == 0):  # First row handling
                # Direct initialization or update from top-left corner
                dp[j] = original_dp[i][j]
            else:
                # Update the current state using the previously computed values
                dp[j] = min(dp[j], dp[j-1]) + cost(i, j)  # Example transition

    return dp

#### Closing Summary
and Common Mistakes/Follow-Ups

By using a single array to simulate the DP table, we significantly reduce memory usage. However, it’s crucial to handle edge cases correctly, especially when dealing with boundary conditions or different initialization requirements for each row/column.

#### Common mistakes
include:

Not properly handling the first row/column.
Forgetting to update the dp array based on the correct previous states.
Misunderstanding the problem constraints and not recognizing where you can reuse space.

For follow-ups, one might ask about the time complexity of this approach or consider if there are any further optimizations possible. If necessary, we could discuss multidimensional spaces or other advanced DP optimization techniques like space-filling curves or 1D to 2D mappings.

**Comparison:**

```
Row	Col	Original Value	Updated DP
1	0	0	0
	1	3	2
	2	8	8
	3	-4	7
Row	Col	Original Value	Updated DP
2	0	2	2
	1	5	7
	2	10	17
	3	1	18
Row	Col	Original Value	Updated DP
3	0	-1	-1
	1	-6	-7
	2	-7	-8
	3	9	1
```

**Code:**

```python
def optimize_dp(n, m):
    dp = [0] * m

    for i in range(1, n):
        if i == 0:
            # Direct initialization for the first row
            dp[0] = original_dp[i][0]
        else:
            # Update current state based on previous states
            for j in range(m):
                dp[j] = min(dp[j], dp[(j-1) % m]) + cost(i, j)

    return dp
```

**Example input / output:**

```text
Input:
- `n = 3`, `m = 4`

Output:
- `[0, 2, 8, 7]` (Example output for the first row after optimization)
```

**Data:**

```json
{
  "original_dp": [[0, 3, 8, -4],
                  [2, 5, 10, 1],
                  [-1, -6, -7, 9]]
}
```

**Flow:**

1. Yes
2. No
3. Initialize DP array
4. Is it the first row/column?
5. Direct initialize or update fromtop-left corner
6. Update current state based onprevious states
7. Store value in dp[j]
8. Loop through columns toupdate values
9. Return optimized DP array

