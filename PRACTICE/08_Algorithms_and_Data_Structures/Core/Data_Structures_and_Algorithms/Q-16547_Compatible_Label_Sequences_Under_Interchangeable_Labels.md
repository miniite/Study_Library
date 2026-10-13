---
type: "practice"
difficulty: "Other"
---

# Compatible Label Sequences Under Interchangeable Labels

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given two sequences of text labels, first and second, together with a list swaps, where each entry swaps[i] = [p, q] declares that label p and label q are directly interchangeable.

Interchangeability is:

reflexive — every label is interchangeable with itself, and
transitive — if p is interchangeable with q and q is interchangeable with r, then p is interchangeable with r.

(It is naturally symmetric as well: if p swaps with q, then q swaps with p.)

Two sequences are called compatible when both of the following hold:

they contain the same number of labels, and
for every position i, first[i] and second[i] are interchangeable — either byte-for-byte identical, or connected directly or transitively through the swap list.

Return true if first and second are compatible, and false otherwise. Note that a label which never appears in any swap is interchangeable only with itself.

Input

first, second: arrays of label strings.
swaps: an array of [p, q] string pairs.

Output

A boolean: true if first and second are compatible, else false.

#### Cons
traints

1 <= first.length, second.length <= 1000
1 <= length of any label <= 20
Labels consist of upper-case and lower-case English letters.
0 <= swaps.length <= 2000
swaps[i].length == 2, and each of its two labels has length 1..20 of English letters.

#### Example
s & test cases
TEXT
Input: first = ["swift","auto","crimson"], second = ["quick","car","crimson"], swaps = [["swift","quick"],["auto","car"]] -> Output: true
Why: "crimson" equals itself; "swift"~"quick" and "auto"~"car" are direct swaps, so every position matches.

Input: first = ["alpha"], second = ["gamma"], swaps = [["alpha","beta"],["beta","gamma"]] -> Output: true
Why: transitivity — the chain alpha-beta-gamma links the two labels even with no direct swap.

Input: first = ["one","two"], second = ["one"], swaps = [] -> Output: false
Why: sequences of different lengths can never be compatible.

Input: first = ["dog"], second = ["cat"], swaps = [["dog","puppy"]] -> Output: false
Why: "cat" appears in no swap, so it is interchangeable only with itself, and "dog" is not "cat".

Input: first = ["north"], second = ["north"], swaps = [] -> Output: true
Why: identical labels are always compatible even with an empty swap list (reflexivity).

Input: first = ["hot","fresh"], second = ["warm","fresh"], swaps = [["hot","warm"],["warm","tepid"]] -> Output: true
Why: "hot"~"warm" directly, and "fresh" matches itself although it never appears in any swap.

#### Approach

Model interchangeability as connectivity in a disjoint-set union (union-find) structure. Give each distinct label that appears in the swap list an integer id, then union the two labels of every swap so that each connected group of mutually interchangeable labels ends up sharing one representative root. To decide compatibility, first reject sequences of unequal length; then, for each position, compare the two labels: if they are identical they are trivially interchangeable and are skipped; otherwise both must appear in the swap graph and must share the same union-find root. Path compression (plus attaching one root under the other) keeps each find near-constant. The underlying pattern is disjoint-set union for dynamic connectivity, i.e. computing transitive-closure equivalence classes without materializing the closure.

Complexity

Time: O((P + L) * alpha(V)) where P = number of swaps, L = sequence length, V = distinct labels, and alpha is the inverse-Ackermann function of union-find with path compression — effectively O(P + L) (the C variant is O((P + L) log V) because it resolves labels via binary search instead of a hash map). Space: O(V) for the disjoint-set structure and the label-to-id map.

Python
PYTHON
from typing import List

def are_label_sequences_compatible(first: List[str], second: List[str],
                                   swaps: List[List[str]]) -> bool:
    if len(first) != len(second):
        return False

    parent = {}

    def add(x: str) -> None:
        if x not in parent:
            parent[x] = x

    def find(x: str) -> str:
        root = x
        while parent[root] != root:
            root = parent[root]
        while parent[x] != root:          # path compression
            parent[x], x = root, parent[x]
        return root

    for a, b in swaps:
        add(a)
        add(b)
        ra, rb = find(a), find(b)
        if ra != rb:
            parent[ra] = rb

    for w1, w2 in zip(first, second):
        if w1 == w2:
            continue
        if w1 not in parent or w2 not in parent or find(w1) != find(w2):
            return False
    return True
JavaScript
JAVASCRIPT
function areLabelSequencesCompatible(first, second, swaps) {
    if (first.length !== second.length) return false;

#### cons
t id = new Map();

#### cons
t add = (s) => { if (!id.has(s)) id.set(s, id.size); };
    for (const [a, b] of swaps) { add(a); add(b); }

#### cons
t parent = Array.from({ length: id.size }, (_, i) => i);

#### cons
t find = (x) => {
        while (parent[x] !== x) {
            parent[x] = parent[parent[x]];   // path compression
            x = parent[x];
        }
        return x;
    };

#### cons
t union = (a, b) => {

#### cons
t ra = find(a), rb = find(b);
        if (ra !== rb) parent[ra] = rb;
    };

    for (const [a, b] of swaps) union(id.get(a), id.get(b));

    for (let i = 0; i < first.length; i++) {
        if (first[i] === second[i]) continue;
        if (!id.has(first[i]) || !id.has(second[i])) return false;
        if (find(id.get(first[i])) !== find(id.get(second[i]))) return false;
    }
    return true;
}
Java
JAVA
import java.util.*;

