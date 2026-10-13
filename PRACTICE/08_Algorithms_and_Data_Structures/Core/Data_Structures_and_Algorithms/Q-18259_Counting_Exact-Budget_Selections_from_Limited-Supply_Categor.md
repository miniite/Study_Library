---
type: "practice"
difficulty: "Other"
---

# Counting Exact-Budget Selections from Limited-Supply Categories

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An event hands out reward tickets that come in n different categories. You are given an integer budget and a 0-indexed 2D integer array categories, where categories[i] = [supply_i, value_i] means category i offers supply_i identical tickets, and each ticket of that category is worth value_i credits.

You want to pick some tickets so that the total credit value of the chosen tickets is exactly budget. From category i you may take anywhere from 0 up to supply_i tickets. Because all tickets within the same category are identical, only the number of tickets you take from each category matters (taking the "first" versus the "second" ticket of a category is the same selection).

Return the number of distinct ways to reach exactly budget credits. Since the count can be very large, return it modulo 10^9 + 7.

Two selections are different if and only if they differ in how many tickets are taken from at least one category.

Input:

budget: the exact total credit value to reach.
categories: an array of [supply_i, value_i] pairs.

Output:

The number of ways to reach exactly budget credits, taken modulo 10^9 + 7.

#### Cons
traints:

1 <= budget <= 1000
n == categories.length
1 <= n <= 50
categories[i].length == 2
1 <= supply_i, value_i <= 50

#### Example
s & test cases
TEXT
1. Input: budget = 5, categories = [[5,1],[3,2],[2,4]] -> Output: 4
   Why: the four selections summing to 5 are (5 of value-1), (3 of value-1 + 1 of value-2), (1 of value-1 + 2 of value-2), and (1 of value-1 + 1 of value-4). Mixed general case.

2. Input: budget = 4, categories = [[2,2],[4,1]] -> Output: 3
   Why: reach 4 with (2 of value-2), (1 of value-2 + 2 of value-1), or (4 of value-1). Multiple categories, small budget.

3. Input: budget = 6, categories = [[3,2]] -> Output: 1
   Why: a single category of value-2 with supply 3 can only make 0, 2, 4, or 6; the only way to hit 6 is to take all 3 tickets. Single unique way, supply-bound edge.

4. Input: budget = 3, categories = [[2,2]] -> Output: 0
   Why: value-2 tickets can only produce even totals (0, 2, 4), so an odd budget of 3 is unreachable. Impossible / zero-answer edge.

5. Input: budget = 5, categories = [[50,1],[50,2],[50,5]] -> Output: 4
   Why: reach 5 via (5 of value-1), (3 of value-1 + 1 of value-2), (1 of value-1 + 2 of value-2), or (1 of value-5). Large supplies that do not constrain the answer.

#### Approach

This is a bounded (multiplicity-limited) knapsack counting problem. Let dp[s] be the number of ways to reach a total of exactly s credits using only the categories processed so far, all reductions taken modulo 1e9+7. Initialize dp[0] = 1 (one empty selection reaches 0) and everything else 0. For each category [supply, value], build a fresh array ndp: for every reachable partial total i, distribute its ways to i + c*value for c = 0,1,...,supply (as long as it stays within budget), which represents "take c tickets of this category on top of the earlier choices." Because ndp is rebuilt from the previous dp rather than updated in place, each category's ticket count is bounded to at most supply automatically, avoiding the overcounting that an in-place unbounded update would cause. After all categories, dp[budget] is the answer. (A prefix-sum trick can shave the inner loop to O(1) amortized, reducing total time to O(n * budget), but the direct triple loop already fits comfortably in the limits.)

Complexity

Time: O(n * budget * maxSupply) in the worst case, since for each of the n categories we scan every partial total from 0..budget and, for each, try up to supply+1 ticket counts. With n <= 50, budget <= 1000, supply <= 50 this is at most ~2.5 million elementary operations. Space: O(budget) — two rolling 1D arrays of size budget+1; the older one is discarded each round.

