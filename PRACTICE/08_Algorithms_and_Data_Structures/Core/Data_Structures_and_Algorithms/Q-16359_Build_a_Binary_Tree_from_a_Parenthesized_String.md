---
type: "practice"
difficulty: "Other"
---

# Build a Binary Tree from a Parenthesized String

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string that encodes a binary tree, and you must rebuild the tree it describes.

The encoding rules are:

The entire string encodes one binary tree.
Each node is written as its integer value (which may be negative), immediately followed by zero, one, or two parenthesized groups.
The first parenthesized group, if present, contains the encoding of the node's left subtree.
The second parenthesized group, if present, contains the encoding of the node's right subtree.
A leaf node is written as just its integer value, with no parentheses.
An empty string encodes an empty tree.

Because the first group is always the left subtree, a node that has a right child but no left child never appears in a valid input.

Return the root of the reconstructed binary tree.

Input

s: a string using only decimal digits, '(', ')', and '-', that is a valid encoding of a binary tree under the rules above.

Output

The root node of the constructed binary tree (or an empty/null tree when the string is empty).

#### Cons
traints

0 <= s.length <= 3 * 10^4
s consists only of the characters '0'–'9', '(', ')', and '-'.
s is always a valid encoding.
Node values fit in a 32-bit signed integer.

For illustration, the outputs below show the tree in level-order (breadth-first) form, listing nodes top-to-bottom, left-to-right.

#### Example
s & test cases
TEXT
Input: s = "7(3(9)(2))(8(5))" -> Output: [7, 3, 8, 9, 2, 5]
Why: root 7 has left subtree "3(9)(2)" (node 3 with children 9 and 2) and right subtree "8(5)" (node 8 whose only child, the left one, is 5); a normal mixed tree.

Input: s = "42" -> Output: [42]
Why: a bare integer with no parentheses is a single leaf node.

Input: s = "" -> Output: []
Why: the empty string encodes an empty tree, so the result is a null/empty root.

Input: s = "-6(2(8)(1))(9(4)(3))" -> Output: [-6, 2, 9, 8, 1, 4, 3]
Why: negative root and a fully populated tree; confirms the parser handles the leading minus sign and both subtrees.

Input: s = "5(3)" -> Output: [5, 3]
Why: a node with only a left child — the single parenthesized group is always the left subtree.

Input: s = "10(-7)" -> Output: [10, -7]
Why: a negative value inside a child group; the sign is parsed within the subtree, not confused with the surrounding syntax.

#### Approach

Recursive-descent parsing with a single shared cursor (a DFS over the grammar). At each node: read an optional minus sign, then read the integer value while the cursor sits on digits, and create the node. If the cursor now points at '(', that group is by rule the left subtree, so consume the '(', recurse to build the left child, then consume the matching ')'. If another '(' immediately follows, that group is the right subtree, so recurse the same way. A leaf simply has no following parentheses. The pattern is classic one-pass expression parsing: keep one moving index so each recursive call resumes exactly where the previous one stopped; the same tree can equivalently be built iteratively with an explicit stack that pushes on a value and pops on ')'.

Complexity

Time: O(n) where n is the string length — every character is consumed exactly once as the shared cursor advances. Space: O(h) for the recursion stack where h is the tree height, which is O(n) in the worst case of a completely skewed tree (plus O(n) for the returned tree itself).

Python
PYTHON
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def buildTree(self, s: str):
        if not s:
            return None
        self.i = 0
        return self._parse(s)

    def _parse(self, s):
        sign = 1
        if s[self.i] == '-':
            sign = -1
            self.i += 1
        val = 0
        while self.i < len(s) and s[self.i].isdigit():
            val = val * 10 + int(s[self.i])
            self.i += 1

        node = TreeNode(sign * val)
        if self.i < len(s) and s[self.i] == '(':   # left subtree
            self.i += 1
            node.left = self._parse(s)
            self.i += 1                             # ')'
        if self.i < len(s) and s[self.i] == '(':   # right subtree
            self.i += 1
            node.right = self._parse(s)
            self.i += 1                             # ')'
        return node
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

