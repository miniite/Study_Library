---
type: "practice"
difficulty: "Deep"
---

# Design an LRU cache with O(1) get and put operations — what data structures would you use?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Redis Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Redis_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an LRU (Least Recently Used) cache that supports O(1) time complexity for both get and put operations, we need to carefully choose appropriate data structures that can efficiently manage cache entries based on their recency of use. The key idea is to leverage a combination of a doubly linked list and a hash map.

In this context, the doubly linked list will help us maintain the order of access, allowing us to quickly identify the least recently used item. Meanwhile, the hash map will enable us to retrieve any cache entry in O(1) time by using its key as the lookup index.

Let's break down the solution step-by-step from first principles:

#### Triage
Understanding the Question

The interviewer is probing our knowledge of data structures and algorithms, specifically focusing on optimizing cache operations. The main assumptions are:

We need to support O(1) time complexity for both get and put.
The LRU policy means that we should remove the least recently used item when the cache exceeds its capacity.

#### Solution
1. Data Structures

To achieve the desired performance, we will use two primary data structures:

Hash Map: To store each key-value pair and allow for quick lookups by keys.
Doubly Linked List: To maintain the order of access, making it easy to remove the least recently used item.
2. Implementation Details

We can design a Cache class with methods like get, put, and helper methods to manage the doubly linked list.

PYTHON
class Node:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = dict()
        self.head = Node(0, 0)  # dummy head node
        self.tail = Node(0, 0)  # dummy tail node
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            self._move_to_head(node)
            return node.val
        return -1  # Key not found

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            node = self.cache[key]
            node.val = value
            self._move_to_head(node)
        else:
            if len(self.cache) == self.capacity:
                self._remove_tail()
            new_node = Node(key, value)
            self.cache[key] = new_node
            self._add_to_head(new_node)

    def _move_to_head(self, node):
        self._remove_node(node)
        self._add_to_head(node)

    def _remove_node(self, node):
        prev = node.prev
        next_ = node.next
        prev.next = next_
        next_.prev = prev

    def _add_to_head(self, node):
        next_ = self.head.next
        self.head.next = node
        node.prev = self.head
        node.next = next_
        next_.prev = node

    def _remove_tail(self):
        tail_node = self.tail.prev
        del self.cache[tail_node.key]
        self._remove_node(tail_node)

#### Closing Summary
and Common Mistakes/Follow-Ups

This design ensures that both get and put operations are performed in constant time. The key points to remember are:

The hash map provides quick lookups.
The doubly linked list maintains the order of access, allowing efficient removal of the least recently used item.

#### Common mistakes
include not properly managing the doubly linked list or not considering edge cases such as when the cache is full during a put operation. During follow-ups, you might discuss trade-offs between using a single hash map vs two maps (for keys and values), or explore more complex eviction policies.