Python
PYTHON
class Solution:
    def countWays(self, budget: int, categories: list[list[int]]) -> int:
        MOD = 1_000_000_007
        dp = [0] * (budget + 1)
        dp[0] = 1

        for supply, value in categories:
            ndp = [0] * (budget + 1)
            for i in range(budget + 1):
                if dp[i] == 0:
                    continue
                c = 0
                while c <= supply and i + c * value <= budget:
                    ndp[i + c * value] = (ndp[i + c * value] + dp[i]) % MOD
                    c += 1
            dp = ndp

        return dp[budget]
JavaScript
JAVASCRIPT
function countWays(budget, categories) {

#### cons
t MOD = 1000000007n;
    let dp = new Array(budget + 1).fill(0n);
    dp[0] = 1n;

    for (const [supply, value] of categories) {

#### cons
t ndp = new Array(budget + 1).fill(0n);
        for (let i = 0; i <= budget; i++) {
            if (dp[i] === 0n) continue;
            for (let c = 0; c <= supply && i + c * value <= budget; c++) {

#### cons
t idx = i + c * value;
                ndp[idx] = (ndp[idx] + dp[i]) % MOD;
            }
        }
        dp = ndp;
    }

    return Number(dp[budget]);
}
Java
JAVA
class Solution {
    public int countWays(int budget, int[][] categories) {
        final long MOD = 1_000_000_007L;
        long[] dp = new long[budget + 1];
        dp[0] = 1;

        for (int[] cat : categories) {
            int supply = cat[0];
            int value = cat[1];
            long[] ndp = new long[budget + 1];
            for (int i = 0; i <= budget; i++) {
                if (dp[i] == 0) continue;
                for (int c = 0; c <= supply && i + (long) c * value <= budget; c++) {
                    int idx = i + c * value;
                    ndp[idx] = (ndp[idx] + dp[i]) % MOD;
                }
            }
            dp = ndp;
        }
        return (int) dp[budget];
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int countWays(int budget, vector<vector<int>> &categories) {

#### cons
t long long MOD = 1000000007LL;
        vector<long long> dp(budget + 1, 0);
        dp[0] = 1;

        for (auto &cat : categories) {
            int supply = cat[0];
            int value = cat[1];
            vector<long long> ndp(budget + 1, 0);
            for (int i = 0; i <= budget; i++) {
                if (dp[i] == 0) continue;
                for (int c = 0; c <= supply && i + (long long)c * value <= budget; c++) {
                    ndp[i + c * value] = (ndp[i + c * value] + dp[i]) % MOD;
                }
            }
            dp = move(ndp);
        }
        return (int)dp[budget];
    }
};
C
C
#include <stdlib.h>

#define MOD 1000000007LL

int countWays(int budget, int **categories, int categoriesSize, int *categoriesColSize) {
    long long *dp = calloc(budget + 1, sizeof(long long));
    long long *ndp = calloc(budget + 1, sizeof(long long));
    dp[0] = 1;

    for (int t = 0; t < categoriesSize; t++) {
        int supply = categories[t][0];
        int value = categories[t][1];
        for (int i = 0; i <= budget; i++) ndp[i] = 0;
        for (int i = 0; i <= budget; i++) {
            if (dp[i] == 0) continue;
            for (int c = 0; c <= supply && i + (long long)c * value <= budget; c++) {
                int idx = i + c * value;
                ndp[idx] = (ndp[idx] + dp[i]) % MOD;
            }
        }
        long long *tmp = dp;
        dp = ndp;
        ndp = tmp;
    }

    int ans = (int)dp[budget];
    free(dp);
    free(ndp);
    return ans;
}

#### Follow-up
questions
Reduce the inner loop with a sliding prefix-sum over dp indices in the same residue class of value, achieving overall O(n * budget) time.
Instead of counting, return one concrete selection (how many tickets from each category) that reaches the budget, or report that none exists.
Count selections that reach at least the budget, or the number of ways to reach every total from 0..budget in one pass.
Handle much larger supplies (e.g., up to 10^9) per category by binary-splitting each category into power-of-two ticket bundles so the DP stays polynomial.

