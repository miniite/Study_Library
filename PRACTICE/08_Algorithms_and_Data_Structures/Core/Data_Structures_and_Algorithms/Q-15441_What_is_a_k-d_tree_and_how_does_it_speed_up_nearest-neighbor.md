---
type: "practice"
difficulty: "Intermediate"
---

# What is a k-d tree and how does it speed up nearest-neighbor search compared to a brute-force scan?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Spatial Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Spatial_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A (k)-dimensional (k-d) tree is a data structure used in computer science, particularly for organizing points in a multidimensional space, such as Euclidean space. It enables efficient nearest-neighbor searches by partitioning the space into regions. In this context, "nearest-neighbor search" refers to finding the closest point or object to a given query point. A brute-force scan involves comparing the query point with every single point in the dataset, which can be very inefficient for large datasets. The (k)-d tree reduces the number of distance calculations needed by organizing points hierarchically and pruning branches that cannot contain the nearest neighbor.

#### Triage
Restating the Question

The core of this question is to explain how a (k)-d tree improves the efficiency of finding the nearest neighbor compared to a brute-force approach. We need to understand the structure of a (k)-d tree, its construction process, and the search mechanism that utilizes these properties.

#### Assumptions
The dataset consists of points in a multidimensional space.
The goal is to find the point closest to a given query point quickly.
Step-by-Step Solution: From First Principles
1. Understanding (k)-d Tree Construction

A (k)-d tree starts with a set of points and recursively splits them into two subsets by partitioning the space along one dimension at each level. At each node, a median is chosen to split the data, ensuring balanced partitions. The splitting axis alternates between dimensions as we go deeper into the tree.

2. Tree Structure
Each internal node in the (k)-d tree represents a hyperplane that splits the space.
Each leaf node contains a subset of points from the dataset.
Internal nodes store the median value along one dimension used to split the data at that level.
3. Search Mechanism

The search for the nearest neighbor involves traversing the tree:

Start at the root and recursively move down the tree based on whether the query point is less than or greater than the splitting hyperplane.
When reaching a leaf node, compute the distance from the query point to all points in that region.
Backtrack through parent nodes, but only consider nodes whose associated hyperplanes are closer than the current best distance found.
4. Efficiency Comparison with Brute-Force Search

A brute-force scan checks the Euclidean distance of each point to the query point, which can be computationally expensive for large datasets. In contrast, a (k)-d tree reduces this by:

Limiting the number of points that need to be checked through effective partitioning.
Reducing redundant calculations by pruning branches.

#### Closing Summary
and Common Mistakes

The key advantage of using a (k)-d tree is its ability to significantly reduce the number of distance computations needed during nearest-neighbor searches. However, it's important to consider:

Balancing: The quality of splits can impact performance; unbalanced trees may degrade efficiency.
Dimensionality Curse: In high-dimensional spaces, the effectiveness of (k)-d trees diminishes due to the concentration of points in a lower-dimensional subspace.

#### Common Mistakes
and Follow-Ups
Misunderstanding the Splitting Process: Overlooking how the tree splits dimensions or not realizing that it alternates between axes.
Ignoring Edge Cases: Failing to consider scenarios where all data lies on one side of a split plane, leading to ineffective partitioning.
Practical Example (Optional)

#### Cons
ider a 2D space with points ((1, 1)), ((3, 3)), and ((5, 5)). A (k)-d tree would first split along the x-axis at median (x = 3), creating two subsets. Then, it would split these subsets by y-values, effectively reducing the number of points that need distance calculations.

**Comparison:**

```
Point	x	y
P1	1	1
P2	3	3
P3	5	5
Query Point	x	y
Q	2	2
Nearest Neighbor	x	y
Closest to Q	3	3
```

**Code:**

```python
class KDTreeNode:
    def __init__(self, point, axis=None):
        self.point = point
        self.axis = axis
        self.left = None
        self.right = None

def build_kd_tree(points, depth=0):
    n = len(points)
    if n == 0:
        return None
    axis = depth % k
    points.sort(key=lambda p: p[axis])
    median_idx = n // 2
    node = KDTreeNode(points[median_idx], axis)
    node.left = build_kd_tree(points[:median_idx], depth + 1)
    node.right = build_kd_tree(points[median_idx+1:], depth + 1)
    return node

def nearest_neighbor(kd_tree, query_point):
    def search(node):
        if not node:
            return None
        best_dist = float('inf')
        closest_point = None
        current_dist = distance(node.point, query_point)

        # Consider the closer branch first
        if current_dist < best_dist:
            closest_point = node.point
            best_dist = current_dist

        axis = node.axis
        split_dim = query_point[axis]
        opposite_axis = (axis + 1) % k

        # Check left and right branches
        if split_dim <= query_point[opposite_axis]:
            closer_branch = node.left
            farther_branch = node.right
        else:
            closer_branch = node.right
            farther_branch = node.left

        closer_node = search(closer_branch)

        # Backtrack if necessary
        if distance(closer_node.point, query_point) > best_dist + (query_point[opposite_axis] - split_dim):
            farther_node = search(farther_branch)
            if distance(farther_node.point, query_point) < best_dist:
                closest_point = farther_node.point
                best_dist = distance(farther_node.point, query_point)

        return closest_point

    k = len(query_point)
    return search(kd_tree)
```

**Example input / output:**

```text
# Sample input data
points = [{"x": 1, "y": 1}, {"x": 3, "y": 3}, {"x": 5, "y": 5}]
query_point = {"x": 2, "y": 2}

# Build k-d tree and find nearest neighbor
kd_tree = build_kd_tree(points)
closest_point = nearest_neighbor(kd_tree, query_point)

print("Closest point:", closest_point)  # Output: Closest point: {'x': 3, 'y': 3}
```

**Data:**

```json
[
  {"x": 1, "y": 1},
  {"x": 3, "y": 3},
  {"x": 5, "y": 5}
]
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is tree empty?
7. Return None
8. Choose split axis
9. Sort points by selected axis
10. Find median point and createnode
11. Split left and right recursively
12. Backtrack if necessary to checkfarther branch
13. Is closer branch closer?
14. Return closest point fromfarther branch
15. Return closest point fromcloser branch
16. End

