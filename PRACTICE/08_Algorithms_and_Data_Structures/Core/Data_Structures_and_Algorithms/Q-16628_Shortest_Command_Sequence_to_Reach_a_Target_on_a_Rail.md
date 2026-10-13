---
type: "practice"
difficulty: "Other"
---

# Shortest Command Sequence to Reach a Target on a Rail

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An automated shuttle sits on an infinite straight rail marked with integer coordinates. It begins at coordinate 0 with velocity +1. Its motion is driven entirely by a string of two commands, S (speed up) and T (turn):

On command S:
coordinate += velocity
velocity *= 2
On command T:
If the current velocity is positive, set velocity = -1.
Otherwise, set velocity = 1.
The coordinate does not change.

For example, running the commands "SST" moves the shuttle through coordinates 0 -> 1 -> 3 -> 3 while its velocity changes 1 -> 2 -> 4 -> -1.

Given an integer target, return the length of the shortest command sequence that leaves the shuttle exactly on coordinate target. The shuttle may pass through negative coordinates on the way. A valid sequence is always guaranteed to exist.

Input

An integer target.

Output

A single integer: the minimum number of commands needed to land exactly on target.

#### Cons
traints

1 <= target <= 10^4

#### Example
s & test cases
TEXT
Input: target = 7 -> Output: 3
Why: `"SSS"` drives 0 -> 1 -> 3 -> 7. Because 7 = 2^3 - 1, three straight accelerations land exactly on it.

Input: target = 6 -> Output: 5
Why: `"SSSTS"` drives 0 -> 1 -> 3 -> 7 -> 7 -> 6. Overshoot to 7, turn, then take one step back.

Input: target = 4 -> Output: 5
Why: `"SSTTS"` drives 0 -> 1 -> 3 -> 3 -> 3 -> 4. Undershoot to 3, two turns, then approach the last unit.

Input: target = 3 -> Output: 2
Why: `"SS"` drives 0 -> 1 -> 3. A power-of-two-minus-one target is reached by pure acceleration.

Input: target = 1 -> Output: 1
Why: `"S"` lands directly on 1 (minimum-size input, single command).

Input: target = 2 -> Output: 4
Why: 2 is not one less than a power of two, so a turn maneuver is unavoidable; the best is 4 commands (e.g. `"SSTS"`-style reversal).

#### Approach

Because velocity doubles on each S, after k consecutive accelerations from a standstill the shuttle covers exactly 2^k - 1 units. So any optimal solution is a sequence of such geometric bursts separated by turns. Build the answer bottom-up with dp[t] = minimum commands to reach coordinate t. For each t, let k be the smallest number of accelerations whose reach 2^k - 1 is at least t. Three cases: (1) if 2^k - 1 == t, the answer is simply k; (2) overshoot to 2^k - 1, spend one turn, and recurse on the leftover distance 2^k - 1 - t; (3) stop early at 2^(k-1) - 1, turn, roll back j accelerations, turn again, and recurse on the still-remaining gap. Taking the minimum over these options yields the optimal length. The core pattern is dynamic programming combined with a power-of-two (binary) decomposition of the distance.

Complexity

Time: O(target * log(target)) — for every coordinate up to target we try O(log target) undershoot split points. Space: O(target) for the DP table.

Python
PYTHON
def shortest_command_count(target: int) -> int:
    INF = float('inf')
    dp = [INF] * (target + 3)
    dp[0] = 0
    if target >= 1:
        dp[1] = 1
    if target >= 2:
        dp[2] = 4

    for t in range(3, target + 1):
        k = t.bit_length()                     # smallest k with 2^k - 1 >= t
        if t == (1 << k) - 1:                   # exact landing
            dp[t] = k
            continue
        # undershoot, turn, roll back j steps, turn, solve remainder
        for j in range(k - 1):
            rem = t - (1 << (k - 1)) + (1 << j)
            dp[t] = min(dp[t], dp[rem] + (k - 1) + j + 2)
        # overshoot, turn, solve leftover
        over = (1 << k) - 1 - t
        if over < t:
            dp[t] = min(dp[t], dp[over] + k + 1)

    return dp[target]