class Solution {
    private int[] parent;

    public boolean areLabelSequencesCompatible(String[] first, String[] second,
                                               List<List<String>> swaps) {
        if (first.length != second.length) return false;

        Map<String, Integer> id = new HashMap<>();
        for (List<String> s : swaps) {
            id.putIfAbsent(s.get(0), id.size());
            id.putIfAbsent(s.get(1), id.size());
        }
        parent = new int[id.size()];
        for (int i = 0; i < parent.length; i++) parent[i] = i;

        for (List<String> s : swaps)
            union(id.get(s.get(0)), id.get(s.get(1)));

        for (int i = 0; i < first.length; i++) {
            if (first[i].equals(second[i])) continue;
            Integer a = id.get(first[i]);
            Integer b = id.get(second[i]);
            if (a == null || b == null || find(a) != find(b))
                return false;
        }
        return true;
    }

    private int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    private void union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra != rb) parent[ra] = rb;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <unordered_map>
using namespace std;

class Solution {
public:
    bool areLabelSequencesCompatible(vector<string>& first, vector<string>& second,
                                     vector<vector<string>>& swaps) {
        if (first.size() != second.size())
            return false;

        unordered_map<string, int> id;
        for (const auto& s : swaps) {
            addNode(s[0], id);
            addNode(s[1], id);
        }
        parent.resize(id.size());
        for (int i = 0; i < (int)parent.size(); ++i) parent[i] = i;

        for (const auto& s : swaps)
            unite(id[s[0]], id[s[1]]);

        for (size_t i = 0; i < first.size(); ++i) {
            if (first[i] == second[i]) continue;
            auto it1 = id.find(first[i]);
            auto it2 = id.find(second[i]);
            if (it1 == id.end() || it2 == id.end() ||
                find(it1->second) != find(it2->second))
                return false;
        }
        return true;
    }

private:
    vector<int> parent;

    void addNode(const string& s, unordered_map<string, int>& id) {
        if (!id.count(s)) {
            int nxt = id.size();
            id[s] = nxt;
        }
    }
    int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
    void unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra != rb) parent[ra] = rb;
    }
};
C
C
#include <stdbool.h>
#include <string.h>
#include <stdlib.h>

static int cmpStr(const void* a, const void* b) {

#### cons
t char* sa = *(const char* const*)a;

#### cons
t char* sb = *(const char* const*)b;
    return strcmp(sa, sb);
}

static int findRoot(int* parent, int x) {
    while (parent[x] != x) {
        parent[x] = parent[parent[x]];   /* path compression (halving) */
        x = parent[x];
    }
    return x;
}

/* Binary-search key in a sorted unique array; -1 if absent. */
static int lookup(char** uniq, int n, const char* key) {
    int lo = 0, hi = n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int c = strcmp(uniq[mid], key);
        if (c == 0) return mid;
        if (c < 0) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

bool areLabelSequencesCompatible(char** first, int firstSize,
                                 char** second, int secondSize,
                                 char*** swaps, int swapsSize, int* swapsColSize) {
    if (firstSize != secondSize) return false;

    /* Collect all labels that appear in swaps, then sort + dedup. */
    int total = swapsSize * 2;
    char** all = (char**)malloc((total > 0 ? total : 1) * sizeof(char*));
    for (int i = 0; i < swapsSize; i++) {
        all[2 * i]     = swaps[i][0];
        all[2 * i + 1] = swaps[i][1];
    }
    qsort(all, total, sizeof(char*), cmpStr);

    char** uniq = (char**)malloc((total > 0 ? total : 1) * sizeof(char*));
    int n = 0;
    for (int i = 0; i < total; i++)
        if (i == 0 || strcmp(all[i], all[i - 1]) != 0)
            uniq[n++] = all[i];

    int* parent = (int*)malloc((n > 0 ? n : 1) * sizeof(int));
    for (int i = 0; i < n; i++) parent[i] = i;

    for (int i = 0; i < swapsSize; i++) {
        int a = lookup(uniq, n, swaps[i][0]);
        int b = lookup(uniq, n, swaps[i][1]);
        int ra = findRoot(parent, a), rb = findRoot(parent, b);
        if (ra != rb) parent[ra] = rb;
    }

    bool ans = true;
    for (int i = 0; i < firstSize && ans; i++) {
        if (strcmp(first[i], second[i]) == 0) continue;
        int a = lookup(uniq, n, first[i]);
        int b = lookup(uniq, n, second[i]);
        if (a < 0 || b < 0 || findRoot(parent, a) != findRoot(parent, b))
            ans = false;
    }

    free(all);
    free(uniq);
    free(parent);
    return ans;
}

#### Follow-up
questions
If the swap list keeps growing (new interchangeable pairs added between queries), how does union-find let you answer many compatibility queries efficiently over time?
How does this differ from the simpler variant where interchangeability is NOT transitive, and which data structure fits that version best?
Could you answer each position with BFS/DFS over an adjacency list instead, and what is the trade-off versus union-find when there are many queries?
How would you additionally report which positions fail, or the minimum number of edits needed to make the two sequences compatible?