function buildTree(s) {
    if (s.length === 0) return null;
    let i = 0;

#### cons
t parse = () => {
        let sign = 1;
        if (s[i] === '-') { sign = -1; i++; }
        let val = 0;
        while (i < s.length && s[i] >= '0' && s[i] <= '9') {
            val = val * 10 + (s.charCodeAt(i) - 48);
            i++;
        }

#### cons
t node = new TreeNode(sign * val);
        if (i < s.length && s[i] === '(') {   // left subtree
            i++;
            node.left = parse();
            i++;                              // ')'
        }
        if (i < s.length && s[i] === '(') {   // right subtree
            i++;
            node.right = parse();
            i++;                              // ')'
        }
        return node;
    };

    return parse();
}
Java
JAVA
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int x) { val = x; }
}

class Solution {
    private int i = 0;

    public TreeNode buildTree(String s) {
        if (s.isEmpty()) return null;
        i = 0;
        return parse(s);
    }

    private TreeNode parse(String s) {
        int sign = 1;
        if (s.charAt(i) == '-') { sign = -1; i++; }
        int val = 0;
        while (i < s.length() && Character.isDigit(s.charAt(i)))
            val = val * 10 + (s.charAt(i++) - '0');

        TreeNode node = new TreeNode(sign * val);
        if (i < s.length() && s.charAt(i) == '(') {   // left subtree
            i++;
            node.left = parse(s);
            i++;                                       // ')'
        }
        if (i < s.length() && s.charAt(i) == '(') {   // right subtree
            i++;
            node.right = parse(s);
            i++;                                       // ')'
        }
        return node;
    }
}
C++
CPP
#include <string>
#include <cctype>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left, *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    TreeNode* buildTree(const string& s) {
        if (s.empty()) return nullptr;
        int i = 0;
        return parse(s, i);
    }

private:
    TreeNode* parse(const string& s, int& i) {
        int sign = 1;
        if (s[i] == '-') { sign = -1; ++i; }
        int val = 0;
        while (i < (int)s.size() && isdigit((unsigned char)s[i]))
            val = val * 10 + (s[i++] - '0');

        TreeNode* node = new TreeNode(sign * val);
        if (i < (int)s.size() && s[i] == '(') {   // left subtree
            ++i;
            node->left = parse(s, i);
            ++i;                                  // ')'
        }
        if (i < (int)s.size() && s[i] == '(') {   // right subtree
            ++i;
            node->right = parse(s, i);
            ++i;                                  // ')'
        }
        return node;
    }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
};

/* Recursive-descent parse using a shared cursor *i into s. */
static struct TreeNode* parse(const char *s, int n, int *i) {
    int sign = 1;
    if (s[*i] == '-') { sign = -1; (*i)++; }
    int val = 0;
    while (*i < n && isdigit((unsigned char)s[*i])) {
        val = val * 10 + (s[*i] - '0');
        (*i)++;
    }
    struct TreeNode *node = (struct TreeNode*)malloc(sizeof(struct TreeNode));
    node->val = sign * val;
    node->left = node->right = NULL;

    if (*i < n && s[*i] == '(') {   /* first group -> left subtree */
        (*i)++;
        node->left = parse(s, n, i);
        (*i)++;                     /* consume ')' */
    }
    if (*i < n && s[*i] == '(') {   /* second group -> right subtree */
        (*i)++;
        node->right = parse(s, n, i);
        (*i)++;                     /* consume ')' */
    }
    return node;
}

struct TreeNode* buildTree(const char *s) {
    int n = (int)strlen(s);
    if (n == 0) return NULL;
    int i = 0;
    return parse(s, n, &i);
}

#### Follow-up
questions
Rewrite the parser iteratively with an explicit stack instead of recursion, pushing a node on each value and popping when a ')' closes a subtree.
Reverse the operation: serialize a given binary tree back into this parenthesized string form.
How would you extend the grammar to represent a node that has a right child but no left child (for example, allowing an empty first group ())?
How would you adapt the parser if values could include a leading '+', surrounding spaces, or be arbitrarily large integers?

