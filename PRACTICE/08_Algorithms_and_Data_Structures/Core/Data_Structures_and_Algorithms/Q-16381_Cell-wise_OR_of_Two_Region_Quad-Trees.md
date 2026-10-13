---
type: "practice"
difficulty: "Other"
---

# Cell-wise OR of Two Region Quad-Trees

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A binary grid is a square matrix of size n x n in which every cell holds either 0 or 1.

Such a grid can be compressed with a quad-tree, a tree in which every internal node has exactly four children. Each node has two boolean attributes:

val: the fill value of the region the node covers — 1 (true) if the whole region is 1s, 0 (false) if the whole region is 0s. For an internal (non-leaf) node val is irrelevant and may be set to either value.
isLeaf: true if the node is a leaf (its region is uniform), false if the node is subdivided into four children.

The node type is:

CODE
Node {
    boolean val;
    boolean isLeaf;
    Node topLeft;
    Node topRight;
    Node bottomLeft;
    Node bottomRight;
}

A quad-tree is built from a square region like this:

If every cell of the current region has the same value, create a leaf: set isLeaf = true, set val to that shared value, and set all four children to null.
Otherwise, set isLeaf = false, split the region into four equal quadrants — topLeft, topRight, bottomLeft, bottomRight (each of side length half the current side) — and recurse to build each child.

You are given the roots of two quad-trees, gridA and gridB, each encoding an n x n binary grid of the same size. Return the root of a quad-tree that encodes the grid obtained by taking the logical OR of the two grids cell by cell (result cell is 1 if either input cell is 1).

The returned tree must itself be a valid quad-tree: any region that ends up uniform must be represented as a single leaf rather than four identical leaves. When isLeaf is false you may set val to either value; both are accepted.

Serialization (for the examples). A quad-tree is serialized by level-order traversal; null marks a missing child. Each node is written as a list [isLeaf, val], where a true boolean is shown as 1 and false as 0.

#### Cons
traints

gridA and gridB are both valid quad-trees, each representing an n x n grid.
n == 2^x where 0 <= x <= 9 (so n is between 1 and 512).
Both trees represent grids of the same dimension n.

#### Example
s & test cases
TEXT
Input: gridA = [[1,1]], gridB = [[1,0]] -> Output: [[1,1]]
Why: Both are 1x1 leaves. OR of a full-1 grid with a full-0 grid is the full-1 grid, returned as a single leaf.

Input: gridA = [[1,0]], gridB = [[1,0]] -> Output: [[1,0]]
Why: Two 1x1 all-zero grids; the OR is all zero, a single 1x1 leaf with value 0.

Input: gridA = [[1,1]], gridB = [[0,1],[1,1],[1,0],[1,0],[1,1]] -> Output: [[1,1]]
Why: gridA is a leaf of all 1s, so the OR is all 1s regardless of gridB; short-circuit to gridA's leaf.

Input: gridA = [[1,0]], gridB = [[0,1],[1,1],[1,0],[1,0],[1,1]] -> Output: [[0,1],[1,1],[1,0],[1,0],[1,1]]
Why: gridA is all 0s, so OR equals gridB exactly; the other tree is returned unchanged.

Input: gridA = [[0,1],[1,1],[1,0],[1,0],[1,0]], gridB = [[0,1],[1,0],[1,1],[1,0],[1,0]] -> Output: [[1,1]]
Why: Each quadrant of the OR becomes 1 (topLeft 1|1=1, topRight 0|1=1, bottomLeft/bottomRight 1|... =1); all four child leaves are 1, so they collapse into one uniform leaf of value 1.

Input: gridA = [[0,1],[1,0],[1,0],[1,1],[1,1]], gridB = [[0,1],[1,0],[1,0],[1,0],[1,0]] -> Output: [[0,1],[1,0],[1,0],[1,1],[1,1]]
Why: topLeft stays 0, topRight stays 0, bottomLeft stays 0, bottomRight 1|0=1; the four child values differ so no collapse — the internal node with its four leaves is kept.

#### Approach

