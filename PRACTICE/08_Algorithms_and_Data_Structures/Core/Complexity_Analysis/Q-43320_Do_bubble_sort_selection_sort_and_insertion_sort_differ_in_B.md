---
type: "practice"
difficulty: "Basic"
---

# Do bubble sort, selection sort, and insertion sort differ in Big-O, and why might one still be preferred?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Sorting Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of whether Bubble Sort, Selection Sort, and Insertion Sort differ in their Big-O complexities, we first need to understand what these algorithms do and then analyze their time and space complexity.

TL;DR:
Bubble Sort has a worst-case and average time complexity of (O(n^2)).
Selection Sort also has a worst-case and average time complexity of (O(n^2)), but it performs fewer swaps.
Insertion Sort has a best-case time complexity of (O(n)) and a worst-case and average time complexity of (O(n^2)).

Despite similar Big-O complexities, these algorithms can be preferred based on specific scenarios like the size of the input array, whether the array is nearly sorted, or the need for minimal swaps.

#### Triaging the Question

The question probes your understanding of sorting algorithm analysis. The key aspects are:

Understanding the time and space complexity of each sort.
Recognizing when one might be preferred over another based on specific conditions.
Step-by-Step Solution
Bubble Sort

Bubble Sort works by repeatedly swapping the adjacent elements if they are in wrong order. Here's a basic implementation:

PYTHON
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        # If no two elements were swapped by inner loop, then break
        if not swapped:
            break

Time Complexity:

Best Case: (O(n)) when the array is already sorted.
Average and Worst Case: (O(n^2)).

Space Complexity: (O(1)), as it's an in-place algorithm.

Selection Sort

Selection Sort works by dividing the input list into two parts: the sublist of items already sorted, which is built up from left to right at the front (left) of the list, and the sublist of items remaining to be sorted that occupy the rest of the list. Initially, the sorted sublist is empty and the unsorted sublist is the entire input list.

PYTHON
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i+1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]

Time Complexity:

Best Case: (O(n^2)).
Average and Worst Case: (O(n^2)).

Space Complexity: (O(1)), as it's also an in-place algorithm.

Insertion Sort

Insertion Sort works by building a sorted array one element at a time. It is much like the way you sort playing cards in your hands. For each iteration, we insert the first unsorted element into its correct position within the already-sorted portion of the list.

PYTHON
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i-1
        while j >=0 and key < arr[j] :
                arr[j+1] = arr[j]
                j -= 1
        arr[j+1] = key

Time Complexity:

Best Case: (O(n)) when the array is already sorted.
Average and Worst Case: (O(n^2)).

Space Complexity: (O(1)), as it's an in-place algorithm.

Why One Might Be Preferred

Bubble Sort: It might be preferred for small datasets or nearly sorted arrays due to its simpler implementation. However, it is generally not used for larger datasets.

Selection Sort: This can be useful when the number of swaps is a critical factor, as it performs fewer swaps than Bubble Sort.

Insertion Sort: Ideal for nearly sorted datasets and small input sizes because of its best-case linear time complexity. It’s also simple and efficient for small arrays but becomes less effective for larger ones.

#### Closing Summary
+ Common Mistakes/Follow-Ups

All three algorithms have similar Big-O complexities, making them suitable for the same general use cases. However, their performance characteristics (e.g., number of swaps) can make one more appropriate than another in specific scenarios.

Mistake: Failing to recognize that while all three algorithms are (O(n^2)), they differ in how many comparisons and swaps are required.

#### Follow-up
Discussing when each might be preferred, such as for nearly sorted data or small arrays.

**Code:**

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        if not swapped:
            break

def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i+1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]

def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i-1
        while j >=0 and key < arr[j] :
                arr[j+1] = arr[j]
                j -= 1
        arr[j+1] = key
```

**Example input / output:**

```text
Input:
arr = [64, 34, 25, 12, 22, 11, 90]

Bubble Sort Output:
[11, 12, 22, 25, 34, 64, 90]

Selection Sort Output:
[11, 12, 22, 25, 34, 64, 90]

Insertion Sort Output:
[11, 12, 22, 25, 34, 64, 90]
```

