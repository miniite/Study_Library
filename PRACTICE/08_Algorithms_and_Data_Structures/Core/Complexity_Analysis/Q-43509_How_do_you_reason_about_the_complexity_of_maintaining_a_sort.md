---
type: "practice"
difficulty: "Deep"
---

# How do you reason about the complexity of maintaining a sorted set for a ranking feature where inserts are O(log n) but you also need O(1) rank lookups, and what structure (skip list) Redis uses under the hood?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Top-k Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Top-k_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reason about the complexity of maintaining a sorted set in a ranking feature where inserts are (O(\log n)) but also require (O(1)) rank lookups, we need to explore data structures that offer both operations efficiently. One such structure is the Skip List, which Redis uses under the hood for its ZSET data type.

#### Triaging the Question

The core of this question revolves around understanding how a data structure can balance between efficient insertion and quick rank-based lookups. The interviewer is probing your knowledge of advanced data structures and your ability to evaluate trade-offs in algorithmic performance.

#### First-Principles Solution
Insertion Complexity ((O(\log n))): We need a structure that allows for logarithmic time insertions, which means we can't simply use an unsorted list or array.
Rank Lookups ((O(1))): For quick rank-based lookups, the data structure must support fast access to elements at arbitrary positions in the sorted order.

Skip List:

Conceptual Overview: Skip Lists are probabilistic data structures that allow for efficient search operations. They maintain a linked list with multiple levels of nodes, where each level has fewer nodes.
Key Idea: The upper levels act as shortcuts, allowing for faster traversal through the structure.
Detailed Solution
Basic Structure

A skip list consists of multiple layers:

Each node can have zero or more forward pointers to nodes at higher levels. These pointers create "skips" that allow us to jump over many nodes in a single step.
The topmost level contains all elements, while the lower levels contain fewer and fewer nodes.
Insertion Process

When inserting an element:

Generate Levels: Randomly decide which levels will have this node by flipping a coin until you reach a "tail" (failure).
Insert at Top Level: Place the node in the top level.
Insert at Lower Levels: For each level that the new node is inserted into, place it between two existing nodes.

This process ensures (O(\log n)) average time for insertions by leveraging randomness to distribute levels efficiently.

Rank Lookup Process

For rank-based lookups:

Start at Top Level: Begin traversal from the head of the top level.
Follow Skips: Use forward pointers to move down multiple nodes in a single step until you reach the bottom level or the target node.
Track Position: Keep track of how many nodes have been skipped and adjust the rank accordingly.

By following these steps, we can achieve (O(1)) average time for rank lookups by leveraging the skip pointers to quickly navigate to the correct position.

#### Trade-offs
Memory Overhead: Skip Lists use more memory due to the additional forward pointers at each level.
Randomness: The performance depends on the randomness of node placement, which can be managed using appropriate algorithms.

#### Closing Summary

In conclusion, a skip list is an efficient data structure for maintaining a sorted set with (O(\log n)) insertions and (O(1)) rank lookups. Redis uses a variant of this structure for its ZSET operations to balance between the need for fast insertions and quick access to elements by their rank.

#### Common Mistakes/Follow-ups
Ignoring Randomness: Forgetting that skip list performance relies on randomness, which can lead to poor performance if not managed properly.
Misunderstanding Levels: Not understanding how levels are generated or why they are beneficial for achieving (O(\log n)) insertions and (O(1)) lookups.

#### Follow-ups
might involve discussing the implementation details of skip lists in Redis, comparing with other data structures like B-trees or treaps, or exploring scenarios where a different structure might be more appropriate.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Skip List Insertion Process
    section Random Level Generation
        rand_level: Generate random level for new node
    section Top Level Insertion
        insert_top: Insert node at top level
    section Lower Level Insertions
        insert_lower_levels: For each lower level, place node between existing nodes
```

**Code:**

```python
# Example Skip List Node
class Node:
    def __init__(self, value, level):
        self.value = value
        self.forward = [None] * level

def insert(skip_list, value):
    # Generate random levels for new node
    new_node_level = 1
    while random.random() < 0.5:
        new_node_level += 1

    new_node = Node(value, new_node_level)

    # Insert at top level
    prev_nodes = [None] * (new_node_level + 1)
    current = skip_list.head

    for i in range(new_node_level - 1, -1, -1):
        while (current is not None and
               current.forward[i] is not None and
               current.forward[i].value < value):
            current = current.forward[i]

        prev_nodes[i] = current

    # Insert at lower levels
    for i in range(new_node_level + 1):
        new_node.forward[i] = (prev_nodes[i].forward[i] if i < len(prev_nodes) else None)

    # Update previous nodes' pointers
    for i in reversed(range(len(prev_nodes))):
        prev_nodes[i].forward[i] = new_node

# Example Rank Lookup Function
def rank_lookup(skip_list, value):
    current = skip_list.head[0]
    count = 0

    while current is not None:
        if current.value < value:
            count += (1 + sum(node.forward[0] is not None for node in skip_list.head))
        elif current.value == value:
            return count
        current = current.forward[0]

    return -1
```

**Example input / output:**

```text
**Insertion Example**
- **Input**: Insert `4` into the skip list.
- **Output**: Node with value `4` inserted at levels `[1]`, creating a new level and adjusting pointers.

**Rank Lookup Example**
- **Input**: Find rank of node with value `3`.
- **Output**: Rank is `2` (considering values 1, 3).
```

**Data:**

```json
{
  "skip_list": {
    "head": [
      {"value": 1, "forward": [null]},
      {"value": 3, "forward": [null]},
      {"value": 5, "forward": [null]}
    ],
    "tail": null
  }
}
```

**Flow:**

1. Start
2. Randomly Generate Level
3. Insert Node at Top Level
4. For Each Lower Level
5. Place Node Between ExistingNodes
6. End Insertion

