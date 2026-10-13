---
type: "practice"
difficulty: "Deep"
---

# Why does an LRU cache need both a hash map and a doubly linked list — what breaks in complexity terms if you drop one?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Time-space Trade-offs](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Time-space_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why an LRU (Least Recently Used) cache requires both a hash map and a doubly linked list, we need to consider the time complexities of various operations involved in caching. A hash map provides O(1) average-time complexity for lookups, insertions, and deletions, making it ideal for key-based operations. The doubly linked list, on the other hand, allows for efficient reordering of nodes based on access frequency, which is crucial for maintaining the LRU behavior.

#### Triage

The question probes into understanding the underlying mechanisms that ensure optimal cache performance in terms of time complexity. Specifically, it challenges you to identify how and why a combination of these two data structures (hash map + doubly linked list) is necessary to maintain both O(1) average-time complexity for key-based operations and efficient reordering.

#### First-Principles Solution

Let's break down the requirements:

1. Hash Map
Purpose: Direct access by keys.
Operations:
Insertion/Deletion: (O(1)) average-time complexity.
Lookup: (O(1)) average-time complexity.
2. Doubly Linked List
Purpose: Maintain the order of elements based on their usage (most recently used to least recently used).
Operations:
Insertion at the head (newly accessed): (O(1)).
Removal from any position and reinsertion: (O(n)) in a singly linked list, but (O(1)) if doubly linked.
Why Both Are Necessary
3. Hash Map Only
Operations: If we rely solely on the hash map for all operations, we would lose the ability to maintain the order of elements.
Insertion/Deletion/Lookup: (O(1)) average-time complexity remains.
Reordering: Unable to reorder nodes based on usage frequency. This means that even if an element is accessed frequently, it won't automatically move to the front of the list.
4. Doubly Linked List Only
Operations: If we rely solely on the doubly linked list for all operations, we would lose direct key-based access.
Insertion/Deletion/Lookup: (O(n)) in a singly linked list; (O(1)) if doubly linked. However, this still means that lookup and insertion/deletion are not as efficient as with a hash map.
Reordering: While reordering is (O(1)), we need to find the node first, which could be slow ((O(n))).
Full Solution

To achieve both O(1) average-time complexity for key-based operations and efficient reordering, we must use a combination of:

Hash Map: For direct key-based access.

Operations: Insertion/Deletion/Lookup: (O(1)) average-time complexity.

Doubly Linked List: For maintaining the order of elements based on their usage frequency.

Operations: Reordering (insert at head, move node to head): (O(1)).

#### Common Mistakes
and Follow-Ups
5. Common Mistakes
Assuming that either data structure alone can maintain both key-based access and ordering efficiently.
Overlooking the trade-offs between time complexities in each operation.
6. Follow-Ups
What happens if we have a large cache size?
How does this solution handle write-heavy workloads versus read-heavy workloads?
Are there any alternatives or optimizations to reduce space complexity while maintaining performance?

By understanding these aspects, you can see why an LRU cache requires both data structures to ensure efficient and optimal operations.

**Comparison:**

| Operation | Cache Content            |
|-----------|--------------------------|
| Initial   | {}                       |
| put(1)    | {1: Node(key=1, val=10)}  |
| put(2)    | {1: Node(key=1, val=10), 2: Node(key=2, val=20)} |
| put(3)    | {1: Node(key=1, val=10), 2: Node(key=2, val=20), 3: Node(key=3, val=30)} (evicts key=1) |
| get(1)    | {}                       |
| put(4)    | {2: Node(key=2, val=20), 3: Node(key=3, val=30), 4: Node(key=4, val=40)} (evicts key=2) |

**Code:**

```python
# Example implementation of a simple LRU Cache using Python

class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}  # Hash map to store (key, node) pairs
        self.head = Node(0, 0)
        self.tail = Node(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._add(node)
            return node.value
        return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self._remove(self.cache[key])
        node = Node(key, value)
        self.cache[key] = node
        self._add(node)

        if len(self.cache) > self.capacity:
            node = self.tail.prev
            del self.cache[node.key]
            self._remove(node)

    def _remove(self, node: Node) -> None:
        prev = node.prev
        next_ = node.next
        prev.next = next_
        next_.prev = prev

    def _add(self, node: Node) -> None:
        prev = self.head
        next_ = self.head.next
        self.head.next = node
        node.prev = self.head
        node.next = next_
        next_.prev = node
```

**Example input / output:**

```text
Initial Cache:
{
  capacity: 3,
  cache: {}
}

Operations:
1. put(1, 10) -> cache = {1: Node(key=1, value=10)}
2. put(2, 20) -> cache = {1: Node(key=1, value=10), 2: Node(key=2, value=20)}
3. put(3, 30) -> cache = {1: Node(key=1, value=10), 2: Node(key=2, value=20), 3: Node(key=3, value=30)} (evicts key=1)
4. get(1) -> returns -1 as key 1 is not present
5. put(4, 40) -> cache = {2: Node(key=2, value=20), 3: Node(key=3, value=30), 4: Node(key=4, value=40)} (evicts key=2)
```

**Data:**

```json
{
  "capacity": 3,
  "operations": [
    {"type": "put", "key": 1, "value": 10},
    {"type": "put", "key": 2, "value": 20},
    {"type": "put", "key": 3, "value": 30},
    {"type": "get", "key": 1}, // returns 10
    {"type": "put", "key": 4, "value": 40} // evicts key 2 as LRU
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Cache Full
8. Update Map
9. Insert Node
10. Initial Cache
11. Cache Full?
12. Add New Entry and UpdateCache
13. Add New Entry and UpdateCache
14. Add New Entry Directly
15. Key Present in Cache?
16. Update Node Position
17. Return Value
18. Insert New Node and UpdateCache
19. Key Present in Doubly LinkedList?
20. Remove Node, Move to Head
21. Add New Node at Head
22. Return Value
23. Move New Node to Head
24. Remove Least Recently UsedNode

