---
type: "practice"
difficulty: "Intermediate"
---

# You're mentoring a junior who reaches for recursion everywhere; how do you teach them to recognize when a problem is better modeled with a stack, a queue, or DP, using a concrete example?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursion and DP Judgment](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When teaching a junior engineer to recognize when a problem is better suited for recursion, stacks, queues, or dynamic programming (DP), it's crucial to start with clear examples and discuss the underlying principles. Let’s consider the classic problem of traversing a binary tree level by level, which can be effectively solved using both queues and DP.

TL;DR

In this mentoring scenario, we will use the example of breadth-first traversal (BFT) of a binary tree to illustrate how a queue can be used to manage the nodes at each level. We’ll also discuss dynamic programming through an analogy with BFT, emphasizing state management and overlapping subproblems.

#### Triaging the Question

The question is fundamentally about understanding data structures and algorithms better by recognizing when certain problems are best solved using queues, stacks, or DP. The key idea is to demonstrate these concepts in a concrete example that highlights their unique characteristics and applications.

#### Assumptions
The junior has some basic understanding of recursion.
They are familiar with the structure of binary trees and basic traversal methods (preorder, inorder, postorder).

#### First-Principles Solution

#### Example
Breadth-First Traversal (BFT) Using a Queue

#### Cons
ider the problem of performing a breadth-first traversal on a binary tree. The goal is to visit nodes level by level from left to right.

Understanding BFT:

BFT visits all nodes in a level before moving to the next.
It ensures that we explore the shortest path first, which can be useful in various applications like finding the shortest path in an unweighted graph or ensuring fairness in accessing resources.

#### Implementation
Using a Queue:

PYTHON
from collections import deque

def bfs_traversal(root):
    if not root:
        return []

    queue = deque([root])
    traversal_result = []

    while queue:
        current_node = queue.popleft()
        traversal_result.append(current_node.val)

        if current_node.left:
            queue.append(current_node.left)
        if current_node.right:
            queue.append(current_node.right)

    return traversal_result

Key Characteristics:

Queue: A queue helps manage nodes level by level, ensuring that each node at the current depth is processed before moving on to deeper levels.
State Management: Each node and its children are stored in a FIFO (First-In-First-Out) manner, making it easy to track which nodes have been visited.

Analogy with DP:

DP as BFT: Think of each level of the tree as a subproblem that needs to be solved before moving on to deeper levels.
State Transition: Just like in DP where we solve smaller subproblems and build up to larger ones, in BFT, we process nodes level by level.

#### Example
Dynamic Programming for Coin Change Problem

Now let’s consider the coin change problem as another example of dynamic programming. The goal is to find the minimum number of coins required to make a certain amount.

Understanding DP:

State: We define dp[i] as the minimum number of coins needed to make up amount i.
Transition: For each coin value, we update dp[i] based on previous states dp[i-coin_value].

#### Implementation
Using a DP Table:

PYTHON
def min_coins(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0

    for coin in coins:
        for x in range(coin, amount + 1):
            dp[x] = min(dp[x], dp[x - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1

Key Characteristics:

DP Table: The dp array stores the minimum number of coins needed for each amount.
Overlapping Subproblems: We reuse previously computed results to avoid redundant calculations, ensuring efficiency.

Comparison with BFT and Stacks:

BFT vs DP: While both manage state transitions, BFT is more about managing nodes in a queue, whereas DP focuses on solving subproblems efficiently.
Stacks: Stacks can be used for problems that require backtracking or depth-first search (DFS), such as finding all paths from the root to leaf nodes.

#### Closing Summary
and Common Mistakes

In this example, we demonstrated how BFT uses a queue to manage levels in a binary tree, while dynamic programming employs a DP table to solve subproblems efficiently. Both techniques involve state management but serve different purposes and are best suited for their respective problems.

#### Common mistakes
include:

Overusing recursion without understanding its limitations.
Misunderstanding the role of states and transitions in both BFT and DP.
Failing to recognize when to use a stack (e.g., DFS) versus a queue or DP.

By understanding these differences, juniors can better choose the right data structure and algorithm for each problem, leading to more efficient and effective solutions.

**Comparison:**

| Node Value | Level |
|------------|-------|
|     1      |   0   |
|     2      |   1   |
|     3      |   1   |
|     4      |   2   |
|     5      |   2   |

**Code:**

```python
from collections import deque

def bfs_traversal(root):
    if not root:
        return []

    queue = deque([root])
    traversal_result = []

    while queue:
        current_node = queue.popleft()
        traversal_result.append(current_node.val)

        if current_node.left:
            queue.append(current_node.left)
        if current_node.right:
            queue.append(current_node.right)

    return traversal_result

def min_coins(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0

    for coin in coins:
        for x in range(coin, amount + 1):
            dp[x] = min(dp[x], dp[x - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1
```

**Example input / output:**

```text
# Example input and output for BFS traversal
result = bfs_traversal(root)
print(result)  # Output: [1, 2, 3, 4, 5]

# Example input and output for Coin Change problem
coins = [1, 2, 5]
amount = 11
min_coins_needed = min_coins(coins, amount)
print(min_coins_needed)  # Output: 3 (minimum number of coins is 3 to make up the amount 11)
```

**Data:**

```python
# Example binary tree node class
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Creating a sample binary tree
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is root None?
7. Return []
8. (root.val, root.left), (root.val,root.right)
9. Push left and right childreninto queue
10. Pop node from queue
11. G
12. Node has children?
13. Push children to queue
14. J
15. Append node value to resultlist

