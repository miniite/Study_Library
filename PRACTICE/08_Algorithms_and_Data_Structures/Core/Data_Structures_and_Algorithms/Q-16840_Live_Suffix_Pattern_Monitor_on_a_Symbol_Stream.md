---
type: "practice"
difficulty: "Other"
---

# Live Suffix Pattern Monitor on a Symbol Stream

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are building a monitor that watches a never-ending stream of lowercase letters arriving one symbol at a time (think of characters typed into a console, or events flowing through a pipe). You are also given a fixed dictionary of alert patterns. After each new symbol arrives, you must report whether any suffix of everything received so far (the most recent symbols, ending exactly at the symbol that just arrived) spells one of the dictionary patterns.

Design a data structure with two operations:

#### Cons
tructor(patterns) — initialize the monitor with an array of pattern strings.
feed(symbol) — accept one new lowercase letter appended to the end of the stream and return true if some non-empty suffix of the stream received so far equals one of the patterns, otherwise return false.

A "suffix of the stream" means a contiguous run of the most recently received symbols that ends at the current symbol. Only suffixes are checked — a pattern that appears somewhere in the middle but does not reach the current end does not count.

Concretely, if patterns = ["go", "ana", "pqr"] and the stream so far is p, q, r, then after feeding r the suffix "pqr" matches, so feed('r') returns true. Later, after the stream becomes p, q, r, g, o, the suffix "go" matches, so that feed('o') returns true.

Return one boolean per feed call, in order.

#### Cons
traints:

1 <= patterns.length <= 2000
1 <= patterns[i].length <= 200
Every patterns[i] consists only of lowercase English letters ('a'–'z').
Each symbol passed to feed is a lowercase English letter.
At most 4 * 10^4 calls are made to feed.

#### Example
s & test cases
TEXT
Input: patterns = ["go","ana","pqr"]; feed 'p','q','r','g','o','a','n','a' -> Output: [false, false, true, false, true, false, false, true]
Why: "pqr" completes at the third symbol; "go" completes at 'o'; "ana" completes at the final 'a'. Normal case exercising several patterns and non-matches.

Input: patterns = ["abc"]; feed 'a','b','c','c' -> Output: [false, false, true, false]
Why: the pattern is only matched when the full "abc" suffix ends at the current symbol; feeding an extra 'c' breaks the suffix, so it goes back to false. Edge case: pattern equals the whole prefix, then trailing garbage.

Input: patterns = ["x"]; feed 'x','y','x','x' -> Output: [true, false, true, true]
Why: a single-character pattern matches whenever that exact letter arrives. Edge case: minimum-length pattern and repeated matches.

Input: patterns = ["mn","lmn"]; feed 'l','m','n' -> Output: [false, true, true]
Why: after 'n' both suffixes "mn" and "lmn" match; only one true is needed. Edge case: overlapping patterns where one is a suffix of the other.

Input: patterns = ["hello"]; feed 'h','e','l' -> Output: [false, false, false]
Why: the stream is shorter than the pattern, so no suffix can match yet. Edge case: pattern longer than the stream received so far.

#### Approach

The key trick is that checking a suffix of the growing stream becomes checking a prefix if you read the recent symbols in reverse. So build a trie of all patterns inserted backwards. Keep a small buffer of the last L received symbols (L = longest pattern), newest first. On each new symbol, start at the trie root and walk through the buffer from newest to oldest: follow the child edge for each symbol; if any edge is missing, no pattern-suffix can match and you stop; if you land on a node marked as a pattern end, a suffix matches and you return true. This is the reversed-trie / suffix-as-prefix pattern; the same problem can also be solved with an Aho-Corasick automaton built on reversed patterns for O(1) amortized per symbol.

Complexity

Time: building the trie is O(sum of all pattern lengths). Each feed is O(L) where L is the length of the longest pattern, because the backward walk stops as soon as a trie branch is missing (and the retained buffer is capped at L). Space: O(sum of pattern lengths) for the trie plus O(L) for the recent-symbol buffer. The per-query cost is independent of how long the stream has been running.

Python
PYTHON
from collections import deque
from typing import List

class PatternMonitor:
    def __init__(self, patterns: List[str]):
        self.root = {}
        self.max_len = 0
        for p in patterns:                       # insert each pattern reversed
            node = self.root
            for ch in reversed(p):
                node = node.setdefault(ch, {})
            node['$'] = True                     # end-of-pattern marker
            self.max_len = max(self.max_len, len(p))
        self.buf = deque()                       # newest symbol on the left

    def feed(self, symbol: str) -> bool:
        self.buf.appendleft(symbol)
        if len(self.buf) > self.max_len:
            self.buf.pop()
        node = self.root
        for ch in self.buf:                      # walk newest -> older
            if ch not in node:
                return False
            node = node[ch]
            if node.get('$'):
                return True
        return False
JavaScript
JAVASCRIPT
class PatternMonitor {

#### cons
tructor(patterns) {
        this.root = {};
        this.maxLen = 0;
        for (const p of patterns) {              // insert each pattern reversed
            let node = this.root;
            for (let i = p.length - 1; i >= 0; i--) {

#### cons
t ch = p[i];
                if (!node[ch]) node[ch] = {};
                node = node[ch];
            }
            node.end = true;                     // 3-char key never collides with a letter
            this.maxLen = Math.max(this.maxLen, p.length);
        }
        this.buf = [];                           // newest symbol at index 0
    }

