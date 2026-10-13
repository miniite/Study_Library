---
type: "practice"
difficulty: "Other"
---

# All Substring Match Positions from a Pattern Set

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Trie](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Trie.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string s and a list of non-empty strings patterns. All strings consist of lowercase English letters, and every string in patterns is unique.

For every occurrence of a pattern as a contiguous substring of s, report the pair of indices [i, j] (both 0-based, inclusive) such that the substring s[i..j] (the characters from position i through position j) is exactly equal to one of the patterns.

Return the list of all such [i, j] pairs, sorted in ascending order — first by the starting index i, and when two pairs share the same i, by the ending index j.

Matches are allowed to overlap: the same character position may participate in several reported pairs, and one starting index may produce several pairs if multiple patterns (of different lengths) all start there.

#### Cons
traints

1 <= s.length <= 100
1 <= patterns.length <= 20
1 <= patterns[i].length <= 50
s and every patterns[i] consist only of lowercase English letters.
All strings in patterns are distinct.

#### Example
s & test cases
TEXT
Input: s = "sheisselling", patterns = ["she","is","sell","hei"] -> Output: [[0,2],[1,3],[3,4],[5,8]]
Why: "she" occurs at 0..2, "hei" at 1..3, "is" at 3..4, "sell" at 5..8; sorting by (i then j) gives this order.

Input: s = "aaaa", patterns = ["a","aa"] -> Output: [[0,0],[0,1],[1,1],[1,2],[2,2],[2,3],[3,3]]
Why: both patterns match repeatedly and overlap; each starting index emits "a" (single char) and, where room remains, "aa", sorted by start then end.

Input: s = "abcabc", patterns = ["abc","bca"] -> Output: [[0,2],[1,3],[3,5]]
Why: "abc" appears at 0..2 and 3..5, "bca" appears once at 1..3; overlapping matches from different patterns are all reported.

Input: s = "hello", patterns = ["xy","zzz"] -> Output: []
Why: none of the patterns occur anywhere in s, so there are no index pairs.

Input: s = "code", patterns = ["code"] -> Output: [[0,3]]
Why: the single pattern equals the whole string, matching exactly once from index 0 to 3.

Input: s = "x", patterns = ["x"] -> Output: [[0,0]]
Why: minimum-size case — a one-character string matched by a one-character pattern.

#### Approach

Insert all patterns into a trie so that shared prefixes are stored once. Then anchor a scan at every starting index i of s and walk down the trie one character at a time following s[i], s[i+1], ...; the moment the current trie node has no child for the next character, no longer pattern can start at i, so stop early. Whenever the walk lands on a node flagged as a pattern end, the substring s[i..j] is one of the patterns, so record [i, j]. Because the outer loop advances i in increasing order and, for a fixed i, the inner walk advances j in increasing order, the pairs are emitted already sorted by (i, j) — no separate sort is needed. The trie is what makes this efficient: instead of testing every pattern against every position independently, all patterns sharing a prefix are matched simultaneously in a single walk. (For very large inputs an Aho-Corasick automaton would give O(n + total), but the trie-walk is optimal and simplest within the given bounds.)

Complexity

Time: O(n * L + total) where n = s.length, L = length of the longest pattern, and total = number of reported pairs. For each of the n starting positions the trie walk advances at most L steps before it either falls off the trie or exhausts s, and each match found is appended in O(1). Space: O(P) for the trie, where P is the total number of characters across all patterns (plus O(total) for the output).

Python
PYTHON
from typing import List

def match_positions(s: str, patterns: List[str]) -> List[List[int]]:
    # Build a trie of all patterns.
    root: dict = {}
    for w in patterns:
        node = root
        for ch in w:
            node = node.setdefault(ch, {})
        node["$"] = True  # marks end of a pattern

    n = len(s)
    res: List[List[int]] = []
    # i grows outward, j grows inward -> pairs are produced in sorted order.
    for i in range(n):
        node = root
        for j in range(i, n):
            ch = s[j]
            if ch not in node:
                break
            node = node[ch]
            if "$" in node:
                res.append([i, j])
    return res
JavaScript
JAVASCRIPT
function matchPositions(s, patterns) {
    // Build a trie of all patterns.

#### cons
t root = {};
    for (const w of patterns) {
        let node = root;
        for (const ch of w) {
            if (!node[ch]) node[ch] = {};
            node = node[ch];
        }
        node.end = true;
    }

#### cons
t n = s.length;

#### cons
t res = [];
    // i outer increasing, j inner increasing -> output already sorted.
    for (let i = 0; i < n; i++) {
        let node = root;
        for (let j = i; j < n; j++) {

#### cons
t ch = s[j];
            if (!node[ch]) break;
            node = node[ch];
            if (node.end) res.push([i, j]);
        }
    }
    return res;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    static class Trie {
        Trie[] next = new Trie[26];
        boolean isEnd = false;
    }

    public List<int[]> matchPositions(String s, String[] patterns) {
        Trie root = new Trie();
        for (String w : patterns) {
            Trie cur = root;
            for (int k = 0; k < w.length(); k++) {
                int idx = w.charAt(k) - 'a';
                if (cur.next[idx] == null) cur.next[idx] = new Trie();
                cur = cur.next[idx];
            }
            cur.isEnd = true;
        }

        int n = s.length();
        List<int[]> res = new ArrayList<>();
        // i outer increasing, j inner increasing -> naturally sorted output.
        for (int i = 0; i < n; i++) {
            Trie cur = root;
            for (int j = i; j < n; j++) {
                int idx = s.charAt(j) - 'a';
                if (cur.next[idx] == null) break;
                cur = cur.next[idx];
                if (cur.isEnd) res.add(new int[]{i, j});
            }
        }
        return res;
    }
}
C++
CPP
#include <vector>
#include <string>
using namespace std;

struct Trie {
    Trie* next[26] = {};
    bool isEnd = false;
};

vector<vector<int>> matchPositions(const string& s, const vector<string>& patterns) {
    Trie* root = new Trie();
    for (const string& w : patterns) {
        Trie* cur = root;
        for (char c : w) {
            int idx = c - 'a';
            if (!cur->next[idx]) cur->next[idx] = new Trie();
            cur = cur->next[idx];
        }
        cur->isEnd = true;
    }

    int n = (int)s.size();
    vector<vector<int>> res;
    // i outer (increasing), j inner (increasing) -> results already sorted.
    for (int i = 0; i < n; i++) {
        Trie* cur = root;
        for (int j = i; j < n; j++) {
            int idx = s[j] - 'a';
            if (!cur->next[idx]) break;
            cur = cur->next[idx];
            if (cur->isEnd) res.push_back({i, j});
        }
    }
    return res;
}
C
C
#include <stdlib.h>
#include <string.h>

typedef struct Trie {
    struct Trie* next[26];
    int isEnd;
} Trie;

static Trie* newNode(void) {
    Trie* t = (Trie*)calloc(1, sizeof(Trie));
    return t;
}

static void freeTrie(Trie* t) {
    if (!t) return;
    for (int k = 0; k < 26; k++) freeTrie(t->next[k]);
    free(t);
}

/* Returns array of [i,j] pairs sorted by (i,j). Caller frees. */
int** matchPositions(char* s, char** patterns, int patternsSize,
                     int* returnSize, int** returnColumnSizes) {
    Trie* root = newNode();
    for (int p = 0; p < patternsSize; p++) {
        Trie* cur = root;
        for (char* c = patterns[p]; *c; c++) {
            int idx = *c - 'a';
            if (!cur->next[idx]) cur->next[idx] = newNode();
            cur = cur->next[idx];
        }
        cur->isEnd = 1;
    }

    int n = (int)strlen(s);
    int cap = 16, cnt = 0;
    int** res = (int**)malloc(sizeof(int*) * cap);

    /* i grows outward, j grows inward -> pairs come out already sorted. */
    for (int i = 0; i < n; i++) {
        Trie* cur = root;
        for (int j = i; j < n; j++) {
            int idx = s[j] - 'a';
            if (!cur->next[idx]) break;
            cur = cur->next[idx];
            if (cur->isEnd) {
                if (cnt == cap) {
                    cap *= 2;
                    res = (int**)realloc(res, sizeof(int*) * cap);
                }
                res[cnt] = (int*)malloc(sizeof(int) * 2);
                res[cnt][0] = i;
                res[cnt][1] = j;
                cnt++;
            }
        }
    }

    freeTrie(root);
    *returnSize = cnt;
    *returnColumnSizes = (int*)malloc(sizeof(int) * cnt);
    for (int k = 0; k < cnt; k++) (*returnColumnSizes)[k] = 2;
    return res;
}

#### Follow-up
questions
Scale it up: s is very long and there are thousands of patterns — replace the per-start trie walk with an Aho-Corasick automaton to achieve O(n + matches).
Return only the count of matching pairs, or the number of distinct positions covered by at least one match.
Support case-insensitive matching or an extended alphabet (Unicode) — how do the trie children and the char indexing change?
Report, for each start index, only the longest matching pattern instead of all matches.

