---
type: "practice"
difficulty: "Intermediate"
---

# Partition an array into two subsets with equal sum, and explain why this is a DP problem.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Subset Sum DP](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Subset_Sum_DP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Alright, let's dive into the problem of partitioning an array into two subsets such that the sum of elements in both subsets is equal. This is not a straightforward task and often requires dynamic programming (DP) to solve efficiently.

#### Triaging the Question

The question is asking us to determine if it's possible to split an array nums into two disjoint sets, each with an equal sum. This problem can be approached using dynamic programming because we're essentially trying to achieve a target value (half of the total sum of the array) by making optimal choices at each step.

#### Assumptions
and First Principles
Input: An integer array nums.
Output: A boolean indicating whether such a partition is possible.

#### Cons
traints:
The length of nums can be up to 100.
Elements in nums are integers.
Step-by-Step Solution

To solve this, we need to determine if the total sum of elements in the array is even and less than or equal to half the size of the input. Here’s a detailed explanation:

Calculate Total Sum: Compute the total sum of all elements in the array.
Check for Even Sum: If the total sum is odd, it's impossible to partition the array into two subsets with equal sums.
Target Value: Define target as half of the total sum (i.e., total_sum // 2). We need to find a subset whose elements sum up to this target.
Dynamic Programming Setup:
Use a boolean DP table where dp[j] indicates whether it's possible to achieve a sum j using some elements from the array.
Initialize dp[0] = True, as achieving a sum of 0 is always possible (using no elements).
Fill the DP Table:
Iterate through each element in the array and update the DP table by considering including or excluding each element.
Result: Check if dp[target] is true.
Code Implementation

Here’s a Python implementation:

PYTHON
def canPartition(nums):
    total_sum = sum(nums)

    # If total sum is odd, it's impossible to partition into two equal subsets
    if total_sum % 2 != 0:
        return False

    target = total_sum // 2
    n = len(nums)

    # Initialize DP table with False values except dp[0] which is True
    dp = [False] * (target + 1)
    dp[0] = True

    for num in nums:
        for j in range(target, num - 1, -1):
            if dp[j - num]:
                dp[j] = True

    return dp[target]

# Example usage
nums = [1, 5, 11, 5]
print(canPartition(nums))  # Output: True

nums = [1, 2, 3, 4]
print(canPartition(nums))  # Output: False
Explanation of the Code
Initialization: We initialize a DP array dp with all values set to False, except for dp[0] which is True.
Filling the DP Table: For each number in the input array, we update the DP table from right to left (decreasing order) to avoid overwriting results that are needed later.
Result Check: Finally, we check if dp[target] is True.

#### Closing Summary

To determine if it's possible to partition an array into two subsets with equal sums, we first calculate the total sum and ensure it’s even. Then, using dynamic programming, we build a DP table to keep track of achievable subset sums. If the target sum can be achieved, we return True; otherwise, False.

#### Common Mistakes
and Follow-Ups
Ignoring Odd Sums: Always check if the total sum is odd; an even sum must exist for such a partition.
Incorrect DP Table Filling Order: Ensure you fill the table in reverse order to avoid using results that haven’t been computed yet.
Target Value Calculation: Make sure to correctly calculate target as half of the total sum.

