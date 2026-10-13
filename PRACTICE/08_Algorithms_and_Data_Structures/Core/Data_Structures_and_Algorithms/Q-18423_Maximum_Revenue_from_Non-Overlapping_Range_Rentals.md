---
type: "practice"
difficulty: "Other"
---

# Maximum Revenue from Non-Overlapping Range Rentals

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Binary Search](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Binary_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A storage facility has m shelves arranged in a single row and numbered 0 to m - 1. You are given a list of rental requests. Each request is a triple [lo, hi, pay], meaning a client wants to rent every shelf in the contiguous block from index lo to index hi (inclusive) and will pay exactly pay units for that whole block.

A shelf can be handed to at most one client, so any two requests you accept must cover disjoint blocks of shelves. You may accept any subset of the requests (possibly none), and some shelves may be left empty. Return the maximum total pay you can collect.

Input

m: the number of shelves in the row.
requests: a list of triples [lo, hi, pay].

Output

A single integer: the largest total pay obtainable from a set of accepted requests whose shelf blocks do not overlap.

#### Cons
traints

1 <= m <= 10^5
1 <= requests.length <= 10^5
Each request has exactly 3 integers.
0 <= lo <= hi <= m - 1
1 <= pay <= 10^3

#### Example
s & test cases
TEXT
Input: m = 6, requests = [[0,1,4],[0,3,5],[2,5,6]] -> Output: 10
Why: accepting [0,1] (pay 4) and [2,5] (pay 6) is disjoint and totals 10, which beats the single [0,3] block worth 5.

Input: m = 6, requests = [[0,1,4],[0,3,20],[2,5,6]] -> Output: 20
Why: the lucrative [0,3] block alone pays 20, more than the disjoint pair [0,1]+[2,5] = 10; picking the big block is optimal.

Input: m = 1, requests = [[0,0,7]] -> Output: 7
Why: minimal case — one shelf and one request; accept it for 7.

Input: m = 4, requests = [[0,1,5],[2,3,5]] -> Output: 10
Why: the two blocks are already disjoint, so both are accepted for a total of 10.

Input: m = 3, requests = [[0,2,4],[0,0,3],[1,1,3],[2,2,3]] -> Output: 9
Why: three single-shelf requests (3+3+3 = 9) beat renting the whole row as one block (4); tests fine-grained partitioning.

Input: m = 5, requests = [[1,2,8]] -> Output: 8
Why: only one request; shelves 0, 3 and 4 stay empty and the answer is simply its pay.

#### Approach

This is weighted interval scheduling framed on a line of positions. Define dp[i] as the maximum revenue achievable using only shelves 0..i-1. Sweep positions left to right. At position i there are two choices: leave shelf i-1 unrented, inheriting dp[i-1]; or accept some request whose block ends exactly at shelf i-1. Accepting a request [lo, hi=i-1, pay] earns pay plus the best revenue from everything strictly before its block, which is dp[lo] (shelves 0..lo-1) — because dp[lo] already excludes shelf lo and beyond, the accepted block never overlaps what it builds on. To find all requests ending at a given shelf in O(1) amortized, bucket them by their hi value up front. The answer is dp[m]. The underlying pattern is DP over sorted intervals by endpoint; an equivalent formulation sorts requests by end and uses binary search to find the last non-conflicting request, which the position-indexed sweep here replaces with direct O(1) predecessor lookup.

Complexity

Time: O(m + R) where m is the number of shelves and R is the number of requests — each shelf position is visited once and each request is examined exactly once when we reach its end shelf. Space: O(m + R) for the dp array and the per-end grouping of requests.

Python
PYTHON
from typing import List

def max_rental_revenue(m: int, requests: List[List[int]]) -> int:
    # Group requests by their end shelf.
    by_end = [[] for _ in range(m)]        # by_end[hi] = [(lo, pay), ...]
    for lo, hi, pay in requests:
        by_end[hi].append((lo, pay))

    # dp[i] = best revenue using shelves 0..i-1.
    dp = [0] * (m + 1)
    for i in range(1, m + 1):
        dp[i] = dp[i - 1]                  # leave shelf i-1 empty
        for lo, pay in by_end[i - 1]:
            cand = dp[lo] + pay            # take a request ending at i-1
            if cand > dp[i]:
                dp[i] = cand
    return dp[m]
JavaScript
JAVASCRIPT
function maxRentalRevenue(m, requests) {
    // Group requests by their end shelf.

#### cons
t byEnd = Array.from({ length: m }, () => []); // byEnd[hi] = [[lo, pay], ...]
    for (const [lo, hi, pay] of requests) byEnd[hi].push([lo, pay]);

    // dp[i] = best revenue using shelves 0..i-1.

#### cons
t dp = new Array(m + 1).fill(0);
    for (let i = 1; i <= m; i++) {
        dp[i] = dp[i - 1];                             // leave shelf i-1 empty
        for (const [lo, pay] of byEnd[i - 1]) {
            dp[i] = Math.max(dp[i], dp[lo] + pay);     // take a request ending here
        }
    }
    return dp[m];
}
Java
JAVA
import java.util.*;

class Solution {
    public int maxRentalRevenue(int m, int[][] requests) {
        // Group requests by their end shelf.
        List<int[]>[] byEnd = new List[m];
        for (int i = 0; i < m; i++) byEnd[i] = new ArrayList<>();
        for (int[] r : requests) byEnd[r[1]].add(new int[]{r[0], r[2]}); // {lo, pay}

        // dp[i] = best revenue using shelves 0..i-1.
        int[] dp = new int[m + 1];
        for (int i = 1; i <= m; i++) {
            dp[i] = dp[i - 1];                          // leave shelf i-1 empty
            for (int[] o : byEnd[i - 1])
                dp[i] = Math.max(dp[i], dp[o[0]] + o[1]); // take a request ending here
        }
        return dp[m];
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxRentalRevenue(int m, vector<vector<int>>& requests) {
        // Group requests by their end shelf.
        vector<vector<pair<int,int>>> byEnd(m);   // byEnd[hi] = {lo, pay}
        for (auto& r : requests)
            byEnd[r[1]].push_back({r[0], r[2]});

        // dp[i] = best revenue using shelves 0..i-1.
        vector<int> dp(m + 1, 0);
        for (int i = 1; i <= m; ++i) {
            dp[i] = dp[i - 1];                     // leave shelf i-1 empty
            for (auto& [lo, pay] : byEnd[i - 1])
                dp[i] = max(dp[i], dp[lo] + pay);  // take a request ending here
        }
        return dp[m];
    }
};
C
C
#include <stdlib.h>

/* requests: requestsSize rows, each {lo, hi, pay}. */
int maxRentalRevenue(int m, int** requests, int requestsSize, int* requestsColSize) {
    /* Bucket request indices by their end shelf using intrusive linked lists. */
    int* head = (int*)malloc(sizeof(int) * m);
    for (int i = 0; i < m; i++) head[i] = -1;
    int* nxt = (int*)malloc(sizeof(int) * requestsSize);
    for (int i = 0; i < requestsSize; i++) {
        int hi = requests[i][1];
        nxt[i] = head[hi];
        head[hi] = i;
    }

    /* dp[i] = best revenue using shelves 0..i-1. */
    int* dp = (int*)calloc(m + 1, sizeof(int));
    for (int i = 1; i <= m; i++) {
        dp[i] = dp[i - 1];                       /* leave shelf i-1 empty */
        for (int j = head[i - 1]; j != -1; j = nxt[j]) {
            int lo = requests[j][0], pay = requests[j][2];
            int cand = dp[lo] + pay;             /* take a request ending at i-1 */
            if (cand > dp[i]) dp[i] = cand;
        }
    }

    int ans = dp[m];
    free(head); free(nxt); free(dp);
    return ans;
}

#### Follow-up
questions
Reconstruct the actual set of accepted requests, not just the maximum revenue.
What changes if the shelf coordinates are huge and sparse (say up to 10^9) so you cannot index dp by position — how do coordinate compression plus sort-by-end and binary search recover the same answer?
Suppose each shelf could be rented to up to two different clients simultaneously; how would the DP state change?
If some requests carried a negative pay (a penalty), how would you adapt the transition?

