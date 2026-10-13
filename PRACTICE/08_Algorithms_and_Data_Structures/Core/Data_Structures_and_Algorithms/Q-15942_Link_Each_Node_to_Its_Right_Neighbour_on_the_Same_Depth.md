---
type: "practice"
difficulty: "Other"
---

# Link Each Node to Its Right Neighbour on the Same Depth

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Linked List](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Linked_List.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given the root of a binary tree. Every node carries four fields: an integer value, a left child pointer, a right child pointer, and an extra next pointer. The tree is arbitrary — it is not guaranteed to be complete or perfect, so a node may have zero, one, or two children.

Wire up the tree so that, after your function runs, each node's next pointer refers to the node immediately to its right on the same depth level. If a node is the last (rightmost) node on its level, its next pointer must remain null.

Initially every next pointer is null. You should set them all in one pass over the tree and return the (unchanged) root.

Input

root: the root of a binary tree whose nodes each expose left, right, and a next pointer that starts as null.

Output

The same root, with every node's next pointer now pointing to its right-hand neighbour on the same level (or null if none). The tree structure (left/right) is otherwise unchanged.

#### Cons
traints

The number of nodes in the tree is in the range [0, 6000].
-100 <= node value <= 100.

Follow up: Solve it using only constant extra space. Recursion is acceptable and its implicit stack does not count toward the extra-space budget, but you must not allocate an auxiliary container proportional to the tree size (for example, no per-level queue).

#### Example
s & test cases
TEXT
Input: root = [6,3,8,2,4,null,1] -> Output: [6,#,3,8,#,2,4,1,#] — Level 0 is just 6; level 1 links 3->8; level 2 has 3's children 2,4 followed by 8's right child 1, so 2->4->1. Each level ends with null, shown as #.

Input: root = [] -> Output: [] — An empty tree has no nodes, so there is nothing to link.

Input: root = [10] -> Output: [10,#] — A single node is alone on its level; its next stays null.

Input: root = [1,2,null,3,null,4] -> Output: [1,#,2,#,3,#,4,#] — A left-leaning skewed tree: every level holds exactly one node, so all next pointers remain null.

Input: root = [7,9,20,null,null,15,17] -> Output: [7,#,9,20,#,15,17,#] — The left child 9 has no children; level 2 comes entirely from 20's children, giving 15->17.

Input: root = [1,2,3,4,5,6,7] -> Output: [1,#,2,3,#,4,5,6,7,#] — A perfect tree: level 2 links 4->5->6->7 even though 5 and 6 come from different parents.

#### Approach

Level-by-level linking using the pointers you have already established. Keep a pointer head to the leftmost node of the level currently being processed. That level is already fully threaded by next pointers (the root level trivially is), so you can walk it left to right with cur = cur.next without any queue. As you walk, append each encountered child (left then right, skipping absent children) onto a fresh chain for the next level; a dummy header node makes the "attach to tail" step uniform and avoids special-casing the first child. When the walk of the current level ends, dummy.next is the leftmost node of the next level — set head to it and repeat. This turns each already-linked level into the scaffolding that links the level beneath it, giving a single linear pass with only O(1) extra memory, which satisfies the constant-space follow-up. (A breadth-first traversal with an explicit per-level queue also works and is easier to reason about, but costs O(width) space and is therefore not optimal for the follow-up.)

Complexity

Time: O(n), where n is the number of nodes — each node is visited exactly once as cur while its level is being scanned, and exactly once as a child being appended to the level below. Space: O(1) auxiliary — only the head, dummy, tail, and cur pointers are used; there is no queue and no recursion, so extra space does not grow with the tree.

Python
PYTHON
class Node:
    def __init__(self, val=0, left=None, right=None, next=None):
        self.val = val
        self.left = left
        self.right = right
        self.next = next

class Solution:
    # Constant extra space: reuse this level's next chain to link the next level.
    def connect(self, root: 'Node') -> 'Node':
        head = root                         # leftmost node of the level above
        while head:
            dummy = Node(0)
            tail = dummy                    # builds the child level's chain
            cur = head
            while cur:
                if cur.left:
                    tail.next = cur.left
                    tail = tail.next
                if cur.right:
                    tail.next = cur.right
                    tail = tail.next
                cur = cur.next
            head = dummy.next               # descend to the level just linked
        return root
JavaScript
JAVASCRIPT
class Node {

#### cons
tructor(val = 0, left = null, right = null, next = null) {
        this.val = val;
        this.left = left;
        this.right = right;
        this.next = next;
    }
}

// Constant extra space: reuse this level's next chain to link the next level.
function connect(root) {
    let head = root;                        // leftmost node of the level above
    while (head) {

#### cons
t dummy = new Node(0);
        let tail = dummy;                   // builds the child level's chain
        for (let cur = head; cur; cur = cur.next) {
            if (cur.left)  { tail.next = cur.left;  tail = tail.next; }
            if (cur.right) { tail.next = cur.right; tail = tail.next; }
        }
        head = dummy.next;                  // descend to the level just linked
    }
    return root;
}
Java
JAVA
class Node {
    int val;
    Node left, right, next;
    Node() {}
    Node(int v) { val = v; }
}

class Solution {
    // Constant extra space: reuse this level's next chain to link the next level.
    public Node connect(Node root) {
        Node head = root;                  // leftmost node of the level above
        while (head != null) {
            Node dummy = new Node();
            Node tail = dummy;             // builds the child level's chain
            for (Node cur = head; cur != null; cur = cur.next) {
                if (cur.left != null)  { tail.next = cur.left;  tail = tail.next; }
                if (cur.right != null) { tail.next = cur.right; tail = tail.next; }
            }
            head = dummy.next;             // descend to the level just linked
        }
        return root;
    }
}
C++
CPP
struct Node {
    int val;
    Node* left;
    Node* right;
    Node* next;
    Node() : val(0), left(nullptr), right(nullptr), next(nullptr) {}
    Node(int v) : val(v), left(nullptr), right(nullptr), next(nullptr) {}
};

class Solution {
public:
    // Constant extra space: reuse this level's next chain to link the next level.
    Node* connect(Node* root) {
        Node* head = root;                 // leftmost node of the level above
        while (head) {
            Node dummy(0);
            Node* tail = &dummy;           // builds the child level's chain
            for (Node* cur = head; cur; cur = cur->next) {
                if (cur->left)  { tail->next = cur->left;  tail = tail->next; }
                if (cur->right) { tail->next = cur->right; tail = tail->next; }
            }
            head = dummy.next;             // descend to the level just linked
        }
        return root;
    }
};
C
C
#include <stdlib.h>

struct Node {
    int val;
    struct Node *left;
    struct Node *right;
    struct Node *next;
};

/* Constant extra space: while processing one level, use the already-built
   next chain of that level to stitch together the next level's children.
   A dummy header lets us append children uniformly. */
struct Node* connect(struct Node* root) {
    struct Node* head = root;              /* leftmost node of the level above */
    while (head) {
        struct Node dummy;
        dummy.next = NULL;
        struct Node* tail = &dummy;        /* builds the child level's chain */
        for (struct Node* cur = head; cur; cur = cur->next) {
            if (cur->left)  { tail->next = cur->left;  tail = tail->next; }
            if (cur->right) { tail->next = cur->right; tail = tail->next; }
        }
        head = dummy.next;                 /* descend to the level just linked */
    }
    return root;
}

#### Follow-up
questions
If the tree were guaranteed to be a perfect binary tree, how would the O(1) solution simplify (no dummy header needed)? 2. Give a recursive constant-space variant and argue why the implicit stack is acceptable here. 3. Generalise to an n-ary tree where each node has a list of children — how does the level-stitching change? 4. Instead of pointing to the right neighbour, connect each node to the leftmost node on the level below — what stays the same?

