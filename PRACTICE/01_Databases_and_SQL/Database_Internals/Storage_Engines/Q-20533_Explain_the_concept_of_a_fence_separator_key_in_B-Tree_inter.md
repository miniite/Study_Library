---
type: "practice"
difficulty: "Basic"
---

# Explain the concept of a fence / separator key in B-Tree internal nodes and how a B+Tree keeps all values in leaves with linked leaf pages. Why does this matter specifically for range scans versus a plain B-Tree?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree vs LSM Storage Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a B-Tree, the concept of a fence or separator key is crucial for efficient range queries and maintaining a well-balanced structure. A B+Tree extends this idea by keeping all data values in leaf nodes, which are linked together into a doubly-linked list. This arrangement significantly enhances performance during range scans.

Firstly, let's break down what these concepts entail:

Fence Key in B-Tree Internal Nodes:

In a B-Tree, internal nodes contain keys that act as fences or separators. Each key acts as the upper limit for one of its child sub-trees.
For example, if an internal node has keys [20, 35, 50], it means:
Values less than or equal to 20 are in the leftmost subtree.
Values between 20 and 35 are in the middle subtree.
Values between 35 and 50 are in the right subtree.
And so on.

Leaf Nodes in B+Tree:

Unlike B-Trees, all data values (keys) reside in leaf nodes of a B+Tree.
Leaf nodes form a doubly-linked list, allowing efficient traversal between them for range queries and sequential access.
This structure ensures that the tree remains balanced, even as more or fewer entries are inserted or deleted.
Why This Matters Specifically for Range Scans

The key difference in how these structures handle range scans is significant:

B-Tree:

During a range scan, you might start at an internal node and traverse multiple paths based on the fence keys.
For instance, if scanning from 25 to 40, you might visit several internal nodes before reaching leaf nodes.
This can be less efficient because it requires traversing more tree levels.

B+Tree:

With a B+Tree, range scans start at the root and directly traverse through the relevant leaf nodes via the linked list.
For example, scanning from 25 to 40 involves visiting only those leaf nodes that fall within this range, leveraging the linked structure for quick navigation.
Detailed Explanation
B-Tree Example:

#### Cons
ider a simple B-Tree with keys [10, 30, 50] in an internal node and values (key, data) pairs like [(20, "A"), (40, "B")] in the leaf nodes. If you perform a range scan between 25 to 35:

Start at the root.
Compare with the fence key 30; since 25 < 30, go left.
Reach another internal node: keys [20, 40]; compare again and find 25 falls in the range between 20 and 40.
Move to the leaf nodes containing values (20, "A") and (40, "B").
B+Tree Example:

For a B+Tree with keys [10, 30, 50] spread across internal nodes and data in the leaf nodes:

Start at the root.
Compare with the fence key 30; since 25 < 30, go left to another node containing values (20, "A") and (40, "B").
Traverse directly through these leaf nodes using links.

#### Closing Summary

The concept of fences in B-Trees ensures efficient range scans by leveraging internal node keys for quick traversal, while B+Trees optimize this process further by placing all data in leaves and linking them into a sequential list. This setup minimizes the number of nodes accessed during a range scan, enhancing overall performance.

#### Common Mistakes/Follow-Ups
Misunderstanding Key Placement: Failing to grasp that fence keys in internal nodes guide traversal but do not contain actual values.
Leaf Node Misconception: Thinking B-Trees also have all data in leaves can lead to confusion about their structure and range scan performance.

By understanding these concepts, you ensure your database design or query optimization strategies are as efficient as possible for various types of queries.

