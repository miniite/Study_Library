---
type: "practice"
difficulty: "Deep"
---

# You must maintain a running median of inference latency over a sliding window (not the whole stream); why does the two-heap trick get harder once elements expire, and what lazy-deletion structure fixes it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Heaps and Running Median](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Heaps_and_Running_Median.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To maintain a running median of inference latency over a sliding window without processing all data, we can use the "two-heap" technique. However, as time progresses, elements expire from the window, making this approach challenging because it requires re-balancing heaps frequently. A lazy-deletion structure addresses these issues by efficiently handling expired elements, ensuring optimal performance.

#### Triaging the Question

The core of the question revolves around the challenges of maintaining a running median in a sliding window setting using two heaps and how to address them with a lazy-deletion mechanism. The interviewer is probing your understanding of data structures, algorithms, and their complexities, as well as your ability to optimize systems for efficiency.

#### Assumptions
Sliding Window: We are dealing with a fixed-size window where elements enter at the front and leave from the back.
Inference Latency: Each element represents an inference latency value.
Median Calculation: The median is required to be calculated over the current contents of this sliding window.

#### First-Principles Solution
Two-Heap Technique

Maintain a Max-Heap and Min-Heap:

max_heap stores the lower half of elements (including the median if the number of elements is odd).
min_heap stores the upper half.

Insertion:

Insert new element into one of the heaps based on its value compared to the current median.
Rebalance heaps if necessary by moving an element from one heap to another.

Deletion:

When elements expire, remove them directly from their respective heaps. This can lead to imbalanced heaps and requires subsequent re-balancing, which is costly in terms of time complexity.
Lazy-Deletion Technique

Mark Expiry with Sentinel Values:

Use sentinel values or marker bits within the heap structure to indicate whether an element has expired.

Heap Operations Adjusted for Expiry:

When performing operations like pop, skip over marked elements without fully removing them.
Periodically clean up the heap by swapping expired elements out of their positions.

Balancing and Cleaning:

Introduce mechanisms to clean up the heaps, such as periodically moving elements between heaps or swapping out entire segments that contain many expired elements.

#### Example
Lazy-Deletion with Sentinel Values
PYTHON
class HeapNode:
    def __init__(self, value):
        self.value = value
        self.expired = False  # Mark whether the element has expired

# Example usage in a max-heap (similar for min-heap)
import heapq

def push(heap, node):
    if not node.expired:  # Only add non-expired nodes
        heapq.heappush(heap, -node.value)  # Max-heap using negative values

def pop(heap):
    while heap and heap[0].expired:  # Skip expired elements
        heapq.heapreplace(heap, -heapq.heappop(heap))  # Replace popped element with a new one
    return -heapq.heappop(heap)  # Return the top value (negated to restore max-heap behavior)

#### Closing Summary
and Common Mistakes/Follow-Ups

The two-heap technique is effective for maintaining a running median but becomes cumbersome when elements expire. Lazy-deletion addresses this by marking expired elements without fully removing them, allowing efficient handling of expiry conditions. Key takeaways include:

Efficient Expiry Handling: Use sentinel values or marker bits to indicate expirations.
Lazy Operations: Skip over expired elements during operations like pop.
Periodic Cleanup: Introduce mechanisms for periodic cleanup and rebalancing.

#### Common mistakes
include failing to handle expiry conditions properly, leading to imbalanced heaps and increased complexity. Follow-up questions might involve discussing trade-offs between different data structures or exploring alternative approaches like using a balanced binary tree or a priority queue with additional metadata.

