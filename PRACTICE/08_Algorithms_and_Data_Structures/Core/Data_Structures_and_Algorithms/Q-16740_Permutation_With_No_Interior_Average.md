---
type: "practice"
difficulty: "Other"
---

# Permutation With No Interior Average

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Divide and Conquer](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Divide_and_Conquer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Given a positive integer m, build any ordering (a permutation) of the integers 1, 2, ..., m that satisfies the following balance rule.

For every pair of positions p and r with p < r, there must be no position q lying strictly between them (p < q < r) whose value equals the arithmetic mean of the two endpoints. Formally, there is no triple of positions p < q < r for which

CODE
2 * value[q] == value[p] + value[r]

Equivalently: no element that sits between two others may equal the average of those two flanking elements. It is guaranteed that at least one such arrangement always exists for the given m; return any one of them.

Input: a single integer m.

Output: an array of length m that contains each of the integers 1..m exactly once and obeys the rule above. Any valid arrangement is accepted.

#### Cons
traints:

1 <= m <= 1000.
The returned array must be a permutation of 1..m (each value appears exactly once).

Note: the forbidden pattern only concerns an element that is positioned between the two whose average it would be; elements outside that span impose no restriction.

#### Example
s & test cases
TEXT
Input: m = 1 -> Output: [1]
Why: a single element has no interior position, so the rule holds trivially.

Input: m = 2 -> Output: [1, 2]
Why: only two elements, so no position can lie strictly between another pair; any order works.

Input: m = 3 -> Output: [1, 3, 2]
Why: the only interior triple is (1, 3, 2); 2*3 = 6 but 1+2 = 3, so 3 is not the average of its neighbors.

Input: m = 4 -> Output: [1, 3, 2, 4]
Why: every span is checked, e.g. positions (1,2,4) give 2*2 = 4 vs 1+4 = 5 (no), and no interior element equals a flanking average.

Input: m = 5 -> Output: [1, 5, 3, 2, 4]
Why: a valid average-free permutation of 1..5; for instance the span from value 1 to value 4 has interiors 5,3,2, none equal to (1+4)/2 = 2.5, and all other spans check out.

Input: m = 6 -> Output: [1, 5, 3, 2, 6, 4]
Why: another valid arrangement of 1..6; the odds are grouped ahead of the evens so any cross-parity span has an odd sum that can never equal an even 2*value[q].

#### Approach

#### Cons
tructive divide-and-conquer built on a parity invariant. Key insight: if a sequence over some value set is already average-free, then mapping every element x -> 2x-1 (making them all odd) and mapping every element x -> 2x (making them all even), and concatenating [odds][evens], produces an average-free sequence over a value set of twice the size. Why: any offending triple needs 2*mid == left + right. Within the odd block or within the even block the property holds by induction, because an affine map x -> 2x-1 or x -> 2x preserves which element is the average of which. Across the two blocks, one endpoint is odd and the other is even, so left + right is odd while 2*mid is even — impossible. So we start from [1] and repeatedly apply the doubling transform, discarding any value that exceeds n, until the sequence covers 1..n exactly. Because each transform only increases values, discarding early keeps every intermediate array within size n.

Complexity

Time: O(n) — there are O(log n) doubling rounds, and the current array roughly doubles each round (1 + 2 + 4 + ... + n), so the summed work is bounded by ~2n. Space: O(n) for the result plus one scratch buffer of the same size.

Python
PYTHON
from typing import List

class Solution:
    def elegantPermutation(self, n: int) -> List[int]:
        res = [1]
        while len(res) < n:
            res = [2 * x - 1 for x in res if 2 * x - 1 <= n] + \
                  [2 * x for x in res if 2 * x <= n]
        return res
JavaScript
JAVASCRIPT
function elegantPermutation(n) {
    let res = [1];
    while (res.length < n) {

#### cons
t next = [];
        for (const x of res) if (2 * x - 1 <= n) next.push(2 * x - 1); // odds
        for (const x of res) if (2 * x     <= n) next.push(2 * x);     // evens
        res = next;
    }
    return res;
}
Java
JAVA
import java.util.ArrayList;
import java.util.List;

class Solution {
    public int[] elegantPermutation(int n) {
        List<Integer> res = new ArrayList<>();
        res.add(1);
        while (res.size() < n) {
            List<Integer> next = new ArrayList<>();
            for (int x : res) if (2 * x - 1 <= n) next.add(2 * x - 1); // odds
            for (int x : res) if (2 * x     <= n) next.add(2 * x);     // evens
            res = next;
        }
        int[] out = new int[n];
        for (int i = 0; i < n; i++) out[i] = res.get(i);
        return out;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> elegantPermutation(int n) {
        vector<int> res = {1};
        while ((int)res.size() < n) {
            vector<int> next;
            next.reserve(res.size() * 2);
            for (int x : res) if (2 * x - 1 <= n) next.push_back(2 * x - 1); // odds
            for (int x : res) if (2 * x     <= n) next.push_back(2 * x);     // evens
            res = move(next);
        }
        return res;
    }
};
C
C
#include <stdlib.h>

/* Returns a heap-allocated average-free permutation of 1..n.
   Caller frees the result; *returnSize is set to n. */
int* elegantPermutation(int n, int* returnSize) {
    int* cur = (int*)malloc(sizeof(int) * n);
    int* nxt = (int*)malloc(sizeof(int) * n);
    int len = 1;
    cur[0] = 1;
    while (len < n) {
        int m = 0;
        for (int i = 0; i < len; ++i) {      /* map x -> 2x-1 (all odds) */
            int v = 2 * cur[i] - 1;
            if (v <= n) nxt[m++] = v;
        }
        for (int i = 0; i < len; ++i) {      /* map x -> 2x   (all evens) */
            int v = 2 * cur[i];
            if (v <= n) nxt[m++] = v;
        }
        int* tmp = cur; cur = nxt; nxt = tmp;
        len = m;
    }
    free(nxt);
    *returnSize = n;
    return cur;
}

#### Follow-up
questions
Prove rigorously that the odds-then-evens doubling transform preserves the average-free property (the parity argument across blocks plus the affine-map argument within blocks).
How does the construction change if the required value range is [0, n-1] instead of [1, n]?
Can you give a memoized/top-down recursion that builds the permutation for a prefix length and reuses subresults?
Generalize the forbidden pattern to disallow any 3-term arithmetic progression by index order — does the same construction still work?

