---
type: "practice"
difficulty: "Deep"
---

# Trace how a query WHERE id = 500 uses a B+ tree index to find the row.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ B-tree and BRIN Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/B-tree_and_BRIN_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To trace how a query WHERE id = 500 uses a B+ tree index to find the row, let's first understand the basic structure and operations of a B+ tree. Then we will walk through the steps in detail from the root node down to finding the leaf node that contains the record with id = 500.

#### Triaging the Question

The question is probing your understanding of how database indexing works, specifically focusing on B+ trees. It requires you to demonstrate a deep understanding of the internal workings of B+ tree structures and their query execution process.

#### Assumptions

The B+ tree is balanced.
Each node can hold multiple keys (typically 10-20 in an index with a depth of 3).
The id column is indexed, so the B+ tree is organized based on this column's values.

#### First-Principles Solution
Step-by-Step Process:

Root Node Access:

Start at the root node (the topmost node in the B+ tree).

Key Comparison:

Compare 500 with the keys stored in the root node.
If 500 is equal to any key, we have found the leaf node containing our target row.

Node Traversal:

If 500 does not match any key, follow the appropriate child pointer (left or right) that would logically contain 500.

Leaf Node Identification:

Continue this process until reaching a leaf node.
In a B+ tree, every leaf node contains actual data rows.

Row Retrieval:

Once in the correct leaf node, locate and return the row with id = 500.
Detailed Example:

Let's assume we have the following simplified B+ tree structure for demonstration purposes:

CODE
            [498] -> Node2
Root -> [516] -> Node3
        [499] -> Node1
Node1: Contains rows with id 497, 498, 499.
Node2: Contains rows with id 500, 501, 502.
Node3: Contains rows with id 516, 517, 518.

Query Execution:

Start at the root node containing keys [498], [516].
Compare 500 with these keys:
500 > [498] and 500 < [516], so we follow the right pointer to Node2.
In Node2, compare 500 with its stored keys [500], [501], [502]:
Directly find 500.

Once at the leaf node containing id = 500, retrieve and return the row.

#### Closing Summary

In a B+ tree, queries like WHERE id = 500 efficiently use the indexed structure to quickly locate the correct leaf node where the target row is stored. This process leverages the balanced nature of the tree to minimize the number of nodes traversed during each query.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to understand that B+ trees are multiway search trees with all leaves at the same level.

#### Follow-Up
Discussing how to handle id values not present in the index, considering edge cases like duplicates and ensuring efficient range queries.

**Comparison:**

| Node         | Keys                     |
|--------------|--------------------------|
| Root Node    | [498, 516]               |
| Node2        | [500, 501, 502]          |

**Code:**

```python
# Example Python function to simulate a B+ tree query
class BPlusTreeNode:
    def __init__(self, keys=None, children=None):
        self.keys = keys if keys else []
        self.children = children if children else []

def find_id_in_b_plus_tree(root_node, target_id):
    current_node = root_node

    while isinstance(current_node, BPlusTreeNode):
        for i, key in enumerate(current_node.keys):
            if target_id == key:
                return current_node
            elif target_id < key:
                current_node = current_node.children[i]
                break
        else:  # No key was found that is less than or equal to the target
            current_node = current_node.children[-1]  # Move to last child if all keys are greater

    return current_node
```

**Example input / output:**

```text
Input:
root_node = {
  "keys": [498, 516],
  "children": [
    null,
    {
      "keys": [500, 501, 502],
      "children": []
    },
    null
  ]
}
target_id = 500

Output:
{
  "keys": [500, 501, 502],
  "children": []
}
```

**Data:**

```json
{
  "root_node": {
    "keys": [498, 516],
    "children": [
      null,
      {
        "keys": [500, 501, 502],
        "children": []
      },
      null
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Root Node
5. Key == 500?
6. Leaf Node
7. Node2
8. Key < 500?
9. Target Row
10. Next Node Child

