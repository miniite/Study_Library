---
type: "practice"
difficulty: "Basic"
---

# Determine whether a number is a power of two using bit manipulation.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine if a given number is a power of two, we can use a clever trick involving bitwise operations. The key idea here is that powers of two have only one bit set in their binary representation (e.g., 2 = 10, 4 = 100, 8 = 1000). Let's break down the solution step-by-step.

First, we need to understand what the question is asking: Given a number n, determine if it is a power of two. The interviewer might be probing for your understanding of bitwise operations and integer properties.

#### Assumptions
We are working with positive integers.
We will ignore the case where the input is 0, as (2^0 = 1) is not considered a power of two in this context (though it could be handled if needed).
Step-by-Step Solution
Check for Zero: If n is zero, return false because no negative powers of two exist.
Bitwise AND Operation: A number that is a power of two will have exactly one bit set to 1 in its binary representation. For example, (4 = 0b100). When you subtract 1 from such a number, all the bits after the single 1 bit flip to 1s (e.g., (4 - 1 = 3 = 0b011)). Therefore, if n is a power of two, performing an AND operation between n and n-1 should result in zero.

Here’s how the logic works:

If n is not a power of two, n & (n - 1) will be non-zero.
If n is a power of two, n & (n - 1) will be zero.
Code Implementation
PYTHON
def is_power_of_two(n):
    # Check if n is greater than 0 and the bitwise AND between n and n-1 is 0
    return n > 0 and (n & (n - 1)) == 0

#### Example
Dry Run

Let’s run a few test cases to ensure our function works correctly.

PYTHON
print(is_power_of_two(1))   # Output: True (2^0)
print(is_power_of_two(2))   # Output: True (2^1)
print(is_power_of_two(3))   # Output: False
print(is_power_of_two(4))   # Output: True (2^2)
print(is_power_of_two(5))   # Output: False

#### Closing Summary
and Common Mistakes
Common Mistake: Forgetting to handle the case where n is zero.
Common Mistake: Incorrectly assuming that any non-zero number might be a power of two, not realizing specific bit patterns are required.

By using bitwise operations, we leverage the unique properties of powers of two and provide an efficient solution with minimal computational overhead.

**Code:**

```python
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0
```

**Example input / output:**

```text
print(is_power_of_two(1))   # True
print(is_power_of_two(2))   # True
print(is_power_of_two(3))   # False
print(is_power_of_two(4))   # True
print(is_power_of_two(5))   # False
```

