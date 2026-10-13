---
type: "practice"
difficulty: "Deep"
---

# Your ANN serving must support deletes (items go out of stock) without a full rebuild; how do tombstones and periodic compaction differ algorithmically from removing a node inline, and what recall drift do tombstones cause?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of supporting deletions in an ANN serving system without a full rebuild, we need to understand the differences between using tombstones, periodic compaction, and inline node removals. The key concepts here are data recall (accuracy) vs. computational efficiency.

Tombstones and periodic compaction offer a balance between maintaining high recall accuracy and avoiding frequent recomputations, whereas inline node removal can lead to immediate accuracy loss but might be more efficient in certain contexts.

First, let's clarify what these terms mean:

Tombstone: A tombstone is a marker indicating that an item has been deleted from the model without physically removing it. This allows us to keep the model structure consistent.
Periodic Compaction: This involves periodically cleaning up and reorganizing the model, which can help reduce storage overhead and improve performance by removing outdated or redundant tombstones.
Inline Node Removal: Directly modifying the model by deleting nodes as they go out of stock.

#### Triaging the Question

The question is essentially probing your understanding of how different deletion strategies impact model accuracy (recall) and computational efficiency. We need to consider the trade-offs between these factors, especially in a machine learning context where recall accuracy can directly affect the system's performance.

#### First-Principles Solution
Tombstones

Using tombstones involves marking items as deleted but not removing them from the data structure. This approach helps maintain consistent model structures and avoids the overhead of retraining or rebuilding the entire model.

Key Points:

Recall Drift: Tombstones can introduce recall drift because some items might be incorrectly classified due to their presence in the model.
Accuracy vs. Overhead: While tombstones help preserve accuracy, they add complexity in terms of storage and computational overhead for maintaining these markers.
Periodic Compaction

Periodic compaction involves cleaning up and reorganizing the data structure at regular intervals. This process can significantly reduce storage requirements and improve performance by removing outdated or redundant tombstones.

Key Points:

Recall Drift: Compaction itself doesn't directly introduce recall drift, but it might if the compaction logic is not perfectly aligned with the model's training.
Efficiency vs. Recall Accuracy: Periodic compaction can help maintain a balance between efficiency and accuracy by reducing storage overhead while minimizing impact on recall.
Inline Node Removal

Inline node removal involves immediately modifying the data structure to reflect deletions, which can be more efficient but might cause immediate accuracy issues as nodes are removed from the model.

Key Points:

Recall Drift: This approach introduces significant recall drift because it directly removes items from the model without any intermediate state.
Efficiency: Inline removal is generally more efficient and avoids the overhead of maintaining tombstones, but it can lead to immediate accuracy loss.

#### Closing Summary

Tombstones offer a balance between maintaining high recall accuracy and avoiding frequent recomputations, making them suitable for scenarios where accuracy is critical. Periodic compaction helps manage storage overhead by cleaning up outdated tombstones, providing a compromise between efficiency and accuracy. Inline node removal is the most efficient but can introduce significant recall drift due to immediate node removals.

#### Common Mistakes/Follow-Ups
Ignoring Compaction Logic: Failing to consider how compaction logic can impact model accuracy.
Overlooking Storage Overhead: Ignoring the storage overhead associated with maintaining tombstones and its potential impact on system performance.
Not Addressing Recall Drift: Focusing solely on computational efficiency without considering the implications for recall accuracy.

**Comparison:**

```
Deletion Strategy	Recall Drift	Computational Efficiency
Tombstones	High	Moderate
Periodic Compaction	Low to Medium	High
Inline Removal	High	Very High
```

**Code:**

```python
# Example of implementing a simple tombstone mechanism in Python

class ANNModel:
    def __init__(self):
        self.items = {}  # Dictionary to store items and their states (alive or tombstoned)
        self.tombstones = set()  # Set to track tombstoned items

    def delete_item(self, item_id):
        if item_id in self.items:
            self.tombstones.add(item_id)  # Mark the item as deleted with a tombstone
            del self.items[item_id]

    def periodic_compaction(self):
        to_remove = self.tombstones.copy()  # Temporary set to avoid modifying during iteration
        for id in to_remove:
            if id in self.items:
                del self.items[id]  # Clean up the tombstoned items
```

**Example input / output:**

```text
Input:
- ANNModel.delete_item("item2")

Output:
- items = {"item1": true, "item3": true}
- tombstones = set()
```

**Data:**

```json
{
    "items": {
        "item1": true,
        "item2": false,  // Tombstone marker
        "item3": true
    },
    "tombstones": ["item2"]
}
```

**Flow:**

1. Delete Item
2. Tombstone Mark
3. Periodic Compaction Cleanup
4. Inline Removal
5. Clean Up Tombstones

