---
type: "practice"
difficulty: "Other"
---

# Time for a Target Customer to Finish at the Service Window

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queue.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A single service window processes a line of customers one request at a time. There are n customers standing in a line, indexed from 0 (the very front of the line) to n - 1 (the very back).

You are given a 0-indexed integer array demand of length n, where demand[i] is the number of separate stamps the customer at position i still needs to collect.

The window works under these rules:

Serving one stamp takes exactly 1 second.
A customer at the front may collect only 1 stamp per turn. After collecting that stamp, if they still need more, they immediately (in zero time) walk to the back of the line and wait for another turn.
The moment a customer has collected all the stamps they need, they leave the line permanently.

Given an index target, return the total number of seconds that elapse from the start until the customer who initially stood at position target collects their last stamp and leaves.

#### Cons
traints:

n == demand.length
1 <= n <= 100
1 <= demand[i] <= 100
0 <= target < n

#### Example
s & test cases
TEXT
- Input: demand = [1,4,2], target = 1 -> Output: 7
  Why: the target needs 4 stamps; everyone at-or-ahead contributes up to 4 seconds, everyone behind up to 3. min(1,4) + min(4,4) + min(2,3) = 1 + 4 + 2 = 7.
- Input: demand = [6,2,2,2], target = 0 -> Output: 12
  Why: target at the very front with the largest demand; min(6,6) + min(2,5)*3 = 6 + 6 = 12. Edge case: target is index 0.
- Input: demand = [3,3,3], target = 1 -> Output: 8
  Why: at-or-ahead give min(3,3)=3 each, the one behind gives min(3,2)=2. 3 + 3 + 2 = 8. Tests the "behind uses cap-1" rule under ties.
- Input: demand = [5], target = 0 -> Output: 5
  Why: single customer, no line rotation; they simply need 5 seconds.
- Input: demand = [2,1,1,1,1], target = 4 -> Output: 5
  Why: cap = 1; indices 0..3 are at-or-ahead giving min(demand,1)=1 each, target gives 1. 1+1+1+1+1 = 5. Tests target at the back with a small demand.

#### Approach

Instead of simulating each second, reason about how many times each customer reaches the window before the target finishes. The target collects exactly demand[target] stamps, so the process ends the instant the target takes its final stamp. Any customer at a position i <= target (at or in front of the target's starting spot) gets a turn in every one of those rounds, but never more than their own demand, so they contribute min(demand[i], demand[target]) seconds. Any customer behind the target (i > target) misses the very last round because the target finishes before reaching them again, so they contribute min(demand[i], demand[target] - 1). Summing these gives the answer in one pass. Pattern: turn a queue/simulation into a per-element amortized counting argument.

Complexity

Time O(n): a single pass over the line. Space O(1): only a running total is kept. The key observation collapses the naive round-by-round simulation (which would be O(n * max(demand))) into one linear scan.

Python
PYTHON
from typing import List

def time_to_finish(demand: List[int], target: int) -> int:
    cap = demand[target]
    total = 0
    for i, d in enumerate(demand):
        limit = cap if i <= target else cap - 1   # those behind get one fewer round
        total += min(d, limit)
    return total
JavaScript
JAVASCRIPT
function timeToFinish(demand, target) {

#### cons
t cap = demand[target];
    let total = 0;
    for (let i = 0; i < demand.length; i++) {

#### cons
t limit = i <= target ? cap : cap - 1;   // customers behind stop one round earlier
        total += Math.min(demand[i], limit);
    }
    return total;
}
Java
JAVA
class Solution {
    public int timeToFinish(int[] demand, int target) {
        int cap = demand[target];
        int total = 0;
        for (int i = 0; i < demand.length; i++) {
            int limit = (i <= target) ? cap : cap - 1;   // customers behind stop one round earlier
            total += Math.min(demand[i], limit);
        }
        return total;
    }
}
C++
CPP
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int timeToFinish(vector<int>& demand, int target) {
        int cap = demand[target];
        int total = 0;
        for (int i = 0; i < (int)demand.size(); i++) {
            int limit = (i <= target) ? cap : cap - 1;
            total += min(demand[i], limit);
        }
        return total;
    }
};
C
C
#include <stddef.h>

/* demand: array of remaining stamps; demandSize: n; target: index of the tracked customer */
int timeToFinish(int* demand, int demandSize, int target) {
    int cap = demand[target];
    int total = 0;
    for (int i = 0; i < demandSize; i++) {
        int limit = (i <= target) ? cap : cap - 1;   /* those behind stop one round earlier */
        total += (demand[i] < limit) ? demand[i] : limit;
    }
    return total;
}

#### Follow-up
questions
What if each customer may collect up to 2 stamps per turn before rejoining the back of the line?
Return the finishing time for every customer, not just one, still in O(n) total.
The line arrives as a stream and you cannot index randomly; give an O(sum of demand) queue simulation and compare it to the O(n) formula.
Customers may join or leave dynamically mid-process; how would the counting argument change?

