---
type: "practice"
difficulty: "Other"
---

# Replace Words With Their Shortest Base Form

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a collection of short lowercase strings called bases, and a line of text line made of lowercase words separated by single spaces.

A word in the line can sometimes be shortened. If one of the base strings is a prefix of the word, the word may be replaced by that base. When several different bases are each a prefix of the same word, replace the word with the shortest such base. If no base is a prefix of a word, that word is left unchanged.

Return the line after performing every possible replacement.

Input:

bases: a list of lowercase strings.
line: a string of lowercase words separated by single spaces.

Output: the transformed line as a single string, with words in the same order and still separated by single spaces.

#### Cons
traints:

1 <= number of bases <= 1000
1 <= length of each base <= 100
Each base consists only of lowercase English letters.
1 <= number of words in line <= 1000
1 <= length of each word <= 1000
1 <= total length of line <= 10^6
line contains only lowercase letters and single spaces; it has no leading or trailing spaces, and exactly one space separates any two consecutive words.

Note: "shortest base" is decided by length. A base that equals the whole word counts as a valid prefix (replacing the word by itself, i.e. no visible change).

#### Example
s & test cases
TEXT
Input: bases = ["sun","moon","star"], line = "sunlight and moonbeam over starlight" -> Output: "sun and moon over star"
Why: each long word has a base as its prefix, so it is replaced by that base; "and"/"over" have no base prefix and stay.

Input: bases = ["a","b","c"], line = "apple berry cherry date" -> Output: "a b c date"
Why: the single-letter bases are prefixes of the first three words; "date" starts with 'd', has no base prefix, so it is unchanged.

Input: bases = ["go","gone"], line = "gone" -> Output: "go"
Why: both "go" and "gone" are prefixes of "gone"; the shorter base "go" wins.

Input: bases = ["cat"], line = "dog fish" -> Output: "dog fish"
Why: no base is a prefix of any word, so the line is returned unchanged.

Input: bases = ["ring"], line = "ring" -> Output: "ring"
Why: the base equals the entire word, so it is replaced by itself (no visible change).

Input: bases = ["work","working"], line = "worker works reworked" -> Output: "work work reworked"
Why: "worker" and "works" begin with "work"; "reworked" begins with "re" (no base prefix), so it is kept as is.

#### Approach

Build a prefix tree (trie) from all base strings, marking the node where each base ends. For every word in the line, walk the word character by character down the trie: the very first end-marked node you reach corresponds to the shortest base that is a prefix of the word, so replace the word with that prefix and stop. If the walk falls off the trie (a needed child is missing) before hitting any end marker, no base is a prefix and the word is kept unchanged. Rejoin the transformed words with single spaces. The trie makes prefix lookup proportional to word length instead of testing each possible prefix separately, and because a trie stores characters shortest-to-longest along a path, the first end marker is automatically the shortest matching base.

Complexity

Time: O(D + L), where D is the total number of characters across all base strings (to build the trie) and L is the total length of the line (each character of every word is visited at most once while walking the trie). Space: O(D) for the trie plus O(L) for the output. This beats the naive "try every prefix against a hash set" approach, which costs O(w^2) per word of length w.

Python
PYTHON
from typing import List

class Solution:
    def shortenWords(self, bases: List[str], line: str) -> str:
        root = {}
        for b in bases:
            node = root
            for ch in b:
                node = node.setdefault(ch, {})
            node['$'] = True   # marks the end of a base string

        def shortest(word: str) -> str:
            node = root
            for i, ch in enumerate(word):
                if ch not in node:
                    return word            # no base along this path
                node = node[ch]
                if '$' in node:
                    return word[:i + 1]     # first end reached = shortest base
            return word

        return ' '.join(shortest(w) for w in line.split(' '))
JavaScript
JAVASCRIPT
function shortenWords(bases, line) {

#### cons
t root = {};
    for (const b of bases) {
        let node = root;
        for (const ch of b) {
            if (!node[ch]) node[ch] = {};
            node = node[ch];
        }
        node.end = true;   // 'end' can never collide with a single-char key
    }

#### cons
t shortest = (word) => {
        let node = root;
        for (let i = 0; i < word.length; i++) {
            node = node[word[i]];
            if (!node) return word;                 // no base along this path
            if (node.end) return word.slice(0, i + 1); // shortest base
        }
        return word;
    };

    return line.split(' ').map(shortest).join(' ');
}
Java
JAVA
import java.util.List;

