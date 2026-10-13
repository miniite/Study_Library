---
type: "practice"
difficulty: "Other"
---

# Design a Prefix-Ranked Query Suggestion Engine

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design a query-suggestion engine for a text box. A user types a query one character at a time. Every query consists of at least one word and is terminated by the special character '#'. Before the terminator, every character is either a lowercase English letter ('a'–'z') or a single space ' '.

You are given a history of previously completed queries: a string array phrases and an integer array counts, both of length n, where phrases[i] is a query that was submitted before and counts[i] is how many times it was submitted.

The engine must support feeding one character at a time. For every fed character other than '#', it returns the top 3 most popular historical queries that begin with everything the user has typed so far in the current query (the current prefix). Ranking rules:

Popularity of a query is the total number of times it has been submitted.
The returned queries are sorted by popularity, most popular first.
If two queries have equal popularity, the one that is lexicographically smaller (by ASCII order, where a space ' ' = 32 sorts before any letter) comes first.
If fewer than 3 queries match the current prefix, return all of them.

When the fed character is '#', the current query is finished: the engine records the just-typed query with popularity 1 (increasing its count if it already existed), then resets so the next fed character starts a brand-new query. Feeding '#' returns an empty list.

Implement a class QuerySuggester:

QuerySuggester(phrases, counts) initializes the engine with the history.
feed(c) accepts one character. Returns [] when c == '#' (and stores the completed query). Otherwise returns the list of up to 3 top matching historical queries for the current prefix. Once the current prefix stops matching any stored query, every further character before the next '#' returns [].

Input

phrases: array of historical query strings.
counts: array of integers, the historical submission count of each phrase.
A stream of calls to feed(c).

Output

For each feed(c): a list of up to 3 query strings (empty when c == '#' or nothing matches).

#### Cons
traints

n == phrases.length == counts.length
1 <= n <= 100
1 <= phrases[i].length <= 100
1 <= counts[i] <= 50
Each fed character c is a lowercase letter, a space ' ', or '#'.
Each completed query (the run of characters ending at a '#') has length in [1, 200]; words are separated by single spaces.
At most 5000 calls are made to feed.

#### Example
s & test cases
TEXT
Setup: QuerySuggester(["good day", "google search", "good luck", "good game"], [5, 3, 2, 2])

Input: feed('g') -> Output: ["good day", "google search", "good game"]
Why: all four phrases start with "g"; ordered by count 5, 3, then the 2-2 tie between "good game" and "good luck" is broken by ASCII ('g'=103 < 'l'=108) so "good game" wins; only the top 3 are returned, dropping "good luck".

Input: feed('o') -> Output: ["good day", "google search", "good game"]
Why: the prefix is now "go" which still matches all four phrases, so the same top 3 is returned.

Input: feed('o') -> Output: ["good day", "google search", "good game"]
Why: the prefix "goo" still matches all four ("google" is g-o-o-g), so the ranking is unchanged.

Input: feed('d') -> Output: ["good day", "good game", "good luck"]
Why: the prefix "good" now excludes "google search"; the three remaining phrases all match and the 2-2 tie is broken lexicographically so "good game" precedes "good luck".

Input: feed('z') -> Output: []
Why: no stored phrase begins with "goodz", so nothing matches and an empty list is returned.

Input: feed('#') -> Output: []
Why: '#' ends the query, so "goodz" is saved with count 1 and the engine resets; a later query typing "goodz" would now surface it.

#### Approach

Store all history in a trie keyed by characters (letters and space). Each terminal node holds the full phrase and its running popularity count. The key optimization: every node caches references to the best up-to-3 terminal (leaf) nodes in its subtree, kept sorted by (count desc, phrase asc). When a phrase is recorded, walk its path once to create/update the leaf's count, then walk it again and offer that leaf to every ancestor's cached list, re-ranking and trimming to 3; because counts only grow, a node never needs to drop a leaf except when a better one displaces it. Answering a keystroke is then just descending one trie edge for the new character and reading the current node's cached list — O(1). A running curr pointer tracks the node for the current prefix; once it falls off the trie (no child edge) it becomes null and stays null until '#' resets it. The underlying pattern is a trie augmented with a per-node top-k cache maintained incrementally as a data stream arrives.

Complexity

Time: building the trie is O(sum of phrase lengths); recording a completed query of length L is O(L) since each of its L path nodes re-ranks a constant-size (<=3) list; a feed of a non-terminating character is O(1) — descend one node and read at most 3 cached suggestions. Space: O(sum of stored phrase characters) for the trie, plus O(1) cached references per node. The cached top-3 per node is what makes each keystroke constant time instead of a fresh subtree scan.

Python
PYTHON
from typing import List

class _Node:
    __slots__ = ("ch", "sentence", "count", "top")

    def __init__(self):
        self.ch = {}
        self.sentence = None
        self.count = 0
        self.top = []                 # best up-to-3 leaf nodes, best first

