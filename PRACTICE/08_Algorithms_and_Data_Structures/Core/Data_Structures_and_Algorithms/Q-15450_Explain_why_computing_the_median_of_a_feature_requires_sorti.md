---
type: "practice"
difficulty: "Basic"
---

# Explain why computing the median of a feature requires sorting or a selection algorithm, and what the complexity of each is.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Complexity of Encodings](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Complexity_of_Encodings.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why computing the median of a feature often involves sorting or using a selection algorithm, we need to first comprehend what the median represents and then explore the different methods to find it efficiently.

TL;DR:

Computing the median requires sorting because the median is the middle value when all values are arranged in order. Alternatively, you can use a selection algorithm like Quickselect, which has an average time complexity of (O(n)), making it efficient for large datasets.

#### Triaging the Question

The question essentially asks why we can't simply compute the median directly without sorting or using some other specialized method, and what the complexities are associated with these methods. The interviewer is likely probing your understanding of basic data structures (like arrays) and algorithms (such as sorting and selection).

#### First-Principles Solution
Median Definition

The median of a set of numbers is the middle value when they are sorted in increasing order. For an odd number of elements, it's straightforward: you simply find the element at index (\left\lfloor \frac{n}{2} \right\rfloor) (integer division). However, for even numbers of elements, we need to average the two middle values.

Why Sorting?

To directly access the median value without sorting would require knowing exactly where it is positioned in the sorted array. Without a sorted order, this position is unknown because:

In an unsorted list, there's no inherent ordering that allows us to jump straight to the middle.
The element at index (\left\lfloor \frac{n}{2} \right\rfloor) would not necessarily be the median.
Complexity of Sorting

The most efficient general-purpose sorting algorithms have a time complexity of (O(n \log n)), such as Merge Sort, QuickSort, or Heap Sort. While this is not trivial for large datasets, it's still more computationally expensive than linear-time algorithms like selection methods.

Selection Algorithms (Like Quickselect)

Quickselect is a specialized algorithm that can find the k-th smallest element in an unordered list, and thus can be used to find the median directly. The key idea here is that we only need to partially sort the array until the desired position is reached. If (k = \left\lfloor \frac{n}{2} \right\rfloor) for even (n), Quickselect can achieve this with an average time complexity of (O(n)).

#### Example

Let's say you have a dataset A = [3, 7, 1, 9, 5]. Without sorting:

You cannot directly determine that the median is 5.
After sorting, A_sorted = [1, 3, 5, 7, 9], and we see that the median is 5 at index 2.

#### Common Mistakes
Forgetting to consider edge cases like even-sized datasets.
Assuming a direct computation method without sorting or selection exists for finding the median efficiently.
Overlooking the (O(n \log n)) complexity of general-purpose sort algorithms versus the linear average time complexity of Quickselect.

#### Closing Summary
and Follow-ups

In summary, computing the median requires either full sorting (with (O(n \log n)) complexity) or a selection algorithm like Quickselect (with an average (O(n)) complexity). Understanding these complexities helps in choosing appropriate methods based on dataset size and performance requirements. Common pitfalls include misjudging the necessity of sorting or overestimating the efficiency of direct computation.

Final Check

This answer covers the core concepts, provides examples, and highlights key differences between sorting and selection algorithms for finding medians.

**Comparison:**

```
Dataset	Sorted Dataset	Median
[3, 7, 1, 9, 5]	[1, 3, 5, 7, 9]	5
[3, 2, 1, 4, 5, 6]	[1, 2, 3, 4, 5, 6]	3.5
```

**Code:**

```python
# Example Quickselect implementation in Python
import random

def quickselect(arr, k):
    if len(arr) == 1:
        return arr[0]

    pivot = random.choice(arr)
    lows = [el for el in arr if el < pivot]
    highs = [el for el in arr if el > pivot]
    pivots = [el for el in arr if el == pivot]

    if k < len(lows):
        return quickselect(lows, k)
    elif k < len(lows) + len(pivots):
        # We found the k-th smallest element
        return pivots[0]
    else:
        return quickselect(highs, k - len(lows) - len(pivots))

# Example usage for finding median of an array
arr = [3, 7, 1, 9, 5]
n = len(arr)
median_index = n // 2

if n % 2 == 0:
    print("Median is:", (quickselect(arr, median_index - 1) + quickselect(arr, median_index)) / 2)
else:
    print("Median is:", quickselect(arr, median_index))
```

**Example input / output:**

```text
Input: `[3, 7, 1, 9, 5]`

Output (for median): `5` (when `n=5`, no need to average two middle values)

Input: `[3, 2, 1, 4, 5, 6]`

Output (for median): `(2 + 3) / 2 = 2.5`
```

**Data:**

```json
{
    "data": [3, 7, 1, 9, 5]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Input Array
6. Choose Pivot
7. Is k < len(Low)?
8. Return Low[k]
9. Is k < len(Low) + len(Pivot)?
10. Return Pivot[0]
11. High = [x for x in A if x >Pivott]; Quickselect(Heights, k -len(Low) - len(Pivot))

