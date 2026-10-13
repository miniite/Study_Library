---
type: "practice"
difficulty: "Intermediate"
---

# How do you analyze an algorithm with three sequential phases: an O(n log n) sort, then an O(n) scan, then an O(1) return?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When analyzing algorithms that consist of multiple sequential phases, we focus on the overall complexity by considering the most significant factor from each phase. In this case, we have three phases with different complexities: an O(n log n) sort, followed by an O(n) scan, and finally an O(1) return.

#### Triaging the Question

The interviewer is primarily probing your understanding of how to combine these individual time complexities and ensuring you can reason about them from first principles. They are also testing your ability to handle different levels of complexity when performing a chain operation.

#### First-Principles Solution

To determine the overall time complexity, we start by looking at each phase individually:

O(n log n) sort: This is the most complex part and will dominate the overall complexity.
O(n) scan: While this is linear, it is less significant than O(n log n).
O(1) return: This is constant time and has no impact on the overall complexity.
Reasoning Step-by-Step

Understanding Each Phase:

The O(n log n) sort phase processes all elements in a collection, resulting in a logarithmic factor that grows with the size of the input.
The O(n) scan phase iterates through each element once, which is linear and grows proportionally to the input size.
The O(1) return phase simply returns a result based on some computation involving constant time operations.

Combining Phases:

In sequential algorithms like this, we consider the sum of their complexities, but only the most significant term remains dominant.
For example, if you had phases with complexities O(n), O(log n), and O(1), the overall complexity would be O(n) because n grows faster than log n.

#### Conclusion

Since O(n log n) is more complex than both O(n) and O(1), the overall time complexity of this algorithm is O(n log n).

#### Closing Summary

The overall time complexity of the given algorithm, consisting of a sort phase with O(n log n), a scan phase with O(n), and a return phase with O(1), is dominated by the sorting step. Therefore, the final time complexity is O(n log n).

#### Common Mistakes/Follow-Ups
A common mistake might be incorrectly adding up all phases without recognizing that only the most significant term dictates the overall complexity.
Interviewers may ask follow-up questions to test your deeper understanding of big O notation and algorithm analysis, such as asking about space complexity or how changes in input size affect the performance.

**Code:**

```python
# Example Python function for illustration (not part of the solution)
def example_algorithm(arr):
    arr.sort()  # O(n log n)
    total = sum(arr)  # O(n)
    return total  # O(1)
```

**Data:**

```json
{
  "input": [3, 1, 4, 1, 5, 9],
  "output": 23
}
```

