---
type: "practice"
difficulty: "Deep"
---

# Your API returns 'nearest available driver' and the naive linear scan over all drivers is too slow at city scale; compare a bucketed grid versus a k-d tree for a workload with high driver-position update churn.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spatial Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Spatial_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where we need to find the nearest available driver in a large, dynamic dataset, both the bucketed grid and the k-d tree offer different trade-offs. The key idea is that while a naive linear scan over all drivers would be too slow for city-scale data, using a spatial data structure can significantly improve performance. However, given high driver-position update churn, we need to consider not just the search time but also the cost of maintaining these structures.

Triaging the Problem

The core question is: Which spatial data structure—bucketed grid or k-d tree—is more suitable for a workload with frequent updates and large-scale city data? The interviewer is probing your understanding of spatial data structures, their trade-offs in terms of complexity, update costs, and query performance. We will assume that the drivers' positions are updated frequently, and we need to find the nearest available driver quickly.

#### First-Principles Solution
Bucketed Grid

#### Overview
A bucketed grid divides the city into small cells (buckets) and stores a list of drivers in each cell.
Insertion/Deletion Complexity:
Inserting or deleting a driver is (O(1)), as it involves updating one or more buckets.
Search Complexity:
To find the nearest available driver, we first identify the bucket containing the driver's position and then search the neighboring buckets.
Update Churn: The frequent insertion/deletion of drivers can be efficiently handled by keeping a linked list for each bucket.
k-d Tree

#### Overview
A k-d tree is a binary tree where each node represents a hyperplane that splits the space into two regions, one on the left and one on the right.
Insertion/Deletion Complexity:
Inserting or deleting a driver involves traversing the tree and possibly rebalancing it, which can be expensive ((O(\log n)) to (O(n))).
Search Complexity:
Finding the nearest available driver involves a search through the k-d tree, which is generally faster than linear scan but can degrade to (O(n)) in the worst case.
Update Churn: Frequent updates are more costly due to potential rebalancing.

#### Closing Summary

Given high driver-position update churn, the bucketed grid appears more suitable because it maintains a simpler structure with efficient insertions and deletions, which can handle frequent changes without significant overhead. The k-d tree, while offering better search performance in some cases, has higher complexity for updates and may not scale as well under high churn.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that a k-d tree will always outperform a bucketed grid due to its inherent efficiency.

#### Follow-Up
Discussing the impact of spatial dimensions (e.g., 2D vs. 3D) and how it affects both structures, as well as considering hybrid approaches like using a combination of buckets and k-d trees.

**Comparison:**

```
Structure	Insert/Update Complexity	Search Complexity
Bucketed Grid	O(1) per update	O(sqrt(n)) + O(k) for search in neighbors
k-d Tree	O(log n) to O(n)	O(log n) to O(n)
```

**Code:**

```
Example code for bucketed grid insertion

def insert_driver(bucket_grid, driver_position): cell = get_cell(bucket_grid, driver_position) cell.append(driver_position)

def remove_driver(bucket_grid, driver_position): cell = get_cell(bucket_grid, driver_position) if driver_position in cell: cell.remove(driver_position)

CODE
Copy

```python
# Example code for k-d tree insertion
class KdNode:
    def __init__(self, point, left=None, right=None):
        self.point = point
        self.left = left
        self.right = right

def insert_kd_tree(kd_root, new_point):
    if not kd_root:
        return KdNode(new_point)

    current_node = kd_root
    while True:
        prev_node = current_node
        axis = len(current_node.point) // 2
        if new_point[axis] < current_node.point[axis]:
            current_node = current_node.left
        else:
            current_node = current_node.right

        if not current_node:
            break

    if new_point[axis] < prev_node.point[axis]:
        prev_node.left = insert_kd_tree(prev_node.left, new_point)
    else:
        prev_node.right = insert_kd_tree(prev_node.right, new_point)

    return kd_root
```

**Example input / output:**

```text
# Input: List of drivers and their positions
drivers = [
    {"latitude": 40.759628, "longitude": -73.981852},
    {"latitude": 40.759630, "longitude": -73.981850}
]

# Output: Nearest available driver
nearest_driver = find_nearest_driver(drivers)
print(nearest_driver)  # Example output: {"latitude": 40.759628, "longitude": -73.981852}
```

**Data:**

```json
{
  "buckets": [
    {"id": 1, "drivers": [{"latitude": 40.759628, "longitude": -73.981852}]},
    {"id": 2, "drivers": []}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Driver Position Update
4. Is Cell Changed?
5. Update Bucket
6. D
7. Return Updated Driver List
8. Rebalance Tree

