---
type: "practice"
difficulty: "Other"
---

# Two-Window Rearrangements to Reach a Palindrome

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prefix Sum](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Prefix_Sum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a 0-indexed string t of even length m, consisting only of lowercase English letters.

You are also given a list of queries. Each query is a 4-tuple [p, q, u, v] describing two index windows:

A left window t[p..q] that lies entirely in the first half: 0 <= p <= q < m / 2.
A right window t[u..v] that lies entirely in the second half: m / 2 <= u <= v < m.

For a single query you may, independently, do both of the following:

Reorder (permute) the characters inside the left window t[p..q] in any way.
Reorder (permute) the characters inside the right window t[u..v] in any way.

Characters outside these two windows stay exactly where they are. After performing the two reorderings, decide whether t can be made a palindrome (a string that reads the same forwards and backwards, i.e. t[k] == t[m-1-k] for every k).

Each query is answered independently: the reorderings from one query do not carry over to the next; every query starts from the original string.

Return a boolean array answer, where answer[i] is true if the i-th query's windows can be rearranged to make t a palindrome, and false otherwise.

Notes:

A window t[x..y] is the contiguous block of characters from index x to index y, both inclusive.
The left window is always confined to [0, m/2) and the right window to [m/2, m), so the two windows never overlap in the original string.

#### Cons
traints

2 <= m == t.length <= 10^5, and m is even.
1 <= queries.length <= 10^5
queries[i].length == 4
0 <= p <= q < m / 2
m / 2 <= u <= v < m
t consists of only lowercase English letters.

#### Example
s & test cases
TEXT
- Input: `t = "pqrqrp"`, queries = `[[0,2,3,5]]` -> Output: `[true]`
  Why: each window is a whole half; the left half `"pqr"` and the mirror of the right half share the same multiset, so both halves can be permuted into a matching palindrome (normal feasible case).
- Input: `t = "bcacab"`, queries = `[[1,2,3,3]]` -> Output: `[true]`
  Why: only a small left window `t[1..2]` and a single fixed right index `t[3]` are touched; permuting the left window alone repairs the mismatched pairs (edge: right window has length 1).
- Input: `t = "ttuuvv"`, queries = `[[0,2,3,5]]` -> Output: `[false]`
  Why: even with both halves fully free, the left half multiset `{t,t,u}` and the right half multiset `{u,v,v}` differ, so the pairs can never all match (impossible multiset case).
- Input: `t = "mnnokpnm"`, queries = `[[0,1,6,7]]` -> Output: `[false]`
  Why: the two windows only cover the outer pair; the characters left fixed in the middle already fail the palindrome pairing, so no allowed rearrangement helps (fixed-region mismatch).
- Input: `t = "sprspsrs"`, queries = `[[1,1,4,4],[1,3,6,7],[1,3,6,6]]` -> Output: `[false,true,true]`
  Why: multiple independent queries on one string give different answers depending on how much freedom the windows provide (mixed-result case).

#### Approach

Fold the string onto itself: let a be the first half and b the reverse of the second half, so t is a palindrome exactly when a[i] == b[i] for every position i in [0, m/2). Mirror the right window into this first-half coordinate system: original right window [u, v] becomes [m-1-v, m-1-u] on b. Now the problem is two free intervals — one that permutes a, one that permutes b — and everything outside them is fixed.

Precompute prefix character counts of a and of b, plus a prefix count of positions where a[i] != b[i], so any range's letter multiset or mismatch count is an O(26) subtraction. For each query, first require every position not covered by either window to already satisfy a[i] == b[i] (checked via the mismatch prefix). Then two cases: if the two free intervals are disjoint, each interval faces only fixed characters on the opposite side, so its own multiset must equal the opposite side's multiset over the same range. If they overlap, the union splits into an only-a-free part, an only-b-free part, and a both-free part; the fixed letters in the only-free parts must be suppliable from the corresponding pool, and after removing that demand the leftover a-pool and leftover b-pool must be identical multisets so the both-free region can be paired up. The core technique is prefix sums over a fixed alphabet combined with careful interval/multiset case analysis.

Complexity

Time: O((m + Q) * 26), where m is the string length and Q the number of queries — building the prefix count and mismatch arrays is O(m * 26), and each query does O(26) work using range look-ups. Space: O(m * 26) for the two prefix-count tables plus the mismatch prefix array. Justification: the alphabet is fixed at 26, so each per-query range count and comparison is effectively constant time.

Python
PYTHON
class Solution:
    def canMakePalindromeQueries(self, t: str, queries: list[list[int]]) -> list[bool]:
        n = len(t)
        h = n // 2
        a = t[:h]
        b = t[h:][::-1]                              # b[i] pairs with a[i]; goal is a == b
        ca = [[0] * 26 for _ in range(h + 1)]
        cb = [[0] * 26 for _ in range(h + 1)]
        ne = [0] * (h + 1)
        for i in range(h):
            ca[i + 1] = ca[i][:]; ca[i + 1][ord(a[i]) - 97] += 1
            cb[i + 1] = cb[i][:]; cb[i + 1][ord(b[i]) - 97] += 1
            ne[i + 1] = ne[i] + (1 if a[i] != b[i] else 0)

        def mism(l, r):
            return 0 if l > r else ne[r + 1] - ne[l]

        def rng(c, l, r):
            return [0] * 26 if l > r else [c[r + 1][k] - c[l][k] for k in range(26)]

        ans = []
        for p, qq, u, v in queries:
            l1, r1 = p, qq
            l2, r2 = n - 1 - v, n - 1 - u           # mirror the right window into first-half coords
            cA, cB = ca, cb
            if l1 > l2:
                l1, l2 = l2, l1
                r1, r2 = r2, r1
                cA, cB = cb, ca
            if r1 < l2:                              # disjoint
                if mism(0, l1 - 1) or mism(r1 + 1, l2 - 1) or mism(r2 + 1, h - 1):
                    ans.append(False); continue
                ok = rng(cA, l1, r1) == rng(cB, l1, r1) and rng(cA, l2, r2) == rng(cB, l2, r2)
            else:                                    # overlap
                hi = max(r1, r2)
                if mism(0, l1 - 1) or mism(hi + 1, h - 1):
                    ans.append(False); continue
                poolA = rng(cA, l1, r1)
                poolB = rng(cB, l2, r2)
                if r1 <= r2:
                    Da = rng(cB, l1, l2 - 1)
                    Db = rng(cA, r1 + 1, r2)
                else:
                    d1 = rng(cB, l1, l2 - 1)
                    d2 = rng(cB, r2 + 1, r1)
                    Da = [d1[k] + d2[k] for k in range(26)]
                    Db = [0] * 26
                ok = True
                for k in range(26):
                    if poolA[k] < Da[k] or poolB[k] < Db[k] or poolA[k] - Da[k] != poolB[k] - Db[k]:
                        ok = False
                        break
            ans.append(ok)
        return ans
JavaScript
JAVASCRIPT
function canMakePalindromeQueries(t, queries) {

#### cons
t n = t.length, h = n >> 1;

#### cons
t ca = Array.from({ length: h + 1 }, () => new Int32Array(26));

#### cons
t cb = Array.from({ length: h + 1 }, () => new Int32Array(26));

#### cons
t ne = new Int32Array(h + 1);
    for (let i = 0; i < h; i++) {
        ca[i + 1].set(ca[i]); ca[i + 1][t.charCodeAt(i) - 97]++;
        cb[i + 1].set(cb[i]); cb[i + 1][t.charCodeAt(n - 1 - i) - 97]++;   // b[i] pairs with a[i]
        ne[i + 1] = ne[i] + (t[i] !== t[n - 1 - i] ? 1 : 0);
    }

#### cons
t mism = (l, r) => (l > r ? 0 : ne[r + 1] - ne[l]);

#### cons
t rng = (c, l, r) => {

#### cons
t o = new Int32Array(26);
        if (l <= r) for (let k = 0; k < 26; k++) o[k] = c[r + 1][k] - c[l][k];
        return o;
    };

#### cons
t eq = (x, y) => { for (let k = 0; k < 26; k++) if (x[k] !== y[k]) return false; return true; };

#### cons
t ans = [];
    for (const q of queries) {
        let l1 = q[0], r1 = q[1];
        let l2 = n - 1 - q[3], r2 = n - 1 - q[2];       // mirror right window into first-half coords
        let cA = ca, cB = cb;
        if (l1 > l2) { [l1, l2] = [l2, l1]; [r1, r2] = [r2, r1]; [cA, cB] = [cB, cA]; }
        let ok;
        if (r1 < l2) {                                   // disjoint
            if (mism(0, l1 - 1) || mism(r1 + 1, l2 - 1) || mism(r2 + 1, h - 1)) { ans.push(false); continue; }
            ok = eq(rng(cA, l1, r1), rng(cB, l1, r1)) && eq(rng(cA, l2, r2), rng(cB, l2, r2));
        } else {                                         // overlap

#### cons
t hi = Math.max(r1, r2);
            if (mism(0, l1 - 1) || mism(hi + 1, h - 1)) { ans.push(false); continue; }

#### cons
t poolA = rng(cA, l1, r1), poolB = rng(cB, l2, r2);
            let Da, Db;
            if (r1 <= r2) { Da = rng(cB, l1, l2 - 1); Db = rng(cA, r1 + 1, r2); }
            else {

#### cons
t d1 = rng(cB, l1, l2 - 1), d2 = rng(cB, r2 + 1, r1);
                Da = new Int32Array(26); Db = new Int32Array(26);
                for (let k = 0; k < 26; k++) Da[k] = d1[k] + d2[k];
            }
            ok = true;
            for (let k = 0; k < 26; k++)
                if (poolA[k] < Da[k] || poolB[k] < Db[k] || poolA[k] - Da[k] !== poolB[k] - Db[k]) { ok = false; break; }
        }
        ans.push(ok);
    }
    return ans;
}
Java
JAVA
class Solution {
  public java.util.List<Boolean> canMakePalindromeQueries(String t, int[][] queries) {
    int n = t.length(), h = n / 2;
    char[] a = new char[h], b = new char[h];
    for (int i = 0; i < h; i++) { a[i] = t.charAt(i); b[i] = t.charAt(n - 1 - i); } // b[i] pairs a[i]
    int[][] ca = new int[h + 1][26], cb = new int[h + 1][26];
    int[] ne = new int[h + 1];
    for (int i = 0; i < h; i++) {
      System.arraycopy(ca[i], 0, ca[i + 1], 0, 26); ca[i + 1][a[i] - 'a']++;
      System.arraycopy(cb[i], 0, cb[i + 1], 0, 26); cb[i + 1][b[i] - 'a']++;
      ne[i + 1] = ne[i] + (a[i] != b[i] ? 1 : 0);
    }
    java.util.List<Boolean> ans = new java.util.ArrayList<>();
    for (int[] q : queries) {
      int l1 = q[0], r1 = q[1];
      int l2 = n - 1 - q[3], r2 = n - 1 - q[2];
      int[][] cA = ca, cB = cb;
      if (l1 > l2) { int tmp; tmp = l1; l1 = l2; l2 = tmp; tmp = r1; r1 = r2; r2 = tmp;
                     int[][] tt = cA; cA = cB; cB = tt; }
      boolean ok;
      if (r1 < l2) {
        if (mism(ne, 0, l1 - 1) > 0 || mism(ne, r1 + 1, l2 - 1) > 0 || mism(ne, r2 + 1, h - 1) > 0) { ans.add(false); continue; }
        ok = eq(rng(cA, l1, r1), rng(cB, l1, r1)) && eq(rng(cA, l2, r2), rng(cB, l2, r2));
      } else {
        int hi = Math.max(r1, r2);
        if (mism(ne, 0, l1 - 1) > 0 || mism(ne, hi + 1, h - 1) > 0) { ans.add(false); continue; }
        int[] poolA = rng(cA, l1, r1), poolB = rng(cB, l2, r2), Da, Db;
        if (r1 <= r2) { Da = rng(cB, l1, l2 - 1); Db = rng(cA, r1 + 1, r2); }
        else {
          int[] d1 = rng(cB, l1, l2 - 1), d2 = rng(cB, r2 + 1, r1);
          Da = new int[26]; for (int k = 0; k < 26; k++) Da[k] = d1[k] + d2[k];
          Db = new int[26];
        }
        ok = true;
        for (int k = 0; k < 26; k++)
          if (poolA[k] < Da[k] || poolB[k] < Db[k] || poolA[k] - Da[k] != poolB[k] - Db[k]) { ok = false; break; }
      }
      ans.add(ok);
    }
    return ans;
  }

  private int mism(int[] ne, int l, int r) { return l > r ? 0 : ne[r + 1] - ne[l]; }

  private int[] rng(int[][] c, int l, int r) {
    int[] o = new int[26];
    if (l <= r) for (int k = 0; k < 26; k++) o[k] = c[r + 1][k] - c[l][k];
    return o;
  }

  private boolean eq(int[] x, int[] y) {
    for (int k = 0; k < 26; k++) if (x[k] != y[k]) return false;
    return true;
  }
}
C++
CPP
class Solution {
 public:
  vector<bool> canMakePalindromeQueries(string t, vector<vector<int>>& queries) {
    int n = t.size(), h = n / 2;
    string a = t.substr(0, h);
    string b = t.substr(h);
    reverse(b.begin(), b.end());                 // b[i] pairs with a[i]; goal is a == b
    vector<array<int, 26>> ca(h + 1), cb(h + 1);
    ca[0].fill(0); cb[0].fill(0);
    vector<int> ne(h + 1, 0);
    for (int i = 0; i < h; ++i) {
      ca[i + 1] = ca[i]; ca[i + 1][a[i] - 'a']++;
      cb[i + 1] = cb[i]; cb[i + 1][b[i] - 'a']++;
      ne[i + 1] = ne[i] + (a[i] != b[i] ? 1 : 0);
    }
    auto mism = [&] (int l, int r) { return l > r ? 0 : ne[r + 1] - ne[l]; };
    auto rng = [&] (vector<array<int, 26>>& c, int l, int r) {
      array<int, 26> o{};
      if (l <= r) for (int k = 0; k < 26; ++k) o[k] = c[r + 1][k] - c[l][k];
      return o;
    };
    vector<bool> ans;
    ans.reserve(queries.size());
    for (auto& q : queries) {
      int l1 = q[0], r1 = q[1];
      int l2 = n - 1 - q[3], r2 = n - 1 - q[2];  // mirror right window into first-half coordinates
      auto* cA = &ca; auto* cB = &cb;
      if (l1 > l2) { swap(l1, l2); swap(r1, r2); swap(cA, cB); }
      bool ok;
      if (r1 < l2) {                             // disjoint
        if (mism(0, l1 - 1) || mism(r1 + 1, l2 - 1) || mism(r2 + 1, h - 1)) { ans.push_back(false); continue; }
        ok = (rng(*cA, l1, r1) == rng(*cB, l1, r1)) && (rng(*cA, l2, r2) == rng(*cB, l2, r2));
      } else {                                   // overlap
        int hi = max(r1, r2);
        if (mism(0, l1 - 1) || mism(hi + 1, h - 1)) { ans.push_back(false); continue; }
        array<int, 26> poolA = rng(*cA, l1, r1), poolB = rng(*cB, l2, r2), Da{}, Db{};
        if (r1 <= r2) { Da = rng(*cB, l1, l2 - 1); Db = rng(*cA, r1 + 1, r2); }
        else {
          array<int, 26> d1 = rng(*cB, l1, l2 - 1), d2 = rng(*cB, r2 + 1, r1);
          for (int k = 0; k < 26; ++k) Da[k] = d1[k] + d2[k];
        }
        ok = true;
        for (int k = 0; k < 26 && ok; ++k)
          if (poolA[k] < Da[k] || poolB[k] < Db[k] || poolA[k] - Da[k] != poolB[k] - Db[k]) ok = false;
      }
      ans.push_back(ok);
    }
    return ans;
  }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

static int mismC(int* ne, int l, int r) { return l > r ? 0 : ne[r + 1] - ne[l]; }

static void rngC(int* c, int l, int r, int* out) {
    for (int k = 0; k < 26; ++k) out[k] = (l > r) ? 0 : (c[(r + 1) * 26 + k] - c[l * 26 + k]);
}

bool* canMakePalindromeQueries(char* t, int** queries, int queriesSize,
                               int* queriesColSize, int* returnSize) {
    int n = (int)strlen(t), h = n / 2;
    int* ca = (int*)calloc((size_t)(h + 1) * 26, sizeof(int));   /* prefix counts of first half  */
    int* cb = (int*)calloc((size_t)(h + 1) * 26, sizeof(int));   /* prefix counts of reversed 2nd half */
    int* ne = (int*)calloc((size_t)(h + 1), sizeof(int));        /* prefix count of mismatched pairs */
    for (int i = 0; i < h; ++i) {
        memcpy(ca + (i + 1) * 26, ca + i * 26, 26 * sizeof(int));
        memcpy(cb + (i + 1) * 26, cb + i * 26, 26 * sizeof(int));
        ca[(i + 1) * 26 + (t[i] - 'a')]++;
        cb[(i + 1) * 26 + (t[n - 1 - i] - 'a')]++;               /* b[i] pairs with a[i] */
        ne[i + 1] = ne[i] + ((t[i] != t[n - 1 - i]) ? 1 : 0);
    }
    bool* ans = (bool*)malloc(sizeof(bool) * queriesSize);
    *returnSize = queriesSize;
    int pa[26], pb[26], qa[26], qb[26], da[26], db[26], d1[26], d2[26];
    for (int i = 0; i < queriesSize; ++i) {
        int l1 = queries[i][0], r1 = queries[i][1];
        int l2 = n - 1 - queries[i][3], r2 = n - 1 - queries[i][2];   /* mirror right window into first-half coords */
        int *cA = ca, *cB = cb;
        if (l1 > l2) { int tmp; tmp = l1; l1 = l2; l2 = tmp; tmp = r1; r1 = r2; r2 = tmp;
                       int* tt = cA; cA = cB; cB = tt; }
        bool ok;
        if (r1 < l2) {                                                /* windows disjoint in mirrored coords */
            if (mismC(ne, 0, l1 - 1) || mismC(ne, r1 + 1, l2 - 1) || mismC(ne, r2 + 1, h - 1)) {
                ans[i] = false; continue;
            }
            rngC(cA, l1, r1, pa); rngC(cB, l1, r1, pb);
            rngC(cA, l2, r2, qa); rngC(cB, l2, r2, qb);
            ok = true;
            for (int k = 0; k < 26; ++k) if (pa[k] != pb[k] || qa[k] != qb[k]) { ok = false; break; }
        } else {                                                     /* windows overlap in mirrored coords */
            int hi = r1 > r2 ? r1 : r2;
            if (mismC(ne, 0, l1 - 1) || mismC(ne, hi + 1, h - 1)) { ans[i] = false; continue; }
            rngC(cA, l1, r1, pa);                                    /* pool of free-A chars */
            rngC(cB, l2, r2, pb);                                    /* pool of free-B chars */
            if (r1 <= r2) {
                rngC(cB, l1, l2 - 1, da);                            /* B fixed in the only-A part */
                rngC(cA, r1 + 1, r2, db);                            /* A fixed in the only-B part */
            } else {
                rngC(cB, l1, l2 - 1, d1);
                rngC(cB, r2 + 1, r1, d2);
                for (int k = 0; k < 26; ++k) { da[k] = d1[k] + d2[k]; db[k] = 0; }
            }
            ok = true;
            for (int k = 0; k < 26; ++k)
                if (pa[k] < da[k] || pb[k] < db[k] || pa[k] - da[k] != pb[k] - db[k]) { ok = false; break; }
        }
        ans[i] = ok;
    }
    free(ca); free(cb); free(ne);
    return ans;
}

#### Follow-up
questions
How does the analysis change if the two windows were allowed to lie anywhere (not confined to their own half), so they could overlap the mirror line?
Instead of yes/no, count the number of distinct palindromic strings reachable for a query.
What if characters could be relabeled (an equivalence between certain letters), not just permuted in place?
If the string can be updated between queries (a character changes), how would you maintain the prefix structures efficiently?

