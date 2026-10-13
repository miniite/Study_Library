---
type: "practice"
difficulty: "Deep"
---

# You are asked to reduce the space complexity of a DP-heavy pricing engine that stores a full O(n*m) table; how do you decide between rolling arrays, recomputation, and just buying more RAM?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Amortized Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Amortized_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When optimizing a DP-heavy pricing engine with an O(n*m) space complexity to reduce memory usage, we need to carefully weigh trade-offs between different approaches such as rolling arrays, recomputation, and simply increasing the RAM. This decision involves understanding the nature of the DP algorithm, the constraints on computation time, and the business context.

The key idea is that by using a more sophisticated approach like rolling arrays or careful recomputation, we can significantly reduce memory usage without sacrificing too much computational efficiency, thereby achieving better overall performance.

**Comparison:**

| n | m | Previous Row (Start) | Current Row (End) |
|---|---|----------------------|------------------|
| 1 | 3 | [0, 0, 0]            | [1, 2, 3]        |
| 2 | 3 | [1, 2, 3]            | [2, 4, 6]        |
| 3 | 3 | [2, 4, 6]            | [3, 7, 9]        |
| 4 | 3 | [3, 7, 9]            | [4, 10, 13]      |
| 5 | 3 | [4, 10, 13]          | [5, 12, 16]      |

**Code:**

```python
# Example of a simple rolling array implementation in Python for a DP problem

def optimized_pricing(n, m):
    if n == 0 or m == 0:
        return 0

    # Initialize the first row (rolling array)
    previous_row = [0] * (m + 1)

    for i in range(1, n + 1):
        current_row = [0] * (m + 1)

        # Fill the current row using the previous row
        for j in range(1, m + 1):
            # Example computation: simple addition; replace with actual logic
            current_row[j] = previous_row[j - 1] + i

        # Update the previous row to be the current row
        previous_row = current_row

    return previous_row[m]

# Example usage
result = optimized_pricing(5, 3)
print(result)  # Output will depend on actual logic but this is a simplified example
```

**Example input / output:**

```text
Input:
- `n = 5`, `m = 3`

Output:
- Result depends on the actual implementation but in this simplified example, the output is `12`.

Dry-run of the optimized_pricing function for n=5, m=3 using a rolling array approach.
```

**Data:**

```json
{
  "n": 5,
  "m": 3,
  "expected_result": 12  // Example expected result based on the provided Python code and logic
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Start
5. Is n = 0 or m = 0?
6. Return 0
7. Initialize previous_row of size(m + 1)
8. For i from 1 to n, initializecurrent_row
9. Is j <= m?
10. Compute current_row[j] =previous_row[j - 1] + i
11. Update previous_row withcurrent_row
12. Increment j and repeat from F
13. Return previous_row[m]

