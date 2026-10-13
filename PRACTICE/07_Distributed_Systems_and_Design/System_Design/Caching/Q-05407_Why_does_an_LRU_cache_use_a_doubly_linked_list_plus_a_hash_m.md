---
type: "practice"
difficulty: "Deep"
---

# Why does an LRU cache use a doubly linked list plus a hash map instead of an array?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Redis Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Redis_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An LRU (Least Recently Used) cache is designed to optimize memory usage by evicting the least recently used items first. The key challenge in implementing such a cache efficiently involves both fast access and quick updates, which are inherently difficult with primitive data structures like arrays. This is where the combination of a doubly linked list and a hash map shines.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Data Structure Selection: Why certain data structures are chosen over others.

#### Performance Considerations
How to balance access time with update efficiency.
Cache Eviction Policies: The rationale behind using LRU specifically.

#### Assumptions
We assume the cache needs efficient get and put operations, along with the ability to maintain a sorted order of recently used items.

#### First Principles Solution
Why Not an Array?
Random Access: Arrays are inefficient for random access. To get or set an element at a specific index requires scanning from the beginning, which is O(n) complexity.
Insertion and Deletion: Inserting or deleting elements in an array involves shifting multiple elements to maintain order, also making these operations costly (O(n)).
Why a Doubly Linked List?
Efficient Updates: Doubly linked lists allow for O(1) time insertion and deletion at any position, which is crucial for maintaining the LRU order.
Flexible Movement: By moving nodes to the front of the list when an element is accessed or added, we can efficiently update the cache's order.
Why a Hash Map?
Fast Lookups: A hash map provides O(1) time complexity for both insertions and lookups, ensuring that accessing elements by key is fast.
Maintaining Order: The doubly linked list maintains the order of access, allowing us to quickly identify the least recently used items.
Full Solution

To implement an LRU cache using a doubly linked list and a hash map:

Hash Map (key -> Node): Maps each key to its corresponding node in the linked list.
Doubly Linked List: Manages the order of access, with nodes representing cache entries.
Key Operations

get(key):

Check if the key exists in the hash map.
If it does, move the associated node to the front of the list and return its value.
If not, return -1 or a similar sentinel value indicating absence.

put(key, value):

Check if the key already exists in the cache:
If yes, update the corresponding node's value, move it to the front, and adjust the hash map entry.
If no, add a new node to the front of the list and update the hash map. Ensure the cache size does not exceed its limit by evicting the least recently used item if necessary.

#### Example
Implementation
PYTHON
class Node:
    def __init__(self, key=None, value=None):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = {}
        self.capacity = capacity
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._add_to_front(node)
            return node.value
        return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
        else:
            if len(self.cache) >= self.capacity:
                removed = self.tail.prev
                del self.cache[removed.key]
                self._remove(removed)

        new_node = Node(key, value)
        self.cache[key] = new_node
        self._add_to_front(new_node)

    def _remove(self, node):
        prev_node = node.prev
        next_node = node.next
        prev_node.next = next_node
        next_node.prev = prev_node

    def _add_to_front(self, node):
        first_node = self.head.next
        self.head.next = node
        node.prev = self.head
        node.next = first_node
        first_node.prev = node

# Example Usage
cache = LRUCache(2)
cache.put(1, 1) # cache is {1=1}
cache.put(2, 2) # cache is {1=1, 2=2}
print(cache.get(1))  # returns 1 (most recent access is 1)
cache.put(3, 3)     # evicts key 2
print(cache.get(2))  # returns -1 (not found)
cache.put(4, 4)     # evicts key 1
print(cache.get(1))  # returns -1 (not found)
print(cache.get(3))  # returns 3
print(cache.get(4))  # returns 4

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Overlooking the importance of maintaining both fast access and update efficiency simultaneously.

#### Follow-Up
Discussing edge cases, such as what happens when updating an existing value or handling concurrent modifications.

This solution ensures efficient get and put operations while adhering to the LRU eviction policy. The combination of a doubly linked list for ordered updates and a hash map for quick lookups is the key to achieving this balance.

**Code:**

```python
class Node:
    def __init__(self, key=None, value=None):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = {}
        self.capacity = capacity
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._add_to_front(node)
            return node.value
        return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
        else:
            if len(self.cache) >= self.capacity:
                removed = self.tail.prev
                del self.cache[removed.key]
                self._remove(removed)

        new_node = Node(key, value)
        self.cache[key] = new_node
        self._add_to_front(new_node)

    def _remove(self, node):
        prev_node = node.prev
        next_node = node.next
        prev_node.next = next_node
        next_node.prev = prev_node

    def _add_to_front(self, node):
        first_node = self.head.next
        self.head.next = node
        node.prev = self.head
        node.next = first_node
        first_node.prev = node
```

**Example input / output:**

```text
cache = LRUCache(2)
cache.put(1, 1) # cache is {1=1}
cache.put(2, 2) # cache is {1=1, 2=2}
print(cache.get(1))  # returns 1 (most recent access is 1)
cache.put(3, 3)     # evicts key 2
print(cache.get(2))  # returns -1 (not found)
cache.put(4, 4)     # evicts key 1
print(cache.get(1))  # returns -1 (not found)
print(cache.get(3))  # returns 3
print(cache.get(4))  # returns 4
```

