---
type: "practice"
difficulty: "Other"
---

# Building a Region Quad Tree from a Binary Grid

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a square grid board of size n x n in which every cell holds either 0 or 1. Represent this grid compactly as a quad tree and return its root.

A quad tree is a tree in which every internal node has exactly four children. Each node stores two fields:

CODE
Node {
    boolean val;       // the cell value this node represents (true for 1, false for 0)
    boolean isLeaf;     // true if this node is a leaf, false if it has four children
    Node topLeft;
    Node topRight;
    Node bottomLeft;
    Node bottomRight;
}

Build the tree from the whole grid using the following recursive rule on any square region:

If every cell inside the current square region has the same value, create a leaf node: set isLeaf = true, set val to that common value (true for 1, false for 0), and set all four children to null.
Otherwise, the region is mixed: create an internal node with isLeaf = false (its val may be either value — both are accepted), then split the square into four equal quadrants and recurse on each. Assign the four resulting subtrees to topLeft, topRight, bottomLeft, and bottomRight, where the split is: top-left quadrant, top-right quadrant, bottom-left quadrant, bottom-right quadrant.

Return the root node of the constructed quad tree.

Input

board: an n x n matrix of 0s and 1s, where n is a power of two.

Output

The root Node of the quad tree that encodes board.

Serialized output format (for reference only). A quad tree is often printed via level-order traversal, where each node is shown as the pair [isLeaf, val] (each field printed as 1 for true, 0 for false) and null marks a missing subtree. You do not need this format to solve the problem; it only helps describe the expected trees below.

#### Cons
traints

n == board.length == board[i].length
n == 2^x where 0 <= x <= 6 (so 1 <= n <= 64)
Every board[i][j] is 0 or 1.

#### Example
s & test cases
TEXT
Input: board = [[1,0],[0,1]] -> Output: [[0,1],[1,1],[1,0],[1,0],[1,1]]
Why: the 2x2 region is mixed, so the root is internal ([isLeaf=0, val=1]); its four single-cell quadrants become leaves top-left=1, top-right=0, bottom-left=0, bottom-right=1. No merge happens because the leaf values differ.

Input: board = [[1,1],[1,1]] -> Output: [[1,1]]
Why: every cell is 1, so the whole grid collapses to a single leaf carrying val=true.

Input: board = [[1]] -> Output: [[1,1]]
Why: a 1x1 grid is trivially uniform, producing one leaf. This is the minimum size (n = 2^0 = 1).

Input: board = [[0,0,0,0],[0,0,0,0],[0,0,0,0],[0,0,0,0]] -> Output: [[1,0]]
Why: a fully uniform 4x4 block of zeros merges all the way up to one leaf with val=false.

Input: board = [[1,1,0,0],[1,1,0,0],[0,0,0,0],[0,0,0,0]] -> Output: [[0,1],[1,1],[1,0],[1,0],[1,0]]
Why: the top-left 2x2 quadrant is all 1s (leaf val=1) while the other three quadrants are all 0s (leaf val=0); the values differ so the root stays internal and no further merging occurs.

#### Approach

This is a classic divide-and-conquer / spatial-partitioning problem solved bottom-up. Recurse on a square region identified by its top-left corner (r, c) and its side length size, halving size at each step down to a single cell (which is always a leaf). After building the four quadrant subtrees, apply the compression rule: if all four children are leaves that share the same value, the entire region is uniform, so discard the children and return a single merged leaf carrying that value; otherwise keep them as an internal node. Building bottom-up and merging on the way up is what makes the algorithm O(n^2): each cell contributes to exactly one leaf, and every internal decision is a constant-time comparison of the four child summaries. The alternative top-down check (scanning the whole region for uniformity before splitting) is correct but costs O(n^2 log n) because upper regions get rescanned repeatedly.

Complexity

Time: O(n^2) — every cell of the grid is touched exactly once at the leaf level, and the merge check at each internal node is O(1); the total number of nodes in a quad tree over n^2 cells is O(n^2), so the overall work is linear in the number of cells. Space: O(n^2) worst case for the output tree (a fully checkerboard grid cannot be compressed) plus O(log n) recursion depth.

Python
PYTHON
class Solution:
    def construct(self, board: List[List[int]]) -> 'Node':
        if not board:
            return None

        def build(r: int, c: int, size: int) -> 'Node':
            if size == 1:
                return Node(board[r][c] == 1, True)

            half = size // 2
            tl = build(r, c, half)
            tr = build(r, c + half, half)
            bl = build(r + half, c, half)
            br = build(r + half, c + half, half)

            # Merge four equal-valued sibling leaves into one leaf.
            if (tl.isLeaf and tr.isLeaf and bl.isLeaf and br.isLeaf
                    and tl.val == tr.val == bl.val == br.val):
                return Node(tl.val, True)

            return Node(True, False, tl, tr, bl, br)

        return build(0, 0, len(board))
