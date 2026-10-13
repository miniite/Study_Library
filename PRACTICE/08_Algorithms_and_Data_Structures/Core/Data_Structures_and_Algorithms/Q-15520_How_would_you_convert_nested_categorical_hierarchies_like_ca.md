---
type: "practice"
difficulty: "Intermediate"
---

# How would you convert nested categorical hierarchies (like category > subcategory) into a tree and traverse it for feature rollups?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursion and DP Judgment](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle converting nested categorical hierarchies into a tree structure and traversing it for feature rollups, we first need to understand that this problem involves data manipulation and graph traversal algorithms. The core idea is to represent hierarchical categories as nodes in a tree and use an algorithm like depth-first search (DFS) or breadth-first search (BFS) to traverse the tree and aggregate features at each level.

Let's break down the process step by step:

Understand the Input Data: We need data in a format that represents the hierarchical structure, such as a nested JSON object.
Tree Construction: Convert the input data into an actual tree data structure.
Traversal and Feature Rollup: Traverse the tree to aggregate features at each node.

#### Triage

The question is asking for:

A method to convert hierarchical categorical data into a tree structure.
An approach to traverse this tree to perform feature rollups, which typically means aggregating (e.g., summing) certain features from child nodes up to their parent nodes.

#### Assumptions

The input will be in JSON or similar nested dictionary format.
We need to implement the conversion logic and the traversal algorithm.

#### First Principles Solution
Step 1: Understanding Input Data

Let's consider an example where we have a hierarchical category structure like this:

JSON
{
    "category_id": "root",
    "name": "Electronics",
    "children": [
        {
            "category_id": "subcat1",
            "name": "Mobiles",
            "children": [
                { "category_id": "item1", "name": "Smartphone A" },
                { "category_id": "item2", "name": "Smartphone B" }
            ]
        },
        {
            "category_id": "subcat2",
            "name": "Televisions",
            "children": [
                { "category_id": "item3", "name": "TV 1" },
                { "category_id": "item4", "name": "TV 2" }
            ]
        }
    ]
}
Step 2: Tree Construction

We need to convert this hierarchical structure into a tree data structure. Here is a Python implementation using dictionaries:

PYTHON
class TreeNode:
    def __init__(self, category_id, name):
        self.category_id = category_id
        self.name = name
        self.children = []
        self.features = {}

def build_tree(data):
    root = TreeNode("root", "Electronics")
    nodes = {root.category_id: root}

    for item in data.get('children', []):
        node_id = item['category_id']
        node_name = item['name']

        if node_id == 'root':
            continue

        parent_id = list(item.keys())[0]
        parent_node = nodes[parent_id]

        new_node = TreeNode(node_id, node_name)
        parent_node.children.append(new_node)
        nodes[node_id] = new_node

    return root
Step 3: Traversal and Feature Rollup

To traverse the tree and perform feature rollups, we can use a depth-first search (DFS) approach:

PYTHON
def dfs(node):
    for child in node.children:
        dfs(child)

    # Perform feature rollup here
    total_features = sum(child.features['total'] for child in node.children if 'total' in child.features)
    node.features['total'] = total_features

tree_root = build_tree(your_hierarchy_data)
dfs(tree_root)

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Not correctly handling the root node, which does not have a parent.

#### Follow-Up
Discuss edge cases like empty nodes or cycles in the hierarchy.

By constructing a tree from hierarchical data and traversing it to aggregate features, we can effectively manage complex categorical hierarchies for various analyses.

**Code:**

```python
class TreeNode:
    def __init__(self, category_id, name):
        self.category_id = category_id
        self.name = name
        self.children = []
        self.features = {}

def build_tree(data):
    root = TreeNode("root", "Electronics")
    nodes = {root.category_id: root}

    for item in data.get('children', []):
        node_id = list(item.keys())[0]
        node_name = item[node_id]['name']

        if node_id == 'root':
            continue

        parent_id = list(item.keys())[0]
        parent_node = nodes[parent_id]

        new_node = TreeNode(node_id, node_name)
        parent_node.children.append(new_node)
        nodes[node_id] = new_node

    return root

def dfs(node):
    for child in node.children:
        dfs(child)

    # Perform feature rollup here
    total_features = sum(child.features['total'] for child in node.children if 'total' in child.features)
    node.features['total'] = total_features
```

**Example input / output:**

```text
tree_root = build_tree(your_hierarchy_data)
dfs(tree_root)

# Example features dictionary after rollup
print(tree_root.features)  # Output: {'total': 4} (assuming each item has a 'total' feature of 1)
```

**Data:**

```json
{
    "category_id": "root",
    "name": "Electronics",
    "children": [
        {
            "subcat1": {
                "name": "Mobiles",
                "children": [
                    { "item1": { "name": "Smartphone A" } },
                    { "item2": { "name": "Smartphone B" } }
                ]
            },
            "subcat2": {
                "name": "Televisions",
                "children": [
                    { "item3": { "name": "TV 1" } },
                    { "item4": { "name": "TV 2" } }
                ]
            }
        }
    ]
}
```

