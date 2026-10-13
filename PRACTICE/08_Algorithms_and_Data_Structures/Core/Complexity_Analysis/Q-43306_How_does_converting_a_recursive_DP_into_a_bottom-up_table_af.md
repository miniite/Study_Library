---
type: "practice"
difficulty: "Intermediate"
---

# How does converting a recursive DP into a bottom-up table affect both time and space complexity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Converting a recursive dynamic programming (DP) solution to a bottom-up approach significantly changes the time and space complexities, often improving them by removing recursion overheads and eliminating stack space usage.

#### Triaging the Question

The core of this question is about understanding how the conversion from a recursive DP implementation to an iterative one affects both time and space complexity. The interviewer might be probing your knowledge of dynamic programming techniques, optimization practices, and trade-offs between different implementations.

#### First Principles Solution

Recursive Dynamic Programming (Time and Space Complexity)

Time Complexity: In most recursive DP solutions, the time complexity is (O(n^2)) or similar due to redundant function calls and overlapping subproblems.
Space Complexity: The space complexity includes both the stack space used for recursion and any additional memory for storing intermediate results. Typically, it can be as high as (O(n)), depending on how many recursive calls are made.

Bottom-Up Dynamic Programming (Time and Space Complexity)

Time Complexity: The time complexity remains the same, i.e., (O(n^2)) or similar, because we are still solving each subproblem once.
Space Complexity: By using an iterative approach with a table to store intermediate results, we can reduce the space overhead. Instead of using stack space for recursion, we use a fixed-size array (or matrix), resulting in (O(n^2)) or similar space complexity.

Key Differences

Recursion Overhead: Recursively solving subproblems involves function call overheads and maintaining an implicit call stack, which can be expensive.
Stack Space vs. Table: Iterative solutions avoid the stack by using a table to store results, thus reducing the risk of stack overflow.

#### Example

#### Cons
ider a simple problem like finding the nth Fibonacci number recursively:

PYTHON
def fib(n):
    if n <= 1:
        return n
    return fib(n-1) + fib(n-2)

This recursive solution has exponential time complexity and linear space due to the call stack. The bottom-up version would be:

PYTHON
def fib_bottom_up(n):
    if n <= 1:
        return n
    dp = [0] * (n+1)
    dp[0], dp[1] = 0, 1
    for i in range(2, n+1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

Here, the time complexity is still (O(n)), but the space complexity reduces to (O(n)).

#### Common Mistakes
and Follow-Ups
Misunderstanding Time Complexity: Some candidates might incorrectly state that iterative DP always has a lower time complexity than recursive DP.
Ignoring In-place Optimization: While converting, make sure to check if in-place optimization can be applied further to reduce space usage.
Overlooking Edge Cases: Ensure you handle edge cases like negative indices or non-positive numbers correctly.

#### Closing Summary

In summary, the conversion from a recursive DP to an iterative bottom-up approach helps mitigate the overheads of recursion and stack usage. While the time complexity remains unchanged, the space complexity is often improved, leading to more efficient solutions in terms of both time and memory.

Self-Check

The solution covers all aspects of the question: it explains the concepts clearly, provides a detailed comparison, and highlights potential pitfalls.

**Comparison:**

| n    | Recursive Calls (Stack) | Bottom-Up Table (Space) |
|------|-------------------------|------------------------|
| 10   | O(2^10)                 | O(n)                   |
| 20   | O(2^20)                 | O(n)                   |

**Code:**

```python
# Recursive Fibonacci
def fib_recursive(n):
    if n <= 1:
        return n
    return fib_recursive(n-1) + fib_recursive(n-2)

# Bottom-up Fibonacci
def fib_bottom_up(n):
    if n <= 1:
        return n
    dp = [0] * (n+1)
    dp[0], dp[1] = 0, 1
    for i in range(2, n+1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

**Example input / output:**

| Input | Output (Bottom-Up) |
|-------|--------------------|
| 1     | 1                  |
| 2     | 1                  |
| 3     | 2                  |
| 4     | 3                  |
| 5     | 5                  |
| 6     | 8                  |

**Data:**

```json
{
  "n": 6,
  "recursive_result": null,
  "bottom_up_result": 8
}
```

**Flow:**

1. Yes
2. No
3. Bottom-Up DP
4. Is n <= 1?
5. Return n
6. Fib(n-1) + Fib(n-2)
7. Initialize dp array
8. For i from 2 to n: dp[i] =dp[i-1] + dp[i-2]
9. Return dp[n]

