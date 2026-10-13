---
type: "practice"
difficulty: "Deep"
---

# Build a min-heap from an unsorted array in O(n) and explain why it is not O(n log n).

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ O(1) Randomized Set](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/O(1)_Randomized_Set.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a min-heap from an unsorted array in O(n), we need to leverage the properties of heap data structures, specifically focusing on how elements can be placed in their correct positions with minimal work. This is achieved through a process known as "heapify." The initial construction involves iterating over the array and performing this heapify operation for each node starting from the last non-leaf node (which is n/2 - 1 where n is the number of elements in the array) all the way up to the root.

Let's break down why building a min-heap from an unsorted array can be done in O(n):

#### Triaging the Question

The question asks for constructing a min-heap from an unsorted array with a time complexity of O(n). The interviewer is probing your understanding of heap properties, specifically the heapify operation and its efficiency. You need to explain both the algorithm and why it's not O(n log n).

#### Assumptions
We are given an array of integers that needs to be transformed into a min-heap.
The input size is n, which is the number of elements in the array.
Step-by-Step Solution From First Principles
Understanding Heapify: A min-heap has a property where for every node i, the value at A[i] is less than or equal to its children, ensuring that smaller values are closer to the root.
Heapify Operation:
The heapify operation is used to restore the heap property by comparing an element with its children and swapping them if necessary.
Building the Heap in O(n):
Start from the last non-leaf node, which is n/2 - 1.
For each node starting from this position down to the root (position 0), perform a heapify operation.
Efficiency of Heapify:
Each call to heapify typically checks at most two children and performs a swap if necessary.
The depth of any subtree is logarithmic in the number of nodes, but since we are only performing this check for n/2 elements (and not all log n levels), the overall complexity is linear.
Detailed Explanation
Heapify Operation: For each node, heapify ensures that it and its children satisfy the min-heap property. If a node violates this, a swap with one of its child nodes corrects it.
Efficiency Analysis:
Each call to heapify can be thought of as processing at most three elements (the parent and two children).
For all non-leaf nodes starting from the bottom up, there are approximately n/2 such calls because leaf nodes do not require heapify.
Thus, the total number of comparisons and swaps is proportional to n, making the overall time complexity O(n).
Why Not O(n log n)
Heapify Complexity: The key insight is that while each call to heapify can be considered as having a complexity of O(log k) where k is the number of nodes in the subtree, we are not performing this for all levels. We only go down to the bottom level.
Linear vs Logarithmic: Each node is processed at most once (or twice with swaps), and the depth of each call does not accumulate into a logarithmic factor.

#### Closing Summary

Building a min-heap from an unsorted array can be done in O(n) time by leveraging the heapify operation starting from the last non-leaf node. This approach ensures that we only process nodes once, making it linear. It is important to understand why this complexity holds and not confuse it with the potential logarithmic nature of individual heapify calls.

#### Common Mistakes
Confusing O(n log n) for Heap Construction: Failing to recognize that while individual heapify operations can be O(log k), the overall process does not accumulate to a higher time complexity.
Overlooking Leaf Nodes: Assuming all nodes, including leaf nodes, require heapify, which is incorrect.

**Comparison:**

```
Node Index	Value	Parent	Left Child	Right Child
0	2	-	1,3	4
1	3	0	5	-
2	9	-	-	-
3	4	0	-	-
4	8	1	-	-
5	5	1	-	-
```

**Code:**

```python
def heapify(arr, n, i):
    smallest = i
    left = 2 * i + 1
    right = 2 * i + 2

    if left < n and arr[i] > arr[left]:
        smallest = left

    if right < n and arr[smallest] > arr[right]:
        smallest = right

    if smallest != i:
        arr[i], arr[smallest] = arr[smallest], arr[i]
        heapify(arr, n, smallest)

def build_min_heap(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
```

**Example input / output:**

```text
# Sample input and output
arr = [5, 3, 8, 4, 2, 9]
build_min_heap(arr)
print("Min-heap:", arr) # Output will be a min-heap representation of the array
```

**Data:**

```json
{
  "array": [5, 3, 8, 4, 2, 9]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Start
5. Is i the root?
6. Heapify with smallest child
7. Check children and swap ifnecessary
8. Are there more nodes toprocess?
9. End

