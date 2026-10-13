---
type: "practice"
difficulty: "Basic"
---

# Rotate an array to the right by k positions in place using the reversal trick.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interval Merging](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Interval_Merging.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To rotate an array to the right by (k) positions in place, we can leverage a clever algorithm that uses reversal operations. This approach ensures that we minimize extra space usage and achieve a time complexity of (O(n)), where (n) is the number of elements in the array.

#### Triage
What's Being Probed

The question asks us to rotate an array by (k) positions to the right in place, using the reversal trick. The interviewer will be testing your understanding of:

Array manipulation algorithms: Specifically, how to reverse parts of the array.
In-place operations: Ensuring no additional space is used beyond a few variables.
Edge cases and optimization: Handling scenarios like when (k) is greater than the length of the array.

#### First-Principles Solution

Calculate Effective Shifts:

If (k \geq n), we can reduce it to (k = k \mod n) because rotating the entire array by its size results in no change.

Reverse Sections:

Reverse the first part of the array from index 0 to (n - k - 1).
Reverse the second part of the array from index (n - k) to (n - 1).
Finally, reverse the entire array.

#### Example
Walkthrough

Let's walk through an example with an array ([1, 2, 3, 4, 5]) and (k = 2).

Initial Array: ([1, 2, 3, 4, 5])
Effective Shifts Calculation:
Since (k = 2 < n) (where (n = 5)), we don't need to reduce it further.
Reverse First Section (([0...2])):
Result: ([3, 2, 1, 4, 5])
Reverse Second Section (([3...4])):
Result: ([3, 2, 5, 4, 1])
Reverse Entire Array:
Result: ([1, 4, 5, 2, 3])

#### Closing Summary
+ Common Mistakes
Ensure (k) is within bounds: Always calculate (k = k \mod n).
Carefully track indices during reversals: Be mindful of which parts to reverse and in what order.
Handle edge cases: When (k) equals the length of the array, simply return the original array as no change will occur.

#### Common Mistakes
Forgetting to reduce (k): Not calculating (k = k \mod n) can lead to unnecessary work or incorrect results.
Improper reversal indices: Misunderstanding which parts need to be reversed and in what order.
Failing edge cases: Not handling scenarios where the array length is small, leading to off-by-one errors.

**Code:**

```python
def rotate(nums, k):
    n = len(nums)
    k %= n  # Ensure k is within bounds

    # Reverse first part of the array
    reverse(nums, 0, n - k - 1)

    # Reverse second part of the array
    reverse(nums, n - k, n - 1)

    # Reverse entire array
    reverse(nums, 0, n - 1)

def reverse(arr, start, end):
    while start < end:
        arr[start], arr[end] = arr[end], arr[start]
        start += 1
        end -= 1

# Example usage
nums = [1, 2, 3, 4, 5]
rotate(nums, 2)
print(nums)  # Output: [1, 4, 5, 2, 3]
```

**Example input / output:**

```text
Input:
nums = [1, 2, 3, 4, 5]
k = 2

Output:
[1, 4, 5, 2, 3]
```

