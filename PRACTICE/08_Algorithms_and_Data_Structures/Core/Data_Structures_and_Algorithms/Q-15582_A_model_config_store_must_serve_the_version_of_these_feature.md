---
type: "practice"
difficulty: "Deep"
---

# A model config store must serve 'the version of these features as of timestamp T' for reproducible training; compare full snapshots, deltas, and a persistent (immutable) tree for storage and read cost.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirements of storing model configurations in such a way that we can serve 'the version of these features as of timestamp T' for reproducible training, we need to consider three primary storage strategies: full snapshots, deltas, and an immutable tree structure. Each approach has its trade-offs in terms of storage cost and read efficiency.

The key idea is to leverage the immutability provided by a persistent tree data structure while ensuring efficient retrieval of configurations as of any given timestamp ( T ). This problem aligns well with concepts from distributed systems and version control, which manage histories and revisions efficiently. Let's break down the solution step-by-step.

**Comparison:**

```
Approach	Full Snapshots	Deltas	Persistent Tree
Storage Cost	High (all versions stored)	Low to Medium (only differences stored)	Moderate (tree structure)
Read Efficiency for Any ( T )	Unfeasible without additional indexing	Difficult but feasible with careful design	Efficient due to immutable tree structure
```

**Code:**

```python
class ConfigTreeNode:
    def __init__(self, config_id, children=None):
        self.config_id = config_id
        self.children = children if children else {}

    def add_child(self, child_node):
        self.children[child_node.config_id] = child_node

    def get_version_at_timestamp(self, timestamp):
        # Dummy implementation; actual logic depends on the underlying storage system
        pass

class ConfigTreeStore:
    def __init__(self):
        self.root = ConfigTreeNode('root')

    def add_config(self, config_id, config_data):
        new_node = ConfigTreeNode(config_id)
        current_node = self.root
        for key in config_data.keys():
            if key not in current_node.children:
                new_child = ConfigTreeNode(key)
                current_node.add_child(new_child)
                current_node = new_child
            else:
                current_node = current_node.children[key]
        # Assign config data to the leaf node

    def get_config_at_timestamp(self, timestamp):
        return self.root.get_version_at_timestamp(timestamp)

# Example usage:
store = ConfigTreeStore()
store.add_config('model-1', {'layer1': 'configA', 'layer2': 'configB'})
```

**Example input / output:**

```text
store = ConfigTreeStore()
store.add_config('model-1', {'layer1': 'configA', 'layer2': 'configB'})
timestamped_config = store.get_config_at_timestamp("2023-09-01T14:00:00Z")
print(timestamped_config)  # Output will be a config object representing the state at that timestamp
```

**Data:**

```json
{
  "timestamp": "2023-09-01T14:00:00Z",
  "config_data": {
    "model-1": {
      "layer1": "configA",
      "layer2": "configB"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Add Config
6. Is Tree Updated?
7. Update Tree
8. Return Current State
9. Get Version at T
10. Has Tree Node for T?
11. Return Node Data
12. Reconstruct History to FindNearest T

