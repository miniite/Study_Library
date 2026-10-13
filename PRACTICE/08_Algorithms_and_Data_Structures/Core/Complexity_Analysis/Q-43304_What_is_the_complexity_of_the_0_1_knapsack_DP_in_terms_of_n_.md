---
type: "practice"
difficulty: "Deep"
---

# What is the complexity of the 0/1 knapsack DP in terms of n items and capacity W, and why is it pseudo-polynomial?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Operation Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Set_Operation_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary

The 0/1 knapsack problem involves selecting a subset of items to maximize the total value while keeping the total weight under a given capacity limit (W). The dynamic programming (DP) solution has time and space complexity dependent on both the number of items (n) and the capacity (W), making it pseudo-polynomial.

#### Triage

The question is asking for the time and space complexities of the 0/1 knapsack problem using a dynamic programming approach. The key points to address are:

Time Complexity: This depends on both the number of items ((n)) and the capacity limit ((W)).
Space Complexity: Also influenced by (n) and (W).
Why it is pseudo-polynomial: This refers to how the time complexity can be polynomial in the size of the input, but potentially exponential in the magnitude of the inputs.

#### First Principles Solution

Problem Overview: The 0/1 knapsack problem involves deciding whether to include or exclude each item based on its weight and value, ensuring the total weight does not exceed (W). A common DP approach uses a table where dp[i][w] represents the maximum value that can be attained with the first (i) items having at most capacity (w).

Dynamic Programming Table Construction:

Initialize a 2D array dp of size ((n+1) \times (W+1)).
Iterate over each item from 0 to (n-1):
For each weight from 0 to (W):
If the current item can fit in the remaining capacity, update dp[i][w] by considering whether including the item increases the value.

Time Complexity: The time complexity of this approach is (O(n \times W)). This is because we have a nested loop where each iteration takes constant time to compute.

Outer loop: (n) iterations (one for each item).
Inner loop: (W+1) iterations (one for each possible capacity from 0 to (W)).

Space Complexity: The space complexity is also (O(n \times W)), as we need to store the DP table of size ((n+1) \times (W+1)).

Why it is Pseudo-polynomial:

The time and space complexities are polynomial in the magnitude of (n) and (W).
However, when the input values (weights and values) can be large numbers, the actual running time can still grow exponentially with respect to the bit length of these numbers.
For example, if (W = 2^{31}), the problem might take significantly longer than polynomial in the size of the input (e.g., the number of bits representing (W)).

#### Closing Summary
and Common Mistakes
Common Mistake: Confusing pseudo-polynomial time complexity with polynomial time complexity. Pseudo-polynomial means it is polynomial in the magnitude, but potentially exponential in the bit length.

#### Follow-up
Understanding how to optimize or approximate the solution for large (W), such as using branch and bound or approximation algorithms like greedy methods.
Depth and Precision

The 0/1 knapsack problem's dynamic programming solution demonstrates a balance between space and time efficiency. While the approach is effective, understanding its limitations in terms of input magnitude is crucial for real-world applications.

**Comparison:**

| w \ i | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 3 | 0 | 0 | 0 | 0 |
| 2 | 0 | 3 | 4 | 0 | 0 | 0 |
| 3 | 0 | 3 | 4 | 5 | 0 | 0 |
| 4 | 0 | 3 | 4 | 5 | 6 | 0 |
| 5 | 0 | 3 | 4 | 5 | 7 | 12 |

**Code:**

```python
def knapsack(n, weights, values, W):
    dp = [[0] * (W + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for w in range(W + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i-1][w], values[i-1] + dp[i-1][w - weights[i-1]])
            else:
                dp[i][w] = dp[i-1][w]

    return dp[n][W]
```

**Example input / output:**

```text
| w | dp[1] | dp[2] | dp[3] | dp[4] | dp[5] |
|---|-------|-------|-------|-------|-------|
| 0 | 0     | 0     | 0     | 0     | 0     |
| 1 | 0     | 3     | 0     | 0     | 0     |
| 2 | 0     | 3     | 4     | 0     | 0     |
| 3 | 0     | 3     | 4     | 5     | 0     |
| 4 | 0     | 3     | 4     | 5     | 6     |
| 5 | 0     | 3     | 4     | 5     | 7     |

Final value: dp[5][9] = 12
```

**Data:**

```text
(n: 5, weights: [2, 3, 4, 5, 6], values: [3, 4, 5, 6, 7], W: 9)
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Initialization
6. Is item i included?
7. Update dp[i][w] =max(dp[i-1][w], value +dp[i-1][w - weight])
8. Check next w
9. End of capacity loop?
10. Check next item i
11. End of items?
12. Return dp[n][W]

