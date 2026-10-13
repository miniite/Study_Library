---
type: "practice"
difficulty: "Intermediate"
---

# Find the next greater element for each item in an array using a monotonic stack.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array In-place Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array_In-place_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding the next greater element for each item in an array, we can use a technique involving a monotonic stack to efficiently track and update our results. This approach ensures that we process each element only once and maintain a stack of elements for which we have yet to find their next greater element.

#### Triage

The question is asking us to implement a function nextGreaterElement that takes an array as input and returns another array where the value at each index (i) in the output array represents the first element in the original array to the right of (A[i]) which is larger than (A[i]). If no such element exists, we set the output value to -1. The interviewer will be probing our understanding of stack usage and efficient algorithm design.

#### First-Principles Solution

We can solve this problem using a single pass through the array with a monotonic stack that keeps track of elements for which the next greater element has not yet been found. Here's how we proceed:

Initialize an empty stack to keep track of indices and a result array filled with -1, as default values.
Iterate over the input array from left to right:
For each element (A[i]), while the stack is not empty and the top of the stack has a value less than or equal to (A[i]), pop elements from the stack. These popped elements have their next greater element found, so update the result array at these indices.
Push the current index onto the stack if it hasn't been processed yet.

This ensures that each element is pushed and popped from the stack exactly once, making our solution (O(n)) in both time and space complexity.

#### Implementation

Here's a step-by-step implementation of this approach:

PYTHON
def nextGreaterElement(nums):
    result = [-1] * len(nums)
    stack = []  # This will store indices

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            prev_index = stack.pop()
            result[prev_index] = num
        stack.append(i)

    return result

#### Example

Given the input array nums = [4, 5, 2, 25], we would get the following operations:

Push index 0 (value: 4) to stack.
Push index 1 (value: 5). Stack is now [0, 1].
Pop index 0 from stack. Set result[0] = 5.
Push index 2 (value: 2). Stack is now [1, 2].
Pop index 2 from stack. Set result[2] = 25.
Push index 3 (value: 25) to stack.

Final output would be: [5, -1, 25, -1].

#### Closing Summary

By using a monotonic stack, we can efficiently find the next greater element for each item in an array. The solution iterates through the array once and uses a stack to keep track of indices that need processing, ensuring linear time complexity.

#### Common Mistakes
& Follow-Ups
Stack Usage: Ensure you're correctly managing the push and pop operations on the stack.
Index Handling: Pay attention to how indices are managed in the result array.
Edge Cases: Consider what happens with monotonically increasing or decreasing sequences, or when all elements are equal.

**Code:**

```python
def nextGreaterElement(nums):
    result = [-1] * len(nums)
    stack = []

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            prev_index = stack.pop()
            result[prev_index] = num
        stack.append(i)

    return result
```

**Data:**

```python
nums = [4, 5, 2, 25]
result = nextGreaterElement(nums)
print(result)  # Output: [5, -1, 25, -1]
```

