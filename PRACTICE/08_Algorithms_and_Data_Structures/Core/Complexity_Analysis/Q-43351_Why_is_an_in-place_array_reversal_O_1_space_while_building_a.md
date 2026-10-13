---
type: "practice"
difficulty: "Basic"
---

# Why is an in-place array reversal O(1) space while building a new reversed array is O(n) space?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Streaming vs In-memory Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Streaming_vs_In-memory_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this explanation, we will delve into why reversing an array in place results in (O(1)) space complexity, whereas creating a new reversed array incurs (O(n)) space complexity.

The key difference lies in how the operations manipulate memory:

In-place Reversal: This operation manipulates the original array without allocating any additional storage. Instead, it uses a small amount of extra space for temporary variables (typically 2 variables).
Building a New Array: Every element from the original array is copied to a new array, which requires linear space proportional to the size of the input.

Let's break down each case:

#### Triage
Understanding the Question

The question is asking us to explain why in-place reversal has constant space complexity ((O(1))), while building a new reversed array requires linear space complexity ((O(n))). This involves understanding how memory allocation and manipulation differ between these two operations.

Step-by-Step Solution
In-place Reversal

Concept: The goal is to reverse the elements of an array without using additional storage, except for a few temporary variables. Here’s how it works:

Initialize Pointers: Set up two pointers at opposite ends of the array (start and end).
Swap Elements: Swap the elements pointed by these two pointers.
Move Pointers: Increment the start pointer and decrement the end pointer, continuing until they meet or cross.

This process does not allocate any new arrays or significant additional memory, thus maintaining constant space complexity (O(1)).

Building a New Reversed Array

Concept: To create a new array that is the reverse of the original, you need to iterate over each element and place it in its correct position in the new array.

Initialize New Array: Create an empty array with the same length as the input array.
Copy Elements: Iterate through the original array from start to end, copying each element into the corresponding position in the new array, but starting from the end of the new array and moving backward.

Since this operation involves creating a new array, it requires additional space proportional to the size of the input array (n), resulting in linear space complexity (O(n)).

#### Closing Summary

In summary, in-place reversal operates on the existing memory layout without requiring any extra storage beyond constant variables, making its space complexity (O(1)). Conversely, building a new reversed array involves creating an entirely separate array, which leads to (O(n)) space usage.

#### Common Mistakes
and Follow-ups
Common Mistake: Confusing in-place reversal with other operations that might seem like they use no extra memory but actually do (e.g., using stack or recursion).

#### Follow-up
Interviewers might ask about optimizing the reversal process further, such as by minimizing the number of swaps or exploring parallel processing techniques.

**Comparison:**

```
Operation	Space Complexity
In-place Reversal	O(1)
New Reversed Array	O(n)
```

**Code:**

```python
# In-place array reversal in Python

def reverse_in_place(arr):
    start = 0
    end = len(arr) - 1
    while start < end:
        arr[start], arr[end] = arr[end], arr[start]
        start += 1
        end -= 1
```

**Example input / output:**

```text
Input:
- Original Array: `[1, 2, 3, 4, 5]`

Output (in-place):
- Reversed Array: `[5, 4, 3, 2, 1]`

Explanation:
- No additional array is created.
- Only a few temporary variables are used to swap elements.

Input:
- Original Array: `[1, 2, 3, 4, 5]`

Output (new reversed array):
- Reversed Array: `[5, 4, 3, 2, 1]`

Explanation:
- A new array is created.
- Each element from the original array is copied to its correct position in the new array.
```

**Data:**

```json
{
  "array": [1, 2, 3, 4, 5],
  "reversed_array_in_place": [5, 4, 3, 2, 1]
}
```

**Flow:**

1. Original Array
2. Start of Loop?
3. Swap Elements
4. Increment/DecrementPointers?
5. End of Reversal
6. New Array
7. Copy Each Element
8. Place in Correct Position
9. Move to Next Index
10. End of Loop?

