---
type: "practice"
difficulty: "Other"
---

# Minimum Starting Charge to Complete Every Job

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A single delivery drone must complete a fixed set of jobs before it lands. Each job is described by two positive integers:

cost — the amount of charge the drone consumes while performing that job.
requirement — the minimum charge the drone must already have on board to be allowed to begin that job.

You can perform the jobs in any order you like. To attempt a job, the drone's current charge must be at least that job's requirement; once the job finishes, its cost is subtracted from the current charge. It is always true that cost <= requirement for every job, so a job never leaves the drone with negative charge if it was legally started.

Choose the starting charge and the order of jobs so that every job can be completed, and return the minimum possible starting charge.

Concretely, for a chosen order the starting charge S works only if, at every job in that order, the charge remaining when you reach it is at least that job's requirement. Equivalently, if spent is the total cost of all jobs done before some job j, then you need S - spent >= requirement_j, i.e. S >= spent + requirement_j. Find the smallest S for which some order makes this hold for all jobs.

Input

A list jobs where each element is a pair [cost, requirement].

Output

A single integer: the minimum starting charge needed to finish all jobs.

#### Cons
traints

1 <= jobs.length <= 10^5
1 <= cost <= requirement <= 10^4

#### Example
s & test cases
TEXT
- Input: `jobs = [[3,6],[4,9],[2,5]]` -> Output: `12`
  Why: doing the largest-slack job first (`[4,9]` needs `9` but only burns `4`), then `[3,6]`, then `[2,5]` peaks at `spent(7)+requirement(5)=12`; no order does better.
- Input: `jobs = [[1,100],[50,60]]` -> Output: `100`
  Why: the job with a huge gap between requirement and cost (`[1,100]`) must be done while charge is highest, so `100` covers it and leaves plenty for `[50,60]`; the reverse order would need `150`.
- Input: `jobs = [[4,4],[2,2],[3,3]]` -> Output: `9`
  Why: every job has zero slack (`cost == requirement`), so there is no buffer and the answer equals the total of all costs `4+2+3=9` regardless of order.
- Input: `jobs = [[5,10]]` -> Output: `10`
  Why: a single job just needs its own `requirement`; the trailing charge does not matter (smallest non-trivial case).
- Input: `jobs = [[1,1]]` -> Output: `1`
  Why: minimum possible values — one unit of charge starts and finishes the only job (boundary case).

#### Approach

Greedy with an exchange argument. For a fixed order, the starting charge must satisfy S >= spent_before(j) + requirement_j for every job j, where spent_before(j) is the sum of costs of all earlier jobs. So the answer for that order is max over j of (spent_before(j) + requirement_j), and we want the order that minimizes this maximum. Consider two adjacent jobs a then b: keeping a first is at least as good as swapping iff cost_a + requirement_b <= cost_b + requirement_a, i.e. requirement_a - cost_a >= requirement_b - cost_b. So sorting all jobs in descending order of slack requirement - cost is optimal — jobs that demand a lot of charge but consume little are scheduled while the drone is fullest. After sorting, sweep once: maintain spent (running total of costs already burned) and take ans = max(ans, spent + requirement) before adding the current cost. This is the classic "minimize the peak prefix requirement" scheduling pattern.

Complexity

Time: O(n log n) — dominated by sorting the n jobs by slack; the single sweep afterward is O(n). Space: O(1) extra beyond the sort (only two running accumulators), or O(log n) for the sort's stack depending on the implementation. Justification: one comparison-based sort plus one linear pass.

Python
PYTHON
from typing import List

class Solution:
    def minimumStartingCharge(self, jobs: List[List[int]]) -> int:
        # Highest slack (requirement - cost) first, so the tightest starts
        # happen while remaining charge is largest.
        jobs.sort(key=lambda j: j[1] - j[0], reverse=True)
        ans = spent = 0
        for cost, requirement in jobs:
            ans = max(ans, spent + requirement)
            spent += cost
        return ans
JavaScript
JAVASCRIPT
function minimumStartingCharge(jobs) {
    // descending by slack = requirement - cost
    jobs.sort((a, b) => (b[1] - b[0]) - (a[1] - a[0]));
    let ans = 0, spent = 0;
    for (const [cost, requirement] of jobs) {
        ans = Math.max(ans, spent + requirement);
        spent += cost;
    }
    return ans;
}
Java
JAVA
import java.util.Arrays;

class Solution {
    public long minimumStartingCharge(int[][] jobs) {
        // descending by slack = requirement - cost (values <= 1e4, no overflow)
        Arrays.sort(jobs, (a, b) -> (b[1] - b[0]) - (a[1] - a[0]));
        long ans = 0, spent = 0;
        for (int[] j : jobs) {
            ans = Math.max(ans, spent + j[1]);
            spent += j[0];
        }
        return ans;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
 public:
  long long minimumStartingCharge(vector<vector<int>>& jobs) {
    // Do jobs with the biggest gap (requirement - cost) while charge is still high.
    sort(jobs.begin(), jobs.end(),
         [] (const vector<int>& a, const vector<int>& b) {
           return (a[1] - a[0]) > (b[1] - b[0]);
         });
    long long ans = 0, spent = 0;
    for (const auto& j : jobs) {
      ans = max(ans, spent + (long long)j[1]);
      spent += j[0];
    }
    return ans;
  }
};
C
C
#include <stdlib.h>

/* sort descending by slack = requirement - cost */
static int cmp(const void* a, const void* b) {

#### cons
t int* x = *(const int* const*)a;

#### cons
t int* y = *(const int* const*)b;
    long sx = (long)x[1] - x[0];
    long sy = (long)y[1] - y[0];
    if (sx < sy) return 1;
    if (sx > sy) return -1;
    return 0;
}

long long minimumStartingCharge(int** jobs, int jobsSize, int* jobsColSize) {
    qsort(jobs, jobsSize, sizeof(int*), cmp);
    long long ans = 0, spent = 0;
    for (int i = 0; i < jobsSize; ++i) {
        long long need = spent + jobs[i][1];   /* charge required to start this job */
        if (need > ans) ans = need;
        spent += jobs[i][0];                    /* consume this job's cost */
    }
    return ans;
}

#### Follow-up
questions
Prove the greedy is optimal with a formal exchange argument on two adjacent jobs.
Besides the minimum starting charge, also return the actual order in which the jobs should be performed.
What changes if some jobs could recharge the drone (a job whose cost is negative / whose requirement is below its cost)?
If you may skip up to k jobs to reduce the starting charge, how would you pick which to drop?

