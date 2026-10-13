---
type: "practice"
difficulty: "Deep"
---

# What is the recursion-tree method for analyzing a recurrence, and how does it help you sum the work per level?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The recursion-tree method is a powerful technique for understanding the time complexity of recursive algorithms by visualizing the cost at each level of the recursion tree. This approach helps us break down the problem into smaller subproblems and sum up their contributions to identify the overall complexity.

To start, let's clarify what we're probing here: The interviewer wants to know if you can apply this method effectively, understand its key steps, and recognize when it is useful. We'll assume a recurrence relation of the form T(n) = aT(n/b) + f(n), where (a \geq 1), (b > 1), and (f(n)) represents the cost outside the recursive calls.

Firstly, let's build the recursion tree for such a relation. Each node in the tree represents an instance of the problem, with its children representing subproblems. The root node has value f(n), corresponding to the initial call. It then splits into (a) child nodes, each having value (f(n/b)). This process continues until we reach the leaf level.

We'll reason step-by-step from first principles:

Identify the Levels: Each level of the recursion tree corresponds to a certain size of the subproblems. The root is at level 0, and the next level down (its children) are at level 1, and so on.
Calculate Costs at Each Level: At each level (i), we determine how many nodes there are and their cost. For example, if a tree has depth (d = \log_b n) (the number of times you can divide (n) by (b)), the work done at level (i) is generally proportional to (f(n/b^i)).
Sum Across Levels: The total time complexity is the sum of costs across all levels from 0 to (d-1). This summation gives us a clear picture of how the overall cost evolves.

For instance, if we have T(n) = 4T(n/2) + n^2, each level will contribute as follows:

Level 0: Cost is (n^2)
Level 1: There are 4 subproblems, each with a cost of ((n/2)^2), so total cost at this level is (4 \times (n/2)^2 = n^2 / 2).
Level 2: Each of the 16 subproblems has a cost of ((n/4)^2), and so on.

Summing these up, we get: [ T(n) = n^2 + \frac{n^2}{2} + \frac{n^2}{8} + ... + O(1) ]

Using geometric series summation, this simplifies to (O(n^2)).

This method is particularly useful when the recurrence splits into a fixed number of subproblems (constant (a)) and each subproblem reduces by the same factor (constant (b)). It’s especially handy for problems like merge sort or quicksort.

In summary, the recursion-tree method provides a concrete way to visualize and analyze recursive algorithms by breaking them down level-by-level. It helps us understand the work done at each level and ultimately gives us the total complexity.

Common pitfalls include not correctly identifying the number of subproblems per level or misestimating the cost of splitting the problem.

**Comparison:**

| Level | Cost per Subproblem             | Total Cost at Level |
|-------|---------------------------------|--------------------|
| 0     | \(n^2\)                         | \(n^2\)            |
| 1     | \((n/2)^2\)                     | \(4 * (n/2)^2 = n^2 / 2\) |
| 2     | \((n/4)^2\)                     | \(16 * (n/4)^2 = n^2 / 8\)|

**Code:**

```python
# Example Python function to illustrate recursion tree method concept (simplified)
def example_recurrence(n):
    if n <= 1:
        return n  # Base case
    else:
        # 4 subproblems, each on half the input size + cost of combining them
        return 4 * example_recurrence(n // 2) + n**2

# Here we would implement the recursion tree manually to calculate the total cost.
```

**Example input / output:**

```text
graph TD;
    A[Root Node] --> B{Cost at Level 0};
    B --> C[n^2];
    C --> D[Level 1: Cost per Subproblem (n/2)^2];
    D --> E[4 * (n/2)^2 = n^2 / 2];
    E --> F[Level 2: Cost per Subproblem (n/4)^2];
    F --> G[16 * (n/4)^2 = n^2 / 8];
```

**Data:**

```json
{
  "recurrence": {
    "a": 4,
    "b": 2,
    "f(n)": "n^2"
  },
  "levels": [
    {"level": 0, "cost_per_subproblem": "n^2", "total_cost": "n^2"},
    {"level": 1, "cost_per_subproblem": "(n/2)^2", "total_cost": "4 * (n/2)^2 = n^2 / 2"},
    {"level": 2, "cost_per_subproblem": "(n/4)^2", "total_cost": "16 * (n/4)^2 = n^2 / 8"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the problem size <= 1?
5. Return n
6. Split into a subproblems, eachon half the input size + cost ofcombining them
7. Calculate total cost for thislevel and recurse
8. Add up costs across all levelsto get final complexity
9. End

