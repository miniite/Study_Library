---
type: "practice"
difficulty: "Deep"
---

# A canvas-based diagram editor slows down as nodes grow; what spatial index would you use for hit-testing and viewport culling so pan/zoom stays smooth at thousands of shapes?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spatial Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Spatial_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of a canvas-based diagram editor slowing down with an increasing number of nodes, we need to focus on optimizing hit-testing and viewport culling operations. The key challenge here is efficiently determining which nodes are visible within the current viewport and handling user interactions like clicking or moving around the canvas.

#### Triaging the Question

The question asks for a spatial index that can be used to improve performance when dealing with thousands of shapes on a canvas. Specifically, we need to ensure that hit-testing (determining if a shape is clicked) and viewport culling (identifying which nodes are visible within the current view) remain efficient.

#### Assumptions
The editor uses a 2D canvas where shapes can be positioned arbitrarily.
Shapes have varying sizes and may overlap.
Users perform frequent interactions like clicking or panning.
The goal is to keep performance smooth even with thousands of shapes.

#### First-Principles Solution
Step 1: Understanding the Problem

In a large number of shapes, hit-testing can become expensive as every shape needs to be checked against each click event. Similarly, viewport culling requires identifying all shapes within a certain bounding box, which becomes inefficient if done naively.

Step 2: Spatial Indexing

A spatial index is crucial for efficiently managing and querying a large number of geometric objects. One effective approach here is using a quadtree (or an octree in 3D) to partition the canvas into smaller regions.

Quadtree/Octree: This hierarchical decomposition divides the space into quadrants or octants, respectively.
Node Structure: Each node can store references to shapes that intersect with it. Leaf nodes directly contain shape references, while internal nodes represent sub-regions.
Step 3: Implementing Quadtree for Hit-Testing

When a user clicks on the canvas:

Determine which quadrant contains the click coordinates.
Check if any shapes in that quadrant or its sub-quadrants are clicked.
If no shape is found, recursively search deeper quadrants until a match is found or there are no more sub-regions.
Step 4: Implementing Quadtree for Viewport Culling

When panning or zooming:

Determine the bounding box of the new viewport.
Traverse the quadtree to find all leaf nodes intersecting this bounding box.
Collect and render only those shapes from these leaf nodes that fall within the viewport.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Summary: Using a quadtree can significantly improve performance by partitioning the space into manageable regions, allowing for efficient hit-testing and viewport culling. This approach ensures that we do not need to check every shape individually, thus maintaining smooth user interactions even with thousands of shapes.

#### Common Mistakes

Not Implementing Hierarchical Caching: Failing to use the hierarchical structure effectively can lead to unnecessary traversal.
Overlapping Shapes Handling: Not accounting for overlapping shapes correctly can cause inaccuracies in hit-testing.
Inadequate Node Management: Improper management of nodes can lead to excessive memory usage or inefficient queries.

#### Follow-Ups

How does the choice between a quadtree and an octree impact performance?
What are some variations in implementing the quadtree for non-uniformly sized shapes?
How would you handle scenarios with dynamic shape additions/removals?

**Comparison:**

```
Node Type	Description
Root	Top-level division of the canvas.
Leaf	Contains direct shape references.
Internal	Divides space further into sub-regions.
```

**Code:**

```python
class QuadTreeNode:
    def __init__(self, x, y, width, height):
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.children = [None] * 4  # Quadtree children (or shapes in leaf nodes)
```

**Example input / output:**

```

Hit-Testing Example: Suppose a user clicks at (150, 250).

PLAINTEXT
Copy
Click coordinates: (150, 250)
Current viewport:
x: 0, y: 0, width: 800, height: 600

Traversal path:
- Root node [0, 0, 800, 600] -> Top-right quadrant
- Node [400, 0, 400, 300]
- Node [450, 250, 100, 100] (Shape found)

Result: Shape at (450, 250) is clicked.

Viewport Culling Example: Suppose the user pans to a new viewport with coordinates (300, 300) and size (500x500).

PLAINTEXT
Copy
New viewport:
x: 300, y: 300, width: 500, height: 500

Traversal path:
- Root node [0, 0, 800, 600] -> Top-right quadrant
- Node [400, 0, 400, 300]
- Nodes within viewport range: [425, 325, 150, 150], [475, 325, 200, 200]

Result: Render shapes at (425, 325) and (475, 325).
```

**Data:**

```json
{
    "quadTree": {
        "x": 0,
        "y": 0,
        "width": 800,
        "height": 600,
        "children": [
            {"x": 0, "y": 0, "width": 400, "height": 300},
            {"x": 400, "y": 0, "width": 400, "height": 300},
            {"x": 0, "y": 300, "width": 400, "height": 300},
            {"x": 400, "y": 300, "width": 400, "height": 300}
        ]
    }
}
```

**Flow:**

1. Yes
2. No
3. Click Coordinates
4. Is Click in Node?
5. Return Shape
6. Traversal to Sub-Region
7. Check Child Nodes
8. Yes
9. No
10. Viewport Coordinates
11. Is Viewport in Node?
12. Render Shapes
13. Traversal to Sub-Region
14. Check Child Nodes