class QuerySuggester:
    def __init__(self, phrases: List[str], counts: List[int]):
        self.root = _Node()
        self.curr = self.root
        self.buf = []
        for s, t in zip(phrases, counts):
            self._insert(s, t)

    def _insert(self, s: str, delta: int) -> None:
        node = self.root
        for c in s:
            node = node.ch.setdefault(c, _Node())
        node.sentence = s
        node.count += delta
        leaf = node
        node = self.root
        for c in s:
            node = node.ch[c]
            self._update(node, leaf)

    @staticmethod
    def _update(node: "_Node", leaf: "_Node") -> None:
        if leaf not in node.top:
            node.top.append(leaf)
        node.top.sort(key=lambda n: (-n.count, n.sentence))
        if len(node.top) > 3:
            node.top.pop()

    def feed(self, c: str) -> List[str]:
        if c == '#':
            self._insert(''.join(self.buf), 1)
            self.buf = []
            self.curr = self.root
            return []
        self.buf.append(c)
        if self.curr is not None:
            self.curr = self.curr.ch.get(c)
        if self.curr is None:
            return []
        return [n.sentence for n in self.curr.top]
JavaScript
JAVASCRIPT
class QuerySuggester {

#### cons
tructor(phrases, counts) {
        this.root = { ch: new Map(), sentence: null, count: 0, top: [] };
        this.curr = this.root;
        this.buf = [];
        for (let i = 0; i < phrases.length; i++) this._insert(phrases[i], counts[i]);
    }

    _insert(s, delta) {
        let node = this.root;
        for (const c of s) {
            if (!node.ch.has(c)) node.ch.set(c, { ch: new Map(), sentence: null, count: 0, top: [] });
            node = node.ch.get(c);
        }
        node.sentence = s;
        node.count += delta;

#### cons
t leaf = node;
        node = this.root;
        for (const c of s) {
            node = node.ch.get(c);
            this._update(node, leaf);
        }
    }

    _update(node, leaf) {
        if (!node.top.includes(leaf)) node.top.push(leaf);
        node.top.sort((a, b) =>
            a.count !== b.count
                ? b.count - a.count
                : (a.sentence < b.sentence ? -1 : a.sentence > b.sentence ? 1 : 0));
        if (node.top.length > 3) node.top.pop();
    }

    feed(c) {
        if (c === '#') {
            this._insert(this.buf.join(''), 1);
            this.buf = [];
            this.curr = this.root;
            return [];
        }
        this.buf.push(c);
        if (this.curr) this.curr = this.curr.ch.get(c) || null;
        if (!this.curr) return [];
        return this.curr.top.map(n => n.sentence);
    }
}
Java
JAVA
import java.util.*;

class QuerySuggester {
    private static class Node {
        Map<Character, Node> ch = new HashMap<>();
        String sentence = null;
        int count = 0;
        List<Node> top = new ArrayList<>();   // best up-to-3 leaf nodes, best first
    }

    private final Node root = new Node();
    private Node curr = root;
    private final StringBuilder buf = new StringBuilder();

    public QuerySuggester(String[] phrases, int[] counts) {
        for (int i = 0; i < phrases.length; i++) insertPhrase(phrases[i], counts[i]);
    }

    private void insertPhrase(String s, int delta) {
        Node node = root;
        for (int i = 0; i < s.length(); i++)
            node = node.ch.computeIfAbsent(s.charAt(i), k -> new Node());
        node.sentence = s;
        node.count += delta;
        Node leaf = node;
        node = root;
        for (int i = 0; i < s.length(); i++) {
            node = node.ch.get(s.charAt(i));
            updateTop(node, leaf);
        }
    }

    private void updateTop(Node node, Node leaf) {
        if (!node.top.contains(leaf)) node.top.add(leaf);
        node.top.sort((a, b) -> a.count != b.count
                ? b.count - a.count
                : a.sentence.compareTo(b.sentence));
        if (node.top.size() > 3) node.top.remove(node.top.size() - 1);
    }

