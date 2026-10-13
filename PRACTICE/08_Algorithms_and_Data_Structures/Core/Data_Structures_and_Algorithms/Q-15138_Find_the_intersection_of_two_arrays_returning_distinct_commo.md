---
type: "practice"
difficulty: "Intermediate"
---

# Find the intersection of two arrays returning distinct common elements, and discuss two approaches.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Combinations](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Combinations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to find the common elements between two arrays while ensuring that each element is unique in the result. There are multiple ways to approach this, but I'll outline two different methods and discuss their trade-offs.

TL;DR
Hash Set Approach: Uses O(n) space complexity and time complexity.
Two-Pointer Approach: Involves sorting and can be more memory-efficient if the arrays are not too large.

#### Triage
Understanding the Question and Assumptions

The question asks us to find common elements between two arrays, but each element in the result should only appear once. We need to consider the constraints such as the size of the arrays and whether they contain duplicates within their own sets.

#### First-Principles Solution
Hash Set Approach
Step 1: Convert one array into a hash set (or dictionary) for O(1) lookups.
Step 2: Iterate through the second array, checking each element against the hash set.
Step 3: If an element is found in both arrays and hasn't been added to the result yet, add it.
PYTHON
def find_intersection_hash_set(arr1, arr2):
    # Convert one array into a hash set for O(1) lookups
    set_arr1 = set(arr1)
    intersection = []

    # Iterate through the second array and check against the set
    for num in arr2:
        if num in set_arr1 and num not in intersection:
            intersection.append(num)

    return intersection

# Example usage
arr1 = [1, 3, 5, 7, 9]
arr2 = [4, 6, 8, 9, 10]
print(find_intersection_hash_set(arr1, arr2))  # Output: [9]
Two-Pointer Approach
Step 1: Sort both arrays.
Step 2: Use two pointers to traverse the sorted arrays and compare elements.
Step 3: If the current elements are equal, add them to the result; otherwise, move the pointer of the smaller element.
PYTHON
def find_intersection_two_pointers(arr1, arr2):
    # Sort both arrays first
    arr1.sort()
    arr2.sort()

    i = j = 0
    intersection = []

    while i < len(arr1) and j < len(arr2):
        if arr1[i] == arr2[j]:
            if not intersection or arr1[i] != intersection[-1]:
                intersection.append(arr1[i])
            i += 1
            j += 1
        elif arr1[i] < arr2[j]:
            i += 1
        else:
            j += 1

    return intersection

# Example usage
arr1 = [1, 3, 5, 7, 9]
arr2 = [4, 6, 8, 9, 10]
print(find_intersection_two_pointers(arr1, arr2))  # Output: [9]

#### Closing Summary
and Common Mistakes/Follow-Ups
Hash Set Approach: This method is straightforward and efficient in terms of time complexity. However, it uses additional space proportional to the size of one array.
Two-Pointer Approach: While this approach is more memory-efficient, it requires sorting the arrays first, which can add overhead if the input sizes are large.

Both approaches have their strengths, but the choice between them depends on specific constraints like available memory and the relative sizes of the input arrays. For small to moderately sized arrays, the hash set method might be preferred due to its simplicity and efficiency. If memory usage is a critical concern and the arrays are not too large, the two-pointer approach could be more suitable.

#### Common Mistakes
Duplicate Elements: Not handling duplicates correctly in the result can lead to incorrect outputs.
Ordering of Results: The order of elements in the result might differ between approaches, so it's important to clarify whether ordered or unordered results are needed.
Edge Cases: Arrays with no common elements should return an empty list.

If there are further questions or specific use cases, please let me know!

**Comparison:**

```
Input Array1	Input Array2	Hash Set Result	Two Pointers Result
[1, 3, 5, 7, 9]	[4, 6, 8, 9, 10]	[9]	[9]
[1, 2, 3, 4, 5]	[5, 6, 7, 8, 9]	[5]	[5]
```

**Code:**

```python
def find_intersection_hash_set(arr1, arr2):
    set_arr1 = set(arr1)
    intersection = []

    for num in arr2:
        if num in set_arr1 and num not in intersection:
            intersection.append(num)

    return intersection

def find_intersection_two_pointers(arr1, arr2):
    arr1.sort()
    arr2.sort()

    i = j = 0
    intersection = []

    while i < len(arr1) and j < len(arr2):
        if arr1[i] == arr2[j]:
            if not intersection or arr1[i] != intersection[-1]:
                intersection.append(arr1[i])
            i += 1
            j += 1
        elif arr1[i] < arr2[j]:
            i += 1
        else:
            j += 1

    return intersection
```

**Example input / output:**

```text
# Example usage with input data from JSON
data = [
    {"array1": [1, 3, 5, 7, 9], "array2": [4, 6, 8, 9, 10]},
    {"array1": [1, 2, 3, 4, 5], "array2": [5, 6, 7, 8, 9]}
]

for entry in data:
    print(f"Input: {entry}")
    result_hash_set = find_intersection_hash_set(**entry)
    result_two_pointers = find_intersection_two_pointers(**entry)
    print(f"Result (Hash Set): {result_hash_set}")
    print(f"Result (Two Pointers): {result_two_pointers}\n")
```

**Data:**

```json
[
    {"array1": [1, 3, 5, 7, 9], "array2": [4, 6, 8, 9, 10]},
    {"array1": [1, 2, 3, 4, 5], "array2": [5, 6, 7, 8, 9]}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Are arrays sorted?
5. Sort arrays
6. Initialize two pointers
7. Convert one array to a hashset
8. Use two-pointer approach
9. Traverse both arrays with twopointers
10. Add element if equal and notin result
11. Move pointer of smallerelement
12. Check for duplicates beforeadding to result
13. Use hash set approach
14. Convert one array to a hashset
15. Iterate through second array
16. Add element if in both arraysand not in result
17. Return intersection
18. Traverse first array with pointeri
19. Compare elements withsecond array using pointer j
20. Add element if equal and notin result
21. Move pointer of smallerelement
22. Return intersection

