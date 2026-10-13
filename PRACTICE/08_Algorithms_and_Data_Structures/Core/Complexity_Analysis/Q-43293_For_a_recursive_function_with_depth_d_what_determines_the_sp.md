---
type: "practice"
difficulty: "Intermediate"
---

# For a recursive function with depth d, what determines the space complexity beyond the time complexity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive vs Iterative Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Recursive_vs_Iterative_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how the space complexity of a recursive function is determined beyond its time complexity, we need to focus on two key aspects: the call stack and any additional data structures used by the function. Let’s break down the solution from first principles.

The space complexity of a recursive function primarily depends on:

The maximum depth of recursion (which determines the size of the call stack).
Any auxiliary space required for other variables or data structures used within each recursive call.

In this context, while time complexity (T(n)) is determined by the number of operations performed and how they scale with input size, space complexity (S(n)) includes both the stack frame overhead and any additional memory used per function invocation.

**Code:**

```python
def factorial(n):
    if n == 0:
        return 1
    else:
        return n * factorial(n - 1)

The factorial function is a classic example. It has a time complexity of O(n) because it makes n recursive calls. However, its space complexity is also O(n) due to the call stack depth.
```

**Example input / output:**

```

No specific input/output examples needed here, but let's consider an example:

Input: n = 5 Output (call stack size): 5 Stack Frames:

CODE
Copy
factorial(5)
    factorial(4)
        factorial(3)
            factorial(2)
                factorial(1)
                    factorial(0) -> return 1
```

**Flow:**

1. Yes
2. No
3. Input n
4. Is n == 0?
5. Return 1
6. n * factorial(n-1)

