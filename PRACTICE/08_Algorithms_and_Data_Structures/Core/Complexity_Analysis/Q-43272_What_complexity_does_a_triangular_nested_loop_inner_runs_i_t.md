---
type: "practice"
difficulty: "Intermediate"
---

# What complexity does a triangular nested loop (inner runs i times) collapse to, and how do you get from n^2/2 to O(n^2)?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When analyzing a triangular nested loop where the inner loop runs i times, we need to determine the time complexity of the entire construct. This is a common scenario in algorithms and data structures, and it's crucial to understand how such loops behave as the input size grows.

Triangulation and Time Complexity

#### Triage
Understanding the Question

The question asks about the time complexity of a triangular nested loop where the innermost loop runs i times for each iteration of the outer loop. We need to derive the total number of operations performed by this construct, which will help us determine its time complexity.

#### Assumptions
and Context
Let ( n ) be the input size.
The outer loop runs from 1 to ( n ).
For each value of ( i ), the inner loop runs exactly ( i ) times.

#### First Principles Solution

To find the total number of operations, we can sum up the operations performed by the inner loop for all values of ( i ). Specifically, when ( i = 1 ), the inner loop runs once; when ( i = 2 ), it runs twice; and so on until ( i = n ), where it runs ( n ) times. This can be expressed as: [ \sum_{i=1}^{n} i ]

The sum of the first ( n ) natural numbers is given by the formula: [ \frac{n(n+1)}{2} ]

This expression simplifies to approximately ( \frac{n^2}{2} + O(n) ). For large values of ( n ), the dominant term is ( \frac{n^2}{2} ), so we can simplify this to ( O(n^2) ).

#### Closing Summary
and Common Mistakes

The time complexity of a triangular nested loop where the inner loop runs i times is ( O(n^2) ). This is because the total number of operations grows quadratically with respect to the input size. A common mistake might be misinterpreting the summation or incorrectly estimating the dominant term.

Why and Edge Cases

Understanding why this complexity arises helps in recognizing similar patterns in other algorithms. For instance, consider a scenario where you are summing elements in an array using nested loops; this pattern often results in ( O(n^2) ).

Common Follow-Ups
How would the complexity change if the inner loop ran from 1 to k for each iteration of the outer loop?
What about if there were more than two nested loops, and each had a different behavior?

**Code:**

```python
def example_function(n):
    total_operations = 0
    for i in range(1, n + 1):  # Outer loop runs from 1 to n
        for j in range(i):      # Inner loop runs i times
            total_operations += 1
    return total_operations
```

**Example input / output:**

```text
For \( n = 5 \):

- When \( i = 1 \), the inner loop runs 1 time.
- When \( i = 2 \), the inner loop runs 2 times.
- When \( i = 3 \), the inner loop runs 3 times.
- When \( i = 4 \), the inner loop runs 4 times.
- When \( i = 5 \), the inner loop runs 5 times.

Total operations: \( 1 + 2 + 3 + 4 + 5 = 15 \)

This is equivalent to:
\[ \frac{5(5+1)}{2} = 15 \]

This confirms our formula.
```

**Flow:**

1. Yes
2. No
3. Start
4. Outer Loop (i from 1 to n)
5. Inner Loop (j from 0 to i-1)
6. Is j < i?
7. Increment Operations
8. End Inner Loop
9. End Outer Loop
10. Total Operations = Sum ofoperations

