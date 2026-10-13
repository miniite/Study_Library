---
type: "practice"
difficulty: "Basic"
---

# Given a stream of key accesses, trace which keys an LRU cache of size 3 evicts and in what order.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this problem, let's break down the scenario where we have a Least Recently Used (LRU) cache with a capacity of 3. Our goal is to trace which keys get evicted and in what order when given a stream of key accesses.

#### Triaging the Question

The core of the question involves understanding how an LRU caching mechanism behaves over time, specifically focusing on eviction patterns. We are essentially simulating the cache's operations based on a sequence of key access requests. The assumptions here include:

Cache size is fixed at 3.
If a key already exists in the cache, its usage count increments and it is marked as recently used.
If the cache is full and a new key is accessed, the least recently used key will be evicted to make space for the new one.

#### First Principles Solution
Initialize Cache: Create an LRU cache with a maximum capacity of 3. This can be represented using a dictionary in memory where keys map to values, but we also need to keep track of usage order.
Maintain Usage Order: Use another data structure (like a doubly linked list) to maintain the order of access for each key efficiently. The head of this list represents the most recently used key, and the tail represents the least recently used key.
Process Key Accesses:
For each key accessed:
If the key is already present in the cache, increment its usage count and move it to the front (most recent) position in the linked list.
Otherwise, if the cache is full, evict the least recently used key from both the dictionary and the linked list. Insert or update the new key at the head of the linked list.
Detailed Steps

Setup:

PYTHON
class Node:
    def __init__(self, key):
        self.key = key
        self.prev = None
        self.next = None

cache = {}
doubly_list_head = Node(None)
doubly_list_tail = Node(None)
doubly_list_head.next, doubly_list_tail.prev = doubly_list_tail, doubly_list_head
capacity = 3

Access Key:

PYTHON
def access(key):
    if key in cache:
        # Move the accessed node to the head of the list
        node = cache[key]
        remove_node(node)
        insert_at_head(node)
        print(f"Key {key} accessed and moved to head.")
    else:
        # Key not found, handle cache miss
        if len(cache) == capacity:
            least_recently_used_key = doubly_list_tail.prev.key
            remove_node(doubly_list_tail.prev)
            del cache[least_recently_used_key]
            print(f"Cache full. Evicting key {least_recently_used_key}.")
        node = Node(key)
        insert_at_head(node)
        cache[key] = node
        print(f"Key {key} accessed and inserted at head.")

Helper Functions:

PYTHON
def remove_node(node):
    prev, next = node.prev, node.next
    prev.next, next.prev = next, prev

def insert_at_head(node):
    head_next = doubly_list_head.next
    doubly_list_head.next = node
    node.prev = doubly_list_head
    node.next = head_next
    head_next.prev = node
Output Example

Given the key accesses: 1 -> 2 -> 3 -> 4 -> 5, we would observe the following:

CODE
Key 1 accessed and inserted at head.
Key 2 accessed and moved to head.
Key 3 accessed and moved to head.
Cache full. Evicting key 1.
Key 4 accessed and inserted at head.
Cache full. Evicting key 2.
Key 5 accessed and inserted at head.

#### Closing Summary

This example demonstrates the operation of an LRU cache, where keys are managed based on their usage frequency. Each access updates the cache's state by moving the accessed key to the front while evicting older keys when necessary.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to update both the dictionary and the doubly linked list simultaneously.

#### Follow-Up
Discussing the time complexity of operations like insertion, deletion, and search in an LRU cache. Exploring alternative eviction policies such as MRU or LFU.

**Code:**

```python
class Node:
    def __init__(self, key):
        self.key = key
        self.prev = None
        self.next = None

cache = {}
doubly_list_head = Node(None)
doubly_list_tail = Node(None)
doubly_list_head.next, doubly_list_tail.prev = doubly_list_tail, doubly_list_head
capacity = 3

def access(key):
    if key in cache:
        node = cache[key]
        remove_node(node)
        insert_at_head(node)
        print(f"Key {key} accessed and moved to head.")
    else:
        if len(cache) == capacity:
            least_recently_used_key = doubly_list_tail.prev.key
            remove_node(doubly_list_tail.prev)
            del cache[least_recently_used_key]
            print(f"Cache full. Evicting key {least_recently_used_key}.")
        node = Node(key)
        insert_at_head(node)
        cache[key] = node
        print(f"Key {key} accessed and inserted at head.")

def remove_node(node):
    prev, next = node.prev, node.next
    prev.next, next.prev = next, prev

def insert_at_head(node):
    head_next = doubly_list_head.next
    doubly_list_head.next = node
    node.prev = doubly_list_head
    node.next = head_next
    head_next.prev = node
```

**Example input / output:**

```text
1 -> 2 -> 3 -> 4 -> 5
Output:
Key 1 accessed and inserted at head.
Key 2 accessed and moved to head.
Key 3 accessed and moved to head.
Cache full. Evicting key 1.
Key 4 accessed and inserted at head.
Cache full. Evicting key 2.
Key 5 accessed and inserted at head.
```

