---
type: "practice"
difficulty: "Basic"
---

# What is the difference between '/' and '//' division operators in Python 3, and what type do they return?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Python 3, the key differences between the / and // division operators lie in their behavior regarding floating-point and integer results. The / operator returns a float result, ensuring that even when dividing integers, you get a precise decimal value. On the other hand, // performs floor division, which returns an integer by rounding down to the nearest whole number.

#### Triaging the Question

The interviewer is likely probing your understanding of basic Python operations and their implications on data types. You should be prepared to explain both operators clearly and provide examples that showcase these differences.

Step-by-Step Solution From First Principles

Operator Behavior:

/ (Floating Division):

This operator divides the first operand by the second.
The result is always a float, even if both operands are integers.

// (Floor Division):

This operator also divides the first operand by the second.
However, it returns an integer that is the largest possible value less than or equal to the actual division result.

#### Example
s:

PYTHON
# Floating Division Example
print(10 / 3)      # Output: 3.3333333333333335

# Floor Division Example
print(10 // 3)     # Output: 3 (integer division)

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the / operator in Python 3 always returns a float result, while // performs integer division and rounds down to the nearest whole number. Understanding these nuances can help avoid unexpected behavior in your code.

#### Common mistakes
include assuming that both operators return the same type of result or failing to recognize when floor division is necessary for integer results.

For follow-ups, you might be asked about edge cases like negative numbers or zero denominators. Make sure to address these scenarios as well:

Edge Cases:

For / (Floating Division):

PYTHON
print(-10 / 3)   # Output: -3.3333333333333335

For // (Floor Division):

PYTHON
print(-10 // 3)  # Output: -4

By exploring these examples and edge cases, you can ensure a comprehensive understanding of the difference between the two division operators in Python.

**Code:**

```python
# Floating Division Example
print(10 / 3)      # Output: 3.3333333333333335

# Floor Division Example
print(10 // 3)     # Output: 3 (integer division)
```

**Example input / output:**

```text
print(-10 / 3)   # Output: -3.3333333333333335

print(-10 // 3)  # Output: -4
```

