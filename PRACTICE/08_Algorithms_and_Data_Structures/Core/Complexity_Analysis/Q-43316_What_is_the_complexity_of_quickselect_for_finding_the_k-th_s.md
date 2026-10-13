---
type: "practice"
difficulty: "Intermediate"
---

# What is the complexity of quickselect for finding the k-th smallest element, and what's its worst case?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Top-k Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Top-k_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Quickselect is a selection algorithm that can be used to find the k-th smallest element in an unordered list. Its average time complexity is ( O(n) ), but it has a worst-case time complexity of ( O(n^2) ). Let's dive into the details step by step.

#### Triage

The question asks for the time and space complexities of quickselect, specifically focusing on its worst-case scenario. Quickselect uses the partitioning method similar to quicksort but only recurses into one side, depending on whether ( k ) is less than or greater than the pivot index. The key idea here is understanding how the pivot choice affects performance.

#### First-Principles Solution

Understanding Quickselect:

Quickselect works by partitioning the array around a chosen pivot such that elements to its left are smaller, and those to its right are larger.
If ( k ) is equal to the pivot index after partitioning, we have found our target element; otherwise, we recursively apply quickselect on the relevant side.

Average Case Analysis:

On average, with random pivots, each recursive call reduces the problem size by about half.
This leads to an expected time complexity of ( O(n) ), as it is similar to a well-balanced binary search tree.

Worst-Case Analysis:

The worst-case scenario occurs when the chosen pivot always divides the array into subarrays where one has 1 element and the other has ( n-2 ) elements.
This happens if we consistently choose the smallest or largest element as the pivot, leading to a partition where ( k ) is never in the desired half.
In this case, each recursive call reduces the problem size by only 1 element, resulting in a worst-case time complexity of ( O(n^2) ).

#### Closing Summary
and Common Mistakes
Key Takeaways: Quickselect's average performance is linear, but it can degrade to quadratic if poor pivot choices are made.
Common Pitfalls: Forgetting the worst-case analysis or assuming quickselect will always perform well due to its average-case behavior. Always consider edge cases like consistently picking the smallest/largest element as the pivot.
Code Example
PYTHON
def quickselect(arr, left, right, k):
    if left == right:
        return arr[left]

    pivot_index = partition(arr, left, right)

    if k == pivot_index:
        return arr[k]
    elif k < pivot_index:
        return quickselect(arr, left, pivot_index - 1, k)
    else:
        return quickselect(arr, pivot_index + 1, right, k)

def partition(arr, left, right):
    pivot = arr[right]
    i = left
    for j in range(left, right):
        if arr[j] <= pivot:
            arr[i], arr[j] = arr[j], arr[i]
            i += 1
    arr[i], arr[right] = arr[right], arr[i]
    return i
Self-Check

The answer is correct and covers all the necessary aspects of quickselect's time complexity.

