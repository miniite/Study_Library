---
type: "practice"
difficulty: "Other"
---

# Bidirectional In-Order Cursor over a Sorted Binary Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary search tree (a binary tree in which, for every node, all values in its left subtree are smaller than the node's value and all values in its right subtree are larger). Design a cursor object, OrderedCursor, that walks the in-order sequence of the tree (the values visited left-subtree, then node, then right-subtree — which for a BST is exactly the values in ascending order) and can move in both directions.

When the cursor is created it is positioned at a virtual slot just before the first (smallest) element — a position that is smaller than every value in the tree. Because of this, the very first forward move lands on the smallest value.

Implement the following operations:

OrderedCursor(root) — build the cursor from the tree root and place it before the first element.
hasNext() — return true if there is at least one element to the right of the cursor, otherwise false.
next() — move the cursor one step to the right and return the value now under the cursor.
hasPrev() — return true if there is at least one element to the left of the cursor, otherwise false.
prev() — move the cursor one step to the left and return the value now under the cursor.

You may assume next() is only called when hasNext() would be true, and prev() is only called when hasPrev() would be true.

#### Cons
traints:

The number of nodes in the tree is in the range [1, 100000].
0 <= node value <= 1000000.
At most 100000 calls in total will be made across hasNext, next, hasPrev, and prev.

Follow up: Solve it without materializing the entire in-order traversal up front — expand the tree lazily as the cursor advances.

#### Example
s & test cases
TEXT
- Input: tree with in-order sequence [4, 8, 11, 18, 25]; ops = [next, next, prev, next, hasNext, next, next, next, hasNext, hasPrev, prev, prev] -> Output: [4, 8, 4, 8, true, 11, 18, 25, false, true, 18, 11]
  Why: interleaves forward/backward moves; after reaching the last value hasNext is false while hasPrev is true.
- Input: tree = single node [42]; ops = [hasPrev, hasNext, next, hasNext, hasPrev] -> Output: [false, true, 42, false, true]
  Why: edge case of one element — before-first has no prev but one next; after next, no next but a prev exists.
- Input: tree with in-order [1, 2]; ops = [next, next, prev, hasPrev] -> Output: [1, 2, 1, true]
  Why: minimal two-element case exercising a backward move.
- Input: right-skewed tree (root 1 -> right 2 -> right 3), in-order [1, 2, 3]; ops = [next, next, next, hasNext, prev, next] -> Output: [1, 2, 3, false, 2, 3]
  Why: verifies lazy expansion along a right spine and that re-advancing after a prev returns already-materialized values.

#### Approach

Use the classic controlled iterative in-order traversal (a stack holding the left spine) but make it resumable and reversible. Keep an array arr of values already emitted, plus a pointer index into it (starting at -1, i.e. before the first element). A forward next() only touches the tree when the pointer sits at the frontier of what has been materialized: pop the top of the stack, append its value, and push the left spine of its right child — the standard in-order step. Otherwise the value was already produced, so next() just advances the pointer. prev() never touches the tree; it simply decrements the pointer into the remembered array. This satisfies the follow-up: the traversal is expanded lazily, one node at a time, rather than precomputed.

Complexity

Time: next() and prev() are amortized O(1) — each node is pushed onto and popped from the stack exactly once over the whole run, so all forward moves together cost O(n); hasNext()/hasPrev() are O(1). Space: O(n) for the materialized value list plus O(h) for the pending stack (h = tree height), i.e. O(n) overall. This is optimal because backward movement requires remembering already-visited values.

Python
PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class OrderedCursor:
    def __init__(self, root: 'TreeNode'):
        self._stack = []       # pending nodes not yet emitted
        self._arr = []         # materialized in-order values
        self._pointer = -1     # index into _arr; -1 == before first
        self._push_left(root)

    def _push_left(self, node):
        while node:
            self._stack.append(node)
            node = node.left

    def hasNext(self) -> bool:
        return self._pointer < len(self._arr) - 1 or bool(self._stack)

    def next(self) -> int:
        if self._pointer == len(self._arr) - 1:   # frontier: expand lazily
            node = self._stack.pop()
            self._arr.append(node.val)
            self._push_left(node.right)
        self._pointer += 1
        return self._arr[self._pointer]

    def hasPrev(self) -> bool:
        return self._pointer > 0

    def prev(self) -> int:
        self._pointer -= 1
        return self._arr[self._pointer]
