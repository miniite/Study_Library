---
type: "practice"
difficulty: "Deep"
---

# Compute how much rainwater can be trapped between bars of varying heights.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Histogram and Stack Problems](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Histogram_and_Stack_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of computing how much rainwater can be trapped between bars of varying heights, we need to determine the volume of water that can be stored above each bar in a histogram-like structure. The key idea is to find the maximum height on either side for every bar and then calculate the difference between the minimum of these maxima and the current bar's height.

#### Triage

The question asks us to implement an algorithm to compute the trapped rainwater given an array of non-negative integers representing the heights of bars. The interviewer might be probing your understanding of space and time complexity, as well as your ability to break down a complex problem into simpler components.

#### First-Principles Solution

We can use two approaches: a brute-force approach with O(n^2) time complexity or an optimal solution with O(n) time complexity. The optimal approach involves precomputing the maximum heights from both left and right ends of the array, then using these values to calculate the trapped water at each position.

Precompute Maximum Heights:

Traverse the array from left to right to compute the maximum height seen so far.
Traverse the array from right to left to compute the maximum height seen so far.

Calculate Trapped Water:

For each bar, calculate the trapped water by taking the minimum of the precomputed maximum heights on both sides and subtracting the current bar's height.

#### Example

Suppose we have an array heights = [0,1,0,2,1,0,1,3,2,1,2,1].

Left Max Heights: [0, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3, 3]
Right Max Heights: [3, 3, 3, 3, 3, 3, 3, 3, 2, 2, 2, 1]

Now, for each bar, we calculate the trapped water:

Bar at index 0: min(0, 3) - 0 = 0
Bar at index 1: min(1, 3) - 1 = 0
Bar at index 2: min(1, 3) - 0 = 1
And so on...

Finally, sum these values to get the total trapped water.

#### Closing Summary

The optimal solution uses O(n) time complexity and O(n) space complexity. It efficiently calculates the trapped water by leveraging precomputed maximum heights from both sides.

#### Common Mistakes/Follow-Ups
Mistake 1: Not correctly computing the left or right max heights.
Mistake 2: Failing to handle edge cases where the input array is empty or has only one element.

#### Follow-Up
Discussing trade-offs between space and time complexity, such as using a single pass with O(1) additional space.

**Comparison:**

| Index | Heights | Left Max | Right Max | Trapped Water |
|-------|---------|----------|-----------|---------------|
| 0     | 0       | 0        | 3         | 0             |
| 1     | 1       | 1        | 3         | 0             |
| 2     | 0       | 1        | 3         | 1             |
| 3     | 2       | 2        | 3         | 0             |
| 4     | 1       | 2        | 3         | 0             |
| 5     | 0       | 2        | 3         | 2             |
| 6     | 1       | 2        | 3         | 1             |
| 7     | 3       | 3        | 3         | 0             |
| 8     | 2       | 3        | 2         | 0             |
| 9     | 1       | 3        | 2         | 1             |
| 10    | 2       | 3        | 2         | 1             |
| 11    | 1       | 3        | 1         | 0             |

**Code:**

```python
def trap(height):
    if not height:
        return 0

    n = len(height)
    left_max = [0] * n
    right_max = [0] * n
    water_trapped = 0

    # Fill left_max array
    left_max[0] = height[0]
    for i in range(1, n):
        left_max[i] = max(left_max[i-1], height[i])

    # Fill right_max array
    right_max[n-1] = height[n-1]
    for i in range(n-2, -1, -1):
        right_max[i] = max(right_max[i+1], height[i])

    # Calculate trapped water
    for i in range(1, n-1):
        water_trapped += min(left_max[i], right_max[i]) - height[i]

    return water_trapped
```

**Example input / output:**

```text
Given the input array `heights = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]`, the function would return `6` as the total trapped water.
```

**Data:**

```json
{
  "heights": [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the array empty?
5. Return 0
6. Initialize left_max andright_max arrays
7. Fill left_max array from left toright
8. For each index i, set left_max[i]= max(left_max[i-1], height[i])
9. Fill right_max array from rightto left
10. For each index i, setright_max[i] =max(right_max[i+1], height[i])
11. Calculate trapped water foreach bar
12. For each index i (1 to n-2), addmin(left_max[i], right_max[i]) -height[i] to total_water
13. Return total_water

