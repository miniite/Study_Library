---
type: "practice"
difficulty: "Deep"
---

# Find the two numbers that appear only once in an array where all others appear twice.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Single Number Bit Tricks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Single_Number_Bit_Tricks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to find the two unique numbers in an array where every other number appears exactly twice. The key idea is to use bitwise operations, specifically XOR, which helps us isolate the unique numbers. Here's a step-by-step breakdown:

Problem Restatement and Assumptions: We are given an array of integers where each integer except two appears exactly twice. Our goal is to find these two unique numbers.
Key Insight: Using the properties of XOR:
(a \oplus 0 = a)
(a \oplus a = 0)
XOR is commutative and associative, so the order of operations doesn't matter.

By leveraging these properties, we can use XOR to find the two unique numbers efficiently. The approach involves three main steps:

First, find any non-zero bit that appears in one of the unique numbers but not the other.
Use this bit to partition the array into two groups and apply XOR within each group.

Let's walk through the solution step by step:

Step-by-Step Solution
XOR All Elements: Compute the XOR of all elements in the array. This will result in a number that is the XOR of the two unique numbers because every pair of duplicate numbers will cancel out, leaving only the two unique numbers.
Identify a Differentiating Bit: Find any bit position where the result from step 1 has a set bit (i.e., a position where one of the unique numbers has a 1 and the other does not). This can be done by taking the bitwise AND of this number with its two's complement.
Partitioning and XORing: Use the differentiating bit to partition the array into two groups. All elements that have the same value in this bit position will go into one group, and those with a differing value will go into another. Compute the XOR of each group separately; these results will be the two unique numbers.
Code Implementation

Here's how we can implement this logic:

PYTHON
def find_single_numbers(nums):
    # Step 1: XOR all elements to get a number that is the XOR of the two unique numbers.
    xor_all = 0
    for num in nums:
        xor_all ^= num

    # Step 2: Identify a differentiating bit (any set bit works).
    diff_bit = xor_all & -xor_all

    # Step 3: Partition and compute XOR within each group.
    res1, res2 = 0, 0
    for num in nums:
        if num & diff_bit:
            res1 ^= num
        else:
            res2 ^= num

    return [res1, res2]

#### Example

Let's consider an example array: [4, 1, 2, 1, 2].

XOR all elements: 4 ^ 1 ^ 2 ^ 1 ^ 2 = 0^4 = 4
Identify a differentiating bit: 4 in binary is 100, so we can use the least significant set bit (which is 4th bit).
Partition and XOR:
Group 1: [4] -> res1 = 4
Group 2: [1, 2, 1, 2] -> res2 = 0

Thus, the unique numbers are 4 and 0.

#### Common Mistakes
and Follow-Ups
Mistake: Not handling edge cases where the array might be empty or contain only one element.

#### Follow-Up
If there were more than two unique numbers, how would you modify your solution?

