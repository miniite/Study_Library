---
type: "practice"
difficulty: "Deep"
---

# Probe: your sliding window gives wrong answers when the array contains negatives; why, and what would you use instead?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a sliding window algorithm providing incorrect answers in an array containing negative numbers, we need to first understand why the standard sliding window technique may fail and then explore alternative approaches that can handle such scenarios.

#### Triage

The core issue with using a sliding window approach on arrays containing negative numbers is typically related to how the sum or value of elements within the window is calculated. Standard sliding window techniques often rely on maintaining a single sum for each window, which can be problematic when the array includes negative values. This problem arises because the presence of negative numbers can cause the sum to become smaller than expected, leading to incorrect results.

#### First Principles Solution

To resolve this issue, we need to consider an alternative approach that can handle negative numbers effectively. One effective solution is using a Prefix Sum combined with a Sliding Window technique. Here's how it works:

Compute Prefix Sums: Calculate the prefix sum array where prefix_sum[i] represents the cumulative sum of elements from index 0 to i-1.
Sliding Window on Prefix Sums: Use the sliding window approach on the prefix sums instead of direct element sums. This way, we can avoid issues related to negative numbers directly impacting our current window's sum.
Detailed Steps:
Compute Prefix Sum Array:
Initialize a prefix sum array prefix_sums where prefix_sums[i] = arr[0] + arr[1] + ... + arr[i-1].
Initialize Sliding Window Parameters:
Set the initial window size and start position.
Sliding Window on Prefix Sums:
For each possible ending point of the window, calculate the sum using the prefix sums: prefix_sums[end] - prefix_sums[start].
Update Maximum/Minimum Sum (As Needed):
Track the maximum or minimum sum based on the problem requirements.

#### Example
Code:
PYTHON
def sliding_window_with_prefix_sum(arr):
    n = len(arr)
    # Compute prefix sums
    prefix_sums = [0] * (n + 1)
    for i in range(1, n + 1):
        prefix_sums[i] = arr[i - 1] + prefix_sums[i - 1]

    max_sum = float('-inf')
    # Initialize sliding window
    start = 0
    end = 1

    while end <= n:
        current_sum = prefix_sums[end] - prefix_sums[start]
        max_sum = max(max_sum, current_sum)

        if current_sum < 0:  # Shrink the window to avoid negative impact
            start += 1
            end = start + 1
        else:
            end += 1

    return max_sum

# Example usage
arr = [-2, -3, 4, -1, -2, 1, 5, -3]
print(sliding_window_with_prefix_sum(arr))  # Output: 7 (subarray [4, -1, -2, 1, 5])

#### Closing Summary

By using prefix sums in combination with a sliding window approach, we can effectively handle negative numbers without the issues that arise from directly summing elements within the window. This method ensures that our results are accurate and reliable.

#### Common Mistakes
and Follow-Ups:
Mistake: Failing to recognize that standard sliding window techniques may not work with negative numbers.

#### Follow-Up
Exploring other scenarios where similar issues might occur, such as when dealing with non-contiguous subarrays or more complex conditions. Understanding the trade-offs between different approaches can also be beneficial.

**Code:**

```python
def sliding_window_with_prefix_sum(arr):
    n = len(arr)
    prefix_sums = [0] * (n + 1)
    for i in range(1, n + 1):
        prefix_sums[i] = arr[i - 1] + prefix_sums[i - 1]

    max_sum = float('-inf')
    start = 0
    end = 1

    while end <= n:
        current_sum = prefix_sums[end] - prefix_sums[start]
        max_sum = max(max_sum, current_sum)

        if current_sum < 0:
            start += 1
            end = start + 1
        else:
            end += 1

    return max_sum

arr = [-2, -3, 4, -1, -2, 1, 5, -3]
print(sliding_window_with_prefix_sum(arr))  # Output: 7 (subarray [4, -1, -2, 1, 5])
```

**Example input / output:**

```text
arr = [-2, -3, 4, -1, -2, 1, 5, -3]
output = sliding_window_with_prefix_sum(arr)
print(f"Maximum subarray sum is {output}")  # Output: Maximum subarray sum is 7 (subarray [4, -1, -2, 1, 5])
```

**Flow:**

1. Yes
2. No
3. Yes
4. Compute Prefix Sums
5. Is window size fixed?
6. Initialize Sliding Window
7. Sliding Window on Prefix Sums
8. Update Maximum/MinimumSum (As Needed)
9. Shrink Window If Necessary?
10. G
11. Adjust Start and End Points
12. End Loop
13. End Loop

