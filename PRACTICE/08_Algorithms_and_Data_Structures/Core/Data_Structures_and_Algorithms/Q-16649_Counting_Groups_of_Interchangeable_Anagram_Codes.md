---
type: "practice"
difficulty: "Other"
---

# Counting Groups of Interchangeable Anagram Codes

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Graph](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Graph.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a list codes of strings. Every string in the list is guaranteed to be an anagram of every other string (same multiset of letters, same length).

Two codes X and Y are said to be directly interchangeable if at least one of the following holds:

X and Y are exactly equal, or
you can turn X into Y by swapping the letters at exactly two distinct positions of X.

Because all the strings are anagrams of one another, "swappable with one transposition" is the same as saying X and Y differ in at most two positions.

Interchangeability spreads transitively into groups: put two codes in the same group whenever they are directly interchangeable, and keep merging, so a code belongs to a group if it is directly interchangeable with at least one other member of that group. (Two codes can therefore land in the same group even if they are not directly interchangeable with each other, as long as a chain of interchangeable codes connects them.)

Return the number of distinct groups.

#### Cons
traints:

1 <= codes.length <= 300
1 <= codes[i].length <= 300
Every codes[i] consists of lowercase English letters only.
All strings in codes have the same length and are anagrams of each other.

#### Example
s & test cases
TEXT
Input: codes = ["abcd","bacd","abdc","dcba"] -> Output: 2
Why: "abcd" links to "bacd" (positions 0,1) and to "abdc" (positions 2,3), so those three form one group; "dcba" differs in all four positions from every other code and is a group of its own.

Input: codes = ["pq","qp"] -> Output: 1
Why: a single transposition turns one into the other, so they merge into one group.

Input: codes = ["xyz"] -> Output: 1
Why: a single code always forms exactly one group.

Input: codes = ["mn","mn"] -> Output: 1
Why: identical strings are directly interchangeable, so the duplicates collapse into one group.

Input: codes = ["abc","acb","bac","bca","cab","cba"] -> Output: 1
Why: every permutation is reachable from another by a single transposition through a chain (e.g. abc-acb-bca and abc-bac-cab), so all six connect into one group.

#### Approach

Treat each code as a node and connect two nodes with an edge whenever they are directly interchangeable; the answer is the number of connected components. The key observation is that since all strings are anagrams, they can never differ in exactly one position (a single mismatch would force a second mismatch elsewhere), so "reachable by one transposition" is equivalent to "differ in at most two positions" — a cheap character-diff test that short-circuits once the count exceeds 2. Feed the qualifying pairs into a disjoint-set-union (union-find) structure with path compression, starting the component count at n and decrementing every time two previously separate roots merge. This union-find over all O(n^2) candidate pairs (connected-components counting) is the canonical pattern; a DFS/BFS flood fill over the same implicit graph is an equally valid alternative with the same complexity.

Complexity

Time O(n^2 * m), where n is the number of codes and m is their common length: we test every unordered pair once, and each interchangeability test scans up to m characters (short-circuiting after the third mismatch). The union-find operations are effectively O(1) amortized with path compression. Space O(n) for the parent array. With n, m <= 300 this is about 300300300/2 ~ 1.4e7 character comparisons, well within limits.

Python
PYTHON
class Solution:
    def countGroups(self, codes):
        n = len(codes)
        parent = list(range(n))

        def find_root(x):
            while parent[x] != x:
                parent[x] = parent[parent[x]]
                x = parent[x]
            return x

        def interchangeable(a, b):
            diff = 0
            for ca, cb in zip(a, b):
                if ca != cb:
                    diff += 1
                    if diff > 2:
                        return False
            return True

        groups = n
        for i in range(n):
            for j in range(i + 1, n):
                if interchangeable(codes[i], codes[j]):
                    ri, rj = find_root(i), find_root(j)
                    if ri != rj:
                        parent[ri] = rj
                        groups -= 1
        return groups
JavaScript
JAVASCRIPT
function countGroups(codes) {

#### cons
t n = codes.length;

#### cons
t parent = Array.from({ length: n }, (_, i) => i);

    function findRoot(x) {
        while (parent[x] !== x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    function interchangeable(a, b) {
        let diff = 0;
        for (let i = 0; i < a.length; ++i)
            if (a[i] !== b[i] && ++diff > 2) return false;
        return true;
    }

    let groups = n;
    for (let i = 0; i < n; ++i)
        for (let j = i + 1; j < n; ++j)
            if (interchangeable(codes[i], codes[j])) {

#### cons
t ri = findRoot(i);

#### cons
t rj = findRoot(j);
                if (ri !== rj) {
                    parent[ri] = rj;
                    --groups;
                }
            }
    return groups;
}
Java
JAVA
class Solution {
    private int[] parent;

    public int countGroups(String[] codes) {
        int n = codes.length;
        parent = new int[n];
        for (int i = 0; i < n; ++i) parent[i] = i;

        int groups = n;
        for (int i = 0; i < n; ++i)
            for (int j = i + 1; j < n; ++j)
                if (interchangeable(codes[i], codes[j])) {
                    int ri = findRoot(i);
                    int rj = findRoot(j);
                    if (ri != rj) {
                        parent[ri] = rj;
                        --groups;
                    }
                }
        return groups;
    }

    private int findRoot(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    private boolean interchangeable(String a, String b) {
        int diff = 0;
        for (int i = 0; i < a.length(); ++i)
            if (a.charAt(i) != b.charAt(i) && ++diff > 2)
                return false;
        return true;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <numeric>
using namespace std;

class Solution {
public:
    int countGroups(vector<string>& codes) {
        int n = (int)codes.size();
        vector<int> parent(n);
        iota(parent.begin(), parent.end(), 0);

        int groups = n;
        for (int i = 0; i < n; ++i)
            for (int j = i + 1; j < n; ++j)
                if (interchangeable(codes[i], codes[j])) {
                    int ri = findRoot(parent, i);
                    int rj = findRoot(parent, j);
                    if (ri != rj) {
                        parent[ri] = rj;
                        --groups;
                    }
                }
        return groups;
    }

private:
    int findRoot(vector<int>& parent, int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    bool interchangeable(const string& a, const string& b) {
        int diff = 0;
        for (int i = 0; i < (int)a.size(); ++i)
            if (a[i] != b[i] && ++diff > 2)
                return false;
        return true;
    }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

static int findRoot(int *parent, int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]];
        x = parent[x];
    }
    return x;
}

static bool interchangeable(const char *a, const char *b) {
    int diff = 0;
    for (int i = 0; a[i] != '\0'; ++i)
        if (a[i] != b[i] && ++diff > 2)
            return false;
    return true;
}

int countGroups(char **codes, int n) {
    int *parent = (int *)malloc(n * sizeof(int));
    for (int i = 0; i < n; ++i) parent[i] = i;

    int groups = n;
    for (int i = 0; i < n; ++i)
        for (int j = i + 1; j < n; ++j)
            if (interchangeable(codes[i], codes[j])) {
                int ri = findRoot(parent, i);
                int rj = findRoot(parent, j);
                if (ri != rj) {
                    parent[ri] = rj;
                    --groups;
                }
            }

    free(parent);
    return groups;
}

#### Follow-up
questions
Prove that two anagrams can differ in 0 or exactly 2 positions but never in exactly 1, and use that to justify the "at most two mismatches" test.
When the word length m is much smaller than the count n, can you speed up pair-finding by generating all one-swap neighbors of each code and looking them up in a hash set instead of comparing every pair?
Add union-by-rank/size on top of path compression and discuss when it matters for the near-linear inverse-Ackermann bound.
Support codes streaming in one at a time, reporting the current group count after each insertion.

