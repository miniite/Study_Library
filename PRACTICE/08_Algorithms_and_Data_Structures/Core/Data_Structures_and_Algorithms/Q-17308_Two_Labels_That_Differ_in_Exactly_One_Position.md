---
type: "practice"
difficulty: "Other"
---

# Two Labels That Differ in Exactly One Position

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hash Table](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hash_Table.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given an array labels containing strings that are all the same length and all distinct. Each string consists only of lowercase English letters.

Return true if there exist two different strings in labels that are identical at every position except exactly one index (i.e. they agree everywhere else and disagree at a single position). Otherwise return false.

In other words, decide whether some pair of strings has a Hamming distance of exactly one.

Input

An array of strings labels, every string having the same length.

Output

A boolean: true if such a pair exists, false otherwise.

#### Cons
traints

The total number of characters across all strings is at most 10^5.
All strings have equal length: labels[i].length == labels[j].length for every i, j.
Every string in labels is unique.
Every string contains only lowercase English letters.

Follow up: Can you solve it in O(n * m) time, where n is the number of strings and m is the length of each string?

#### Example
s & test cases
TEXT
- Input: labels = ["stone","shone","stony"] -> Output: true
  Why: "stone" and "shone" match everywhere except index 1 ('t' vs 'h').
- Input: labels = ["go","by","me"] -> Output: false
  Why: every pair differs in both positions, so no pair has distance exactly one.
- Input: labels = ["code","cope","cove","card"] -> Output: true
  Why: "code" and "cope" agree except at index 2 ('d' vs 'p').
- Input: labels = ["a"] -> Output: false
  Why: fewer than two strings, so no pair can be formed.
- Input: labels = ["ab","aa"] -> Output: true
  Why: shortest non-trivial case; they differ only at the last position.
- Input: labels = ["xy","yx"] -> Output: false
  Why: the two strings disagree at both indices, a Hamming distance of two.

#### Approach

For every column position i, two strings differ by exactly one character at i precisely when they are equal at all other positions. "Blanking" position i and comparing the rest turns the problem into a duplicate search. To do the comparison in O(1) instead of O(m), precompute a polynomial (rolling) hash of every string; the hash with position i removed is H - value(s[i]) * base^(m-1-i). Iterate position by position, inserting each masked hash into a set for that column; if a masked hash repeats, two distinct strings share the same "blanked" form and therefore differ only at that one index, so return true. Double hashing (two independent moduli) drives the false-positive probability negligibly low. The pattern is rolling-hash fingerprinting combined with a hash-set duplicate scan.

Complexity

Time: O(n * m). Each string's full polynomial hash is built once in O(n * m) total; then for each of the m positions we scan all n strings deriving a masked hash in O(1). Space: O(n) for the per-position hash set (plus O(m) for the precomputed powers and O(n) for the stored hashes).

Python
PYTHON
from typing import List

def differ_by_one(labels: List[str]) -> bool:
    n = len(labels)
    if n < 2:
        return False
    m = len(labels[0])
    if m == 0:
        return False

    MOD1, MOD2, B = 1_000_000_007, 1_000_000_009, 131
    p1 = [1] * m
    p2 = [1] * m
    for k in range(1, m):
        p1[k] = p1[k - 1] * B % MOD1
        p2[k] = p2[k - 1] * B % MOD2

    H1 = [0] * n
    H2 = [0] * n
    for idx, s in enumerate(labels):
        h1 = h2 = 0
        for ch in s:
            v = ord(ch) - 96  # 'a' -> 1
            h1 = (h1 * B + v) % MOD1
            h2 = (h2 * B + v) % MOD2
        H1[idx], H2[idx] = h1, h2

    for i in range(m):
        seen = set()
        w1, w2 = p1[m - 1 - i], p2[m - 1 - i]
        for idx in range(n):
            v = ord(labels[idx][i]) - 96
            mh1 = (H1[idx] - v * w1) % MOD1
            mh2 = (H2[idx] - v * w2) % MOD2
            key = (mh1, mh2)
            if key in seen:
                return True
            seen.add(key)
    return False
JavaScript
JAVASCRIPT
function differByOne(labels) {

#### cons
t n = labels.length;
    if (n < 2) return false;

#### cons
t m = labels[0].length;
    if (m === 0) return false;

#### cons
t MOD1 = 1000000007n, MOD2 = 1000000009n, B = 131n;

#### cons
t p1 = new Array(m), p2 = new Array(m);
    p1[0] = 1n; p2[0] = 1n;
    for (let k = 1; k < m; k++) {
        p1[k] = (p1[k - 1] * B) % MOD1;
        p2[k] = (p2[k - 1] * B) % MOD2;
    }

#### cons
t H1 = new Array(n), H2 = new Array(n);
    for (let idx = 0; idx < n; idx++) {
        let h1 = 0n, h2 = 0n;

#### cons
t s = labels[idx];
        for (let j = 0; j < m; j++) {

#### cons
t v = BigInt(s.charCodeAt(j) - 96); // 'a' -> 1
            h1 = (h1 * B + v) % MOD1;
            h2 = (h2 * B + v) % MOD2;
        }
        H1[idx] = h1; H2[idx] = h2;
    }

    for (let i = 0; i < m; i++) {

#### cons
t seen = new Set();

#### cons
t w1 = p1[m - 1 - i], w2 = p2[m - 1 - i];
        for (let idx = 0; idx < n; idx++) {

#### cons
t v = BigInt(labels[idx].charCodeAt(i) - 96);

#### cons
t mh1 = ((H1[idx] - v * w1) % MOD1 + MOD1) % MOD1;

#### cons
t mh2 = ((H2[idx] - v * w2) % MOD2 + MOD2) % MOD2;

#### cons
t key = (mh1 * MOD2 + mh2).toString();
            if (seen.has(key)) return true;
            seen.add(key);
        }
    }
    return false;
}
Java
JAVA
import java.util.HashSet;
import java.util.Set;

class Solution {
    public boolean differByOne(String[] labels) {
        int n = labels.length;
        if (n < 2) return false;
        int m = labels[0].length();
        if (m == 0) return false;

        final long MOD1 = 1000000007L, MOD2 = 1000000009L, B = 131;
        long[] p1 = new long[m], p2 = new long[m];
        p1[0] = p2[0] = 1;
        for (int k = 1; k < m; k++) {
            p1[k] = p1[k-1] * B % MOD1;
            p2[k] = p2[k-1] * B % MOD2;
        }

        long[] H1 = new long[n], H2 = new long[n];
        for (int idx = 0; idx < n; idx++) {
            long h1 = 0, h2 = 0;
            String s = labels[idx];
            for (int j = 0; j < m; j++) {
                long v = s.charAt(j) - 'a' + 1;
                h1 = (h1 * B + v) % MOD1;
                h2 = (h2 * B + v) % MOD2;
            }
            H1[idx] = h1; H2[idx] = h2;
        }

        for (int i = 0; i < m; i++) {
            Set<Long> seen = new HashSet<>();
            long w1 = p1[m-1-i], w2 = p2[m-1-i];
            for (int idx = 0; idx < n; idx++) {
                long v = labels[idx].charAt(i) - 'a' + 1;
                long mh1 = ((H1[idx] - v * w1) % MOD1 + MOD1) % MOD1;
                long mh2 = ((H2[idx] - v * w2) % MOD2 + MOD2) % MOD2;
                long key = mh1 * MOD2 + mh2;
                if (!seen.add(key)) return true;
            }
        }
        return false;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <unordered_set>
using namespace std;

class Solution {
public:
    bool differByOne(vector<string>& labels) {
        int n = labels.size();
        if (n < 2) return false;
        int m = labels[0].size();
        if (m == 0) return false;

#### cons
t long long MOD1 = 1000000007LL, MOD2 = 1000000009LL, B = 131;
        vector<long long> p1(m), p2(m);
        p1[0] = p2[0] = 1;
        for (int k = 1; k < m; ++k) {
            p1[k] = p1[k-1] * B % MOD1;
            p2[k] = p2[k-1] * B % MOD2;
        }

        vector<long long> H1(n), H2(n);
        for (int idx = 0; idx < n; ++idx) {
            long long h1 = 0, h2 = 0;
            for (int j = 0; j < m; ++j) {
                long long v = labels[idx][j] - 'a' + 1;
                h1 = (h1 * B + v) % MOD1;
                h2 = (h2 * B + v) % MOD2;
            }
            H1[idx] = h1; H2[idx] = h2;
        }

        for (int i = 0; i < m; ++i) {
            unordered_set<unsigned long long> seen;
            seen.reserve(n * 2);
            long long w1 = p1[m-1-i], w2 = p2[m-1-i];
            for (int idx = 0; idx < n; ++idx) {
                long long v = labels[idx][i] - 'a' + 1;
                long long mh1 = ((H1[idx] - v * w1) % MOD1 + MOD1) % MOD1;
                long long mh2 = ((H2[idx] - v * w2) % MOD2 + MOD2) % MOD2;
                unsigned long long key =
                    (unsigned long long)mh1 * (unsigned long long)MOD2 + (unsigned long long)mh2;
                if (!seen.insert(key).second) return true;
            }
        }
        return false;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define MOD1 1000000007LL
#define MOD2 1000000009LL
#define BASE 131LL

bool differByOne(char** labels, int n) {
    if (n < 2) return false;
    int m = (int)strlen(labels[0]);
    if (m == 0) return false;

    long long *p1 = malloc(sizeof(long long) * m);
    long long *p2 = malloc(sizeof(long long) * m);
    p1[0] = p2[0] = 1;
    for (int k = 1; k < m; k++) {
        p1[k] = p1[k-1] * BASE % MOD1;
        p2[k] = p2[k-1] * BASE % MOD2;
    }

    /* Full polynomial hash of each string under two moduli. */
    long long *H1 = malloc(sizeof(long long) * n);
    long long *H2 = malloc(sizeof(long long) * n);
    for (int idx = 0; idx < n; idx++) {
        long long h1 = 0, h2 = 0;

#### cons
t char *s = labels[idx];
        for (int j = 0; j < m; j++) {
            long long v = s[j] - 'a' + 1;
            h1 = (h1 * BASE + v) % MOD1;
            h2 = (h2 * BASE + v) % MOD2;
        }
        H1[idx] = h1; H2[idx] = h2;
    }

    /* Power-of-two capacity for a linear-probing hash set of masked keys. */
    int cap = 1;
    while (cap < 2 * n) cap <<= 1;
    unsigned long long *keys = malloc(sizeof(unsigned long long) * cap);
    char *used = malloc(cap);

    bool found = false;
    for (int i = 0; i < m && !found; i++) {
        memset(used, 0, cap);
        long long w1 = p1[m-1-i], w2 = p2[m-1-i];
        for (int idx = 0; idx < n; idx++) {
            long long v = labels[idx][i] - 'a' + 1;
            long long mh1 = ((H1[idx] - v * w1) % MOD1 + MOD1) % MOD1;
            long long mh2 = ((H2[idx] - v * w2) % MOD2 + MOD2) % MOD2;
            unsigned long long key =
                (unsigned long long)mh1 * (unsigned long long)MOD2 + (unsigned long long)mh2;
            unsigned int h = (unsigned int)(key & (cap - 1));
            bool hit = false;
            while (used[h]) {
                if (keys[h] == key) { hit = true; break; }
                h = (h + 1) & (cap - 1);
            }
            if (hit) { found = true; break; }
            used[h] = 1; keys[h] = key;
        }
    }

    free(p1); free(p2); free(H1); free(H2); free(keys); free(used);
    return found;
}

int main(void) {
    char* a[] = {"stone", "shone", "stony"};
    printf("%s\n", differByOne(a, 3) ? "true" : "false");   /* true */
    return 0;
}

#### Follow-up
questions
How would you eliminate any chance of a hash collision producing a wrong answer (e.g. explicit verification or three moduli)?
Return the actual pair of matching strings (or their indices) rather than just a boolean.
Generalize to detect pairs whose Hamming distance is at most k instead of exactly one.
What changes if the strings can have different lengths, so alignment is no longer positional?

