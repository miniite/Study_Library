---
type: "practice"
difficulty: "Other"
---

# Longest Run of Identical Characters Under Point Updates

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Segment Tree](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Segment_Tree.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed string text consisting of lowercase English letters. You are also given a string newChars of length k and an integer array positions of length k, which together describe k update operations applied one after another.

The i-th update sets the character of text at index positions[i] to the character newChars[i]. Updates are cumulative: each one modifies the string produced by all previous updates.

After each update is applied, you must determine the length of the longest contiguous block of text that consists of a single repeating character (i.e., the longest maximal run of identical letters).

Return an integer array answer of length k, where answer[i] is that longest-run length after the i-th update has been performed.

#### Cons
traints:

1 <= text.length <= 10^5
text consists of lowercase English letters.
k == newChars.length == positions.length
1 <= k <= 10^5
newChars consists of lowercase English letters.
0 <= positions[i] < text.length

#### Example
s & test cases
TEXT
- Input: `text = "abbba", newChars = "ba", positions = [4,0]` -> Output: `[4,5]`. After update 1 the string is `"abbbb"` (index 4 becomes 'b'), longest run `"bbbb"` = 4. After update 2 the string is `"bbbbb"` (index 0 becomes 'b'), longest run = 5. Normal case where runs grow.
- Input: `text = "abcde", newChars = "aaaa", positions = [1,2,3,4]` -> Output: `[2,3,4,5]`. Each update extends a growing run of 'a' from the left: `"aacde"` -> `"aaade"` -> `"aaaae"` -> `"aaaaa"`. Shows a run merging step by step.
- Input: `text = "ppppp", newChars = "q", positions = [2]` -> Output: `[2]`. Overwriting the middle 'p' with 'q' splits `"ppppp"` into `"ppqpp"`, whose longest single-character run drops to 2. Shows a split shrinking the answer.
- Input: `text = "z", newChars = "a", positions = [0]` -> Output: `[1]`. Minimal string of length 1; after the update it is `"a"`, longest run = 1. Edge case n = 1.

#### Approach

A naive rescan of the whole string after every update is O(n*k) and too slow for 10^5 updates. The key is that "longest run of one repeating character" is a mergeable range statistic, so a segment tree maintains it under point updates. Each node stores, for its segment: the leftmost and rightmost characters, the length of the longest identical-character prefix, the longest such suffix, the longest run anywhere inside, and the segment length. Merging a left child L with a right child R: the answer is the max of the two children's best runs, plus a possible run crossing the boundary (L.suffix + R.prefix) but only when L.rightChar == R.leftChar; the merged prefix extends past L only if L is entirely one character that matches R's left char (and symmetrically for the suffix). A point update overwrites a leaf's character and re-pulls the O(log n) ancestors. The root's "longest run" field is the answer reported after each update. This is the canonical "segment tree storing prefix/suffix/best plus boundary chars to combine contiguous-run info" pattern (the same idea used for maximum-subarray or longest-balanced-segment trees).

Complexity

Time: O((n + k) log n) — building the tree is O(n) and each of the k point updates walks one root-to-leaf path recomputing O(1) merges per node, so O(log n) each. Space: O(n) for the segment tree arrays. Justification: the tree has O(n) nodes and every operation touches only one path of height log n.

Python
PYTHON
import sys
from typing import List

class Solution:
    def longestRepeating(self, text: str, newChars: str, positions: List[int]) -> List[int]:
        sys.setrecursionlimit(1 << 20)
        n = len(text)
        pre = [0] * (4 * n)
        suf = [0] * (4 * n)
        lng = [0] * (4 * n)
        ln  = [0] * (4 * n)          # segment length
        lc  = [''] * (4 * n)         # leftmost char
        rc  = [''] * (4 * n)         # rightmost char
        s = list(text)

        def pull(node):
            l, r = 2 * node, 2 * node + 1
            ln[node] = ln[l] + ln[r]
            lc[node] = lc[l]
            rc[node] = rc[r]
            lng[node] = max(lng[l], lng[r])
            if rc[l] == lc[r]:
                lng[node] = max(lng[node], suf[l] + pre[r])
            pre[node] = pre[l]
            if pre[l] == ln[l] and rc[l] == lc[r]:
                pre[node] = ln[l] + pre[r]
            suf[node] = suf[r]
            if suf[r] == ln[r] and rc[l] == lc[r]:
                suf[node] = ln[r] + suf[l]

        def build(node, lo, hi):
            if lo == hi:
                pre[node] = suf[node] = lng[node] = ln[node] = 1
                lc[node] = rc[node] = s[lo]
                return
            mid = (lo + hi) // 2
            build(2 * node, lo, mid)
            build(2 * node + 1, mid + 1, hi)
            pull(node)

        def update(node, lo, hi, idx, ch):
            if lo == hi:
                lc[node] = rc[node] = ch
                return
            mid = (lo + hi) // 2
            if idx <= mid:
                update(2 * node, lo, mid, idx, ch)
            else:
                update(2 * node + 1, mid + 1, hi, idx, ch)
            pull(node)

        build(1, 0, n - 1)
        ans = []
        for ch, idx in zip(newChars, positions):
            update(1, 0, n - 1, idx, ch)
            ans.append(lng[1])
        return ans
JavaScript
JAVASCRIPT
/**
 * @param {string} text
 * @param {string} newChars
 * @param {number[]} positions
 * @return {number[]}
 */
function longestRepeating(text, newChars, positions) {

#### cons
t n = text.length;

#### cons
t pre = new Int32Array(4 * n);

#### cons
t suf = new Int32Array(4 * n);

#### cons
t lng = new Int32Array(4 * n);

#### cons
t len = new Int32Array(4 * n);

#### cons
t lc = new Array(4 * n).fill('');

#### cons
t rc = new Array(4 * n).fill('');

#### cons
t s = text.split('');

    function pull(node) {

#### cons
t l = 2 * node, r = 2 * node + 1;
        len[node] = len[l] + len[r];
        lc[node] = lc[l];
        rc[node] = rc[r];
        lng[node] = Math.max(lng[l], lng[r]);
        if (rc[l] === lc[r]) lng[node] = Math.max(lng[node], suf[l] + pre[r]);
        pre[node] = pre[l];
        if (pre[l] === len[l] && rc[l] === lc[r]) pre[node] = len[l] + pre[r];
        suf[node] = suf[r];
        if (suf[r] === len[r] && rc[l] === lc[r]) suf[node] = len[r] + suf[l];
    }
    function build(node, lo, hi) {
        if (lo === hi) {
            pre[node] = suf[node] = lng[node] = len[node] = 1;
            lc[node] = rc[node] = s[lo];
            return;
        }

#### cons
t mid = (lo + hi) >> 1;
        build(2 * node, lo, mid);
        build(2 * node + 1, mid + 1, hi);
        pull(node);
    }
    function update(node, lo, hi, idx, ch) {
        if (lo === hi) { lc[node] = rc[node] = ch; return; }

#### cons
t mid = (lo + hi) >> 1;
        if (idx <= mid) update(2 * node, lo, mid, idx, ch);
        else update(2 * node + 1, mid + 1, hi, idx, ch);
        pull(node);
    }

    build(1, 0, n - 1);

#### cons
t ans = [];
    for (let i = 0; i < positions.length; i++) {
        update(1, 0, n - 1, positions[i], newChars[i]);
        ans.push(lng[1]);
    }
    return ans;
}
Java
JAVA
class Solution {
    private int[] pre, suf, lng, len;
    private char[] lc, rc;

    private void pull(int node) {
        int l = 2 * node, r = 2 * node + 1;
        len[node] = len[l] + len[r];
        lc[node] = lc[l];
        rc[node] = rc[r];
        lng[node] = Math.max(lng[l], lng[r]);
        if (rc[l] == lc[r]) lng[node] = Math.max(lng[node], suf[l] + pre[r]);
        pre[node] = pre[l];
        if (pre[l] == len[l] && rc[l] == lc[r]) pre[node] = len[l] + pre[r];
        suf[node] = suf[r];
        if (suf[r] == len[r] && rc[l] == lc[r]) suf[node] = len[r] + suf[l];
    }
    private void build(int node, int lo, int hi, char[] s) {
        if (lo == hi) {
            pre[node] = suf[node] = lng[node] = len[node] = 1;
            lc[node] = rc[node] = s[lo];
            return;
        }
        int mid = (lo + hi) / 2;
        build(2 * node, lo, mid, s);
        build(2 * node + 1, mid + 1, hi, s);
        pull(node);
    }
    private void update(int node, int lo, int hi, int idx, char ch) {
        if (lo == hi) { lc[node] = rc[node] = ch; return; }
        int mid = (lo + hi) / 2;
        if (idx <= mid) update(2 * node, lo, mid, idx, ch);
        else update(2 * node + 1, mid + 1, hi, idx, ch);
        pull(node);
    }
    public int[] longestRepeating(String text, String newChars, int[] positions) {
        int n = text.length();
        pre = new int[4 * n]; suf = new int[4 * n];
        lng = new int[4 * n]; len = new int[4 * n];
        lc = new char[4 * n]; rc = new char[4 * n];
        build(1, 0, n - 1, text.toCharArray());

        int k = positions.length;
        int[] ans = new int[k];
        for (int i = 0; i < k; i++) {
            update(1, 0, n - 1, positions[i], newChars.charAt(i));
            ans[i] = lng[1];
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

class Solution {
    vector<int> pre, suf, lng, len;
    vector<char> lc, rc;

    void pull(int node) {
        int l = 2 * node, r = 2 * node + 1;
        len[node] = len[l] + len[r];
        lc[node] = lc[l];
        rc[node] = rc[r];
        lng[node] = max(lng[l], lng[r]);
        if (rc[l] == lc[r]) lng[node] = max(lng[node], suf[l] + pre[r]);
        pre[node] = pre[l];
        if (pre[l] == len[l] && rc[l] == lc[r]) pre[node] = len[l] + pre[r];
        suf[node] = suf[r];
        if (suf[r] == len[r] && rc[l] == lc[r]) suf[node] = len[r] + suf[l];
    }
    void build(int node, int lo, int hi, const string& s) {
        if (lo == hi) {
            pre[node] = suf[node] = lng[node] = len[node] = 1;
            lc[node] = rc[node] = s[lo];
            return;
        }
        int mid = (lo + hi) / 2;
        build(2 * node, lo, mid, s);
        build(2 * node + 1, mid + 1, hi, s);
        pull(node);
    }
    void update(int node, int lo, int hi, int idx, char ch) {
        if (lo == hi) { lc[node] = rc[node] = ch; return; }
        int mid = (lo + hi) / 2;
        if (idx <= mid) update(2 * node, lo, mid, idx, ch);
        else update(2 * node + 1, mid + 1, hi, idx, ch);
        pull(node);
    }
public:
    vector<int> longestRepeating(string text, string newChars, vector<int>& positions) {
        int n = text.size();
        pre.assign(4 * n, 0); suf.assign(4 * n, 0);
        lng.assign(4 * n, 0); len.assign(4 * n, 0);
        lc.assign(4 * n, 0);  rc.assign(4 * n, 0);
        build(1, 0, n - 1, text);

        vector<int> ans;
        ans.reserve(positions.size());
        for (size_t i = 0; i < positions.size(); i++) {
            update(1, 0, n - 1, positions[i], newChars[i]);
            ans.push_back(lng[1]);
        }
        return ans;
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Segment tree with merge info, stored in file-scope arrays. */
static int *Pre, *Suf, *Lng, *Len;
static char *LC, *RC; /* leftmost / rightmost char of each segment */

static void pull(int node) {
    int l = 2 * node, r = 2 * node + 1;
    Len[node] = Len[l] + Len[r];
    LC[node] = LC[l];
    RC[node] = RC[r];
    Lng[node] = Lng[l] > Lng[r] ? Lng[l] : Lng[r];
    if (RC[l] == LC[r]) {
        int cross = Suf[l] + Pre[r];
        if (cross > Lng[node]) Lng[node] = cross;
    }
    Pre[node] = Pre[l];
    if (Pre[l] == Len[l] && RC[l] == LC[r]) Pre[node] = Len[l] + Pre[r];
    Suf[node] = Suf[r];
    if (Suf[r] == Len[r] && RC[l] == LC[r]) Suf[node] = Len[r] + Suf[l];
}

static void build(int node, int lo, int hi, const char* s) {
    if (lo == hi) {
        Pre[node] = Suf[node] = Lng[node] = Len[node] = 1;
        LC[node] = RC[node] = s[lo];
        return;
    }
    int mid = (lo + hi) / 2;
    build(2 * node, lo, mid, s);
    build(2 * node + 1, mid + 1, hi, s);
    pull(node);
}

static void update(int node, int lo, int hi, int idx, char ch) {
    if (lo == hi) { LC[node] = RC[node] = ch; return; }
    int mid = (lo + hi) / 2;
    if (idx <= mid) update(2 * node, lo, mid, idx, ch);
    else update(2 * node + 1, mid + 1, hi, idx, ch);
    pull(node);
}

/* Returns a heap array of length positionsSize (caller frees). */
int* longestRepeating(char* text, char* newChars, int* positions,
                      int positionsSize, int* returnSize) {
    int n = (int)strlen(text);
    Pre = (int*)malloc(sizeof(int) * 4 * n);
    Suf = (int*)malloc(sizeof(int) * 4 * n);
    Lng = (int*)malloc(sizeof(int) * 4 * n);
    Len = (int*)malloc(sizeof(int) * 4 * n);
    LC  = (char*)malloc(4 * n);
    RC  = (char*)malloc(4 * n);
    build(1, 0, n - 1, text);

    int* ans = (int*)malloc(sizeof(int) * positionsSize);
    for (int i = 0; i < positionsSize; i++) {
        update(1, 0, n - 1, positions[i], newChars[i]);
        ans[i] = Lng[1];
    }
    free(Pre); free(Suf); free(Lng); free(Len); free(LC); free(RC);
    *returnSize = positionsSize;
    return ans;
}

#### Follow-up
questions
How does the node merge change if you must also answer arbitrary range queries "longest run within [l, r]" instead of always the whole string?
Could you support range-assign updates (set an entire interval to one character) efficiently? What lazy-propagation info would each node need?
How would you adapt this to count the number of distinct maximal runs, or the total number of runs, after each update?
What breaks if you tried a balanced BST / ordered set of run boundaries instead of a segment tree, and what are the trade-offs?