    feed(symbol) {
        this.buf.unshift(symbol);
        if (this.buf.length > this.maxLen) this.buf.pop();
        let node = this.root;
        for (const ch of this.buf) {             // walk newest -> older
            if (!node[ch]) return false;
            node = node[ch];
            if (node.end) return true;
        }
        return false;
    }
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class PatternMonitor {
    private static class Node {
        Node[] next = new Node[26];
        boolean end = false;
    }

    private final Node root = new Node();
    private final Deque<Character> buf = new ArrayDeque<>(); // newest at head
    private int maxLen = 0;

    // Store each pattern reversed; a backward walk then matches a suffix.
    public PatternMonitor(String[] patterns) {
        for (String p : patterns) {
            Node node = root;
            for (int i = p.length() - 1; i >= 0; --i) {
                int idx = p.charAt(i) - 'a';
                if (node.next[idx] == null) node.next[idx] = new Node();
                node = node.next[idx];
            }
            node.end = true;
            maxLen = Math.max(maxLen, p.length());
        }
    }

    public boolean feed(char symbol) {
        buf.addFirst(symbol);
        if (buf.size() > maxLen) buf.removeLast();
        Node node = root;
        for (char ch : buf) {                 // newest -> older
            int idx = ch - 'a';
            if (node.next[idx] == null) return false;
            node = node.next[idx];
            if (node.end) return true;
        }
        return false;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <deque>
using namespace std;

class PatternMonitor {
    struct Node {
        Node* next[26] = {};
        bool end = false;
    };
    Node* root = new Node();
    deque<char> buf;   // newest symbol at the front
    int maxLen = 0;

public:
    // Insert each pattern REVERSED so a backward walk matches a suffix.
    PatternMonitor(vector<string>& patterns) {
        for (const string& p : patterns) {
            Node* node = root;
            for (int i = (int)p.size() - 1; i >= 0; --i) {
                int idx = p[i] - 'a';
                if (!node->next[idx]) node->next[idx] = new Node();
                node = node->next[idx];
            }
            node->end = true;
            maxLen = max(maxLen, (int)p.size());
        }
    }

    bool feed(char symbol) {
        buf.push_front(symbol);
        if ((int)buf.size() > maxLen) buf.pop_back();
        Node* node = root;
        for (char ch : buf) {                 // walk newest -> older
            int idx = ch - 'a';
            if (!node->next[idx]) return false;
            node = node->next[idx];
            if (node->end) return true;
        }
        return false;
    }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

typedef struct Node {
    struct Node* next[26];
    bool end;
} Node;

static Node* nodeNew(void) { return (Node*)calloc(1, sizeof(Node)); }

typedef struct {
    Node* root;
    char* buf;   /* newest symbol at index 0 */
    int size;
    int maxLen;
} PatternMonitor;

/* Build a trie of the patterns stored REVERSED, so that walking backward
   from the newest stream symbol matches a suffix. */
PatternMonitor* pmCreate(char** patterns, int patternsSize) {
    PatternMonitor* m = (PatternMonitor*)malloc(sizeof(PatternMonitor));
    m->root = nodeNew();
    m->maxLen = 0;
    for (int i = 0; i < patternsSize; ++i) {
        Node* node = m->root;
        int len = (int)strlen(patterns[i]);
        for (int j = len - 1; j >= 0; --j) {
            int idx = patterns[i][j] - 'a';
            if (!node->next[idx]) node->next[idx] = nodeNew();
            node = node->next[idx];
        }
        node->end = true;
        if (len > m->maxLen) m->maxLen = len;
    }
    m->buf = (char*)malloc((m->maxLen > 0 ? m->maxLen : 1) * sizeof(char));
    m->size = 0;
    return m;
}

bool pmFeed(PatternMonitor* m, char symbol) {
    /* keep only the last maxLen symbols; shift right, newest at front */
    if (m->size < m->maxLen) m->size++;
    for (int i = m->size - 1; i > 0; --i) m->buf[i] = m->buf[i - 1];
    m->buf[0] = symbol;

    Node* node = m->root;
    for (int i = 0; i < m->size; ++i) {
        int idx = m->buf[i] - 'a';
        if (!node->next[idx]) return false;
        node = node->next[idx];
        if (node->end) return true;
    }
    return false;
}

static void nodeFree(Node* n) {
    if (!n) return;
    for (int i = 0; i < 26; ++i) nodeFree(n->next[i]);
    free(n);
}

void pmFree(PatternMonitor* m) {
    nodeFree(m->root);
    free(m->buf);
    free(m);
}

#### Follow-up
questions
How do you bound memory so it does not grow with the stream length? (Only keep the last L symbols, since no pattern longer than L can match.)
What if the alphabet is arbitrary Unicode instead of 26 lowercase letters? (Use hash-map children instead of a fixed array.)
Can you make each query O(1) amortized regardless of pattern lengths? (Build an Aho-Corasick automaton over reversed patterns and follow failure links.)
Extend it to also report which pattern(s) matched, not just a boolean.