JavaScript
JAVASCRIPT
function construct(board) {
    if (!board || board.length === 0) return null;

    // Recurse to single cells, then merge four equal-valued sibling leaves.
    function build(r, c, size) {
        if (size === 1) {
            return new Node(board[r][c] === 1, true, null, null, null, null);
        }

#### cons
t half = size >> 1;

#### cons
t tl = build(r, c, half);

#### cons
t tr = build(r, c + half, half);

#### cons
t bl = build(r + half, c, half);

#### cons
t br = build(r + half, c + half, half);

        if (tl.isLeaf && tr.isLeaf && bl.isLeaf && br.isLeaf &&
            tl.val === tr.val && tl.val === bl.val && tl.val === br.val) {
            return new Node(tl.val, true, null, null, null, null);
        }
        return new Node(false, false, tl, tr, bl, br);
    }

    return build(0, 0, board.length);
}
Java
JAVA
class Solution {
  public Node construct(int[][] board) {
    if (board.length == 0) return null;
    return build(board, 0, 0, board.length);
  }

  // Recurse to single cells, then merge four equal-valued sibling leaves.
  private Node build(int[][] board, int r, int c, int size) {
    if (size == 1)
      return new Node(board[r][c] == 1, true);

    int half = size / 2;
    Node tl = build(board, r, c, half);
    Node tr = build(board, r, c + half, half);
    Node bl = build(board, r + half, c, half);
    Node br = build(board, r + half, c + half, half);

    if (tl.isLeaf && tr.isLeaf && bl.isLeaf && br.isLeaf
        && tl.val == tr.val && tl.val == bl.val && tl.val == br.val)
      return new Node(tl.val, true);       // collapse into one leaf

    return new Node(true, false, tl, tr, bl, br);
  }
}
C++
CPP
class Solution {
 public:
  Node* construct(vector<vector<int>>& board) {
    if (board.empty()) return nullptr;
    return build(board, 0, 0, board.size());
  }

 private:
  // Recurse to single cells, then merge four equal-valued sibling leaves.
  Node* build(vector<vector<int>>& board, int r, int c, int size) {
    if (size == 1)
      return new Node(board[r][c] == 1, true);

    int half = size / 2;
    Node* tl = build(board, r, c, half);
    Node* tr = build(board, r, c + half, half);
    Node* bl = build(board, r + half, c, half);
    Node* br = build(board, r + half, c + half, half);

    if (tl->isLeaf && tr->isLeaf && bl->isLeaf && br->isLeaf &&
        tl->val == tr->val && tl->val == bl->val && tl->val == br->val)
      return new Node(tl->val, true);      // collapse into one leaf

    return new Node(true, false, tl, tr, bl, br);
  }
};
C
C
#include <stdlib.h>

struct Node {
    int val;
    int isLeaf;
    struct Node* topLeft;
    struct Node* topRight;
    struct Node* bottomLeft;
    struct Node* bottomRight;
};

/* Allocate a node; children default to NULL. */
static struct Node* makeNode(int val, int isLeaf) {
    struct Node* node = (struct Node*)malloc(sizeof(struct Node));
    node->val = val;
    node->isLeaf = isLeaf;
    node->topLeft = node->topRight = node->bottomLeft = node->bottomRight = NULL;
    return node;
}

/* Build the quad tree bottom-up: recurse to single cells, then merge
   four sibling leaves that agree on their value into one leaf. */
static struct Node* build(int** board, int r, int c, int size) {
    if (size == 1)
        return makeNode(board[r][c], 1);

    int half = size / 2;
    struct Node* tl = build(board, r, c, half);
    struct Node* tr = build(board, r, c + half, half);
    struct Node* bl = build(board, r + half, c, half);
    struct Node* br = build(board, r + half, c + half, half);

    if (tl->isLeaf && tr->isLeaf && bl->isLeaf && br->isLeaf &&
        tl->val == tr->val && tl->val == bl->val && tl->val == br->val) {
        int val = tl->val;
        free(tl); free(tr); free(bl); free(br);
        return makeNode(val, 1);
    }

    struct Node* node = makeNode(1, 0);   /* internal val is arbitrary */
    node->topLeft = tl;
    node->topRight = tr;
    node->bottomLeft = bl;
    node->bottomRight = br;
    return node;
}

struct Node* construct(int** board, int boardSize, int* boardColSize) {
    if (boardSize == 0) return NULL;
    return build(board, 0, 0, boardSize);
}

#### Follow-up
questions
Given two quad trees over grids of the same size, compute the quad tree of their element-wise logical OR (or AND) directly on the trees without expanding them back to full grids.
How would you extend the structure to non-binary grids (arbitrary integers), and when does the compression rule still pay off?
Support a point update: flip a single cell in the underlying grid and repair the tree in O(log n) instead of rebuilding it from scratch.
Analyze the worst-case tree size and describe a grid pattern for which no compression is possible.

