---
type: "practice"
difficulty: "Other"
---

# Detect Inconsistent Ratio Equations

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Union Find](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Union_Find.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a collection of ratio statements over string-labeled quantities. There are n statements. The i-th statement is given as a pair of labels pairs[i] = [left, right] together with a positive real number ratios[i], and it asserts that

CODE
value(left) / value(right) = ratios[i]

Every label is a short lowercase string. The same label may appear in many statements, and a statement may even relate a label to itself. Nothing in the input tells you the actual value of any label — only these pairwise ratios.

Your job is to decide whether the whole collection can be satisfied at once. Concretely, is there an assignment of a positive real number to every label such that all of the ratio statements hold simultaneously?

Return true if the statements are inconsistent — that is, they force at least one contradiction and no valid assignment exists.
Return false if the statements are consistent — some assignment satisfies every statement.

When you compare two ratios that the equations imply, treat them as equal if their absolute difference is strictly less than 1e-5. All intermediate values fit comfortably in IEEE double precision, so you do not need any special high-precision arithmetic.

#### Cons
traints

1 <= n <= 100
pairs.length == ratios.length == n
Each pairs[i] has exactly two labels.
1 <= left.length, right.length <= 5, each label consists of lowercase English letters.
0.0 < ratios[i] <= 10.0, and each ratios[i] has at most two decimal places.

#### Example
s & test cases
TEXT
Input: pairs = [["x","y"],["y","z"],["x","z"]], ratios = [4, 0.25, 1] -> Output: false
Why: x/y = 4 and y/z = 0.25 imply x/z = 4 * 0.25 = 1, which matches the third statement exactly, so the system is consistent (e.g. x=1, y=0.25, z=1).

Input: pairs = [["p","q"],["p","r"],["r","q"]], ratios = [2, 5, 0.5] -> Output: true
Why: p/q = 2 and p/r = 5 imply r/q = (p/5)/(p/2) = 2/5 = 0.4, but the third statement claims r/q = 0.5, a contradiction.

Input: pairs = [["a","b"]], ratios = [3] -> Output: false
Why: a single statement can never contradict itself, so a consistent assignment always exists (minimum input size).

Input: pairs = [["m","n"],["m","n"]], ratios = [2, 2] -> Output: false
Why: a duplicated statement that agrees with itself adds no new constraint, so the system stays consistent.

Input: pairs = [["m","n"],["m","n"]], ratios = [2, 3] -> Output: true
Why: the same pair is asserted to have two different ratios (2 and 3), which cannot both hold — a direct contradiction.

Input: pairs = [["g","g"]], ratios = [2] -> Output: true
Why: a label divided by itself must equal 1, but the statement claims g/g = 2, so it is inconsistent (self-loop edge case).

#### Approach

Model each label as a node in a graph and each equation a / b = r as a weighted relation between a and b. Use a weighted (relative-size) union-find: for every node x store weight[x] = value(x) / value(parent[x]), so after path compression weight[x] = value(x) / value(root). Process the equations one by one. To incorporate a / b = r, find the roots of a and b. If they already share a root, the tree already fixes their ratio as weight[a] / weight[b]; if that differs from r by at least 1e-5, the new equation contradicts the earlier ones and the answer is true. Otherwise merge the two components, setting the new edge weight so the asserted ratio holds (weight[root_a] = r * weight[b] / weight[a]). If no equation ever conflicts with the accumulated ratios, the system is consistent. This is the classic weighted-union-find / DFS-on-a-division-graph pattern; a plain DFS/BFS that builds the graph and recomputes each ratio would also work but is asymptotically slower per query.

Complexity

Time: O(n * α(v)) where n is the number of equations and v (<= 2n) the number of distinct labels; weighted union-find with path compression makes each find effectively near-constant (α is the inverse Ackermann function). Space: O(v) for the parent/weight tables and the label-to-id map.

Python
PYTHON
from typing import List

def has_contradiction(pairs: List[List[str]], ratios: List[float]) -> bool:
    parent = {}
    weight = {}  # weight[x] = value(x) / value(parent[x])

    def add(x: str) -> None:
        if x not in parent:
            parent[x] = x
            weight[x] = 1.0

    def find(x: str) -> str:
        if parent[x] != x:
            orig = parent[x]
            root = find(orig)
            weight[x] *= weight[orig]
            parent[x] = root
        return parent[x]

    for (a, b), r in zip(pairs, ratios):
        add(a)
        add(b)
        ra, rb = find(a), find(b)
        if ra == rb:
            # value(a)/value(b) implied by the tree = weight[a]/weight[b]
            if abs(weight[a] / weight[b] - r) >= 1e-5:
                return True
        else:
            # attach ra under rb so that value(a)/value(b) == r
            parent[ra] = rb
            weight[ra] = r * weight[b] / weight[a]
    return False
JavaScript
JAVASCRIPT
function hasContradiction(pairs, ratios) {

#### cons
t id = new Map();

#### cons
t par = [];

#### cons
t wt = [];   // wt[x] = value(x) / value(par[x])

#### cons
t getId = (s) => {
        if (id.has(s)) return id.get(s);

#### cons
t k = par.length;
        id.set(s, k);
        par.push(k);
        wt.push(1.0);
        return k;
    };

#### cons
t find = (x) => {
        if (par[x] === x) return x;

#### cons
t op = par[x];

#### cons
t root = find(op);
        wt[x] *= wt[op];
        par[x] = root;
        return root;
    };

    for (let i = 0; i < pairs.length; i++) {

#### cons
t a = getId(pairs[i][0]);

#### cons
t b = getId(pairs[i][1]);

#### cons
t r = ratios[i];

#### cons
t ra = find(a), rb = find(b);
        if (ra === rb) {
            if (Math.abs(wt[a] / wt[b] - r) >= 1e-5) return true;
        } else {
            par[ra] = rb;
            wt[ra] = r * wt[b] / wt[a];
        }
    }
    return false;
}
Java
JAVA
import java.util.HashMap;
import java.util.Map;

class Solution {
    private int[] par;
    private double[] wt;   // wt[x] = value(x) / value(par[x])

    private int find(int x) {
        if (par[x] == x) return x;
        int op = par[x];
        int root = find(op);
        wt[x] *= wt[op];
        par[x] = root;
        return root;
    }

    public boolean hasContradiction(String[][] pairs, double[] ratios) {
        int n = pairs.length;
        par = new int[2 * n];
        wt  = new double[2 * n];
        Map<String,Integer> id = new HashMap<>();
        int cnt = 0;

        for (String[] p : pairs)
            for (String s : p)
                if (!id.containsKey(s)) {
                    id.put(s, cnt);
                    par[cnt] = cnt;
                    wt[cnt]  = 1.0;
                    cnt++;
                }

        for (int i = 0; i < n; i++) {
            int a = id.get(pairs[i][0]);
            int b = id.get(pairs[i][1]);
            double r = ratios[i];
            int ra = find(a), rb = find(b);
            if (ra == rb) {
                if (Math.abs(wt[a] / wt[b] - r) >= 1e-5) return true;
            } else {
                par[ra] = rb;
                wt[ra]  = r * wt[b] / wt[a];
            }
        }
        return false;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <unordered_map>
#include <cmath>
using namespace std;

class Solution {
    vector<int>    par;
    vector<double> wt;   // wt[x] = value(x) / value(par[x])

    int find(int x) {
        if (par[x] == x) return x;
        int op = par[x];
        int root = find(op);
        wt[x] *= wt[op];
        par[x] = root;
        return root;
    }

public:
    bool hasContradiction(vector<vector<string>>& pairs, vector<double>& ratios) {
        unordered_map<string,int> id;
        auto getId = [&] (const string& s) {
            auto it = id.find(s);
            if (it != id.end()) return it->second;
            int k = (int)id.size();
            id[s] = k;
            par.push_back(k);
            wt.push_back(1.0);
            return k;
        };

        for (size_t i = 0; i < pairs.size(); i++) {
            int a = getId(pairs[i][0]);
            int b = getId(pairs[i][1]);
            double r = ratios[i];
            int ra = find(a), rb = find(b);
            if (ra == rb) {
                if (fabs(wt[a] / wt[b] - r) >= 1e-5) return true;
            } else {
                par[ra] = rb;
                wt[ra]  = r * wt[b] / wt[a];
            }
        }
        return false;
    }
};
C
C
#include <stdbool.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

#define MAXN 256

static int   par[MAXN];
static double wt[MAXN];   /* wt[x] = value(x) / value(par[x]) */

static int find(int x) {
    if (par[x] == x) return x;
    int op = par[x];
    int root = find(op);
    wt[x] *= wt[op];      /* value(x)/value(op) * value(op)/value(root) */
    par[x] = root;
    return root;
}

static int getId(char** labels, int* cnt, const char* s) {
    for (int i = 0; i < *cnt; i++)
        if (strcmp(labels[i], s) == 0) return i;
    int id = (*cnt)++;
    labels[id] = strdup(s);
    par[id] = id;
    wt[id]  = 1.0;
    return id;
}

/* pairs: array of `n` 2-element string arrays; ratios: parallel array of n doubles. */
bool hasContradiction(char*** pairs, int n, double* ratios) {
    char* labels[MAXN];
    int cnt = 0;
    bool contradiction = false;

    for (int i = 0; i < n; i++) {
        int a = getId(labels, &cnt, pairs[i][0]);
        int b = getId(labels, &cnt, pairs[i][1]);
        double r = ratios[i];
        int ra = find(a), rb = find(b);
        if (ra == rb) {
            /* value(a)/value(b) implied by the tree = wt[a]/wt[b] */
            if (fabs(wt[a] / wt[b] - r) >= 1e-5) { contradiction = true; break; }
        } else {
            /* attach ra under rb so value(a)/value(b) == r holds */
            par[ra] = rb;
            wt[ra]  = r * wt[b] / wt[a];
        }
    }

    for (int i = 0; i < cnt; i++) free(labels[i]);
    return contradiction;
}

#### Follow-up
questions
Instead of just detecting a contradiction, return the index of the first equation that conflicts, or a full consistent assignment of values.
Support ad-hoc division queries after all equations are processed: given x and y, return value(x)/value(y) or report that it is unknown.
Handle the equations as a stream, answering "consistent so far?" after each new equation is added.
Discuss why an absolute tolerance of 1e-5 is used and where floating-point error could accumulate on a long chain of ratios.

