---
type: "practice"
difficulty: "Intermediate"
---

# Find the length of the longest increasing subsequence and mention the O(n log n) improvement.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Longest Increasing Subsequence](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Longest_Increasing_Subsequence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding the length of the longest increasing subsequence (LIS) with an (O(n \log n)) time complexity, we need to understand both the core concept and some efficient algorithms that can achieve this goal.

#### Triage

The question is asking for a solution to find the length of the longest increasing subsequence in an array. The key challenge here is achieving (O(n \log n)) efficiency, which is significantly better than the naive (O(n^2)) approach using dynamic programming (DP).

#### Assumptions
and Key Idea
We assume that the input is a sequence of integers.
The goal is to find the length of the LIS, not necessarily the actual subsequence itself.

The key idea behind achieving (O(n \log n)) time complexity involves leveraging binary search in combination with an array (or list) to keep track of the smallest possible tail values for increasing subsequences of different lengths. This allows us to efficiently update and query this information using a balanced data structure like a Binary Indexed Tree (BIT) or a Segment Tree.

#### Solution
Step 1: Understanding Dynamic Programming Approach

In a naive (O(n^2)) approach, we can use dynamic programming where dp[i] represents the length of the longest increasing subsequence ending at index i. The recurrence relation is: [ dp[i] = \max(dp[j]) + 1 \text{ for all } j < i \text{ and } arr[j] < arr[i] ]

Step 2: Achieving (O(n \log n)) Efficiency

To achieve the desired efficiency, we can use a technique involving binary search and maintaining an auxiliary array tails. The idea is to keep track of the smallest possible tail values for increasing subsequences of different lengths.

Initialization: Create an empty list tails where each element represents the minimum possible last value of all increasing subsequences of that length.
Processing Each Element:
For each element in the array, use binary search to find the position at which this element can extend some subsequence.
Replace the first element in tails greater than or equal to the current element with it. This way, we maintain the smallest possible tail values for increasing subsequences of different lengths.
Step 3: Implementation

Here is a Python implementation that achieves (O(n \log n)) time complexity:

PYTHON
def length_of_LIS(nums):
    if not nums:
        return 0

    tails = [nums[0]]

    for num in nums[1:]:
        pos = binary_search(tails, num)

        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num

    return len(tails)

def binary_search(arr, target):
    low, high = 0, len(arr) - 1

    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1

    return low
Step 4: Explanation of the Code
tails is an array that keeps track of the smallest possible last values for increasing subsequences.
binary_search function efficiently finds the position to insert or replace elements in tails.
The length of tails at the end gives us the length of the longest increasing subsequence.

#### Closing Summary

The (O(n \log n)) solution uses a combination of binary search and dynamic programming concepts. By maintaining an auxiliary array and using binary search, we can efficiently update and query the smallest possible tail values for increasing subsequences. This approach ensures that we only process each element once, leading to significant time savings.

#### Common Mistakes
Forgetting to use tails as a reference point in the binary search.
Not updating tails correctly when performing binary search.
Misunderstanding the purpose of tails and not realizing it needs to keep track of smallest possible tail values.

