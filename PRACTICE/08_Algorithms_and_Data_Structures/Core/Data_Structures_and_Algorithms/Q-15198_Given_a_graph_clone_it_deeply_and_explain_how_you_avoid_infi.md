---
type: "practice"
difficulty: "Intermediate"
---

# Given a graph, clone it deeply and explain how you avoid infinite loops on cycles.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cycle Detection in Linked Lists](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Cycle_Detection_in_Linked_Lists.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clone a graph deeply while avoiding infinite loops due to cycles, we need to understand that each node in the graph can have multiple outgoing edges, forming potentially complex structures including cycles. The goal is to create an exact copy of this structure without falling into an endless loop when encountering cycles.

#### Triage

The question is asking for a deep cloning mechanism that ensures no infinite loops occur during the process. We need to assume:

Each node has unique identifiers (IDs) which allow us to detect cycles.
The graph can contain multiple nodes and edges, possibly forming one or more cycles.
We must avoid visiting the same node twice in the current traversal.

#### First-Principles Solution
Identify Nodes and Edges: Traverse each node exactly once while creating a new copy of every node encountered.
Track Visited Nodes: Use a hash map to keep track of nodes that have already been cloned, ensuring no infinite loops by checking if a node is being visited for the second time during the current traversal.
Recursive or Iterative Cloning: Implement this using either recursion (DFS) or an iterative approach with a stack.

#### Implementation

Here’s how we can implement this in JavaScript:

JAVASCRIPT
class GraphNode {

#### cons
tructor(value, neighbors = []) {
        this.value = value;
        this.neighbors = neighbors;
    }
}

function cloneGraph(node, visitedNodes = new Map()) {
    if (!node) return null;

    // Check if the node is already visited and cloned
    if (visitedNodes.has(node)) {
        return visitedNodes.get(node);
    }

    // Create a new node with the same value as the original

#### cons
t newNode = new GraphNode(node.value);

    // Store the mapping between the old and new nodes
    visitedNodes.set(node, newNode);

    // Recursively clone all neighbors
    for (const neighbor of node.neighbors) {
        newNode.neighbors.push(cloneGraph(neighbor, visitedNodes));
    }

    return newNode;
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

The solution ensures that each node is cloned exactly once by maintaining a visitedNodes map. This prevents the algorithm from visiting nodes multiple times during a single traversal path and thus avoids cycles.

#### Common mistakes
include:

Not checking if a node has already been visited, leading to infinite loops.
Failing to store references between old and new nodes properly.
Incorrectly handling edge cases like disconnected or empty graphs.

#### Follow-ups
might involve discussing performance optimizations (e.g., using BFS instead of DFS), handling directed vs. undirected graphs, or dealing with large graphs where memory efficiency is critical.

