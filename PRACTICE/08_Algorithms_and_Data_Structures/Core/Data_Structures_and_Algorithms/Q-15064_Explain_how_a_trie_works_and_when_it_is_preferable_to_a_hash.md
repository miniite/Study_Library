---
type: "practice"
difficulty: "Deep"
---

# Explain how a trie works and when it is preferable to a hash map for string lookups.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

Two linked parts: (1) explain the trie data structure — its shape, how insert/search/delete work, and its complexity; and (2) make the engineering call: when is a trie preferable to a hash map for string lookups? "Deep understanding" means the interviewer wants more than "a trie stores strings as a tree." They want the mechanics, the complexity in terms of key length vs. number of keys, and — crucially — the operations a trie enables that a hash map fundamentally cannot do efficiently (prefix queries, ordered traversal, longest-prefix match). They're also probing whether you know the downsides (memory, cache behavior) so your recommendation is balanced.

Clarifying questions: What's the alphabet size (26 lowercase? full Unicode? bytes)? Do we need only exact contains, or also prefix operations (autocomplete)? What's the read/write mix and dataset size? These determine whether a trie is worth it.

What a trie is — from first principles

A trie (prefix tree, "retrieval" tree) is a tree where each node represents a prefix, and each edge is labeled with one character. The path from the root to a node spells out a string. So all strings sharing a common prefix share the same initial path and nodes — the structure factors out shared prefixes.

Key structural facts:

The root represents the empty string.
Each node has up to Σ children (Σ = alphabet size), typically stored as a fixed-size array indexed by character, or a hash map / sorted list of char → child for sparse/large alphabets.
A node carries an isEnd flag marking that the path to it is a complete inserted word (needed to distinguish "car" being stored from "car" merely being a prefix of "card").
Operations
Insert(word): start at root; for each character, follow the child edge if it exists, else create it; at the last character, set isEnd = true. Cost O(L) where L = len(word).
Search(word) (exact contains): walk the characters; if any edge is missing, return false; at the end return the node's isEnd. Cost O(L).
StartsWith(prefix): same walk but at the end just return "node exists" (ignore isEnd). Cost O(P) for prefix length P. This is the trie's superpower.
Delete(word): walk to the end, clear isEnd; then unwind, pruning nodes that have no children and are not word-ends. O(L).

Notice all these are O(length of the key) and, importantly, independent of n, the number of stored keys — the trie never compares against other keys. A hash map is also effectively independent of n on average, but it must still hash the entire key (O(L)) and compare on collision (O(L)), so the naive "hash map is O(1)" claim really means O(L) amortized average, plus worst-case O(n·L) under adversarial collisions.

Complexity: trie vs. hash map — the honest comparison
Operation	Trie	Hash map
Insert / exact lookup	O(L) worst-case, deterministic	O(L) average (hash + compare), O(n·L) adversarial worst-case
Prefix search / autocomplete	O(P + matches)	O(n·L) — must scan every key
Ordered / lexicographic traversal	O(total chars) via DFS	O(n log n) — must extract and sort
Memory	Higher: one node per char in worst case, child pointers	Lower per key; just the key + slot overhead
Cache locality	Poor (pointer chasing)	Better (contiguous buckets)

The two headline takeaways:

For pure exact lookup, both are O(L); the hash map is usually faster in practice (fewer pointer dereferences, better cache locality) and smaller. So if all you need is contains, a hash map is the default.
The trie wins decisively on operations a hash map can't do without a full scan: prefix queries, ordered iteration, and longest-prefix match.
When a trie is preferable — the actual answer

Choose a trie when you need any of these:

Prefix / autocomplete / typeahead: "give me all words starting with pre." A trie walks to the pre node in O(P) then DFS-collects the subtree — O(P + size of subtree). A hash map has no notion of prefixes; it would have to scan all n keys, O(n·L). This is the canonical trie use case (search suggestions, IDE autocomplete, command completion).
Longest-prefix match: networking IP routing tables and URL/route matching need "the longest stored prefix of this input." Tries (and their compressed variant, the radix/Patricia trie) do this naturally in O(L).
Ordered / lexicographic enumeration: a trie DFS yields all keys in sorted order for free; a hash map must collect and sort (O(n log n)).
Many keys sharing long common prefixes (e.g., dictionaries, dictionaries of URLs, dictionaries of DNA k-mers): the trie shares prefix nodes, so it can be more memory-efficient than storing each full string, and it deduplicates prefixes for free.
Spell-check / fuzzy match / word games (Boggle, Scrabble): the trie lets you prune whole branches the instant a prefix is invalid, which a hash-set of words cannot.
Deterministic worst-case latency requirements: O(L) is guaranteed with no hashing and no collision cliffs — useful in adversarial or real-time contexts.
When to prefer the hash map (the counter-case — important for balance)
Only exact lookups, no prefix/ordering needs → hash map is simpler, faster, less memory.
Large/Unicode alphabet with sparse tries → array-per-node wastes huge memory; a hash map avoids that. (You can mitigate with map-of-children tries or a DAWG/compressed trie, but a plain hash map is easier.)
Cache-sensitive hot paths — the trie's pointer chasing causes cache misses; a hash lookup touches one or two buckets.
Short keys, small dataset — the trie's structural overhead isn't worth it.
Optimizations worth naming at this level
Compressed / radix (Patricia) trie: collapse chains of single-child nodes into one edge holding a substring — cuts node count and depth dramatically. Used in routing, IP tables, and git object storage.
DAWG (directed acyclic word graph): also merge common suffixes, shrinking memory further for large fixed dictionaries.
Ternary search trie: stores children as a small BST per node — memory-friendly for large alphabets while keeping ordered traversal.
Array vs. map children: fixed 26-array for lowercase English (fast, some waste) vs. hash/sorted-list children for large alphabets (compact, slightly slower).

