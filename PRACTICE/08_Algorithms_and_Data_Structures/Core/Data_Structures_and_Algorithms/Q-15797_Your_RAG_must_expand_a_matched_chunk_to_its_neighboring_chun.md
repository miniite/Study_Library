---
type: "practice"
difficulty: "Intermediate"
---

# Your RAG must expand a matched chunk to its neighboring chunks for context; why model parent-document-to-child-chunks as a tree or adjacency list, and what's the lookup cost to fetch a window around a hit?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursion and DP Judgment](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing a retrieval-Augmented Generation (RAG) system that expands matched chunks by accessing neighboring chunks, modeling the parent-document-to-child-chunks relationship as either a tree or an adjacency list offers several advantages. Trees provide a hierarchical structure that is intuitive and easy to navigate, while adjacency lists offer flexibility in representing complex relationships between chunks. The lookup cost for fetching a window around a hit (i.e., retrieving context from neighboring chunks) depends on the choice of data structure.

Firstly, let's understand why trees or adjacency lists are chosen over other data structures like hash maps. Trees and adjacency lists are more suitable because they naturally represent hierarchical and relational data, respectively. In a tree, each node has a parent (the document), and children (child chunks). This structure is intuitive for organizing hierarchical information.

Now, let's discuss the advantages of both approaches:

Trees:

Intuitive Hierarchical Structure: Trees allow us to represent the parent-document-to-child-chunks relationship in a natural way.
Efficient Path Traversal: Tree traversal algorithms like depth-first search (DFS) and breadth-first search (BFS) can be used efficiently to navigate through the tree structure.
Balanced Trees: If balanced trees are used, operations such as insertion, deletion, and lookup have logarithmic time complexity.

Adjacency Lists:

Flexibility in Relationships: Adjacency lists allow for a more flexible representation of relationships between chunks, which can be beneficial when the context is not strictly hierarchical.
Efficient Lookup: In an adjacency list, accessing neighboring chunks directly from a chunk's neighbors has constant time complexity.

Next, let’s consider the lookup cost to fetch a window around a hit. The goal is to efficiently retrieve the context surrounding the matched chunk. Here are the steps for both data structures:

Trees

Initialization:

Build the tree structure by inserting each chunk as a node with pointers to its parent and children.

Finding Neighbors:

When a chunk is hit, start traversing up (to find siblings) or down (to find child chunks).
For example, if we need to fetch 3 chunks before and after the hit, traverse up and down from the hit node until the desired number of neighbors are collected.
Adjacency Lists

Initialization:

Build an adjacency list where each chunk points to its neighboring chunks.

Finding Neighbors:

Directly access the neighbors of the hit chunk using the adjacency list. This step is efficient and has a time complexity of O(1).
Lookup Cost Analysis
Trees: The lookup cost in a balanced tree structure for fetching ( n ) neighbors is approximately ( O(n \log n) ), where each level of traversal (up or down) takes logarithmic time.
Adjacency Lists: The lookup cost to fetch the same number of neighbors using an adjacency list is constant, ( O(1) ).

#### Trade-offs
Trees are more suitable when a hierarchical structure is needed and operations like path traversals are frequent. However, they might be less efficient for simple context retrieval.
Adjacency Lists offer better performance for direct neighbor lookups but may require additional effort to represent complex hierarchies.

In conclusion, the choice between a tree or an adjacency list depends on the specific requirements of the RAG system. Trees are useful when maintaining a hierarchical structure is important, while adjacency lists provide faster context retrieval at the cost of potentially more complex data modeling.

**Code:**

