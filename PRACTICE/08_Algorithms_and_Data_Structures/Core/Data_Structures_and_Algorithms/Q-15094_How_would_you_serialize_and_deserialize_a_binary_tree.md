---
type: "practice"
difficulty: "Deep"
---

# How would you serialize and deserialize a binary tree?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tree Serialization](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree_Serialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Design two functions — serialize(root) -> string and deserialize(string) -> root — that turn an arbitrary binary tree into a flat, transmittable/storable representation and reconstruct the exact same tree back. The round-trip must be lossless: deserialize(serialize(t)) structurally equals t, including shape and null positions.

What the interviewer is probing for (level = "deep understanding"): Do you understand why naive approaches fail? A classic trap: a single in-order traversal, or even pre-order without null markers, cannot uniquely reconstruct a tree — multiple distinct trees produce the same sequence. The fix is to encode the null/missing children (or use two traversals for a BST/unique-value tree). They also want: choice of traversal (pre-order/BFS both work cleanly), correct O(n) time/space, handling of the empty tree and duplicate values, and awareness of the delimiter/encoding pitfalls. Leetcode 297 is the canonical version.

#### Assumptions
/clarifications I'd state:

Values may be arbitrary integers, possibly negative, possibly duplicated → I cannot rely on "BST + unique values + two-traversal" trick in general; I'll use the robust traversal-with-null-markers approach, which works for any binary tree.
I'll pick a delimiter (comma) and an explicit null token (#) and note the encoding caveat if values could contain the delimiter.
Output is a string; a real system might use a compact binary/length-prefixed format, but the algorithm is identical.

Plan: state the core insight (structure needs null markers) → pre-order recursive solution (cleanest) → show BFS/level-order as the common alternative → complexity → edge cases and production considerations → why the single-traversal trap fails.

1. Core insight: you must encode structure, not just values

A tree has two kinds of information: values and shape. A single traversal ordering (say pre-order [1,2,3]) records values but throws away shape — you can't tell whether 2 is 1's left child or right child, or whether 3 is a child of 1 or of 2. Multiple trees collapse to the same value list.

Two ways to recover shape:

