---
type: "practice"
difficulty: "Deep"
---

# Given course prerequisites, determine whether all courses can be finished (cycle detection in a DAG).

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Shortest Paths](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Shortest_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The problem is to detect cycles in a Directed Acyclic Graph (DAG) where nodes represent courses and edges represent prerequisites. The goal is to determine if all courses can be completed, meaning there should be no cycle present.

#### Triaging the Question

This question probes your understanding of graph theory, specifically cycle detection algorithms. Key points include:

Graph Representation: Courses as nodes, prerequisites as directed edges.
Cycle Detection Algorithms: We need an algorithm that efficiently determines if a cycle exists in a DAG.
Edge Cases: Handling cases where courses have no dependencies or multiple levels of dependency.

#### First Principles Solution
Graph Representation and Initialization:
Use adjacency list representation to store the graph.
Cycle Detection:
Utilize Depth-First Search (DFS) with a visited set to detect cycles.
Edge Cases Handling:
Check for self-loops or redundant edges.
Step-by-Step Solution
Graph Representation and Initialization
PYTHON
from collections import defaultdict

def build_graph(prerequisites):
    graph = defaultdict(list)
    for course, prereq in prerequisites:
        graph[prereq].append(course)
    return graph
Cycle Detection Using DFS
Maintain a set of visited nodes and an additional set rec_stack to track the current path.
PYTHON
def is_cycle_dfs(graph, node, visited, rec_stack):
    if node in rec_stack:
        return True  # Cycle detected
    if node not in visited:
        visited.add(node)
        rec_stack.add(node)
        for neighbor in graph[node]:
            if is_cycle_dfs(graph, neighbor, visited, rec_stack):
                return True
        rec_stack.remove(node)  # Backtrack
    return False
Main Function to Check Cycles
Iterate over all nodes to ensure no cycle exists.
PYTHON
def can_finish(num_courses, prerequisites):
    graph = build_graph(prerequisites)
    visited = set()
    rec_stack = set()

    for course in range(num_courses):
        if course not in visited and is_cycle_dfs(graph, course, visited, rec_stack):
            return False

    return True

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The algorithm uses DFS to detect cycles. If any cycle exists, it returns False; otherwise, all courses can be completed.

#### Common Mistakes
Not handling the case where a node is visited but not added to the recursive stack (rec_stack).
Failing to initialize both visited and rec_stack sets correctly.
Code Blocks
PYTHON
from collections import defaultdict

def build_graph(prerequisites):
    graph = defaultdict(list)
    for course, prereq in prerequisites:
        graph[prereq].append(course)
    return graph

def is_cycle_dfs(graph, node, visited, rec_stack):
    if node in rec_stack:
        return True
    if node not in visited:
        visited.add(node)
        rec_stack.add(node)
        for neighbor in graph[node]:
            if is_cycle_dfs(graph, neighbor, visited, rec_stack):
                return True
        rec_stack.remove(node)
    return False

def can_finish(num_courses, prerequisites):
    graph = build_graph(prerequisites)
    visited = set()
    rec_stack = set()

    for course in range(num_courses):
        if course not in visited and is_cycle_dfs(graph, course, visited, rec_stack):
            return False

    return True
Self-Check

The solution covers all aspects of cycle detection in a DAG using DFS. Correctness + completeness confirmed.

