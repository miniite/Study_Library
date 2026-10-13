---
type: "practice"
difficulty: "Intermediate"
---

# Find the peak element in an array where a peak is greater than its neighbors, in O(log n).

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ O(1) Randomized Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/O(1)_Randomized_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find a peak element in an array with the constraint of (O(\log n)) time complexity, we can leverage binary search. The key idea here is to compare elements in a way that reduces the search space efficiently, similar to how we would perform binary search on a sorted list.

The question essentially asks us to find an index i such that the element at this index (arr[i]) is greater than its neighbors: arr[i-1] < arr[i] > arr[i+1]. The constraints allow for logarithmic complexity, suggesting we use binary search. We will make comparisons in a way that narrows down our search space by half with each iteration.

#### Triage

The problem is asking us to find a peak element using an (O(\log n)) algorithm. To achieve this, we need to understand how binary search can be adapted for this scenario. Key assumptions include:

The array has at least one peak.
The boundaries of the array are valid (i.e., arr[0] and arr[n-1] are treated as non-peak elements).

#### First-Principles Solution

Initialization: Start with indices low = 0 and high = n - 1.

Binary Search Loop:

Calculate the midpoint mid = (low + high) // 2.
Compare arr[mid] with its neighbors.
If arr[mid-1] < arr[mid] > arr[mid+1], then arr[mid] is a peak, return it.
Otherwise, adjust the search space:
If arr[mid] < arr[mid+1], move to the right half by setting low = mid + 1.
Otherwise, move to the left half by setting high = mid.

Termination: When low equals high, the loop terminates, and that index points to a peak element.

#### Closing Summary

By using binary search, we reduce the problem size logarithmically, ensuring an (O(\log n)) time complexity. This approach is efficient and leverages the properties of the array to quickly find a peak element without needing to compare every single element.

#### Common Mistakes/Follow-Ups
Off-by-One Errors: Be careful with indices when calculating mid and handling boundaries.
Edge Cases: Consider what happens if there are multiple peaks, or if the array is strictly increasing or decreasing (these should still have a peak by definition).
Comparison Validity: Ensure that all comparisons are valid; for example, avoid out-of-bounds errors.

