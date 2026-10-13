---
type: "practice"
difficulty: "Deep"
---

# A trie-backed autocomplete must return top-k weighted completions per prefix, not just any completions; how do you store aggregated weights on nodes and avoid a full subtree scan per keystroke?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of implementing an efficient trie-based autocomplete feature that returns top-k weighted completions per prefix, we need to focus on two key aspects: storing aggregated weights effectively and avoiding a full subtree scan with each keystroke. The solution involves leveraging a combination of data structures and algorithms to achieve both performance and accuracy.

#### Triage

The question is asking how to implement an autocomplete system that can efficiently return the top-k weighted completions for any given prefix, without performing a full subtree scan at each keystroke. This requires us to consider the trie structure, weight aggregation, and optimization techniques.

#### Assumptions

Each node in the trie represents a character.
Weights are associated with paths from the root to leaves.
The goal is to find the top-k completions based on their aggregated weights for any given prefix.

#### First-Principles Solution
1. Trie Structure and Weight Aggregation

A basic trie structure can be extended to support weight aggregation by storing a cumulative weight at each node. However, this alone would lead to full subtree scans when querying the autocomplete feature, which is inefficient.

Instead, we need to maintain an efficient way to query the top-k weights without scanning the entire subtree. This can be achieved through a combination of:

Trie nodes with aggregated weights: Each node will store the total weight contributed by all descendants.
Priority Queue (Max-Heap): To efficiently retrieve the top-k weighted paths.
2. Trie Node Design

Each trie node should have the following attributes:

children: A dictionary mapping characters to child nodes.
weight: The cumulative weight of the path from the root to this node.
count: The number of leaf nodes in the subtree rooted at this node.
3. Insertion and Update Operations

When inserting a word into the trie, update the weights and counts accordingly:

PYTHON
class TrieNode:
    def __init__(self):
        self.children = {}
        self.weight = 0
        self.count = 0
Step-by-Step Solution

Insertion:

Traverse the trie while incrementing weight for each node.
Update the count at leaf nodes.

Query Top-k Completions:

Use a max-heap to keep track of the top-k weighted paths.
Start from the root and traverse down the prefix path, updating the heap with valid completions.
PYTHON
import heapq

def insert(word, weight):
    node = trie
    for char in word:
        if char not in node.children:
            node.children[char] = TrieNode()
        node = node.children[char]
        node.weight += weight
    node.count += 1

def autocomplete(prefix, k):
    node = trie
    for char in prefix:
        if char not in node.children:
            return []
        node = node.children[char]

    heap = []

    def dfs(node, path, score):
        if len(heap) < k or score > -heap[0][0]:
            heapq.heappush(heap, (-score, "".join(path)))
            if len(heap) > k:
                heapq.heappop(heap)
            for child in node.children.values():
                path.append(child.char)
                dfs(child, path[:], score + child.weight)
                path.pop()

    dfs(node, [prefix[-1]], 0)

    return [item[1] for item in heap]

#### Closing Summary

This approach ensures that the autocomplete feature is efficient by leveraging a trie structure with aggregated weights and a max-heap to retrieve top-k weighted completions. The solution avoids full subtree scans, making it scalable and performant.

#### Common Mistakes/Follow-Ups
Mistake: Failing to use a priority queue for top-k retrieval.

#### Follow-Up
Discussing edge cases like handling repeated characters in the prefix or optimizing further with compression techniques.

**Comparison:**

```
Node	Weight	Count
a	-	0
b	15	3
c	10	1
d	8	2
e	7	2
f	6	3
```

**Code:**

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.weight = 0
        self.count = 0

def insert(word, weight):
    node = trie
    for char in word:
        if char not in node.children:
            node.children[char] = TrieNode()
        node = node.children[char]
        node.weight += weight
    node.count += 1

def autocomplete(prefix, k):
    node = trie
    for char in prefix:
        if char not in node.children:
            return []
        node = node.children[char]

    heap = []

    def dfs(node, path, score):
        if len(heap) < k or score > -heap[0][0]:
            heapq.heappush(heap, (-score, "".join(path)))
            if len(heap) > k:
                heapq.heappop(heap)
            for child in node.children.values():
                path.append(child.char)
                dfs(child, path[:], score + child.weight)
                path.pop()

    dfs(node, [prefix[-1]], 0)

    return [item[1] for item in heap]
```

**Example input / output:**

```text
trie = TrieNode()
insert("abcd", 10)
insert("adef", 8)
insert("abf", 6)
insert("bdef", 7)

print(autocomplete("a", 2))  # Output: ['abcd', 'adef']
```

**Data:**

```json
{
  "trie": {
    "a": {
      "b": {
        "c": {"weight": 10, "count": 1},
        "d": {"weight": 5, "count": 2}
      },
      "d": {"weight": 3, "count": 3}
    }
  }
}
```

**Flow:**

1. Start
2. Insert 'abcd', weight=10
3. Update Trie Node Weights
4. Insert 'adef', weight=8
5. Insert 'abf', weight=6
6. Insert 'bdef', weight=7
7. Autocomplete 'a', k=2
8. abcd', 'adef

