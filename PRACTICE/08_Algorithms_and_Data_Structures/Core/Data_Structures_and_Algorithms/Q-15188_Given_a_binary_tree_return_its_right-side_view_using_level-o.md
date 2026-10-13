---
type: "practice"
difficulty: "Intermediate"
---

# Given a binary tree, return its right-side view using level-order traversal.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Tree Traversal](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Tree_Traversal.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem of returning the right-side view of a binary tree using level-order traversal, we need to understand what "right-side view" means and how level-order traversal works. The right-side view of a binary tree is essentially the nodes you see if you look at the tree from the right side. At each level, only the last node encountered will be part of the final answer.

First, let's clarify the requirements:

We need to perform a level-order (breadth-first) traversal.
For each level, we capture the rightmost node.
The result should be a list of nodes from left to right at each level, starting with the top level.

Now, let's break down the solution step by step:

Initialize a Queue: Use a queue to facilitate the level-order traversal. Start by adding the root node to the queue.
Process Each Level: For each level, process all nodes in that level from left to right.
Capture Rightmost Node: At the end of processing each level, the last node added to the result list will be the rightmost one for that level.

Here is a step-by-step first-principles solution:

PYTHON
from collections import deque

def right_side_view(root):
    if not root:
        return []

    queue = deque([root])
    result = []

    while queue:
        level_size = len(queue)

        # Process all nodes in the current level
        for i in range(level_size):
            node = queue.popleft()

            # The last node of each level will be part of the right-side view
            if i == level_size - 1:
                result.append(node.val)

            # Add children to the queue
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

    return result

This solution works as follows:

We start by checking if the root is None. If it is, we return an empty list.
We initialize a queue with the root node.
For each level, we determine the number of nodes (level_size) at that level using the current size of the queue.
We then process all nodes in the current level. The last node processed for each level (i.e., when i == level_size - 1) will be added to our result list.
Finally, we add the left and right children of each node to the queue for the next level.

This approach ensures that we capture only the rightmost nodes from each level in a level-order traversal.

**Code:**

```python
from collections import deque

def right_side_view(root):
    if not root:
        return []

    queue = deque([root])
    result = []

    while queue:
        level_size = len(queue)

        for i in range(level_size):
            node = queue.popleft()

            if i == level_size - 1:
                result.append(node.val)

            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

    return result
```

**Example input / output:**

```text
# Example input tree (in level order)
tree = [
    1,
    2, 3,
    4, None, 5, 6
]

root = TreeNode(tree[0])
queue = deque([root])

for i in range(1, len(tree), 2):
    node = queue.popleft()

    left_val = tree[i]
    right_val = tree[i + 1] if (i + 1) < len(tree) else None

    if left_val is not None:
        node.left = TreeNode(left_val)
        queue.append(node.left)

    if right_val is not None:
        node.right = TreeNode(right_val)
        queue.append(node.right)

# Expected output: [1, 3, 6]
right_side_view(root) == [1, 3, 6]
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. Yes
6. No
7. Yes
8. No
9. Start
10. Is root None?
11. Return []
12. Initialize queue with root
13. Get level size from queue
14. Process nodes in current level?(i < level_size)
15. G
16. Is last node of this level? (i ==level_size - 1)
17. Append node.val to result
18. J
19. Add children to queue if exist
20. Next node in level? (i <level_size)
21. Is next node the last of currentlevel? (i == level_size - 1)
22. End