JavaScript
JAVASCRIPT
class TreeNode {

#### cons
tructor(val = 0, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

class OrderedCursor {

#### cons
tructor(root) {
        this.stack = [];       // pending nodes not yet emitted
        this.arr = [];         // materialized in-order values
        this.pointer = -1;     // index into arr; -1 == before first
        this._pushLeft(root);
    }

    _pushLeft(node) {
        while (node) {
            this.stack.push(node);
            node = node.left;
        }
    }

    hasNext() {
        return this.pointer < this.arr.length - 1 || this.stack.length > 0;
    }

    next() {
        if (this.pointer === this.arr.length - 1) {   // frontier: expand lazily

#### cons
t node = this.stack.pop();
            this.arr.push(node.val);
            this._pushLeft(node.right);
        }
        this.pointer++;
        return this.arr[this.pointer];
    }

    hasPrev() {
        return this.pointer > 0;
    }

    prev() {
        this.pointer--;
        return this.arr[this.pointer];
    }
}
Java
JAVA
import java.util.*;

class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val; this.left = left; this.right = right;
    }
}

class OrderedCursor {
    private final Deque<TreeNode> stack = new ArrayDeque<>();
    private final List<Integer> arr = new ArrayList<>();
    private int pointer = -1;   // index into arr; -1 == before first

    public OrderedCursor(TreeNode root) {
        pushLeft(root);
    }

    private void pushLeft(TreeNode node) {
        while (node != null) {
            stack.push(node);
            node = node.left;
        }
    }

    public boolean hasNext() {
        return pointer < arr.size() - 1 || !stack.isEmpty();
    }

    public int next() {
        if (pointer == arr.size() - 1) {   // at the frontier: expand lazily
            TreeNode node = stack.pop();
            arr.add(node.val);
            pushLeft(node.right);
        }
        pointer++;
        return arr.get(pointer);
    }

    public boolean hasPrev() {
        return pointer > 0;
    }

    public int prev() {
        pointer--;
        return arr.get(pointer);
    }
}
C++
CPP
#include <vector>
#include <stack>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class OrderedCursor {
    stack<TreeNode*> st;   // pending nodes not yet emitted
    vector<int> arr;       // materialized in-order values
    int pointer = -1;      // index into arr; -1 == before first

    void pushLeft(TreeNode* node) {
        while (node) { st.push(node); node = node->left; }
    }

public:
    OrderedCursor(TreeNode* root) { pushLeft(root); }

    bool hasNext() {
        return pointer < (int)arr.size() - 1 || !st.empty();
    }

    int next() {
        if (pointer == (int)arr.size() - 1) {   // at the frontier: expand lazily
            TreeNode* node = st.top(); st.pop();
            arr.push_back(node->val);
            pushLeft(node->right);
        }
        ++pointer;
        return arr[pointer];
    }

    bool hasPrev() { return pointer > 0; }

    int prev() {
        --pointer;
        return arr[pointer];
    }
};
C
C
#include <stdlib.h>
#include <stdbool.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

typedef struct {
    struct TreeNode **stack;  /* pending nodes whose value is not yet emitted */
    int stackTop;
    int *arr;                 /* values already materialized, in order        */
    int arrSize;
    int arrCap;
    int pointer;              /* index into arr; -1 means before the first    */
} OrderedCursor;

static void pushLeft(OrderedCursor *it, struct TreeNode *node) {
    while (node) {
        it->stack[it->stackTop++] = node;
        node = node->left;
    }
}

OrderedCursor *orderedCursorCreate(struct TreeNode *root) {
    OrderedCursor *it = (OrderedCursor *)malloc(sizeof(OrderedCursor));
    it->stack = (struct TreeNode **)malloc(sizeof(struct TreeNode *) * 100001);
    it->stackTop = 0;
    it->arrCap = 16;
    it->arr = (int *)malloc(sizeof(int) * it->arrCap);
    it->arrSize = 0;
    it->pointer = -1;
    pushLeft(it, root);
    return it;
}

bool orderedCursorHasNext(OrderedCursor *it) {
    return it->pointer < it->arrSize - 1 || it->stackTop > 0;
}

int orderedCursorNext(OrderedCursor *it) {
    /* if the cursor is at the frontier, materialize one more in-order value */
    if (it->pointer == it->arrSize - 1) {
        struct TreeNode *node = it->stack[--it->stackTop];
        if (it->arrSize == it->arrCap) {
            it->arrCap *= 2;
            it->arr = (int *)realloc(it->arr, sizeof(int) * it->arrCap);
        }
        it->arr[it->arrSize++] = node->val;
        pushLeft(it, node->right);
    }
    it->pointer++;
    return it->arr[it->pointer];
}

bool orderedCursorHasPrev(OrderedCursor *it) {
    return it->pointer > 0;
}

int orderedCursorPrev(OrderedCursor *it) {
    it->pointer--;
    return it->arr[it->pointer];
}

void orderedCursorFree(OrderedCursor *it) {
    free(it->stack);
    free(it->arr);
    free(it);
}

#### Follow-up
questions
Prove the amortized O(1) bound for next() given that a single call can push a whole left spine.
Add a peek() that returns the value the cursor would land on next without moving it.
Support a hasPrev()/prev() pair that also works if the cursor may sit before the first element after many backward steps.
Generalize to jump k steps forward or backward efficiently; what changes if the tree is modified between calls?