    public List<String> feed(char c) {
        if (c == '#') {
            insertPhrase(buf.toString(), 1);
            buf.setLength(0);
            curr = root;
            return new ArrayList<>();
        }
        buf.append(c);
        if (curr != null) curr = curr.ch.get(c);
        if (curr == null) return new ArrayList<>();
        List<String> res = new ArrayList<>();
        for (Node n : curr.top) res.add(n.sentence);
        return res;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;

class QuerySuggester {
    struct Node {
        unordered_map<char, Node*> ch;
        string sentence;
        int count = 0;
        vector<Node*> top;                 // best up-to-3 leaf nodes, best first
    };
    Node* root;
    Node* curr;
    string buf;

    static bool better(Node* a, Node* b) { // does a rank before b?
        if (a->count != b->count) return a->count > b->count;
        return a->sentence < b->sentence;
    }
    void updateTop(Node* node, Node* leaf) {
        auto& t = node->top;
        if (find(t.begin(), t.end(), leaf) == t.end()) t.push_back(leaf);
        sort(t.begin(), t.end(), better);
        if (t.size() > 3) t.pop_back();
    }
    void insertPhrase(const string& s, int delta) {
        Node* node = root;
        for (char c : s) {
            if (!node->ch.count(c)) node->ch[c] = new Node();
            node = node->ch[c];
        }
        node->sentence = s;
        node->count += delta;
        Node* leaf = node;
        node = root;
        for (char c : s) { node = node->ch[c]; updateTop(node, leaf); }
    }

public:
    QuerySuggester(vector<string>& phrases, vector<int>& counts) {
        root = new Node();
        curr = root;
        for (size_t i = 0; i < phrases.size(); ++i) insertPhrase(phrases[i], counts[i]);
    }
    vector<string> feed(char c) {
        if (c == '#') { insertPhrase(buf, 1); buf.clear(); curr = root; return {}; }
        buf.push_back(c);
        if (curr) curr = curr->ch.count(c) ? curr->ch[c] : nullptr;
        if (!curr) return {};
        vector<string> res;
        for (Node* n : curr->top) res.push_back(n->sentence);
        return res;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

#define CH 27   /* 26 lowercase letters + a single space */

typedef struct TrieNode {
    struct TrieNode* children[CH];
    char* sentence;                 /* non-NULL only at a terminal node */
    int count;
    struct TrieNode* top[3];        /* references to the best up-to-3 leaf nodes */
    int topSize;
} TrieNode;

typedef struct {
    TrieNode* root;
    TrieNode* curr;                 /* NULL once the current prefix has no match */
    char* buf;                      /* the query typed so far */
    int len, cap;
} QuerySuggester;

static int slot(char c) { return c == ' ' ? 26 : c - 'a'; }

static TrieNode* newNode(void) { return (TrieNode*)calloc(1, sizeof(TrieNode)); }

/* returns 1 if node a should rank AFTER node b (lower count, or equal count but larger sentence) */
static int worse(TrieNode* a, TrieNode* b) {
    if (a->count != b->count) return a->count < b->count;
    return strcmp(a->sentence, b->sentence) > 0;
}

static void sort3(TrieNode* node) {                 /* insertion sort of up to 3 entries */
    for (int i = 1; i < node->topSize; i++) {
        TrieNode* key = node->top[i];
        int j = i - 1;
        while (j >= 0 && worse(node->top[j], key)) { node->top[j + 1] = node->top[j]; j--; }
        node->top[j + 1] = key;
    }
}

static void updateTop(TrieNode* node, TrieNode* leaf) {
    for (int i = 0; i < node->topSize; i++)
        if (node->top[i] == leaf) { sort3(node); return; }   /* already tracked: re-rank */
    if (node->topSize < 3) { node->top[node->topSize++] = leaf; sort3(node); return; }
    sort3(node);                                              /* top[2] is now the weakest */
    if (worse(node->top[2], leaf)) { node->top[2] = leaf; sort3(node); }
}

static void insertPhrase(QuerySuggester* qs, const char* s, int delta) {
    TrieNode* node = qs->root;
    for (const char* p = s; *p; ++p) {
        int c = slot(*p);
        if (!node->children[c]) node->children[c] = newNode();
        node = node->children[c];
    }
    if (!node->sentence) node->sentence = strdup(s);
    node->count += delta;
    TrieNode* leaf = node;
    node = qs->root;
    for (const char* p = s; *p; ++p) {
        node = node->children[slot(*p)];
        updateTop(node, leaf);
    }
}

QuerySuggester* querySuggesterCreate(char** phrases, int phrasesSize, int* counts, int countsSize) {
    QuerySuggester* qs = (QuerySuggester*)calloc(1, sizeof(QuerySuggester));
    qs->root = newNode();
    qs->curr = qs->root;
    qs->cap = 256;
    qs->buf = (char*)malloc(qs->cap);
    for (int i = 0; i < phrasesSize; i++) insertPhrase(qs, phrases[i], counts[i]);
    return qs;
}

/* returns a malloc'd array of strdup'd strings; *returnSize holds its length */
char** querySuggesterFeed(QuerySuggester* qs, char c, int* returnSize) {
    if (c == '#') {
        qs->buf[qs->len] = '\0';
        insertPhrase(qs, qs->buf, 1);
        qs->len = 0;
        qs->curr = qs->root;
        *returnSize = 0;
        return NULL;
    }
    if (qs->len + 1 >= qs->cap) { qs->cap *= 2; qs->buf = (char*)realloc(qs->buf, qs->cap); }
    qs->buf[qs->len++] = c;
    qs->curr = qs->curr ? qs->curr->children[slot(c)] : NULL;
    if (!qs->curr) { *returnSize = 0; return NULL; }
    int k = qs->curr->topSize;
    char** res = (char**)malloc(sizeof(char*) * (k > 0 ? k : 1));
    for (int i = 0; i < k; i++) res[i] = strdup(qs->curr->top[i]->sentence);
    *returnSize = k;
    return res;
}

#### Follow-up
questions
What if the history streams in continuously with millions of phrases — how do you bound trie memory and keep each node's top-k fresh under constant updates?
How would you support typo-tolerant matching (edit distance / fuzzy prefix) rather than exact prefix matches?
How would you generalize to return the top k where k is configurable and possibly large, instead of a fixed 3?
How would you shard the trie across machines and merge per-shard top-k results into a single global ranking?