#### Common mistakes
/ follow-up probes
Claiming the trie is "always faster than a hash map" — false for exact lookup; hash maps usually win there.
Forgetting the isEnd flag, so "car" as a stored word can't be distinguished from "car" as a prefix of "card".
Ignoring memory blow-up for large alphabets (the #1 reason people abandon naive tries).

#### Follow-ups
"How would you autocomplete top-k suggestions?" → store a frequency/rank at nodes or attach a heap per prefix, DFS the subtree and pick top-k. "How to save memory?" → radix/compressed trie or DAWG. "Concurrent tries?" → per-node locking / lock-free CAS on child arrays, or a persistent trie (HAMT) for immutable sharing.
Summary — what a strong answer emphasizes

Define the trie as a character-edge prefix tree with an isEnd flag; state that insert/search/prefix are O(key length), independent of the number of keys; and then make the crisp trade-off: for exact lookup use a hash map (faster, smaller); use a trie when you need prefix search / autocomplete, longest-prefix match, ordered enumeration, or heavy prefix sharing — the things a hash map can only do by scanning all keys. Naming the radix-trie/DAWG optimizations and the memory-vs-cache downsides is what makes it a deep, senior-grade answer.

**Comparison:**

```
Need	Prefer	Why
Only exact contains	Hash map	O(L) avg, faster in practice, less memory, better cache locality
Autocomplete / typeahead / prefix scan	Trie	O(P + matches) vs hash map's O(n·L) full scan
Longest-prefix match (IP routing, route matching)	Trie / radix trie	natural O(L) longest-match
Ordered / lexicographic iteration	Trie	DFS yields sorted keys for free (hash map: O(n log n))
Many keys with shared long prefixes	Trie	prefix nodes shared → memory saved, dedup for free
Large/Unicode sparse alphabet, exact only	Hash map	trie array-per-node wastes memory; hash avoids it
Deterministic worst-case latency	Trie	no hashing/collision cliffs; guaranteed O(L)
```

**Code:**

```python
class TrieNode:
    def __init__(self):
        self.children = {}     # char -> TrieNode (dict handles any alphabet)
        self.is_end = False    # true if a word ends here (distinguishes 'car' from prefix of 'card')

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:          # O(L)
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:          # exact contains, O(L)
        node = self._walk(word)
        return node is not None and node.is_end

    def starts_with(self, prefix: str) -> bool:   # prefix exists, O(P)  -- the trie's edge over a hash map
        return self._walk(prefix) is not None

    def _walk(self, s: str):
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node

    def autocomplete(self, prefix: str):          # all words with prefix, O(P + subtree)
        node = self._walk(prefix)
        out = []
        if node is None:
            return out
        def dfs(n, path):
            if n.is_end:
                out.append(prefix + path)
            for ch, child in sorted(n.children.items()):   # sorted -> lexicographic order
                dfs(child, path + ch)
        dfs(node, "")
        return out
```

**Example input / output:**

```text
Given inserts: "car", "card", "cat", "dog"

search("car")        -> True   (walk c->a->r, node.is_end == True)
search("ca")         -> False  (node exists but is_end == False -> it's only a prefix)
starts_with("ca")    -> True   (node c->a exists)
starts_with("do")    -> True
starts_with("z")     -> False  (no 'z' child of root)
autocomplete("ca")   -> ["car", "card", "cat"]   (walk to 'ca', DFS subtree, sorted)
autocomplete("do")   -> ["dog"]

Hash-set equivalent for autocomplete("ca") would require scanning ALL keys and testing
str.startswith("ca") on each -> O(n * L), vs trie's O(P + matches).
```

**Data:**

```

Trie built from ["car", "card", "cat", "dog"] (isEnd nodes marked *):

CODE
Copy
(root)
 ├─ c ─ a ─ r*        "car"
 │            └─ d*   "card"
 │        └─ t*       "cat"
 └─ d ─ o ─ g*        "dog"

Note "car" shares the "c-a-r" path with "card"; only the trailing 'd' node is extra — this prefix sharing is the memory win when many keys share prefixes.
```

**Flow:**

1. No, only exact contains
2. Yes
3. Yes
4. No
5. String lookup requirement
6. Need prefix search /autocompleteordered iteration /longest-prefix match?
7. Use HASH MAPO(L) avg, smaller,cache-friendly
8. Alphabet large/sparse ormemory tight?
9. Compressed radix trie / DAWG/ternary trie or map-of-children
10. Plain array-children TRIEO(L) ops, O(P+matches) prefix