Divide and conquer over the two trees in lockstep. At each step compare the two corresponding regions: if either node is a leaf, the OR short-circuits — a leaf whose value is 1 forces the entire region to 1 (return that leaf), and a leaf whose value is 0 contributes nothing (return the other region's subtree). If neither is a leaf, recurse on the four aligned quadrants and combine the results. Finally, if all four returned children are leaves carrying the same value, the merged region is uniform, so collapse them into a single leaf to keep the output a canonical/minimal quad-tree; otherwise build an internal node holding the four children. The pattern is recursive tree merging with a bottom-up compaction step.

Complexity

Time: O(m) where m is the total number of nodes visited, bounded by the size of the smaller/finer tree and at most O(n^2) in the worst case (a fully detailed n x n grid has on the order of n^2 leaves). Each pair of corresponding nodes is examined once, and the four-leaf collapse check is O(1). Space: O(x) = O(log n) for the recursion stack (the tree height, at most 9 here), plus O(m) for the newly allocated result nodes.

Python
PYTHON
class Node:
    def __init__(self, val=False, isLeaf=False,
                 topLeft=None, topRight=None, bottomLeft=None, bottomRight=None):
        self.val = val
        self.isLeaf = isLeaf
        self.topLeft = topLeft
        self.topRight = topRight
        self.bottomLeft = bottomLeft
        self.bottomRight = bottomRight

class Solution:
    def logicalOr(self, a: 'Node', b: 'Node') -> 'Node':
        # A uniform leaf short-circuits the OR.
        if a.isLeaf:
            return a if a.val else b
        if b.isLeaf:
            return b if b.val else a

        tl = self.logicalOr(a.topLeft, b.topLeft)
        tr = self.logicalOr(a.topRight, b.topRight)
        bl = self.logicalOr(a.bottomLeft, b.bottomLeft)
        br = self.logicalOr(a.bottomRight, b.bottomRight)

        # Merge four identical leaves into one.
        if (tl.isLeaf and tr.isLeaf and bl.isLeaf and br.isLeaf and
                tl.val == tr.val == bl.val == br.val):
            return Node(tl.val, True)

        return Node(False, False, tl, tr, bl, br)
JavaScript
JAVASCRIPT
class Node {

#### cons
tructor(val, isLeaf, topLeft = null, topRight = null,
                bottomLeft = null, bottomRight = null) {
        this.val = val;
        this.isLeaf = isLeaf;
        this.topLeft = topLeft;
        this.topRight = topRight;
        this.bottomLeft = bottomLeft;
        this.bottomRight = bottomRight;
    }
}

function logicalOr(a, b) {
    // A uniform leaf short-circuits the OR.
    if (a.isLeaf) return a.val ? a : b;
    if (b.isLeaf) return b.val ? b : a;

#### cons
t tl = logicalOr(a.topLeft,     b.topLeft);

#### cons
t tr = logicalOr(a.topRight,    b.topRight);

#### cons
t bl = logicalOr(a.bottomLeft,  b.bottomLeft);

#### cons
t br = logicalOr(a.bottomRight, b.bottomRight);

    // Merge four identical leaves into one.
    if (tl.isLeaf && tr.isLeaf && bl.isLeaf && br.isLeaf &&
        tl.val === tr.val && tl.val === bl.val && tl.val === br.val) {
        return new Node(tl.val, true);
    }

    return new Node(false, false, tl, tr, bl, br);
}
Java
JAVA
class Node {
    public boolean val;
    public boolean isLeaf;
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;

    public Node() {}

    public Node(boolean val, boolean isLeaf) {
        this.val = val;
        this.isLeaf = isLeaf;
    }

    public Node(boolean val, boolean isLeaf,
                Node topLeft, Node topRight, Node bottomLeft, Node bottomRight) {
        this.val = val;
        this.isLeaf = isLeaf;
        this.topLeft = topLeft;
        this.topRight = topRight;
        this.bottomLeft = bottomLeft;
        this.bottomRight = bottomRight;
    }
}

class Solution {
    public Node logicalOr(Node a, Node b) {
        // A uniform leaf short-circuits the OR.
        if (a.isLeaf) return a.val ? a : b;
        if (b.isLeaf) return b.val ? b : a;

        Node tl = logicalOr(a.topLeft,     b.topLeft);
        Node tr = logicalOr(a.topRight,    b.topRight);
        Node bl = logicalOr(a.bottomLeft,  b.bottomLeft);
        Node br = logicalOr(a.bottomRight, b.bottomRight);

        // Merge four identical leaves into one.
        if (tl.isLeaf && tr.isLeaf && bl.isLeaf && br.isLeaf &&
            tl.val == tr.val && tl.val == bl.val && tl.val == br.val)
            return new Node(tl.val, true);

        return new Node(false, false, tl, tr, bl, br);
    }
}
C++
CPP
class Node {
public:
    bool val;
    bool isLeaf;
    Node* topLeft;
    Node* topRight;
    Node* bottomLeft;
    Node* bottomRight;
    Node(bool v, bool leaf,
         Node* tl = nullptr, Node* tr = nullptr,
         Node* bl = nullptr, Node* br = nullptr)
        : val(v), isLeaf(leaf),
          topLeft(tl), topRight(tr), bottomLeft(bl), bottomRight(br) {}
};

class Solution {
public:
    Node* logicalOr(Node* a, Node* b) {
        // A uniform leaf short-circuits the OR.
        if (a->isLeaf) return a->val ? a : b;
        if (b->isLeaf) return b->val ? b : a;

        Node* tl = logicalOr(a->topLeft,     b->topLeft);
        Node* tr = logicalOr(a->topRight,    b->topRight);
        Node* bl = logicalOr(a->bottomLeft,  b->bottomLeft);
        Node* br = logicalOr(a->bottomRight, b->bottomRight);

        // Merge four identical leaves into one.
        if (tl->isLeaf && tr->isLeaf && bl->isLeaf && br->isLeaf &&
            tl->val == tr->val && tl->val == bl->val && tl->val == br->val)
            return new Node(tl->val, true);

        return new Node(false, false, tl, tr, bl, br);
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>

struct Node {
    bool val;
    bool isLeaf;
    struct Node* topLeft;
    struct Node* topRight;
    struct Node* bottomLeft;
    struct Node* bottomRight;
};

static struct Node* newLeaf(bool val) {
    struct Node* n = (struct Node*)malloc(sizeof(struct Node));
    n->val = val;
    n->isLeaf = true;
    n->topLeft = n->topRight = n->bottomLeft = n->bottomRight = NULL;
    return n;
}

static struct Node* newInternal(struct Node* tl, struct Node* tr,
                                struct Node* bl, struct Node* br) {
    struct Node* n = (struct Node*)malloc(sizeof(struct Node));
    n->val = false;
    n->isLeaf = false;
    n->topLeft = tl;
    n->topRight = tr;
    n->bottomLeft = bl;
    n->bottomRight = br;
    return n;
}

struct Node* logicalOr(struct Node* a, struct Node* b) {
    /* If one region is a uniform leaf, OR short-circuits:
       a 1-leaf dominates; a 0-leaf yields the other subtree. */
    if (a->isLeaf) return a->val ? a : b;
    if (b->isLeaf) return b->val ? b : a;

    struct Node* tl = logicalOr(a->topLeft,     b->topLeft);
    struct Node* tr = logicalOr(a->topRight,    b->topRight);
    struct Node* bl = logicalOr(a->bottomLeft,  b->bottomLeft);
    struct Node* br = logicalOr(a->bottomRight, b->bottomRight);

    /* Collapse four identical leaves back into a single leaf. */
    if (tl->isLeaf && tr->isLeaf && bl->isLeaf && br->isLeaf &&
        tl->val == tr->val && tl->val == bl->val && tl->val == br->val)
        return newLeaf(tl->val);

    return newInternal(tl, tr, bl, br);
}

#### Follow-up
questions
Implement the logical AND of the two grids instead of OR (now a 0-leaf short-circuits and a 1-leaf defers to the other subtree).
Support XOR, and explain why XOR cannot short-circuit on a single leaf the way OR and AND can.
Given a quad-tree, reconstruct and print the underlying n x n binary matrix.
Build a quad-tree from a raw n x n matrix, and discuss when the quad-tree is smaller than the matrix versus when it is larger.

