---
type: "practice"
difficulty: "Deep"
---

# Find the maximum in every sliding window of size k using a deque.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sums and Sliding Windows](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sums_and_Sliding_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding the maximum in every sliding window of size (k) using a deque, we need to ensure that our solution is both efficient and easy to understand. The key idea here is to use a double-ended queue (deque) to maintain indices of useful elements for each sliding window in such a way that the front of the deque always contains the index of the maximum element in the current window.

#### Triaging the Question

The question asks us to find the maximum value within every window of size (k) as it slides through an array. The interviewer is likely probing your understanding of data structures, specifically deques, and how they can be used efficiently for such problems. We will assume that the input array is non-empty, and (1 \leq k \leq n), where (n) is the length of the array.

#### First-Principles Solution

To solve this problem, we use a sliding window approach combined with a deque to keep track of the indices of elements in the current window. Here’s how:

Initialize: Start by inserting the first (k) elements into the deque and find their maximum.
Slide Window: For each subsequent element up to the last element, maintain the deque such that it always contains indices of elements from which we can extract the max in constant time.
Maintain Deque: Ensure that the deque is ordered in decreasing order of values, i.e., the front of the deque always points to the index with the maximum value within the current window.
Detailed Steps
Initialization:
Initialize an empty deque and a result list for storing the maximums.
Iterate over the first (k-1) elements and add their indices to the deque while ensuring that the deque is ordered in decreasing order of values.
Sliding Window:
For each element from index (k) to (n-1):
Add the current element's index to the deque, maintaining its order with the other elements.
The front of the deque always points to the maximum value within the window; add this to our result list.
Remove indices that are out of the current window from the front of the deque.
Pseudocode
PYTHON
def max_in_sliding_window(nums, k):
    n = len(nums)
    if n == 0 or k <= 0:
        return []

    # Initialize deque and result list
    deq = collections.deque()
    maxes = []

    for i in range(n):
        # Remove indices that are out of the current window from the front
        while deq and deq[0] < i - k + 1:
            deq.popleft()

        # Maintain deque to be ordered in decreasing order of values
        while deq and nums[i] >= nums[deq[-1]]:
            deq.pop()

        # Add current index to the front of the deque
        deq.append(i)

        # If we have processed at least k elements, add the max to result list
        if i >= k - 1:
            maxes.append(nums[deq[0]])

    return maxes

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is that by using a deque, we can maintain an ordered sequence of indices which allows us to efficiently find the maximum in each sliding window. Common mistakes include not maintaining the correct order within the deque or forgetting to remove out-of-window indices.

For follow-ups, one could discuss edge cases like (k = 1), or explore optimizations if the array is very large and contains repeated values.

**Code:**

```python
from collections import deque

def max_in_sliding_window(nums, k):
    n = len(nums)
    if n == 0 or k <= 0:
        return []

    # Initialize deque and result list
    deq = deque()
    maxes = []

    for i in range(n):
        # Remove indices that are out of the current window from the front
        while deq and deq[0] < i - k + 1:
            deq.popleft()

        # Maintain deque to be ordered in decreasing order of values
        while deq and nums[i] >= nums[deq[-1]]:
            deq.pop()

        # Add current index to the front of the deque
        deq.append(i)

        # If we have processed at least k elements, add the max to result list
        if i >= k - 1:
            maxes.append(nums[deq[0]])

    return maxes
```

**Example input / output:**

```text
# Example input and output
nums = [1, 3, -1, -3, 5, 3, 6, 7]
k = 3
print(max_in_sliding_window(nums, k))  # Output: [3, 3, 5, 5, 6, 7]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Is index out of window?
9. Remove front element fromdeque
10. Is current element >= lastelement in deque?
11. Add current index to back ofdeque
12. Remove last element fromdeque and repeat step 4
13. Process first k-1 elements,adding indices to deque whilemaintaining order
14. Is index >= k-1?
15. Add front of deque (max) toresult list
16. End

