---
type: "practice"
difficulty: "Other"
---

# Clear the Row of Colored Beads With the Fewest Inserts

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Backtracking](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Backtracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A single horizontal row holds a line of colored beads. Each bead has one of five colors, encoded by an uppercase letter drawn from the set {R, G, B, Y, P}. You also hold a small supply of spare beads in reserve.

You clear the row by taking turns. On each turn:

Choose any one bead from your reserve and insert it at any position in the row — between two existing beads, or at either end.
Immediately after inserting, look at the row. If it now contains a maximal group of three or more consecutive beads of the same color, that entire group is removed.
If removing a group causes beads that were previously separated to become adjacent and form a new group of three or more of the same color, that group is removed as well, and this chain reaction continues until no group of three or more remains.
If the row becomes empty, you have cleared it and the game ends.

You repeat turns until the row is empty or your reserve is exhausted.

Given the starting row row and the reserve string reserve (each character is the color of one spare bead you hold), return the minimum number of beads you must insert to completely clear the row. If the row cannot be cleared using the beads available, return -1.

You may use any subset of the reserve in any order; you do not have to use all of it.

Input

row: a string of bead colors describing the initial row.
reserve: a string of bead colors you hold in reserve.

Output

The minimum number of insertions needed to clear the row, or -1 if it is impossible.

#### Cons
traints

1 <= row.length <= 16
1 <= reserve.length <= 5
Both strings consist only of the characters R, G, B, Y, and P.
The starting row is guaranteed to contain no group of three or more consecutive beads of the same color.

#### Example
s & test cases
TEXT
- Input: `row = "PRRBBP"`, `reserve = "RB"` -> Output: `-1`
  Why: inserting `R` clears the `RRR` leaving `PBBP`, then inserting `B` clears the `BBB` leaving `PP`; the reserve is empty and two `P` beads remain, so it cannot be finished.
- Input: `row = "PPRRBBPP"`, `reserve = "PRBRP"` -> Output: `2`
  Why: insert `R` to make `PPRRRBBPP` -> `PPBBPP`, then insert `B` to make `PPBBBPP` -> `PPPP` which is itself a group of four and collapses to empty; only 2 beads were needed.
- Input: `row = "G"`, `reserve = "GGGGG"` -> Output: `2`
  Why: insert `G` to get `GG`, insert another `G` to get `GGG`, which is removed, clearing the row in 2 inserts.
- Input: `row = "RRGGRR"`, `reserve = "G"` -> Output: `1`
  Why: inserting one `G` makes `RRGGGRR`; the `GGG` is removed, the surrounding `RRRR` becomes adjacent and collapses too (chain reaction), clearing everything with a single insert.
- Input: `row = "R"`, `reserve = "R"` -> Output: `-1`
  Why: the best you can build is `RR`, only two of a color, which is never removed; the reserve runs out.
- Input: `row = "RGBYP"`, `reserve = "R"` -> Output: `-1`
  Why: every color appears once; a single insert can at most create a pair, never a group of three, so nothing can be removed.

#### Approach

This is a shortest-sequence-of-moves search solved with depth-first backtracking plus memoization (a game-state DP). Two helpers drive it. First, collapse normalizes any row by repeatedly deleting every maximal run of three or more identical beads and letting the freed neighbors merge, which reproduces the chain-reaction rule and gives a canonical row so equal states hash together. Second, dfs(board, hand) first collapses the board; an empty board costs 0. It prunes the reserve down to colors that actually appear on the board (any other insert is useless), and if nothing usable remains the state is infeasible. Otherwise it tries inserting each still-available distinct color into every gap, recursing on the resulting collapsed board with that bead removed from the reserve, and takes 1 + min over all children. Because two reserve beads of the same color are interchangeable, only one insertion per distinct color per gap is explored. Results are cached on the (collapsed board, remaining reserve) key so overlapping subproblems are solved once; -1 is returned when no branch clears the board.

Complexity

Let n be the row length (<= 16) and m the reserve size (<= 5). Each turn consumes exactly one reserve bead, so the recursion depth is at most m; at every node we branch over O(n) insertion gaps times at most 5 distinct colors. The raw search is therefore exponential, bounded by roughly O((5n)^m), but memoization on the state (collapsed row, remaining reserve) collapses the many paths that reach the same configuration, keeping it fast within the tiny limits. Space is O(number of distinct memoized states) plus O(n) per recursion frame for the string copies.

Python
PYTHON
import math
from functools import lru_cache

class Solution:
    def minInserts(self, row: str, reserve: str) -> int:
        def collapse(s: str) -> str:
            # Remove every maximal run of length >= 3, chaining reactions.
            start = 0
            n = len(s)
            for i in range(n):
                if s[i] != s[start]:
                    if i - start >= 3:
                        return collapse(s[:start] + s[i:])
                    start = i
            if n - start >= 3:
                return collapse(s[:start])
            return s

        @lru_cache(maxsize=None)
        def dfs(board: str, hand: str) -> float:
            board = collapse(board)
            if not board:
                return 0
            present = set(board)
            hand = "".join(c for c in hand if c in present)  # usable colors only
            if not hand:
                return math.inf

            best = math.inf
            for i in range(len(board) + 1):
                seen = set()
                for j, c in enumerate(hand):
                    if c in seen:                             # one color per gap
                        continue
                    seen.add(c)
                    nb = board[:i] + c + board[i:]
                    nh = hand[:j] + hand[j + 1:]
                    best = min(best, 1 + dfs(nb, nh))
            return best

        res = dfs(row, "".join(sorted(reserve)))
        return -1 if res == math.inf else res
JavaScript
JAVASCRIPT
function minInserts(row, reserve) {

#### cons
t startHand = reserve.split('').sort().join('');

#### cons
t memo = new Map();

    // Collapse every maximal run of length >= 3, chaining reactions.
    function collapse(s) {

#### cons
t n = s.length;
        let start = 0;
        for (let i = 0; i < n; ++i) {
            if (s[i] !== s[start]) {
                if (i - start >= 3) return collapse(s.slice(0, start) + s.slice(i));
                start = i;
            }
        }
        if (n - start >= 3) return collapse(s.slice(0, start));
        return s;
    }

    function dfs(rawBoard, hand) {

#### cons
t board = collapse(rawBoard);
        if (board.length === 0) return 0;

#### cons
t present = new Set(board);
        let h = '';
        for (const c of hand) if (present.has(c)) h += c;    // usable colors only
        if (h.length === 0) return Infinity;

#### cons
t key = board + '|' + h;
        if (memo.has(key)) return memo.get(key);

        let best = Infinity;
        for (let i = 0; i <= board.length; ++i) {

#### cons
t seen = new Set();
            for (let j = 0; j < h.length; ++j) {

#### cons
t c = h[j];
                if (seen.has(c)) continue;                    // one color per gap
                seen.add(c);

#### cons
t nb = board.slice(0, i) + c + board.slice(i);

#### cons
t nh = h.slice(0, j) + h.slice(j + 1);

#### cons
t res = dfs(nb, nh);
                if (res !== Infinity) best = Math.min(best, res + 1);
            }
        }
        memo.set(key, best);
        return best;
    }

#### cons
t r = dfs(row, startHand);
    return r === Infinity ? -1 : r;
}
Java
JAVA
import java.util.*;

class Solution {
    public int minInserts(String row, String reserve) {
        char[] rc = reserve.toCharArray();
        Arrays.sort(rc);
        int r = dfs(row, new String(rc), new HashMap<>());
        return r == Integer.MAX_VALUE ? -1 : r;
    }

    private int dfs(String rawBoard, String hand, Map<String, Integer> memo) {
        String board = collapse(rawBoard);
        if (board.isEmpty()) return 0;

        Set<Character> present = new HashSet<>();
        for (char c : board.toCharArray()) present.add(c);

        StringBuilder hb = new StringBuilder();
        for (char c : hand.toCharArray())
            if (present.contains(c)) hb.append(c);          // usable colors only
        String h = hb.toString();
        if (h.isEmpty()) return Integer.MAX_VALUE;

        String key = board + "|" + h;
        if (memo.containsKey(key)) return memo.get(key);

        int best = Integer.MAX_VALUE;
        for (int i = 0; i <= board.length(); ++i) {
            Set<Character> seen = new HashSet<>();
            for (int j = 0; j < h.length(); ++j) {
                if (!seen.add(h.charAt(j))) continue;        // one color per gap
                String nb = board.substring(0, i) + h.charAt(j) + board.substring(i);
                String nh = h.substring(0, j) + h.substring(j + 1);
                int res = dfs(nb, nh, memo);
                if (res != Integer.MAX_VALUE) best = Math.min(best, res + 1);
            }
        }
        memo.put(key, best);
        return best;
    }

    // Collapse every maximal run of length >= 3, chaining reactions.
    private String collapse(String s) {
        int n = s.length(), start = 0;
        for (int i = 0; i < n; ++i)
            if (s.charAt(i) != s.charAt(start)) {
                if (i - start >= 3)
                    return collapse(s.substring(0, start) + s.substring(i));
                start = i;
            }
        if (n - start >= 3) return collapse(s.substring(0, start));
        return s;
    }
}
C++
CPP
#include <string>
#include <algorithm>
#include <unordered_map>
#include <unordered_set>
#include <climits>
using namespace std;

class Solution {
public:
    int minInserts(string row, string reserve) {
        sort(reserve.begin(), reserve.end());
        unordered_map<string, int> memo;
        int r = dfs(row, reserve, memo);
        return r == INT_MAX ? -1 : r;
    }

private:
    // Collapse every maximal run of length >= 3, chaining reactions.
    string collapse(string s) {
        int n = (int)s.size(), start = 0;
        for (int i = 0; i < n; ++i)
            if (s[i] != s[start]) {
                if (i - start >= 3)
                    return collapse(s.substr(0, start) + s.substr(i));
                start = i;
            }
        if (n - start >= 3) return collapse(s.substr(0, start));
        return s;
    }

    int dfs(string board, string hand, unordered_map<string, int>& memo) {
        board = collapse(board);
        if (board.empty()) return 0;

        unordered_set<char> present(board.begin(), board.end());
        string h;
        for (char c : hand) if (present.count(c)) h += c;   // usable colors only
        if (h.empty()) return INT_MAX;

        string key = board + "|" + h;
        auto it = memo.find(key);
        if (it != memo.end()) return it->second;

        int best = INT_MAX;
        for (int i = 0; i <= (int)board.size(); ++i) {
            unordered_set<char> seen;
            for (int j = 0; j < (int)h.size(); ++j) {
                if (seen.count(h[j])) continue;             // one color per gap
                seen.insert(h[j]);
                string nb = board.substr(0, i) + h[j] + board.substr(i);
                string nh = h.substr(0, j) + h.substr(j + 1);
                int res = dfs(nb, nh, memo);
                if (res != INT_MAX) best = min(best, res + 1);
            }
        }
        memo[key] = best;
        return best;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <limits.h>

/* ---- fixed-capacity string->int hash map (open addressing) ---- */
#define CAP 262144  /* power of two */
typedef struct { char* key; int val; } Entry;
static Entry g_map[CAP];

static unsigned long hashStr(const char* s) {
    unsigned long h = 1469598103934665603UL;
    for (; *s; ++s) { h ^= (unsigned char)*s; h *= 1099511628211UL; }
    return h;
}
static int mapGet(const char* k, int* found) {
    unsigned long h = hashStr(k) & (CAP - 1);
    while (g_map[h].key) {
        if (strcmp(g_map[h].key, k) == 0) { *found = 1; return g_map[h].val; }
        h = (h + 1) & (CAP - 1);
    }
    *found = 0; return 0;
}
static void mapPut(const char* k, int v) {
    unsigned long h = hashStr(k) & (CAP - 1);
    while (g_map[h].key) {
        if (strcmp(g_map[h].key, k) == 0) { g_map[h].val = v; return; }
        h = (h + 1) & (CAP - 1);
    }
    g_map[h].key = strdup(k); g_map[h].val = v;
}

/* Fully collapse every maximal run of length >= 3, with chain reactions.
   Returns a newly malloc'd string. */
static char* collapse(const char* s) {
    int n = (int)strlen(s), start = 0;
    for (int i = 0; i < n; ++i) {
        if (s[i] != s[start]) {
            if (i - start >= 3) {
                int keep = n - i;
                char* tmp = malloc(start + keep + 1);
                memcpy(tmp, s, start);
                memcpy(tmp + start, s + i, keep);
                tmp[start + keep] = '\0';
                char* r = collapse(tmp);
                free(tmp);
                return r;
            }
            start = i;
        }
    }
    if (n - start >= 3) {              /* trailing run */
        char* tmp = malloc(start + 1);
        memcpy(tmp, s, start);
        tmp[start] = '\0';
        char* r = collapse(tmp);
        free(tmp);
        return r;
    }
    char* r = malloc(n + 1);
    memcpy(r, s, n + 1);
    return r;
}

static int dfs(const char* rawBoard, const char* hand) {
    char* board = collapse(rawBoard);
    if (board[0] == '\0') { free(board); return 0; }

    int present[128] = {0};
    for (char* p = board; *p; ++p) present[(int)*p] = 1;

    char fhand[8]; int fn = 0;
    for (const char* p = hand; *p; ++p)
        if (present[(int)*p]) fhand[fn++] = *p;   /* only colors that occur */
    fhand[fn] = '\0';
    if (fn == 0) { free(board); return INT_MAX; }

    char key[64];
    snprintf(key, sizeof(key), "%s|%s", board, fhand);
    int found; int cached = mapGet(key, &found);
    if (found) { free(board); return cached; }

    int bn = (int)strlen(board);
    int best = INT_MAX;
    for (int i = 0; i <= bn; ++i) {
        char seen[128] = {0};
        for (int j = 0; j < fn; ++j) {
            char c = fhand[j];
            if (seen[(int)c]) continue;           /* try each color once per gap */
            seen[(int)c] = 1;

            char nb[40];
            memcpy(nb, board, i);
            nb[i] = c;
            memcpy(nb + i + 1, board + i, bn - i);
            nb[bn + 1] = '\0';

            char nh[8];
            memcpy(nh, fhand, j);
            memcpy(nh + j, fhand + j + 1, fn - j - 1);
            nh[fn - 1] = '\0';

            int res = dfs(nb, nh);
            if (res != INT_MAX && res + 1 < best) best = res + 1;
        }
    }
    mapPut(key, best);
    free(board);
    return best;
}

int minInserts(const char* row, const char* reserve) {
    for (int i = 0; i < CAP; ++i)
        if (g_map[i].key) { free(g_map[i].key); g_map[i].key = NULL; }

    char h[8]; strcpy(h, reserve);
    int m = (int)strlen(h);
    for (int a = 0; a < m; ++a)                    /* canonicalize the reserve */
        for (int b = a + 1; b < m; ++b)
            if (h[b] < h[a]) { char t = h[a]; h[a] = h[b]; h[b] = t; }

    int r = dfs(row, h);
    return r == INT_MAX ? -1 : r;
}

int main(void) {
    printf("%d\n", minInserts("PRRBBP", "RB"));     /* -1 */
    printf("%d\n", minInserts("PPRRBBPP", "PRBRP")); /* 2 */
    printf("%d\n", minInserts("G", "GGGGG"));        /* 2 */
    printf("%d\n", minInserts("RRGGRR", "G"));       /* 1 */
    return 0;
}

#### Follow-up
questions
How would you reconstruct the actual list of (position, color) insertions that achieve the minimum, not just the count?
What extra pruning keeps this practical if the row length grows (e.g., only insert next to a matching run, and skip inserts that can never lead to a removal)?
Suppose removed groups scored points and you wanted to maximize score rather than minimize inserts — how does the recurrence change?
What if a group of only two identical beads (instead of three) is enough to be removed? How do collapse and the branching adapt?

