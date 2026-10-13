---
type: "practice"
difficulty: "Deep"
---

# Explain how union-find with union-by-rank achieves near-constant amortized time.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Set Intersection Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Set_Intersection_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Union-Find is a powerful data structure used to manage disjoint sets efficiently, and it can be optimized using union-by-rank to achieve nearly constant-time operations on average. This optimization ensures that each operation (find or union) has an amortized time complexity of (O(\log n)), which is effectively close to (O(1)).

#### Triaging the Question

The question asks about how union-by-rank in the Union-Find algorithm achieves near-constant amortized time. The core concepts being probed are understanding the basic operations (find and union) of Union-Find, the role of path compression, and the mechanism by which union-by-rank optimizes these operations.

#### First Principles Solution

The Union-Find problem involves efficiently managing a collection of disjoint sets. The standard operations are:

Find: Determine which set a particular element is in.
Union: Join two sets into one.

To achieve efficient performance, we can use several optimizations: path compression and union-by-rank. union-by-rank specifically aims to balance the tree structure formed by the union-find operations to reduce the height of the trees.

Path Compression:

In path compression, every node on the way from a given node to its root gets updated so that it points directly to the root.
This significantly flattens the tree, making subsequent queries faster because the paths are compressed.

Union-by-Rank:

union-by-rank ensures that when we merge two sets, the set with smaller rank (depth) is always attached as a child of the set with larger rank. If ranks are equal, it can be arbitrary.
This helps in maintaining balanced trees and prevents deep nesting.
Detailed Explanation:

Initialization:

Each element starts its own set and has a rank initially set to 0 or 1.

Find Operation (Path Compression):

When find is called on an element, follow the parent pointers until reaching the root.
During this traversal, update each node’s parent pointer to point directly to the root, effectively compressing the path.

Union Operation:

To union two sets, find their roots using find.
Attach the tree with smaller rank under the one with larger rank. If ranks are equal, increment the rank of the new root.
This ensures that trees remain relatively flat and operations don't become too deep.

#### Example

#### Cons
ider a set of elements [0, 1, 2, 3] where initially each element is its own parent:

PYTHON
parents = [0, 1, 2, 3]
ranks = [0, 0, 0, 0]

Perform union(0, 1) and union(1, 2). After union-by-rank:

First, find the root of both elements:

find(0) -> 0
find(2) -> 2

Union the sets by attaching one under the other with rank updates.

Amortized Analysis:

The key insight is that path compression ensures paths are compressed in future queries. The union-by-rank strategy keeps tree heights small, meaning each operation (especially find) has a very low cost due to the reduced number of nodes involved.

#### Common Mistakes
and Follow-Ups:

Mistakes:

Failing to implement path compression properly.
Not understanding that union-by-rank only ensures balance at union, but not necessarily for each find operation.

#### Follow-Ups

How does this compare with other data structures?
What are the implications of these operations on real-world applications like disjoint set management in graph algorithms?

#### Closing Summary

Union-Find with union-by-rank and path compression provides near-constant amortized time for operations, making it highly efficient. By balancing trees and compressing paths, the overall performance is significantly improved.

**Comparison:**

| Operation   | Find (Path Compression) | Union (Union-by-Rank) |
|-------------|------------------------|----------------------|
| Time Complexity | O(\log n) with compression | Amortized O(\log n) |

**Code:**

```python
def find(parents, i):
    if parents[i] != i:
        parents[i] = find(parents, parents[i])  # Path compression
    return parents[i]

def union(parents, ranks, i, j):
    root_i = find(parents, i)
    root_j = find(parents, j)

    if root_i == root_j:
        return

    if ranks[root_i] > ranks[root_j]:
        parents[root_j] = root_i
    elif ranks[root_i] < ranks[root_j]:
        parents[root_i] = root_j
    else:  # Equal rank
        parents[root_j] = root_i
        ranks[root_i] += 1
```

**Example input / output:**

```text
- **Input**: `union(0, 1)`, `find(2)`
- **Output**:
  - After `union(0, 1)`: `parents = [0, 0, 2, 3]`, `ranks = [0, 1, 0, 0]`
  - After `find(2)`: `root of 2 is now 2`

- **Input**: `union(1, 2)`
- **Output**:
  - After `union(1, 2)`: `parents = [0, 0, 2, 3]`, `ranks = [0, 1, 1, 0]`
```

**Data:**

```json
{
  "parents": [0, 1, 2, 3],
  "ranks": [0, 0, 0, 0]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Find
6. Is parent[i] = i?
7. Path Compression
8. Return parent[i]
9. Update parent[j] toparent[parent[j]]
10. Union
11. Are roots same?
12. Attach smaller rank tree underlarger one
13. Return
14. Increment root's rank if equalranks

