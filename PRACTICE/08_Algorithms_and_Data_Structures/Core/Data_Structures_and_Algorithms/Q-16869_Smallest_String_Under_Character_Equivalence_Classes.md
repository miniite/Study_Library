---
type: "practice"
difficulty: "Other"
---

# Smallest String Under Character Equivalence Classes

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two strings first and second of the same length, and a third string target.

For every position i, the characters first[i] and second[i] are declared equivalent. Equivalence obeys the usual rules of an equivalence relation:

Reflexive: every character is equivalent to itself.
Symmetric: if x is equivalent to y, then y is equivalent to x.
Transitive: if x is equivalent to y and y is equivalent to z, then x is equivalent to z.

These rules partition the lowercase letters into equivalence classes (groups of mutually equivalent characters). Any character may be freely replaced by any other character in its class.

Transform target by replacing each of its characters with the lexicographically smallest character in that character's equivalence class, and return the resulting string. The result is the smallest string obtainable from target under these equivalences.

Input

first, second: equal-length strings of lowercase English letters describing the equivalences.
target: a string of lowercase English letters to transform.

Output

The lexicographically smallest string equivalent to target.

#### Cons
traints

1 <= first.length, second.length, target.length <= 1000
first.length == second.length
first, second, and target consist only of lowercase English letters.

#### Example
s & test cases
TEXT
Input: first = "cat", second = "dog", target = "toad" -> Output: "gaac"
Why: Pairs (c,d), (a,o), (t,g) form classes {c,d}, {a,o}, {g,t}; smallest of each is c, a, g. So t->g, o->a, a->a, d->c.

Input: first = "abc", second = "bcd", target = "dad" -> Output: "aaa"
Why: Transitivity chains a=b, b=c, c=d into one class {a,b,c,d} with smallest a; every letter maps to a.

Input: first = "xy", second = "yz", target = "wut" -> Output: "wut"
Why: Class {x,y,z} does not include w, u, or t, so target is unchanged (no applicable equivalence).

Input: first = "aa", second = "bb", target = "b" -> Output: "a"
Why: Repeated identical pair still just unions a and b; the single character b becomes a.

Input: first = "ab", second = "ba", target = "ab" -> Output: "aa"
Why: Symmetric pairs (a,b) and (b,a) both union a and b; both letters map to the smaller, a.

#### Approach

Model the 26 lowercase letters as nodes in a Disjoint Set Union (Union-Find). Each pair (first[i], second[i]) unions two letters into the same equivalence class. The key trick is to always keep the numerically smallest letter as the class root: when uniting, attach the larger root beneath the smaller one. After processing all pairs, the root of any character's set is guaranteed to be the lexicographically smallest character in that class, so translating each character of target to find(c) yields the smallest equivalent string in a single pass. Path compression keeps the finds near-constant. Pattern: Union-Find / equivalence classes with a min-root tie-break.

Complexity

Time: O((n + m) * α(26)) which is effectively O(n + m), where n is the length of the equivalence strings and m is the length of target; each union/find is near-constant thanks to path compression on a fixed 26-node forest. Space: O(1) — the disjoint-set array has a fixed size of 26 regardless of input.

Python
PYTHON
class Solution:
    def smallestEquivalentString(self, first: str, second: str, target: str) -> str:
        parent = list(range(26))

        def find(x: int) -> int:
            while parent[x] != x:
                parent[x] = parent[parent[x]]  # path halving
                x = parent[x]
            return x

        def unite(a: int, b: int) -> None:
            ra, rb = find(a), find(b)
            if ra < rb:
                parent[rb] = ra
            else:
                parent[ra] = rb

        for a, b in zip(first, second):
            unite(ord(a) - 97, ord(b) - 97)

        return ''.join(chr(97 + find(ord(c) - 97)) for c in target)
JavaScript
JAVASCRIPT
function smallestEquivalentString(first, second, target) {

#### cons
t parent = Array.from({ length: 26 }, (_, i) => i);

#### cons
t find = (x) => {
        while (parent[x] !== x) {
            parent[x] = parent[parent[x]]; // path halving
            x = parent[x];
        }
        return x;
    };

#### cons
t unite = (a, b) => {

#### cons
t ra = find(a), rb = find(b);
        if (ra < rb) parent[rb] = ra;
        else parent[ra] = rb;
    };

    for (let i = 0; i < first.length; i++)
        unite(first.charCodeAt(i) - 97, second.charCodeAt(i) - 97);

    let out = '';
    for (const c of target)
        out += String.fromCharCode(97 + find(c.charCodeAt(0) - 97));
    return out;
}
Java
JAVA
class Solution {
    private int[] parent;

    public String smallestEquivalentString(String first, String second, String target) {
        parent = new int[26];
        for (int i = 0; i < 26; i++) parent[i] = i;
        for (int i = 0; i < first.length(); i++)
            unite(first.charAt(i) - 'a', second.charAt(i) - 'a');

        StringBuilder sb = new StringBuilder();
        for (char c : target.toCharArray())
            sb.append((char) ('a' + find(c - 'a')));
        return sb.toString();
    }

    private int find(int x) {
        return parent[x] == x ? x : (parent[x] = find(parent[x]));
    }

    private void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra < rb) parent[rb] = ra;
        else parent[ra] = rb;
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
    string smallestEquivalent(string first, string second, string target) {
        parent.resize(26);
        iota(parent.begin(), parent.end(), 0);
        for (size_t i = 0; i < first.size(); ++i)
            unite(first[i] - 'a', second[i] - 'a');

        string out;
        out.reserve(target.size());
        for (char c : target)
            out += char('a' + find(c - 'a'));
        return out;
    }

private:
    vector<int> parent;

    int find(int x) {
        return parent[x] == x ? x : parent[x] = find(parent[x]);
    }

    void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra < rb) parent[rb] = ra;
        else parent[ra] = rb;
    }
};
C
C
#include <string.h>
#include <stdlib.h>

static int parent[26];

static int find(int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]]; /* path halving */
        x = parent[x];
    }
    return x;
}

static void unite(int a, int b) {
    int ra = find(a), rb = find(b);
    if (ra == rb) return;
    /* Attach the larger root under the smaller so roots stay minimal. */
    if (ra < rb) parent[rb] = ra;
    else parent[ra] = rb;
}

/* Caller is responsible for free()-ing the returned buffer. */
char *smallestEquivalent(const char *first, const char *second, const char *target) {
    for (int i = 0; i < 26; ++i) parent[i] = i;
    for (int i = 0; first[i]; ++i)
        unite(first[i] - 'a', second[i] - 'a');

    int n = (int)strlen(target);
    char *out = (char *)malloc((size_t)n + 1);
    for (int i = 0; i < n; ++i)
        out[i] = (char)('a' + find(target[i] - 'a'));
    out[n] = '\0';
    return out;
}

#### Follow-up
questions
Why must the smaller letter always be chosen as the root, and how would you prove the result is truly lexicographically smallest?
How would this change if the alphabet were arbitrarily large (Unicode) rather than fixed at 26 letters?
Could you solve it with graph traversal (BFS/DFS over a 26-node graph) instead of Union-Find, and what are the tradeoffs?
What if some equivalences were later removed — how hard is dynamic disconnection compared to union-only?

