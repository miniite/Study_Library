---
type: "practice"
difficulty: "Basic"
---

# How would you swap two numbers without a temporary variable, and why is XOR-swap risky?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Swapping and Bit Tricks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Swapping_and_Bit_Tricks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To swap two numbers without using a temporary variable, one common approach is to use the XOR bitwise operation. However, it's important to understand that while this method works in most cases, there are edge cases where it can fail, such as when dealing with negative integers or specific values.

#### Triaging the Question

The question asks for an algorithmic solution using bitwise operations and also prompts you to identify potential risks associated with this approach. The interviewer is likely probing your understanding of bitwise operators and their limitations in different scenarios.

#### First-Principles Solution

To swap two numbers (a) and (b), we can use the XOR bitwise operation as follows:

Compute (a = a \oplus b).
Compute (b = a \oplus b) (now (b) will be the original value of (a \oplus b)).
Compute (a = a \oplus b) (now (a) will be the original value of (b)).

Here’s how this works in practice:

Initial values: (a = 5), (b = 10)
Step 1: (a = 5 \oplus 10 = -7) (in binary, (101 \oplus 1010 = -1001))
Step 2: (b = -7 \oplus 10 = 5) (now (b) holds the original value of (a))
Step 3: (a = -7 \oplus 10 = 10) (now (a) holds the original value of (b))

So, after these steps, the values of (a) and (b) are swapped.

#### Common Mistakes
and Follow-Ups
Edge Cases: The XOR-swap method can fail with certain edge cases, such as when dealing with negative integers due to overflow in two's complement representation.
Specific Values: If both numbers are zero or if they have specific values that lead to incorrect results (e.g., (a = b)), the swap might not work as expected.

#### Closing Summary

In conclusion, while the XOR-swap technique is a clever way to swap two numbers without using additional storage, it has limitations and potential risks. It's essential to understand these constraints when deciding whether this approach is suitable for your specific use case.

