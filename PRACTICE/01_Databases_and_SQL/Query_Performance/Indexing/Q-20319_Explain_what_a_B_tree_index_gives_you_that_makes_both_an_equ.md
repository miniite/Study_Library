---
type: "practice"
difficulty: "Basic"
---

# Explain what a B+ tree index gives you that makes both an equality lookup and a date-range scan fast in a warehouse table.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ B-tree and BRIN Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/B-tree_and_BRIN_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, a B+ tree index is a data structure used to efficiently manage and search large datasets, particularly within warehouse tables where both exact lookups and range queries are common. The core idea of B+ trees is to balance the load on disk accesses, minimizing I/O operations which are relatively slow compared to in-memory operations.

The question asks specifically about how a B+ tree index facilitates fast equality lookups and date-range scans (range queries) in warehouse tables. To understand this, we need to break down what makes B+ trees so effective for these types of queries.

**Comparison:**

```
Level	Node Type	Keys	Children
Leaf
Non-leaf		[k1, k2]	[c1, c2, c3]
```

**Code:**

```python
# Example code snippet to illustrate the structure of a simple B+ tree node
class BPlusTreeNode:
    def __init__(self, level):
        self.keys = []
        self.children = []
        self.is_leaf = False  # Indicates if it's a leaf or an internal node

    def insert_key(self, key):
        # Logic to insert keys and maintain the B+ tree properties
        pass

# Example of how a B+ tree might be constructed for equality lookups
class EqualityLookupBPlusTree:
    def __init__(self, max_degree=4):
        self.root = BPlusTreeNode(max_degree)

    def insert(self, key, value):
        # Insert logic here
        pass

    def search(self, key):
        current_node = self.root
        while not isinstance(current_node, BPlusTreeNode) and len(current_node.keys) > 0:
            if key <= current_node.keys[0]:
                return current_node.get_value(key)
            else:
                next_node = current_node.children[1]
                # Continue down the tree
```

**Example input / output:**

```text
Input:
- Query 1: Find all sales for product_id=42 (equality lookup)
- Query 2: List sales between '2023-01-01' and '2023-12-31' (range scan)

Output for Query 1:
- Efficiently locates the node containing entries with product_id=42, reads only relevant entries.

Output for Query 2:
- Begins from the root, traverses to leaf nodes that contain sale_date within the range and collects matching records.
```

**Data:**

```json
{
    "table": {
        "name": "sales_data",
        "columns": [
            {"name": "order_id", "type": "int"},
            {"name": "customer_name", "type": "text"},
            {"name": "product_id", "type": "int"},
            {"name": "sale_date", "type": "timestamp"}
        ],
        "index": {
            "name": "sales_index",
            "columns": ["product_id", "sale_date"],
            "type": "B+ tree"
        }
    }
}
```

**Flow:**

1. Yes
2. No
3. Root Node
4. Is Leaf?
5. Leaf Node
6. Non-leaf Node
7. Insert Key/Range Query

