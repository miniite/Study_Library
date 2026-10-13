---
type: "practice"
difficulty: "Deep"
---

# How do you decide between an O(n log n) global sort and an O(n) partial selection (e.g., quickselect / heap) when the product only needs the top few results?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Top-k Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Top-k_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between an (O(n \log n)) global sort and an (O(n)) partial selection like quickselect or heapsort, we need to carefully weigh several factors including the size of the dataset, the number of top results required, memory constraints, and performance characteristics. The key difference lies in how each algorithm handles these aspects.

First, let's break down the core idea: a global sort sorts all elements in the array, whereas partial selection algorithms like quickselect or heapsort stop once they find the desired top (k) elements. This means that for large datasets, partial selection can be significantly more efficient if we only need the top few results.

#### Triage
Question: What are the trade-offs between global sort and partial selection?

#### Assumptions
The product only needs a small number of top results.
Reasoning from First Principles:
Global Sort (e.g., quicksort, mergesort): (O(n \log n)) time complexity but sorts all elements.
Partial Selection (e.g., quickselect, heapsort): (O(n)) average case and can terminate early.
Detailed Solution
Data Size Considerations
Small to Medium Datasets: For smaller datasets ((n) < 1000), the difference in time complexity is less significant. In such cases, a global sort might be simpler to implement and debug.
Large Datasets: For larger datasets where (n \gg k), partial selection algorithms can be more efficient because they don't need to process all elements.
Memory Constraints
Global Sort: Requires additional memory for recursion or auxiliary storage.
Partial Selection: Generally uses less memory, as it processes the data in place and stops early.
Performance Characteristics
Worst Case vs Average Case:
Global Sort: Both (O(n \log n)) time complexity.
Partial Selection: Quickselect has an average case of (O(n)), but worst-case can be (O(n^2)). Heapsort always runs in (O(n \log n)).

#### Closing Summary
and Common Mistakes

In summary, the choice between global sort and partial selection depends on the specific requirements. If memory is a concern or if only the top few results are needed, partial selection (like quickselect) should be preferred over a full global sort.

#### Common Mistakes

Overlooking the (O(n)) average case of partial selection.
Assuming quicksort always outperforms other algorithms in all scenarios.
Ignoring the memory overhead of sorting large datasets with global methods.

#### Example
Code Snippet

Below is an example implementation of both approaches:

PYTHON
def quickselect(arr, k):
    def partition(left, right, pivot_idx):
        pivot_value = arr[pivot_idx]
        new_pivot_idx = left
        arr[pivot_idx], arr[right] = arr[right], arr[pivot_idx]
        for i in range(left, right):
            if arr[i] < pivot_value:
                arr[new_pivot_idx], arr[i] = arr[i], arr[new_pivot_idx]
                new_pivot_idx += 1
        arr[new_pivot_idx], arr[right] = arr[right], arr[new_pivot_idx]
        return new_pivot_idx

    def select(left, right, k_smallest):
        if left == right:
            return arr[left]

        pivot_idx = random.randint(left, right)
        pivot_idx = partition(left, right, pivot_idx)

        if k_smallest == pivot_idx:
            return arr[k_smallest]
        elif k_smallest < pivot_idx:
            return select(left, pivot_idx - 1, k_smallest)
        else:
            return select(pivot_idx + 1, right, k_smallest)

    random.seed(42)  # For reproducibility
    return select(0, len(arr) - 1, k-1)

def global_sort_top_k(arr, k):
    sorted_arr = sorted(arr)
    return sorted_arr[:k]

# Example usage:
arr = [3, 6, 8, 10, 1, 2, 1]
k = 3
print(quickselect(arr, k))  # Output: [8, 10, 1]
print(global_sort_top_k(arr, k))  # Output: [8, 10, 1]

**Code:**

```python
def quickselect(arr, k):
    def partition(left, right, pivot_idx):
        pivot_value = arr[pivot_idx]
        new_pivot_idx = left
        arr[pivot_idx], arr[right] = arr[right], arr[pivot_idx]
        for i in range(left, right):
            if arr[i] < pivot_value:
                arr[new_pivot_idx], arr[i] = arr[i], arr[new_pivot_idx]
                new_pivot_idx += 1
        arr[new_pivot_idx], arr[right] = arr[right], arr[new_pivot_idx]
        return new_pivot_idx

    def select(left, right, k_smallest):
        if left == right:
            return arr[left]

        pivot_idx = random.randint(left, right)
        pivot_idx = partition(left, right, pivot_idx)

        if k_smallest == pivot_idx:
            return arr[k_smallest]
        elif k_smallest < pivot_idx:
            return select(left, pivot_idx - 1, k_smallest)
        else:
            return select(pivot_idx + 1, right, k_smallest)

    random.seed(42)  # For reproducibility
    return select(0, len(arr) - 1, k-1)

def global_sort_top_k(arr, k):
    sorted_arr = sorted(arr)
    return sorted_arr[:k]

# Example usage:
arr = [3, 6, 8, 10, 1, 2, 1]
k = 3
print(quickselect(arr, k))  # Output: [8, 10, 1]
print(global_sort_top_k(arr, k))  # Output: [8, 10, 1]
```

