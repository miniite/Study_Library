---
type: "practice"
difficulty: "Intermediate"
---

# Given an array of positive numbers, find the smallest subarray with a sum at least a target.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Maximum Subarray](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Maximum_Subarray.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify the shortest contiguous subarray in an array of positive numbers that has a sum greater than or equal to a given target. The key idea is to use a sliding window approach, which is both efficient and straightforward for this type of question.

The problem can be broken down as follows:

Understanding the Question: We are looking for the smallest subarray (contiguous segment) in an array where the sum of its elements meets or exceeds a specified target.

#### Assumptions
The input array is non-empty and contains only positive numbers.
The target value is a positive integer.

Now, let's reason step-by-step to arrive at the solution using first principles:

Initialize two pointers: start pointing to the beginning of the array (0), and end also at 0. These will represent the current subarray we are examining.

Maintain a variable current_sum to keep track of the sum of elements in the current window defined by the start and end pointers.

Use a loop that runs until either:

The end pointer reaches the end of the array, or
We find a subarray with a sum greater than or equal to the target.

Inside this loop:

Add the value at the current end position to current_sum.
If current_sum meets or exceeds the target, we have found a valid subarray. Now check if it is smaller than any previously found subarrays by storing its length and adjusting the window's start pointer.
Continue moving the end pointer rightwards until current_sum falls below the target.

After exiting the loop, return the smallest subarray found or a message indicating no such subarray exists.

This approach ensures that each element is processed at most twice (once when added and once when potentially removed), making it an efficient O(n) solution in terms of time complexity.

**Code:**

```python
def min_subarray_with_target_sum(arr, target):
    start = 0
    end = 0
    current_sum = 0
    min_length = float('inf')

    while end < len(arr):
        current_sum += arr[end]

        while current_sum >= target:
            if (end - start + 1) < min_length:
                min_length = end - start + 1

            current_sum -= arr[start]
            start += 1

        end += 1

    return min_length if min_length != float('inf') else None
```

**Example input / output:**

```text
For input array `[2, 3, 4, 6]` with target `7`, the function returns `2`. This corresponds to the subarray `[3, 4]`.
```

**Data:**

```json
{
    "arr": [2, 3, 4, 6],
    "target": 7,
    "expected_output": 2 // The subarray [3, 4] has the smallest length of 2 and its sum is 7.
}
```

**Flow:**

1. No
2. Yes
3. Initialize start and end pointers
4. Set current_sum = 0
5. Increment end pointer
6. current_sum += arr[end]
7. current_sum < target
8. Current subarray is valid
9. min_length update
10. Decrement current_sum byarr[start]
11. Increment start pointer
12. End of array reached or novalid subarray found
13. Return min_length

