---
type: "practice"
difficulty: "Other"
---

# Guaranteeing a Win in the Adjacent Down-Toggle Game

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Game Theory](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Game_Theory.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Two players share a strip of toggle switches in a row, described by a string strip of 'U' (a switch pushed up) and 'D' (a switch pushed down).

The players alternate turns, and you move first. On a turn, a player must choose a position where two adjacent switches are both up (the substring "UU") and push both of them down, turning that "UU" into "DD". A player who, on their turn, has no adjacent up-pair to flip loses (the other player wins).

Assuming both players play optimally, return true if you (the first player) can guarantee a win, and false otherwise.

Input: a string strip containing only 'U' and 'D'.

Output: a boolean — true if the first player has a forced win, false otherwise.

#### Cons
traints:

1 <= strip.length <= 60
Each character of strip is either 'U' or 'D'.

Follow up: state the runtime complexity of your method.

#### Example
s & test cases
TEXT
Input: strip = "UUUU" -> Output: true
Why: the first player flips the middle pair (indices 1-2) to reach "UDDU", which has no adjacent up-pair, so the opponent cannot move and loses.

Input: strip = "UU" -> Output: true
Why: the only move turns "UU" into "DD"; the opponent then has no move, so the first player wins immediately.

Input: strip = "U" -> Output: false
Why: there is no adjacent up-pair at all, so the first player cannot move on turn one and loses.

Input: strip = "UDU" -> Output: false
Why: the two up-switches are separated by a down-switch, so no move exists and the first player loses.

Input: strip = "UUDUU" -> Output: false
Why: two independent up-pairs of equal size; whatever pair the first player flips, the opponent flips the other and leaves no move — a mirroring loss.

Input: strip = "UUUUU" -> Output: false
Why: a single run of five up-switches is a losing position (its game value is zero); every first move hands the opponent a winning reply.

#### Approach

This is an impartial combinatorial game, so apply the Sprague-Grundy theorem. Only maximal runs of up-switches matter, and they are mutually independent: a move inside one run of length k flips an interior "UU" to "DD", splitting that run into two smaller runs of lengths a and k-2-a (with a >= 0). Compute each run's Grundy value bottom-up: grundy[k] = mex over a of (grundy[a] XOR grundy[k-2-a]), where mex is the smallest non-negative integer missing from the reachable set, and grundy[0] = grundy[1] = 0 (no move). The whole position's value is the XOR (Nim-sum) of the Grundy values of all its runs; under normal play the first player has a forced win exactly when that XOR is nonzero. A simpler but exponential alternative is memoized minimax directly over the string states — correct, but far slower.

Complexity

Let n be the strip length and L the longest run of up-switches (L <= n). Time: O(n + L^2) = O(n^2) worst case — scanning for runs is O(n), and computing each Grundy value grundy[k] costs O(k), so building the whole table is O(L^2). Space: O(L) for the Grundy table (plus a reusable O(L) "seen" buffer). This is polynomial, versus the exponential O(n * 2^n) blow-up of a naive minimax over raw string states.

Python
PYTHON
from typing import List

class Solution:
    def canWin(self, strip: str) -> bool:
        # Lengths of each maximal run of up-switches.
        runs: List[int] = [len(seg) for seg in strip.split('D') if seg]
        if not runs:
            return False

        max_len = max(runs)
        grundy = [0] * (max_len + 1)
        for k in range(2, max_len + 1):
            reachable = set()
            for a in range(0, k - 1):          # a from 0..k-2
                reachable.add(grundy[a] ^ grundy[k - 2 - a])
            mex = 0
            while mex in reachable:
                mex += 1
            grundy[k] = mex

        total = 0
        for r in runs:
            total ^= grundy[r]
        return total != 0
JavaScript
JAVASCRIPT
function canWin(strip) {

#### cons
t runs = [];
    let cur = 0, maxLen = 0;
    for (const c of strip) {
        if (c === 'U') { cur++; if (cur > maxLen) maxLen = cur; }
        else { if (cur > 0) runs.push(cur); cur = 0; }
    }
    if (cur > 0) runs.push(cur);
    if (maxLen < 2) return false;

#### cons
t grundy = new Array(maxLen + 1).fill(0);
    for (let k = 2; k <= maxLen; k++) {

#### cons
t seen = new Array(2 * maxLen + 4).fill(false);
        for (let a = 0; a <= k - 2; a++) {
            seen[grundy[a] ^ grundy[k - 2 - a]] = true;
        }
        let mex = 0;
        while (seen[mex]) mex++;
        grundy[k] = mex;
    }

    let total = 0;
    for (const r of runs) total ^= grundy[r];
    return total !== 0;
}
Java
JAVA
import java.util.*;

class Solution {
    public boolean canWin(String strip) {
        List<Integer> runs = new ArrayList<>();
        int cur = 0, maxLen = 0;
        for (int i = 0; i < strip.length(); i++) {
            if (strip.charAt(i) == 'U') { cur++; maxLen = Math.max(maxLen, cur); }
            else { if (cur > 0) runs.add(cur); cur = 0; }
        }
        if (cur > 0) runs.add(cur);
        if (maxLen < 2) return false;

        int[] grundy = new int[maxLen + 1];
        for (int k = 2; k <= maxLen; k++) {
            boolean[] seen = new boolean[2 * maxLen + 4];
            for (int a = 0; a <= k - 2; a++)
                seen[grundy[a] ^ grundy[k - 2 - a]] = true;
            int mex = 0;
            while (seen[mex]) mex++;
            grundy[k] = mex;
        }

        int total = 0;
        for (int r : runs) total ^= grundy[r];
        return total != 0;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    bool canWin(const string& strip) {
        vector<int> runs;
        int cur = 0, maxLen = 0;
        for (char c : strip) {
            if (c == 'U') { cur++; maxLen = max(maxLen, cur); }
            else { if (cur) runs.push_back(cur); cur = 0; }
        }
        if (cur) runs.push_back(cur);
        if (maxLen < 2) return false;

        vector<int> grundy(maxLen + 1, 0);
        for (int k = 2; k <= maxLen; ++k) {
            vector<char> seen(2 * maxLen + 4, 0);
            for (int a = 0; a <= k - 2; ++a)
                seen[grundy[a] ^ grundy[k - 2 - a]] = 1;
            int mex = 0;
            while (seen[mex]) ++mex;
            grundy[k] = mex;
        }

        int total = 0;
        for (int r : runs) total ^= grundy[r];
        return total != 0;
    }
};
C
C
#include <string.h>
#include <stdbool.h>
#include <stdlib.h>

/* Sprague-Grundy: each maximal run of 'U' of length k is an independent
   sub-game. A move on a run of length k splits it into runs of length a and
   k-2-a (a >= 0). The Grundy value of length k is the mex over all such splits
   of grundy[a] ^ grundy[k-2-a]. The first player wins iff the XOR of the
   Grundy values of all runs is nonzero. */
bool canWin(char* strip) {
    int n = (int)strlen(strip);

    int maxLen = 0, cur = 0;
    for (int i = 0; i < n; i++) {
        if (strip[i] == 'U') { cur++; if (cur > maxLen) maxLen = cur; }
        else cur = 0;
    }
    if (maxLen < 2) return false;              /* no run can host a move */

    int* grundy = calloc(maxLen + 1, sizeof(int));
    int cap = 2 * maxLen + 4;                  /* safe bound for any XOR value */
    char* seen = malloc(cap);

    for (int k = 2; k <= maxLen; k++) {
        memset(seen, 0, cap);
        for (int a = 0; a <= k - 2; a++)
            seen[grundy[a] ^ grundy[k - 2 - a]] = 1;
        int mex = 0;
        while (mex < cap && seen[mex]) mex++;
        grundy[k] = mex;
    }

    int total = 0;
    cur = 0;
    for (int i = 0; i <= n; i++) {
        if (i < n && strip[i] == 'U') {
            cur++;
        } else {
            if (cur > 0) total ^= grundy[cur];
            cur = 0;
        }
    }

    free(grundy);
    free(seen);
    return total != 0;
}

#### Follow-up
questions
Derive and justify the runtime complexity of your solution (why the Grundy table is O(L^2)).
Implement the memoized-minimax version over raw string states and explain why it is exponential in the worst case.
Solve the misere variant, where the player who makes the last move loses — how do the winning conditions change?
Count how many distinct winning first moves exist, not just whether a win is possible.

