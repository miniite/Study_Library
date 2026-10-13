---
type: "practice"
difficulty: "Intermediate"
---

# Explain how radix sort works and why it is not a comparison sort.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Counting and Radix Sort](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Counting_and_Radix_Sort.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Radix sort is a non-comparison-based sorting algorithm that sorts data with integer keys by grouping keys by individual digits that share the same significant position and value. This makes radix sort particularly efficient when dealing with large integers or fixed-length strings, where direct comparison can be costly.

#### Triage

The question asks you to explain how radix sort works and why it is not a comparison sort. The core idea here is understanding the fundamental mechanics of radix sort and contrasting its approach with traditional comparison-based sorting algorithms like quicksort or mergesort.

#### Assumptions
Radix sort works on integers, but can also be applied to strings.
It sorts from least significant digit (LSD) to most significant digit (MSD), or vice versa.
The key difference lies in its lack of direct comparison of elements during the sorting process.

#### First Principles Solution
Step 1: Understanding Radix Sort

Radix sort works by processing individual digits (or characters for strings). Here’s a high-level breakdown:

Divide: Separate the input into buckets based on each digit.
Conquer: Process each bucket individually.
Combine: Combine the results back into the original array.
Step 2: Implementation Details

To implement radix sort, you typically use an auxiliary array or a series of counting sorts for each digit (0-9). Here’s a simple example using LSD:

PYTHON
def counting_sort(arr, exp1):
    n = len(arr)
    output = [0] * n
    count = [0] * 10

    # Store count of occurrences in count[]
    for i in range(n):
        index = (arr[i] // exp1) % 10
        count[index] += 1

    # Change count[i] so that count[i] now contains actual position of this digit in output[]
    for i in range(1, 10):
        count[i] += count[i - 1]

    # Build the output array
    i = n - 1
    while i >= 0:
        index = (arr[i] // exp1) % 10
        output[count[index] - 1] = arr[i]
        count[index] -= 1
        i -= 1

    # Copy the output array to arr[], so that arr[] now contains sorted numbers according to current digit
    for i in range(n):
        arr[i] = output[i]

def radix_sort(arr):
    max_num = max(arr)
    exp = 1
    while max_num // exp > 0:
        counting_sort(arr, exp)
        exp *= 10

# Example usage
arr = [170, 45, 75, 90, 802, 24, 2, 66]
radix_sort(arr)
print("Sorted array is", arr)
Step 3: Why Radix Sort is Not a Comparison Sort

Comparison-based sorting algorithms (like quicksort and mergesort) compare elements directly to determine their relative order. Radix sort does not rely on such comparisons:

Non-comparative Nature: Radix sort sorts numbers digit by digit, which means it doesn’t directly compare two numbers. Instead, it groups them based on digits.
Efficiency for Certain Data Types: For large integers or fixed-length strings, radix sort can be more efficient because the number of comparisons required is significantly reduced.

#### Closing Summary
and Common Mistakes

Radix sort works by sorting data based on individual digits, making it a non-comparison-based algorithm. This approach provides efficiency benefits when dealing with specific types of data (like large integers or fixed-length strings), but it has limitations in terms of adaptability to different data distributions.

#### Common Mistakes

Forgetting to handle the case where all elements have leading zeros.
Misunderstanding the difference between LSD and MSD radix sort, which can affect performance depending on the input distribution.
Not properly implementing the counting sort for each digit place.

**Code:**

```python
def counting_sort(arr, exp1):
    n = len(arr)
    output = [0] * n
    count = [0] * 10

    # Store count of occurrences in count[]
    for i in range(n):
        index = (arr[i] // exp1) % 10
        count[index] += 1

    # Change count[i] so that count[i] now contains actual position of this digit in output[]
    for i in range(1, 10):
        count[i] += count[i - 1]

    # Build the output array
    i = n - 1
    while i >= 0:
        index = (arr[i] // exp1) % 10
        output[count[index] - 1] = arr[i]
        count[index] -= 1
        i -= 1

    # Copy the output array to arr[], so that arr[] now contains sorted numbers according to current digit
    for i in range(n):
        arr[i] = output[i]

def radix_sort(arr):
    max_num = max(arr)
    exp = 1
    while max_num // exp > 0:
        counting_sort(arr, exp)
        exp *= 10

# Example usage
arr = [170, 45, 75, 90, 802, 24, 2, 66]
radix_sort(arr)
print("Sorted array is", arr)
```

**Example input / output:**

```text
arr = [170, 45, 75, 90, 802, 24, 2, 66]
radix_sort(arr)
print("Sorted array is", arr) # Output: Sorted array is [2, 24, 45, 66, 75, 90, 170, 802]
```

**Flow:**

1. Start
2. Input Array
3. Find Maximum Value
4. Determine Radix (LSD or MSD)
5. Initialize Exponent = 1
6. While Max Number // Exp > 0Loop
7. Counting Sort Based onCurrent Digit Place
8. Update Exponent *= 10
9. End While Loop
10. Output Sorted Array
11. End