class Solution {
    private static class TrieNode {
        TrieNode[] child = new TrieNode[26];
        boolean isEnd = false;
    }

    private final TrieNode root = new TrieNode();

    public String shortenWords(List<String> bases, String line) {
        for (String b : bases) insert(b);

        String[] words = line.split(" ");
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < words.length; i++) {
            if (i > 0) sb.append(' ');
            sb.append(shortestBase(words[i]));
        }
        return sb.toString();
    }

    private void insert(String word) {
        TrieNode node = root;
        for (int k = 0; k < word.length(); k++) {
            int i = word.charAt(k) - 'a';
            if (node.child[i] == null) node.child[i] = new TrieNode();
            node = node.child[i];
        }
        node.isEnd = true;
    }

    private String shortestBase(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            node = node.child[word.charAt(i) - 'a'];
            if (node == null) return word;                 // no base on this path
            if (node.isEnd) return word.substring(0, i + 1); // shortest base
        }
        return word;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <array>
#include <sstream>
using namespace std;

class Solution {
public:
    string shortenWords(vector<string>& bases, string line) {
        for (const string& b : bases) insert(b);

        string ans;
        istringstream iss(line);
        for (string w; iss >> w; ) {
            if (!ans.empty()) ans += ' ';
            ans += shortestBase(w);
        }
        return ans;
    }

private:
    struct TrieNode {
        array<TrieNode*, 26> child{};
        bool isEnd = false;
    };
    TrieNode* root = new TrieNode();

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int i = c - 'a';
            if (!node->child[i]) node->child[i] = new TrieNode();
            node = node->child[i];
        }
        node->isEnd = true;
    }

    string shortestBase(const string& word) {
        TrieNode* node = root;
        for (int i = 0; i < (int)word.size(); ++i) {
            node = node->child[word[i] - 'a'];
            if (!node) return word;                 // no base along this path
            if (node->isEnd) return word.substr(0, i + 1);  // shortest base
        }
        return word;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct TrieNode {
    struct TrieNode* children[26];
    int isEnd;   /* 1 if a base string ends here */
} TrieNode;

static TrieNode* newNode(void) {
    return (TrieNode*)calloc(1, sizeof(TrieNode));
}

static void insert(TrieNode* root, const char* word) {
    TrieNode* node = root;
    for (int i = 0; word[i]; ++i) {
        int c = word[i] - 'a';
        if (!node->children[c]) node->children[c] = newNode();
        node = node->children[c];
    }
    node->isEnd = 1;
}

/* Length of the shortest base that is a prefix of word[0..len), or -1 if none. */
static int shortestBaseLen(TrieNode* root, const char* word, int len) {
    TrieNode* node = root;
    for (int i = 0; i < len; ++i) {
        int c = word[i] - 'a';
        if (!node->children[c]) return -1;
        node = node->children[c];
        if (node->isEnd) return i + 1;   /* first end reached = shortest base */
    }
    return -1;
}

static void freeTrie(TrieNode* node) {
    if (!node) return;
    for (int i = 0; i < 26; ++i) freeTrie(node->children[i]);
    free(node);
}

char* shortenWords(char** bases, int basesCount, const char* line) {
    TrieNode* root = newNode();
    for (int i = 0; i < basesCount; ++i) insert(root, bases[i]);

    int n = (int)strlen(line);
    char* out = (char*)malloc(n + 1);   /* output never longer than input */
    int oi = 0, i = 0;
    while (i < n) {
        int j = i;
        while (j < n && line[j] != ' ') ++j;      /* word occupies [i, j) */
        int wlen = j - i;
        int slen = shortestBaseLen(root, line + i, wlen);
        int keep = (slen == -1) ? wlen : slen;
        memcpy(out + oi, line + i, keep);
        oi += keep;
        if (j < n) out[oi++] = ' ';               /* preserve the separator */
        i = j + 1;
    }
    out[oi] = '\0';
    freeTrie(root);
    return out;
}

#### Follow-up
questions
If two bases of the same (shortest) length are both prefixes, break the tie by lexicographic order — how does the trie walk change?
What if you must replace each word with the LONGEST matching base instead of the shortest? (Walk to the deepest end marker.)
Support mixed-case letters or a larger alphabet — how do you size or replace the fixed 26-child array?
Stream the line word by word (it may be too large to hold in memory) while reusing the same trie.

