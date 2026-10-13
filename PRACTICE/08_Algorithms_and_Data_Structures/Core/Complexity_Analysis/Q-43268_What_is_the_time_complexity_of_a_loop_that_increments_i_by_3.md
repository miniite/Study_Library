---
type: "practice"
difficulty: "Basic"
---

# What is the time complexity of a loop that increments i by 3 each iteration until it reaches n?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the time complexity of a loop that increments i by 3 each iteration until it reaches n, we need to break down the problem and analyze its underlying structure step-by-step.

#### Triaging the Question

The core of this question is to determine how many iterations the loop will take before reaching or exceeding n. The key aspects the interviewer might be probing are:

Understanding Iterative Processes: How do loops work in terms of iteration counts?
Mathematical Analysis: Can we derive a mathematical formula for the number of iterations?
Step-by-Step Solution

First, let's consider the loop structure:

PLAINTEXT
for (i = 0; i < n; i += 3) {
    // Some operations
}

Here, i starts at 0 and increments by 3 until it reaches or exceeds n. The goal is to determine how many iterations this will take.

Mathematical Insight

The number of iterations can be derived from the arithmetic sequence formula. Specifically, we need to find the smallest integer ( k ) such that: [ 0 + 3k \geq n ] This simplifies to: [ k \geq \frac{n}{3} ] Since k must be an integer, we take the ceiling of the division: [ k = \left\lceil \frac{n}{3} \right\rceil ]

#### Closing Summary

The time complexity of this loop is ( O(\frac{n}{3}) ), which simplifies to ( O(n) ). This means that as n grows, the number of iterations increases linearly with n.

#### Common Mistakes
and Follow-Ups
Incorrect Simplification: A common mistake might be incorrectly assuming the time complexity is ( O(1) ) or ( O(\log n) ), especially if one does not properly analyze the arithmetic progression.
Boundary Conditions: Be sure to consider edge cases like n being a multiple of 3, close to but just below a multiple of 3, etc.

#### Example
Calculation

For instance, if n = 10, then: [ k = \left\lceil \frac{10}{3} \right\rceil = \left\lceil 3.33 \right\rceil = 4 ] So the loop runs 4 times.

Self-Check

This analysis covers both the conceptual and mathematical aspects of time complexity for the given problem, ensuring correctness and completeness.

**Code:**

```plaintext
for (i = 0; i < n; i += 3) {
    // Some operations
}
```

