---
type: "practice"
difficulty: "Other"
---

# Compact Encode and Decode of a Binary Search Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Encoding (or "serialization") is the process of turning an in-memory data structure into a flat string of characters so it can be written to a file, kept in a buffer, or shipped over a network. Decoding (or "deserialization") is the reverse: taking such a string and rebuilding the exact structure it described.

Design a pair of routines, encode and decode, for a binary search tree (BST):

encode(root) takes the root of a BST and returns a single string.
decode(text) takes a string produced by encode and returns the root of a tree that is structurally identical to the original.

There is no restriction on the format of the string you choose, as long as decode(encode(root)) reconstructs the same BST. Because the input is guaranteed to be a valid binary search tree (every node's value is greater than all values in its left subtree and less than all values in its right subtree, and all values are distinct), you should exploit that ordering to keep the encoded string as compact as possible — in particular, you do not need to store any placeholder markers for missing children.

Input / Output: encode receives the root node (which may be null for an empty tree) and returns a string. decode receives that string and returns the reconstructed root node.

#### Cons
traints:

The number of nodes is in the range [0, 10^4].
0 <= Node.val <= 10^4.
All node values are distinct, and the tree is a valid binary search tree.
Your encoding should avoid per-null markers; rely on the BST ordering property to rebuild structure.

#### Example
s & test cases
TEXT
Input: root = [8, 4, 12, 2, 6, 10, 14] -> Output: decode(encode(root)) rebuilds the same BST; encoded string is "8 4 2 6 12 10 14"
Why: a pre-order walk lists each node before its children, and the BST ordering lets decode place every value uniquely without null markers.

Input: root = [] -> Output: decode(encode(root)) is an empty tree; encoded string is ""
Why: an empty tree serializes to the empty string and round-trips back to null.

Input: root = [7] -> Output: decode(encode(root)) is a single node with value 7; encoded string is "7"
Why: a one-node tree needs only its single value.

Input: root = [3, null, 5, null, null, null, 9] (a right-leaning chain 3 -> 5 -> 9) -> Output: round-trips to the same chain; encoded string is "3 5 9"
Why: pre-order emits 3, 5, 9; on decode each larger value is forced into the right subtree by the ascending bounds.

Input: root = [9, 7, null, 5] (a left-leaning chain 9 -> 7 -> 5) -> Output: round-trips to the same chain; encoded string is "9 7 5"
Why: each smaller value falls into the left subtree, reconstructing the descending chain.

Input: root = [50, 30, 70, 20, 40, 60, 80] -> Output: round-trips exactly; encoded string is "50 30 20 40 70 60 80"
Why: a full balanced BST reconstructs uniquely from its pre-order sequence using value bounds.

#### Approach

Store just the pre-order traversal of the BST — the sequence of values, root before children. The key insight is that a pre-order listing of a BST is enough to rebuild it uniquely without any null markers, because the ordering property tells you exactly where each subsequent value belongs. On decode, thread a value window [lo, hi] through the stream: the next token becomes the current node only if it falls inside the window; its left subtree is built with window [lo, value] and its right subtree with [value, hi]. A value outside the window signals the end of the current subtree, so the recursion unwinds to an ancestor that can accept it. This yields a maximally compact encoding (only the numbers, no #/null placeholders) and an O(n) reconstruction. It is the bounded-BST-from-preorder pattern.

Complexity

Time: O(n) for both encode and decode — encode does one pre-order pass and decode consumes each token exactly once. Space: O(n) for the produced string / value list plus O(h) recursion stack, where h is the tree height (O(log n) balanced, O(n) skewed).

Python
PYTHON
from collections import deque
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Codec:
    def encode(self, root: Optional[TreeNode]) -> str:
        out = []

        def pre(node):
            if not node:
                return
            out.append(str(node.val))
            pre(node.left)
            pre(node.right)

        pre(root)
        return " ".join(out)

    def decode(self, text: str) -> Optional[TreeNode]:
        if not text:
            return None
        vals = deque(int(x) for x in text.split())

        def build(lo, hi):
            if not vals or not (lo <= vals[0] <= hi):
                return None
            v = vals.popleft()
            node = TreeNode(v)
            node.left = build(lo, v)
            node.right = build(v, hi)
            return node

        return build(float("-inf"), float("inf"))
JavaScript
JAVASCRIPT
function TreeNode(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
}

// Pre-order encode, space separated, no null markers.
function encode(root) {

#### cons
t out = [];
    (function pre(node) {
        if (!node) return;
        out.push(node.val);
        pre(node.left);
        pre(node.right);
    })(root);
    return out.join(" ");
}

// Rebuild by walking the pre-order stream under a shrinking [lo, hi] window.
function decode(text) {
    if (!text) return null;

#### cons
t vals = text.split(" ").map(Number);
    let i = 0;
    function build(lo, hi) {
        if (i >= vals.length || vals[i] < lo || vals[i] > hi) return null;

#### cons
t v = vals[i++];

#### cons
t node = new TreeNode(v);
        node.left = build(lo, v);
        node.right = build(v, hi);
        return node;
    }
    return build(-Infinity, Infinity);
}
Java
JAVA
import java.util.*;

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

public class Codec {
    // Pre-order encode, space separated, no null markers.
    public String encode(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        pre(root, sb);
        return sb.toString();
    }

    // Rebuild by consuming the pre-order stream under a shrinking value window.
    public TreeNode decode(String text) {
        if (text.isEmpty()) return null;
        Queue<Integer> q = new ArrayDeque<>();
        for (String tok : text.trim().split("\\s+"))
            q.offer(Integer.parseInt(tok));
        return build(Long.MIN_VALUE, Long.MAX_VALUE, q);
    }

    private void pre(TreeNode node, StringBuilder sb) {
        if (node == null) return;
        sb.append(node.val).append(' ');
        pre(node.left, sb);
        pre(node.right, sb);
    }

    private TreeNode build(long lo, long hi, Queue<Integer> q) {
        if (q.isEmpty()) return null;
        long v = q.peek();
        if (v < lo || v > hi) return null;
        q.poll();
        TreeNode node = new TreeNode((int) v);
        node.left  = build(lo, v, q);
        node.right = build(v, hi, q);
        return node;
    }
}
C++
CPP
#include <string>
#include <sstream>
#include <vector>
#include <climits>
using namespace std;

struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Codec {
public:
    // Pre-order encode, values separated by spaces (no null markers).
    string encode(TreeNode* root) {
        string s;
        pre(root, s);
        return s;
    }

    // Rebuild by threading a monotonic value window through the pre-order stream.
    TreeNode* decode(string text) {
        istringstream iss(text);
        vector<int> vals;
        for (int x; iss >> x;) vals.push_back(x);
        int idx = 0;
        return build(vals, idx, LONG_MIN, LONG_MAX);
    }

private:
    void pre(TreeNode* node, string& s) {
        if (!node) return;
        s += to_string(node->val) + " ";
        pre(node->left, s);
        pre(node->right, s);
    }

    TreeNode* build(vector<int>& vals, int& idx, long lo, long hi) {
        if (idx == (int)vals.size() || vals[idx] < lo || vals[idx] > hi)
            return nullptr;
        int v = vals[idx++];
        TreeNode* node = new TreeNode(v);
        node->left  = build(vals, idx, lo, v);
        node->right = build(vals, idx, v, hi);
        return node;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

/* ---- encode: pre-order, space separated ---- */
static void encodeHelper(struct TreeNode* node, char* buf, int* len) {
    if (node == NULL) return;
    *len += sprintf(buf + *len, "%d ", node->val);
    encodeHelper(node->left, buf, len);
    encodeHelper(node->right, buf, len);
}

char* encode(struct TreeNode* root) {
    /* up to 1e4 nodes, each value <= 7 chars + space */
    char* buf = (char*)malloc(1 << 20);
    int len = 0;
    buf[0] = '\0';
    encodeHelper(root, buf, &len);
    return buf;
}

/* ---- decode: rebuild using BST value bounds ---- */
static struct TreeNode* decodeHelper(long lo, long hi, char** p) {
    while (**p == ' ') (*p)++;
    if (**p == '\0') return NULL;

    char* r = *p;
    long sign = 1, val = 0;
    if (*r == '-') { sign = -1; r++; }
    char* start = r;
    while (*r >= '0' && *r <= '9') { val = val * 10 + (*r - '0'); r++; }
    val *= sign;
    if (r == start) return NULL;            /* nothing numeric left */
    if (val < lo || val > hi) return NULL;  /* belongs to an ancestor's other side */

    *p = r;                                 /* consume this token */
    struct TreeNode* node = (struct TreeNode*)malloc(sizeof(struct TreeNode));
    node->val = (int)val;
    node->left  = decodeHelper(lo, val, p);
    node->right = decodeHelper(val, hi, p);
    return node;
}

struct TreeNode* decode(char* text) {
    char* p = text;
    return decodeHelper(LONG_MIN, LONG_MAX, &p);
}

#### Follow-up
questions
How does your encoding change if the tree is a general binary tree (not a BST)? Now you do need null markers or a paired traversal, since ordering no longer pins down structure.
Make the format binary/byte-packed instead of decimal text to shrink it further, and discuss the size trade-off.
Support duplicate values or negative values — how do the value-window bounds need to change?
Reconstruct iteratively with an explicit stack instead of recursion to avoid stack overflow on a skewed 10^4-node tree.