JavaScript
JAVASCRIPT
function shortestCommandCount(target) {

#### cons
t dp = new Array(target + 3).fill(Infinity);
    dp[0] = 0;
    if (target >= 1) dp[1] = 1;
    if (target >= 2) dp[2] = 4;

    for (let t = 3; t <= target; t++) {

#### cons
t k = 32 - Math.clz32(t);           // smallest k with 2^k - 1 >= t
        if (t === (1 << k) - 1) {                // exact landing
            dp[t] = k;
            continue;
        }
        // undershoot, turn, roll back j steps, turn, solve remainder
        for (let j = 0; j < k - 1; j++) {

#### cons
t rem = t - (1 << (k - 1)) + (1 << j);
            dp[t] = Math.min(dp[t], dp[rem] + (k - 1) + j + 2);
        }
        // overshoot, turn, solve leftover

#### cons
t over = (1 << k) - 1 - t;
        if (over < t) dp[t] = Math.min(dp[t], dp[over] + k + 1);
    }
    return dp[target];
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public int shortestCommandCount(int target) {
        int[] dp = new int[target + 3];
        Arrays.fill(dp, Integer.MAX_VALUE);
        dp[0] = 0;
        if (target >= 1) dp[1] = 1;
        if (target >= 2) dp[2] = 4;

        for (int t = 3; t <= target; t++) {
            int k = 32 - Integer.numberOfLeadingZeros(t); // smallest k with 2^k - 1 >= t
            if (t == (1 << k) - 1) {                       // exact landing
                dp[t] = k;
                continue;
            }
            // undershoot, turn, roll back j steps, turn, solve remainder
            for (int j = 0; j < k - 1; j++) {
                int rem = t - (1 << (k - 1)) + (1 << j);
                dp[t] = Math.min(dp[t], dp[rem] + (k - 1) + j + 2);
            }
            // overshoot, turn, solve leftover
            int over = (1 << k) - 1 - t;
            if (over < t) dp[t] = Math.min(dp[t], dp[over] + k + 1);
        }
        return dp[target];
    }
}
C++
CPP
#include <vector>
#include <climits>
using namespace std;

class Solution {
public:
    int shortestCommandCount(int target) {
        vector<int> dp(target + 3, INT_MAX);
        dp[0] = 0;
        if (target >= 1) dp[1] = 1;
        if (target >= 2) dp[2] = 4;

        for (int t = 3; t <= target; t++) {
            int k = 32 - __builtin_clz(t);          // smallest k with 2^k - 1 >= t
            if (t == (1 << k) - 1) {                 // exact landing
                dp[t] = k;
                continue;
            }
            // undershoot, turn, roll back j steps, turn, solve remainder
            for (int j = 0; j < k - 1; j++) {
                int rem = t - (1 << (k - 1)) + (1 << j);
                dp[t] = min(dp[t], dp[rem] + (k - 1) + j + 2);
            }
            // overshoot, turn, solve leftover
            int over = (1 << k) - 1 - t;
            if (over < t) dp[t] = min(dp[t], dp[over] + k + 1);
        }
        return dp[target];
    }
};
C
C
#include <limits.h>

int shortestCommandCount(int target) {
    int size = target + 3;
    int dp[size];
    for (int i = 0; i < size; i++) dp[i] = INT_MAX;
    dp[0] = 0;
    if (target >= 1) dp[1] = 1;
    if (target >= 2) dp[2] = 4;

    for (int t = 3; t <= target; t++) {
        int k = 0;
        while ((1 << k) <= t) k++;          /* smallest k with 2^k - 1 >= t */
        if (t == (1 << k) - 1) {            /* land exactly with k accelerations */
            dp[t] = k;
            continue;
        }
        /* undershoot to 2^(k-1)-1, turn, roll back j steps, turn again, solve remainder */
        for (int j = 0; j < k - 1; j++) {
            int rem = t - (1 << (k - 1)) + (1 << j);
            if (dp[rem] != INT_MAX) {
                int cand = dp[rem] + (k - 1) + j + 2;
                if (cand < dp[t]) dp[t] = cand;
            }
        }
        /* overshoot to 2^k-1, turn, solve the leftover distance */
        int over = (1 << k) - 1 - t;
        if (over < t && dp[over] != INT_MAX) {
            int cand = dp[over] + k + 1;
            if (cand < dp[t]) dp[t] = cand;
        }
    }
    return dp[target];
}

#### Follow-up
questions
Solve the same problem with BFS over (position, velocity) states and argue why the reachable search space stays bounded.
Reconstruct and return the actual optimal command string, not just its length.
How does the DP scale if target can be as large as 10^9? What alternative (memoized recursion / bounded search) keeps it tractable?
Suppose a turn also cost distance, or reset speed to a value other than 1 — how would the recurrence change?

