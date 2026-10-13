---
type: "practice"
difficulty: "Basic"
---

# Sort an array containing only 0s, 1s, and 2s in a single pass (Dutch national flag).

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Counting and Radix Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Counting_and_Radix_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the Dutch National Flag problem, we need to sort an array consisting of three distinct values—typically 0s, 1s, and 2s—in a single pass without using extra space. The goal is to achieve linear time complexity (O(n)) and constant space complexity (O(1)). This approach leverages a three-way partitioning technique.

#### Triage
What the Question Is Asking

The Dutch National Flag problem asks for an in-place sorting algorithm that can handle arrays with three unique values. The key is to perform this in a single pass, ensuring efficient runtime and minimal space usage. The interviewer might be probing your understanding of array manipulation techniques, partitioning strategies, and optimization principles.

#### First-Principles Solution

Let's break down the solution step by step:

Initialization: We need three pointers: low to track the boundary for 0s, high to mark the boundary for 2s, and a current pointer i that starts at the beginning of the array.
Traversal: As we traverse the array with the current pointer:
If the value is 0, swap it with the element at low, increment both low and current.
If the value is 2, swap it with the element at high, decrement high. The current element remains in place.
If the value is 1, simply move to the next element by incrementing current.

This approach ensures that all 0s are moved to the start of the array and all 2s are pushed towards the end. By maintaining these boundaries, we achieve a single pass through the array.

Detailed Explanation

Let's illustrate this with an example:

Initial Array: [0, 1, 2, 0, 1, 2]
Pointer Initialization:
low = 0, current = 0, high = n-1 (where n is the length of the array).
Step-by-step Process:
Start with i = 0. The value at index 0 is 0, so swap it with the element at low and increment both:
Array: [0, 1, 2, 0, 1, 2], low = 1, current = 1.
Next, i = 1. The value is 1, so move to the next element.
i = 2. The value is 2, swap it with the element at high and decrement high:
Array: [0, 1, 2, 0, 1, 2], high = n-2.
Continue this process until current reaches high.

By the end of the traversal, all 0s will be at the start, and all 2s will be towards the end.

#### Closing Summary

The Dutch National Flag problem is a classic example of in-place partitioning that can be solved with three pointers. This technique ensures linear time complexity and constant space usage, making it highly efficient for large arrays.

#### Common Mistakes/Follow-ups
Mistake: Failing to handle the case where current meets high, which can lead to redundant operations.

#### Follow-up
Discussing how this algorithm can be extended or modified for different problems, such as sorting an array with more than three unique values.

**Comparison:**

```
Step	Action	Array
0	Initial	[0, 1, 2, 0, 1, 2]
1	Swap	[0, 1, 2, 0, 1, 2], low = 1, mid = 1, high = 5
2	Skip	[0, 1, 2, 0, 1, 2]
3	Swap	[0, 1, 2, 0, 1, 2], low = 2, mid = 2, high = 5
...	Repeat	...
```

**Code:**

```python
def sort_array(nums):
    low = 0
    mid = 0
    high = len(nums) - 1

    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 2:
            nums[high], nums[mid] = nums[mid], nums[high]
            high -= 1
        else:  # nums[mid] == 1
            mid += 1

    return nums
```

**Example input / output:**

```text
- **Input**: `[0, 1, 2, 0, 1, 2]`
- **Output**: `[0, 0, 1, 1, 2, 2]`
```

**Data:**

```json
{
    "input": [0, 1, 2, 0, 1, 2],
    "output": [0, 0, 1, 1, 2, 2]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. No
6. Yes
7. Start
8. Is current == high?
9. Is nums[current] == 0?
10. Swap low and current,increment low and mid
11. Is nums[current] == 2?
12. Swap high and current,decrement high
13. Increment current
14. End

