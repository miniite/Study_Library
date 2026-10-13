---
type: "practice"
difficulty: "Intermediate"
---

# You inherited an LRU cache implemented with an array; explain, in complexity terms, exactly what's slow and how you'd move get and put to O(1).

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Time-space Trade-offs](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Time-space_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with an LRU (Least Recently Used) cache implemented using an array, one of the primary issues is that lookups are not efficient, typically operating in (O(n)) time complexity due to the need to search through the array for a specific key. This can be particularly problematic as the number of elements in the cache grows.

To explain this further and address the question at hand, we'll start by triaging what exactly is being asked: your task is to analyze the current implementation's performance bottlenecks and propose an optimized solution that allows both get and put operations to run in constant time (O(1)). This means you need to leverage data structures with fast access times, such as hash tables or hash maps.

Let’s assume the current LRU cache is implemented using a simple array of key-value pairs. Here are some assumptions we'll make:

Each element in the array contains both a key and its corresponding value.
The array itself does not maintain any additional metadata like pointers to previous/next nodes, making it impossible to quickly identify recently used items.

Now, let’s break down the solution step by step from first principles:

Understanding Current Performance:

For get(key), you would need to iterate through the entire array to find the key and return its value.
For put(key, value) operations, if a key already exists, it would require finding that element in the array (which is (O(n))), updating the value, and potentially reordering elements. If not present, adding an entry at the end of the array also requires iterating over the entire list.

Proposed Solution:

To achieve constant time complexity for both get and put, we need a data structure that can provide fast access to keys while maintaining the order of usage.
A combination of a hash map (for (O(1)) lookups) and doubly linked list (to maintain LRU ordering with (O(1)) time for insertions and deletions) is ideal.

#### Implementation
Steps:

Initialization: Create both a hash map to store key-value pairs, where keys are the actual cache data, and values are pointers to nodes in a doubly linked list.
Get Operation:
Use the key from the input to find the corresponding value in the hash map (this is (O(1))).
Update the node's position in the doubly linked list so that it reflects its recent usage by moving the node to a new head of the list.
Return the value stored in the node.
Put Operation:
Check if the key already exists in the hash map (this is (O(1))).
If it does, remove the node from its current position and update the pointer.
Add a new node at the head of the doubly linked list with the given key-value pair, updating the corresponding entry in the hash map.
If the cache exceeds its capacity, remove the least recently used item (the tail node) to maintain space.

By combining these two data structures, we can achieve (O(1)) time complexity for both get and put.

**Comparison:**

| Node | Key | Value | Prev | Next |
|------|-----|-------|------|------|
| 1    | 10  |       | null | 2    |
| 2    | 20  |       | 1    | 3    |
| 3    | 30  |       | 2    | null |

**Code:**

```python
class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache_map = dict()
        self.cache_list = DoublyLinkedList()

    def get(self, key):
        if key not in self.cache_map:
            return -1  # Key does not exist

        node = self.cache_map[key]
        value = node.value  # Extract the value from the node
        self._move_to_head(node)  # Move to head of the list to mark as recently used
        return value

    def put(self, key, value):
        if key in self.cache_map:
            node = self.cache_map[key]
            node.value = value  # Update the value of existing node
            self._move_to_head(node)  # Move it to head of list for recent usage
        else:
            new_node = DoublyLinkedListNode(key, value)
            if len(self.cache_map) >= self.capacity:
                tail_node = self.cache_list.remove_tail()
                del self.cache_map[tail_node.key]  # Remove from cache map too

            self.cache_map[key] = new_node
            self.cache_list.add_to_head(new_node)

    def _move_to_head(self, node):
        if node.prev is not None:
            node.prev.next = node.next
        if node.next is not None:
            node.next.prev = node.prev
        self.cache_list.add_to_head(node)
```

**Example input / output:**

```text
Input:
- Cache capacity = 2
- Initial cache state:
  - cache_map = {1: 10, 2: 20}
  - cache_list.head = (3, 30)

Operations:
- get(2)
- put(4, 40)

Output:
- After get(2): Cache state is now
  - cache_map = {1: 10, 4: 40, 2: 20}
  - cache_list.head = (4, 40), next = (3, 30)

- After put(4, 40): Cache state is now
  - cache_map = {1: 10, 4: 40, 2: 20}
  - cache_list.head = (4, 40), next = (3, 30)
```

**Data:**

```json
{
  "cache_map": {
    "1": {"prev": null, "next": "2", "value": 10},
    "2": {"prev": "1", "next": "3", "value": 20},
    "3": {"prev": "2", "next": null, "value": 30}
  },
  "cache_list": {
    "head": {"key": 3, "value": 30, "next": {"key": 2, "value": 20}},
    "tail": {"key": 1, "value": 10}
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Cache Map
6. Key Lookup in Cache Map
7. Node Found?
8. Insert New Node at Head andRemove Tail Node
9. Move Node to Head of List
10. Return Value
11. Doubly Linked List
12. Key Lookup in Doubly LinkedList
13. Node Found?
14. J
15. Update Node Value and Moveto Head

