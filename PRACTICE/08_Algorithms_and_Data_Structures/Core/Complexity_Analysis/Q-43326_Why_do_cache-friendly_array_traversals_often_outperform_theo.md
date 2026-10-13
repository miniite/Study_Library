---
type: "practice"
difficulty: "Deep"
---

# Why do cache-friendly array traversals often outperform theoretically-equivalent linked-list ones despite the same Big-O?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Time-space Trade-offs](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Time-space_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why cache-friendly array traversals often outperform theoretically-equivalent linked-list traversals, we need to delve into how modern computer architectures operate and how these data structures interact with memory systems. This performance difference is primarily due to the way CPUs handle data access through caches.

TL;DR

Array traversals are generally more cache-friendly, leading to better locality of reference and reduced cache misses, which can significantly outperform linked-list traversal even when both have the same Big-O complexity.

#### Triage
Restating the Question and Assumptions

The question is probing our understanding of how memory systems interact with different data structures. The key assumptions are:

Both array and linked list traversals are considered "theoretically equivalent" in terms of time complexity.
We focus on modern CPU architectures, where cache performance significantly affects overall execution speed.

#### First Principles Solution

To reason from first principles, let's break down the problem step by step:

Understanding Cache Performance:

Modern CPUs use various levels of caching (L1, L2, L3) to store frequently accessed data.
The goal is to minimize cache misses and maximize hit rates.

Array Traversal:

Arrays are stored in contiguous memory locations.
When traversing an array, the CPU can predictively load nearby memory addresses due to spatial locality.
This leads to higher cache hit rates because adjacent elements in the array are likely to be located within the same cache lines.

Linked List Traversal:

Linked lists store data and pointers separately, with each node pointing to the next node.
To traverse a linked list, you must follow these pointer references, which can lead to arbitrary memory accesses scattered throughout memory.
This results in poor spatial locality, increasing the likelihood of cache misses.

Big-O Complexity:

Both traversals have O(n) complexity in terms of time and space.
However, the constant factors hidden in Big-O notation become significant when considering real-world performance.

Cache Misses and Hits:

A cache miss is an expensive operation as it involves fetching data from slower main memory.
Cache hits are much faster because they come directly from the CPU cache.
Detailed Analysis

Let's analyze a simple example to illustrate these points:

Array Example

#### Cons
ider traversing an array of 10,000 elements:

PYTHON
for i in range(10000):
    # Do something with arr[i]
The CPU can predict the next memory address and load it before accessing it.
This reduces cache misses and improves overall performance.
Linked List Example

Now consider traversing a linked list of 10,000 nodes:

PYTHON
current = head
while current is not None:
    # Do something with current.value
    current = current.next
For each node, the CPU must follow the pointer to the next node.
The memory addresses of these nodes can be scattered throughout memory, leading to many cache misses.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, while both array and linked list traversals have O(n) complexity, the performance difference lies in how they interact with the CPU's cache. Array traversal benefits from spatial locality, reducing cache misses and improving overall execution speed. Linked list traversal suffers from poor spatial locality, leading to more cache misses.

#### Common mistakes
include:

Overlooking the impact of cache behavior on algorithm performance.
Assuming that Big-O complexity alone determines real-world performance without considering constant factors.
Ignoring how memory access patterns affect CPU performance.

#### Follow-ups
might involve discussing:

Techniques to optimize linked list traversal for better cache utilization (e.g., using hash maps or pre-fetching).

#### Trade-offs
between different data structures in various use cases.
How different caching strategies can mitigate some of the issues with linked lists.

**Comparison:**

```
Data Structure	Memory Layout	Spatial Locality	Cache Hits/ Misses
Array	Contiguous memory	High	Low
Linked List	Discontiguous memory	Low	High
```

**Code:**

```python
# Example array and linked list traversal

array = [i for i in range(10000)]
linked_list = [(None, j) for j in range(10000)]

def traverse_array(arr):
    start_time = time.time()
    for i in range(len(arr)):
        # Do something with arr[i]
        pass
    end_time = time.time()
    print("Array traversal took:", end_time - start_time)

def traverse_linked_list(llist):
    current = llist[0][1]
    start_time = time.time()
    while current is not None:
        # Do something with current.value
        next_node, current = current
    end_time = time.time()
    print("Linked list traversal took:", end_time - start_time)
```

**Example input / output:**

```

Input:

PYTHON
Copy
traverse_array(array)
traverse_linked_list(linked_list)

Output:

CODE
Copy
Array traversal took: [time_value]
Linked list traversal took: [larger_time_value]
```

**Data:**

```json
{
  "array": [0, 1, 2, ..., 9999],
  "linked_list": [
    {"next": 1},
    {"next": 2},
    ...
    {"next": null}
  ]
}
```

**Flow:**

1. Array Traversal
2. High Cache Hits, Low CacheMisses
3. Linked List Traversal
4. Low Cache Hits, High CacheMisses

