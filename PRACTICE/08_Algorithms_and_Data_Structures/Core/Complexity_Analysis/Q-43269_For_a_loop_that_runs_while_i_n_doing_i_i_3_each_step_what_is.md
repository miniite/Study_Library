---
type: "practice"
difficulty: "Intermediate"
---

# For a loop that runs while i < n doing i = i * 3 each step, what is the complexity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Nested Loop Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Nested_Loop_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, let's start with a brief overview and then dive into the details.

#### Overview

The given loop iterates while i < n and in each iteration multiplies i by 3. We need to determine the time complexity of this loop. The key insight is that the value of i grows exponentially, which can help us derive an accurate time complexity.

#### Triaging the Question

The question is asking for the time complexity of a specific loop structure. The interviewer might be probing your understanding of how exponential growth impacts the number of iterations and thus the overall time complexity. We will assume that n represents the initial value or boundary condition, and we'll reason step-by-step from first principles.

#### First-Principles Solution

Let's analyze the loop step by step:

Initial Condition: Assume i = 0 initially.
Condition for Loop Exit: The loop continues as long as i < n.
Iteration Operation: In each iteration, i = i * 3.

We need to determine how many times the loop runs until i exceeds or equals n. Let's denote the number of iterations by k. After k iterations, we have: [ i_k = 0 \times 3^k < n \leq 1 \times 3^k ]

This simplifies to finding the smallest integer k such that: [ 3^k \geq n ] Taking the logarithm base 3 on both sides, we get: [ k \geq \log_3(n) ] Since ( k ) must be an integer, we take the ceiling of this value: [ k = \lceil \log_3(n) \rceil ]

#### Closing Summary
and Common Mistakes

The time complexity is directly related to the number of iterations k, which is approximately (\log_3(n)). Therefore, the time complexity of the loop is (O(\log_3(n))).

#### Common Mistakes

Failing to recognize that each iteration grows exponentially.
Assuming a linear relationship between i and the number of iterations.
Not converting the problem into a logarithmic form.
Detailed Explanation

Let's consider some examples:

If ( n = 8 ), we have:

Iteration 1: ( i = 0 \times 3^1 = 0 < 8 )
Iteration 2: ( i = 0 \times 3^2 = 0 < 8 ) (still less than 8, so iteration continues)

If ( n = 27 ), we have:

Iteration 1: ( i = 0 \times 3^1 = 0 < 27 )
Iteration 2: ( i = 0 \times 3^2 = 0 < 27 )
Iteration 3: ( i = 0 \times 3^3 = 27 \geq 27 ) (loop stops here)

In both cases, the number of iterations is logarithmic in n.

Summary

The time complexity of the given loop is (O(\log_3(n))).

**Code:**

```python
# Example code for the loop
def loop_complexity(n):
    i = 0
    while i < n:
        i *= 3
```

**Example input / output:**

```text
Input: `n = 27`
Output: The number of iterations is approximately \(\lceil \log_3(27) \rceil = 3\).
```

**Flow:**

1. No
2. Yes
3. Initial i=0
4. i < n?
5. Exit loop
6. i *= 3

