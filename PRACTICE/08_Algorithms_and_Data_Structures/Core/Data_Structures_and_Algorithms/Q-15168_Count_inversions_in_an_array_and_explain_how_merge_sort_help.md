---
type: "practice"
difficulty: "Intermediate"
---

# Count inversions in an array and explain how merge sort helps compute them.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Merge Sort and Counting Inversions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Merge_Sort_and_Counting_Inversions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To count the number of inversions in an array, we need to determine the total number of pairs (i, j) such that i < j and arr[i] > arr[j]. A common approach is to use a modified merge sort algorithm, which not only sorts the array but also counts the inversions during the merging process. This method ensures that we can achieve an efficient solution with a time complexity of (O(n \log n)).

The key idea here is understanding how merge sort works and modifying it to count inversions. Let's break down the steps:

Divide: Split the array into two halves.
Conquer: Recursively sort and count inversions in both halves.
Combine: Merge the sorted halves while counting split inversions.

**Code:**

```python
def merge_sort_and_count(arr, temp_arr, left, right):
    inv_count = 0
    if right > left:
        mid = (left + right) // 2

        # Left subarray inversion count
        inv_count += merge_sort_and_count(arr, temp_arr, left, mid)

        # Right subarray inversion count
        inv_count += merge_sort_and_count(arr, temp_arr, mid + 1, right)

        # Merge the sorted halves and count split inversions
        inv_count += merge(arr, temp_arr, left, mid, right)

    return inv_count

def merge(arr, temp_arr, left, mid, right):
    i = left     # Starting index for left subarray
    j = mid + 1  # Starting index for right subarray
    k = left     # Starting index to be sorted

    inv_count = 0

    # Copy data to temp arrays and merge while counting split inversions
    while i <= mid and j <= right:
        if arr[i] <= arr[j]:
            temp_arr[k] = arr[i]
            i += 1
        else:
            temp_arr[k] = arr[j]
            inv_count += (mid - i + 1)  # Count split inversions
            j += 1

        k += 1

    # Copy the remaining elements of left subarray, if any
    while i <= mid:
        temp_arr[k] = arr[i]
        i += 1
        k += 1

    # Copy the remaining elements of right subarray, if any
    while j <= right:
        temp_arr[k] = arr[j]
        j += 1
        k += 1

    # Copy back sorted portion to original array
    for loop_var in range(left, right + 1):
        arr[loop_var] = temp_arr[loop_var]

    return inv_count

def count_inversions(arr):
    n = len(arr)
    temp_arr = [0] * n
    return merge_sort_and_count(arr, temp_arr, 0, n - 1)

# Example usage:
arr = [8, 4, 2, 1]
print("Number of inversions are:", count_inversions(arr))
```

**Example input / output:**

```text
For the array `arr = [8, 4, 2, 1]`:
- The input is `[8, 4, 2, 1]`.
- After processing by `count_inversions`, the output is `"Number of inversions are: 6"`.
```

**Flow:**

1. Yes
2. No
3. Input Array
4. Divide array into two halves
5. Sort and count inversions inleft half
6. Sort and count inversions inright half
7. Merge the sorted halves
8. Are elements from leftsubarray > current element?
9. Count as split inversion, moveto next element in left array
10. Move to next element in rightarray
11. Continue merging andcounting inversions
12. Merge back into original array
13. C1,C2
14. Tally total inversions
15. Return total inversions count

