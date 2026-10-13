---
type: "practice"
difficulty: "Other"
---

# Count Dissatisfied Participants After Pairing

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Matrix](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Matrix.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A workshop has n participants, numbered 0 to n - 1, where n is always even. Every participant has ranked all of the other participants by how much they would like to be teamed with them.

You are given ranking, a list of n lists. ranking[a] is the list of the other n - 1 participants, ordered from most preferred to least preferred by participant a (someone appearing earlier is preferred over someone appearing later). No participant appears in their own list, and each list contains every other participant exactly once.

The participants have been split into teams of two, described by teams, a list of n / 2 pairs. The pair teams[k] = [a, b] means participant a is teamed with b and b is teamed with a. Every participant belongs to exactly one team.

A participant x (teamed with y) is dissatisfied if there exists some other participant u (teamed with v) such that both of the following hold:

x prefers u over their own teammate y, and
u prefers x over their own teammate v.

Intuitively, x and u would both rather be teamed with each other than with the teammates they were assigned.

Return the number of dissatisfied participants.

Input

n: an even integer, the number of participants.
ranking: n lists, each a permutation of the other n - 1 participants in preference order.
teams: n / 2 pairs partitioning all n participants.

Output

An integer: how many participants are dissatisfied.

#### Cons
traints

2 <= n <= 500, and n is even.
ranking.length == n, ranking[a].length == n - 1.
0 <= ranking[a][j] <= n - 1; values in ranking[a] are unique and never equal a.
teams.length == n / 2, each teams[k] has length 2 with distinct endpoints in [0, n - 1].
Each participant appears in exactly one team.

#### Example
s & test cases
TEXT
- Input: `n = 2, ranking = [[1],[0]], teams = [[0,1]]` -> Output: `0`
  Why: with only one other person each, nobody has an alternative to prefer (minimum-size case).
- Input: `n = 4, ranking = [[1,2,3],[3,2,0],[3,1,0],[1,2,0]], teams = [[0,1],[2,3]]` -> Output: `2`
  Why: participant 1 prefers 3 over teammate 0 and 3 prefers 1 over teammate 2; symmetrically participant 3 is also dissatisfied, while 0 and 2 are content.
- Input: `n = 4, ranking = [[1,3,2],[2,3,0],[1,3,0],[0,2,1]], teams = [[1,3],[0,2]]` -> Output: `4`
  Why: every participant has a mutually-preferred alternative to their assigned teammate, so all four are dissatisfied (worst case).
- Input: `n = 4, ranking = [[1,2,3],[0,2,3],[3,0,1],[2,0,1]], teams = [[0,1],[2,3]]` -> Output: `0`
  Why: each participant was teamed with their single most-preferred choice, so no mutually-preferred alternative can exist (best case).
- Input: `n = 4, ranking = [[3,1,2],[0,2,3],[1,0,3],[0,1,2]], teams = [[0,1],[2,3]]` -> Output: `2`
  Why: participant 0 prefers 3 over teammate 1 and 3 prefers 0 over teammate 2; participant 3 mirrors this, while 1 and 2 stay content (partial-dissatisfaction case).

#### Approach

The key transformation is turning "does A prefer B over C?" into an O(1) test. Build rank[a][b] = the position of b in a's preference list, so a smaller value means more preferred. Also record each participant's partner from the team list. Now a participant x (teamed with y) is dissatisfied precisely when some u that x prefers over y also prefers x over its own teammate v = partner[u]. Because x's preference list is ordered best-first, only the people appearing before y in ranking[x] are candidates — so we walk ranking[x] and stop the moment we reach y. For each candidate u, a single comparison rank[u][x] < rank[u][partner[u]] decides mutual preference. This is a simulation / preference-matrix technique closely related to detecting instability in the stable-matching setting.

Complexity

Time: O(n^2) — building the rank lookup is O(n^2), and for each of the n participants we scan the prefix of their preference list up to their teammate, which is O(n) per participant, O(n^2) overall. Space: O(n^2) — the rank matrix storing each participant's preference position for every other participant. Justification: constant-time preference comparisons via rank reduce each dissatisfaction test from a linear search to an O(1) check.

Python
PYTHON
class Solution:
    def countDissatisfied(self, n: int, ranking: list[list[int]],
                          teams: list[list[int]]) -> int:
        rank = [[0] * n for _ in range(n)]
        for a in range(n):
            for pos, b in enumerate(ranking[a]):
                rank[a][b] = pos          # smaller position = more preferred
        partner = [0] * n
        for a, b in teams:
            partner[a] = b
            partner[b] = a
        count = 0
        for x in range(n):
            y = partner[x]
            for u in ranking[x]:
                if u == y:
                    break                 # nobody after y is preferred over y
                v = partner[u]
                if rank[u][x] < rank[u][v]:
                    count += 1
                    break
        return count
JavaScript
JAVASCRIPT
function countDissatisfied(n, ranking, teams) {

#### cons
t rank = Array.from({ length: n }, () => new Array(n).fill(0));
    for (let a = 0; a < n; ++a)
        for (let pos = 0; pos < ranking[a].length; ++pos)
            rank[a][ranking[a][pos]] = pos;   // smaller position = more preferred

#### cons
t partner = new Array(n).fill(0);
    for (const [a, b] of teams) {
        partner[a] = b;
        partner[b] = a;
    }
    let count = 0;
    for (let x = 0; x < n; ++x) {

#### cons
t y = partner[x];
        let dissatisfied = false;
        for (const u of ranking[x]) {
            if (u === y) break;               // nobody after y is preferred over y

#### cons
t v = partner[u];
            if (rank[u][x] < rank[u][v]) { dissatisfied = true; break; }
        }
        if (dissatisfied) ++count;
    }
    return count;
}
Java
JAVA
class Solution {
  public int countDissatisfied(int n, int[][] ranking, int[][] teams) {
    int[][] rank = new int[n][n];
    for (int a = 0; a < n; ++a)
      for (int pos = 0; pos < ranking[a].length; ++pos)
        rank[a][ranking[a][pos]] = pos;
    int[] partner = new int[n];
    for (int[] t : teams) {
      partner[t[0]] = t[1];
      partner[t[1]] = t[0];
    }
    int count = 0;
    for (int x = 0; x < n; ++x) {
      int y = partner[x];
      boolean dissatisfied = false;
      for (int u : ranking[x]) {
        if (u == y) break;
        int v = partner[u];
        if (rank[u][x] < rank[u][v]) { dissatisfied = true; break; }
      }
      if (dissatisfied) ++count;
    }
    return count;
  }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
 public:
  int countDissatisfied(int n, vector<vector<int>>& ranking,
                        vector<vector<int>>& teams) {
    vector<vector<int>> rank(n, vector<int>(n, 0));
    for (int a = 0; a < n; ++a)
      for (int pos = 0; pos < (int)ranking[a].size(); ++pos)
        rank[a][ranking[a][pos]] = pos;
    vector<int> partner(n);
    for (auto& t : teams) {
      partner[t[0]] = t[1];
      partner[t[1]] = t[0];
    }
    int count = 0;
    for (int x = 0; x < n; ++x) {
      int y = partner[x];
      bool dissatisfied = false;
      for (int u : ranking[x]) {
        if (u == y) break;
        int v = partner[u];
        if (rank[u][x] < rank[u][v]) { dissatisfied = true; break; }
      }
      if (dissatisfied) ++count;
    }
    return count;
  }
};
C
C
#include <stdlib.h>

int countDissatisfied(int n, int** ranking, int* rankingColSizes,
                      int** teams, int teamsSize, int* teamsColSizes) {
    int** rank = (int**)malloc(n * sizeof(int*));
    for (int a = 0; a < n; ++a) {
        rank[a] = (int*)calloc(n, sizeof(int));
        for (int pos = 0; pos < n - 1; ++pos)
            rank[a][ranking[a][pos]] = pos;   /* smaller position = more preferred */
    }
    int* partner = (int*)malloc(n * sizeof(int));
    for (int t = 0; t < teamsSize; ++t) {
        partner[teams[t][0]] = teams[t][1];
        partner[teams[t][1]] = teams[t][0];
    }
    int count = 0;
    for (int x = 0; x < n; ++x) {
        int y = partner[x];
        int dissatisfied = 0;
        for (int p = 0; p < n - 1; ++p) {
            int u = ranking[x][p];
            if (u == y) break;                /* nobody after y is preferred over y */
            int v = partner[u];
            if (rank[u][x] < rank[u][v]) { dissatisfied = 1; break; }
        }
        if (dissatisfied) ++count;
    }
    for (int a = 0; a < n; ++a) free(rank[a]);
    free(rank);
    free(partner);
    return count;
}

#### Follow-up
questions
Instead of a count, return the list of dissatisfied participants, or every mutually-preferred pair (x, u) that causes dissatisfaction.
How would you construct a pairing that minimizes (or eliminates) dissatisfaction — how does this relate to stable matching?
Can you lower the space from O(n^2) while keeping the O(1) preference comparison, or trade time for space if n is large?
What changes if preference lists may be partial (a participant ranks only some others and is indifferent to the rest)?

