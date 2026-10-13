---
type: "practice"
difficulty: "Intermediate"
---

# Given a 2D grid, find the number of distinct islands considering their shapes.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Shortest Paths](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Shortest_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify and count unique island shapes in a 2D grid. The key is to treat each island as a unique shape based on its structure rather than its position. We'll use Depth-First Search (DFS) or Breadth-First Search (BFS) to traverse the islands and serialize their shapes into a canonical form that can be compared.

#### Triage

The question asks for the number of distinct islands in a 2D grid, where "distinct" means considering shape similarity rather than position. The interviewer is probing our understanding of graph traversal techniques (DFS/BFS), serialization methods to represent island shapes, and handling edge cases like boundaries and disconnected components.

#### First-Principles Solution
Graph Traversal: Use DFS or BFS to traverse each island.
Shape Serialization: Record the shape as a canonical form that can be compared across islands.
Hashing Shapes: Use a hash set to store unique shapes.
Edge Cases: Handle boundary conditions and ensure we don't revisit cells.
Detailed Steps
Initialize a Visited Set: To avoid revisiting cells during traversal.
Define Directions: Define the directions for moving around the grid (up, down, left, right).
Traverse Each Cell: Start DFS/BFS from each unvisited '1' cell to find an island.
Serialize Island Shape: During traversal, record the shape as a string or list of coordinates relative to the starting point.
Hash and Count Shapes: Add serialized shapes to a hash set to count distinct ones.

#### Example

#### Cons
ider the following grid:

CODE
1 0 0 1
1 1 0 0
0 0 1 0
0 1 1 1
Start DFS/BFS from (0,0):
Serialized shape: [(0,0), (-1,-1), (-1,-2)]
Continue until all islands are traversed.

#### Common Mistakes/Follow-Ups
Mistake: Not handling edge cases like boundary conditions or disconnected components.

#### Follow-Up
Discuss trade-offs between DFS and BFS in terms of memory usage and time complexity.

