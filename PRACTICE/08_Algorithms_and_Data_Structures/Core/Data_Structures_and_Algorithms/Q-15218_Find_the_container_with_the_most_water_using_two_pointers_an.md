---
type: "practice"
difficulty: "Deep"
---

# Find the container with the most water using two pointers, and justify why moving the shorter side is correct.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Two Pointers](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Two_Pointers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the "Container With Most Water" problem, you'll need to understand how to maximize the volume of water that can be contained between two vertical lines on a graph where the x-axis represents the distance and the y-axis represents the height of each line. The key is using a two-pointer approach: one pointer starts at the beginning (left) and the other at the end (right). By comparing the heights at both pointers, you decide which pointer to move inward based on the principle that moving the shorter side can potentially increase the width, thus maximizing the water volume.

This problem primarily tests your understanding of algorithm efficiency, optimization techniques, and logical reasoning. We'll start by defining the two-pointer approach and then delve into the logic behind why this method works best for solving this challenge.

**Comparison:**

```
Heights	Expected Output
[1,8,6,2,5,4,8,3,7]	49
[1,1]	1
[4,3,2,1,4]	16
[1,2,1]	2
```

**Code:**

```python
def max_area(height):
    left, right = 0, len(height) - 1
    max_area = 0

    while left < right:
        # Calculate current area
        current_area = min(height[left], height[right]) * (right - left)

        # Update max_area if current_area is greater
        max_area = max(max_area, current_area)

        # Move the pointer that points to the shorter line inward
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return max_area
```

**Example input / output:**

```text
For the input `[1,8,6,2,5,4,8,3,7]`:
- Initial state: `left=0`, `right=8`.
- First iteration: `current_area = min(1, 7) * (9 - 1) = 1 * 8 = 8`. Move the left pointer to `1` since `height[left] < height[right]`.
- Second iteration: `current_area = min(8, 7) * (8 - 2) = 7 * 6 = 42`. Stay at these pointers as moving either would not increase the area.
- Final result: `max_area = 49`.

For the input `[1,1]`:
- Initial state: `left=0`, `right=1`.
- First iteration: `current_area = min(1, 1) * (2 - 1) = 1 * 1 = 1`. Stay at these pointers as both heights are equal.
- Final result: `max_area = 1`.

For the input `[4,3,2,1,4]`:
- Initial state: `left=0`, `right=4`.
- First iteration: `current_area = min(4, 4) * (5 - 1) = 4 * 4 = 16`. Stay at these pointers as moving either would not increase the area.
- Final result: `max_area = 16`.

For the input `[1,2,1]`:
- Initial state: `left=0`, `right=2`.
- First iteration: `current_area = min(1, 1) * (3 - 1) = 1 * 2 = 2`. Move the left pointer to `1` since `height[left] < height[right]`.
- Final result: `max_area = 2`.
```

**Data:**

```json
[
  {"heights": [1,8,6,2,5,4,8,3,7], "expected_output": 49},
  {"heights": [1,1], "expected_output": 1},
  {"heights": [4,3,2,1,4], "expected_output": 16},
  {"heights": [1,2,1], "expected_output": 2}
]
```

**Flow:**

1. Initialize left and right pointersat start and end
2. Calculate current area
3. Update max_area if larger thanprevious
4. Move the shorter pointerinward
5. Repeat until pointers meet
6. Return max_area