```python
# Example code for building and traversing a tree structure
class ChunkNode:
    def __init__(self, chunk_id):
        self.chunk_id = chunk_id
        self.parent = None
        self.children = []

def build_tree(chunks):
    root = None
    nodes = {}

    for chunk in chunks:
        node = ChunkNode(chunk['id'])
        nodes[chunk['id']] = node

        if 'parent' in chunk and chunk['parent'] in nodes:
            parent_node = nodes[chunk['parent']]
            parent_node.children.append(node)
            node.parent = parent_node
        else:
            root = node

    return root

def fetch_neighbors(root, hit_id, num_neighbors=3):
    # Traverse up to find siblings and ancestors
    def traverse_up(node, depth, neighbors):
        if not node or depth <= 0:
            return
        if node.chunk_id != hit_id and len(neighbors) < num_neighbors:
            neighbors.append(node)
        for child in reversed(node.children):
            traverse_up(child, depth - 1, neighbors)

    # Traverse down to find children
    def traverse_down(node, depth, neighbors):
        if not node or depth <= 0:
            return
        if node.chunk_id != hit_id and len(neighbors) < num_neighbors:
            neighbors.append(node)
        for child in node.children:
            traverse_down(child, depth - 1, neighbors)

    hit_node = nodes[hit_id]
    neighbors = []

    # Traverse up to find siblings and ancestors
    traverse_up(hit_node.parent, 3, neighbors)

    # Traverse down to find children
    traverse_down(hit_node, 3, neighbors)

    return [node.chunk_id for node in neighbors]

# Example adjacency list representation
class AdjacencyListChunk:
    def __init__(self, chunk_id):
        self.chunk_id = chunk_id
        self.neighbors = []

def build_adjacency_list(chunks):
    nodes = {}

    for chunk in chunks:
        node = AdjacencyListChunk(chunk['id'])
        nodes[chunk['id']] = node

        if 'parent' in chunk and chunk['parent'] in nodes:
            parent_node = nodes[chunk['parent']]
            parent_node.neighbors.append(node)
        else:
            root = node

    return nodes[root.chunk_id]

def fetch_neighbors_adjacency_list(root, hit_id, num_neighbors=3):
    hit_node = root
    neighbors = []

    # Traverse up to find siblings and ancestors
    current = hit_node.parent
    while current and len(neighbors) < num_neighbors:
        neighbors.append(current)
        current = current.parent

    # Traverse down to find children
    for child in hit_node.neighbors:
        if len(neighbors) >= num_neighbors:
            break
        neighbors.append(child)

    return [node.chunk_id for node in neighbors]
```

**Example input / output:**

```
Example input and output for tree traversal

chunks = [ {"id": "chunk1", "parent": null, "text": "Introduction to the document"}, {"id": "chunk2", "parent": "chunk1", "text": "Background information"}, {"id": "chunk3", "parent": "chunk1", "text": "Main content part 1"}, {"id": "chunk4", "parent": "chunk2", "text": "More detailed background"} ]

root = build_tree(chunks) hit_id = "chunk2" neighbors = fetch_neighbors(root, hit_id)

print(f"Neighbors around chunk {hit_id}: {neighbors}")

Output: Neighbors around chunk chunk2: ['chunk1', 'chunk3']
CODE
Copy
```python
# Example input and output for adjacency list traversal
chunks = [
  {"id": "chunk1", "parent": null, "text": "Introduction to the document"},
  {"id": "chunk2", "parent": "chunk1", "text": "Background information"},
  {"id": "chunk3", "parent": "chunk1", "text": "Main content part 1"},
  {"id": "chunk4", "parent": "chunk2", "text": "More detailed background"}
]

root = build_adjacency_list(chunks)
hit_id = "chunk2"
neighbors = fetch_neighbors_adjacency_list(root, hit_id)

print(f"Neighbors around chunk {hit_id}: {neighbors}")
# Output: Neighbors around chunk chunk2: ['chunk1', 'chunk3']
```

**Data:**

```json
[
  {
    "id": "chunk1",
    "parent": null,
    "text": "Introduction to the document"
  },
  {
    "id": "chunk2",
    "parent": "chunk1",
    "text": "Background information"
  },
  {
    "id": "chunk3",
    "parent": "chunk1",
    "text": "Main content part 1"
  },
  {
    "id": "chunk4",
    "parent": "chunk2",
    "text": "More detailed background"
  }
]
```