Two complementary traversals (e.g., in-order + pre-order). This uniquely reconstructs a tree only if all values are distinct (you split the in-order array at the root found from pre-order). It breaks with duplicate values and needs O(n) index lookups. Good for a specific "reconstruct from traversals" question, but not the general serialize/deserialize contract.
One traversal + explicit null markers. Record a placeholder (#) for every missing child. Now the sequence carries the full shape, works with any values including duplicates and negatives, and reconstructs deterministically. This is the robust, preferred answer.
2. Preferred solution: pre-order DFS with null markers

Serialize (pre-order): visit root, then recurse left, then right. For a null node, emit #. Join tokens with a delimiter.

Tree 1 / (2, 3 with children 4,5) → 1,2,#,#,3,4,#,#,5,#,#.

Deserialize: read tokens left to right (a queue/iterator). Because we wrote them in pre-order, the first token is the current node. If it's #, this subtree is null; return. Otherwise create the node, then recursively build its left subtree (consuming the next tokens), then its right subtree. The pre-order order guarantees each recursive call consumes exactly the tokens belonging to its subtree — no index math needed.

This is elegant because serialize and deserialize are mirror recursions over the same token stream.

3. Alternative: BFS / level-order (also standard, sometimes preferred)

Serialize with a queue: pop a node, emit its value; if null, emit # and don't enqueue children; else enqueue both children. Deserialize with a queue of tokens: the first token is the root; then process the token stream, attaching the next two tokens as left/right children of the front node, skipping #. Level-order is nice when you want a "compact leftmost" representation or want to trim trailing #s (LeetCode/HackerRank style [1,2,3,null,null,4,5]), and it avoids deep recursion (no stack-overflow risk on skewed trees).

Both DFS and BFS are O(n) time and O(n) space; the difference is stack (DFS) vs. queue (BFS) and the recursion-depth consideration.

4. Complexity
Time: O(n) for both directions — each node (and each null slot, of which there are n+1 for n nodes) is visited exactly once.
Space: O(n) for the output string, plus O(h) recursion/queue overhead (h = height; O(n) worst case for a skewed tree with DFS, which is a reason to prefer iterative BFS for adversarial inputs).
5. Edge cases and production considerations
Empty tree: serialize(None) → "#"; deserialize must return None. Test this explicitly.
Single node / skewed tree: verify no off-by-one on null markers.
Duplicate values: handled — the null-marker approach doesn't rely on uniqueness (this is where two-traversal breaks).
Negative numbers / multi-digit / large ints: the delimiter-separated token approach handles them; a fixed-width or purely character approach would not.
Delimiter collision: if node values can themselves contain the delimiter (e.g., serialized strings with commas), you must escape them or use a length-prefixed encoding (len:value) — a mature answer calls this out. For integer trees, comma + # is fine.
Cross-language / versioning: a real system would prefer a documented, self-describing format (JSON, Protobuf, or a length-prefixed binary) rather than an ad-hoc string, so the schema can evolve. The traversal algorithm underneath is the same; only the framing changes.
BST-specific optimization: for a BST, you can serialize pre-order without null markers and reconstruct using value bounds, saving the # tokens — worth mentioning as an optimization when the interviewer specifies a BST.
6. Why the "single in-order traversal" trap fails (worth volunteering)

Serializing only the in-order sequence [2,1,3] is ambiguous: it matches infinitely many trees (any BST-shaped or skewed arrangement with that in-order). Even pre-order alone ([1,2,3]) is ambiguous without nulls. Demonstrating that you know this and choosing null-markers or two-traversals-for-unique-values is exactly the "deep understanding" signal.

Summary — what a strong answer emphasizes

Serialization must preserve both values and shape. The robust, general solution is one traversal (pre-order DFS or level-order BFS) with explicit null markers (#) and a delimiter, giving lossless O(n) round-trip for any binary tree — including duplicates and negatives — where a single traversal without nulls is ambiguous. Present the pre-order recursion (serialize emits # for nulls; deserialize consumes a token queue, #→null, else node then build left/right), give BFS as the alternative that avoids deep recursion, state O(n)/O(n), and cover empty-tree, duplicate-value, delimiter-escaping, and BST-without-nulls optimizations. That combination — correct algorithm + the ambiguity insight + production framing — is what makes it a complete senior answer.

#### Common mistakes
/ probes: omitting null markers (ambiguous); relying on unique values without saying so; forgetting the empty tree; using split in a way that mis-handles negative numbers or empty tokens; not mentioning delimiter escaping; deep recursion overflow on skewed trees.

**Comparison:**

```
Approach	Reconstructs any tree?	Handles duplicates?	Time	Space	Notes
Pre-order + null markers (DFS)	yes	yes	O(n)	O(n), O(h) stack	cleanest; recursion depth = height
Level-order + null markers (BFS)	yes	yes	O(n)	O(n) queue	no deep recursion; compact/trimmable form
In-order alone	no (ambiguous)	—	O(n)	O(n)	shape lost — do not use
Pre-order alone (no nulls)	no (ambiguous)	—	O(n)	O(n)	shape lost — do not use
Pre-order + in-order (two traversals)	yes	no (needs unique values)	O(n) w/ index map	O(n)	good for "rebuild from traversals" only
BST: pre-order, no nulls, value bounds	yes (BST only)	yes	O(n)	O(n)	optimization: skips # tokens
```

**Code:**

```
Robust serialize/deserialize for ANY binary tree — pre-order DFS + null markers.
(LeetCode 297 style.) Works with duplicates, negatives, and the empty tree.

from collections import deque

class TreeNode: def init(self, val=0, left=None, right=None): self.val = val self.left = left self.right = right

class Codec: def serialize(self, root) -> str: out = [] def dfs(node): if node is None: out.append("#") # explicit null marker preserves shape return out.append(str(node.val)) dfs(node.left) dfs(node.right) dfs(root) return ",".join(out) # e.g. "1,2,#,#,3,4,#,#,5,#,#"

CODE
Copy
def deserialize(self, data: str):
    tokens = iter(data.split(","))
    def build():
        val = next(tokens)
        if val == "#":
            return None
        node = TreeNode(int(val))
        node.left = build()          # pre-order: left subtree consumes its tokens
        node.right = build()         # then right subtree
        return node
    return build()
CODE
Copy

```python
# Alternative: BFS / level-order — avoids deep recursion on skewed trees.
class CodecBFS:
    def serialize(self, root) -> str:
        if root is None:
            return "#"
        out, q = [], deque([root])
        while q:
            node = q.popleft()
            if node is None:
                out.append("#")
            else:
                out.append(str(node.val))
                q.append(node.left)
                q.append(node.right)
        return ",".join(out)

    def deserialize(self, data: str):
        tokens = data.split(",")
        if tokens[0] == "#":
            return None
        root = TreeNode(int(tokens[0]))
        q, i = deque([root]), 1
        while q:
            node = q.popleft()
            if tokens[i] != "#":
                node.left = TreeNode(int(tokens[i]))
                q.append(node.left)
            i += 1
            if tokens[i] != "#":
                node.right = TreeNode(int(tokens[i]))
                q.append(node.right)
            i += 1
        return root
```

**Example input / output:**

```

Sample tree:

CODE
Copy
        1
       / \
      2   3
         / \
        4   5

Pre-order DFS round trip:

CODE
Copy
serialize(root)  -> "1,2,#,#,3,4,#,#,5,#,#"

deserialize dry-run (tokens consumed left->right):
  build(): val=1 -> node(1)
    left  = build(): val=2 -> node(2)
              left  = build(): "#" -> None
              right = build(): "#" -> None
    right = build(): val=3 -> node(3)
              left  = build(): val=4 -> node(4); left "#"->None; right "#"->None
              right = build(): val=5 -> node(5); left "#"->None; right "#"->None
  returns the identical tree  ✓ (shape + values match)

Edge cases:

CODE
Copy
serialize(None)           -> "#"       ;  deserialize("#")           -> None
serialize(node(7))        -> "7,#,#"   ;  deserialize("7,#,#")       -> single node 7
serialize(dup-tree 1/1/1) -> "1,1,#,#,1,#,#"  (duplicates handled — no unique-value assumption)

Why a null-less single traversal is ambiguous:

CODE
Copy
pre-order-without-nulls "1,2,3" could be:      1        or       1
                                              /                    \
                                             2                      2
                                            /                        \
                                           3                          3
  ...and more. Cannot reconstruct uniquely -> that is why "#" markers are required.
```

**Data:**

```text
# Pre-order DFS serialized form of the sample tree:
"1,2,#,#,3,4,#,#,5,#,#"

# BFS/level-order serialized form of the SAME tree:
"1,2,3,#,#,4,5,#,#,#,#"

# Empty tree (both codecs):
"#"
```

**Flow:**

1. Deserialize [deserialize - consume token stream]
2. yes
3. no
4. read next token
5. token == '#'?
6. return null
7. create node(val)
8. node.left = build()
9. node.right = build()
10. return node
11. Serialize [serialize - pre-order DFS]
12. yes
13. no
14. visit node
15. node is null?
16. emit '#'
17. emit value
18. recurse left
19. recurse right

