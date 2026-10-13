---
type: "practice"
difficulty: "Other"
---

# Circular Route Starting Depot

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Greedy](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Greedy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A delivery vehicle runs a circular loop that passes through n depots numbered 0 to n - 1. When the vehicle stops at depot i, it can load supply[i] units of fuel into its tank (the tank has unlimited capacity). Driving from depot i forward to the next depot (i + 1) mod n consumes burn[i] units of fuel.

The vehicle starts with an empty tank at some depot of your choosing and must drive one full lap in the forward direction, returning to the depot it started from. At every leg of the journey the tank must never go negative (you can only drive to the next depot if the fuel currently in the tank is at least the amount that leg burns).

Given the two integer arrays supply and burn, return the index of the depot from which the vehicle can complete exactly one full lap. If no such starting depot exists, return -1. If a valid starting depot exists, it is guaranteed to be unique.

Input / output. Input is two equal-length integer arrays supply and burn. Output is a single integer: a starting index in [0, n - 1], or -1.

#### Cons
traints

n == supply.length == burn.length
1 <= n <= 100000
0 <= supply[i], burn[i] <= 10000

#### Example
s & test cases
TEXT
Input: supply = [4,6,7,4], burn = [6,5,3,5] -> Output: 1
Why: Net per depot is [-2,+1,+4,-1] (total +2 >= 0, so a start exists). Starting at 1: tank 6, ->2 leaves 1, +7=8 ->3 leaves 5, +4=9 ->0 leaves 4, +4=8 ->1 leaves 2. Full lap succeeds; depots 0 fails immediately.

Input: supply = [3,2,2], burn = [4,3,2] -> Output: -1
Why: Net per depot is [-1,-1,0], total = -2 < 0, so total fuel is less than total burn and no start can finish a lap.

Input: supply = [7], burn = [3] -> Output: 0
Why: Single depot; supply 7 covers its own burn 3, so start at index 0 and drive the trivial one-depot loop back to itself.

Input: supply = [2,3,4], burn = [5,1,1] -> Output: 1
Why: Depot 0 has a large deficit (-3). Starting at 1: tank 3 ->2 leaves 2, +4=6 ->0 leaves 5, +2=7 ->1 leaves 2. Only index 1 works.

Input: supply = [5,1,2,3,4], burn = [4,4,1,5,1] -> Output: 2
Why: Net [+1,-3,+1,-2,+3], total 0 (exactly enough). The greedy reset lands on 2, the unique feasible start.

Input: supply = [0,0], burn = [0,0] -> Output: 0
Why: Every leg burns 0 and every depot supplies 0; the tank never goes negative, so index 0 (the earliest) is the unique answer.

#### Approach

Two greedy insights make one pass enough. First, feasibility: a full lap is possible iff the total surplus sum(supply[i] - burn[i]) >= 0; if total fuel is less than total burn, no start can work, so return -1. Second, choosing the start: scan left to right keeping a running tank of surpluses from the current candidate start. If the tank ever drops below 0 at depot i, then none of the depots from the current candidate through i can be the answer — because starting anywhere in that window would have hit the same shortfall no later — so reset the candidate start to i + 1 and zero the tank. When total surplus is non-negative, the last surviving candidate is exactly the unique valid start. This is the classic prefix-sum-with-reset greedy (a.k.a. the running-minimum / Kadane style argument applied to a circular route).

Complexity

Time: O(n) — a single left-to-right pass over the two arrays. Space: O(1) — only three integer accumulators are used regardless of input size.

Python
PYTHON
from typing import List

class Solution:
    def canCompleteCircuit(self, supply: List[int], burn: List[int]) -> int:
        net = 0
        tank = 0
        start = 0
        for i in range(len(supply)):
            diff = supply[i] - burn[i]
            net += diff
            tank += diff
            if tank < 0:              # candidate start fails; move past i
                start = i + 1
                tank = 0
        return -1 if net < 0 else start
JavaScript
JAVASCRIPT
function canCompleteCircuit(supply, burn) {
    let net = 0, tank = 0, start = 0;
    for (let i = 0; i < supply.length; i++) {

#### cons
t diff = supply[i] - burn[i];
        net += diff;
        tank += diff;
        if (tank < 0) {              // restart candidate at next depot
            start = i + 1;
            tank = 0;
        }
    }
    return net < 0 ? -1 : start;
}
Java
JAVA
class Solution {
    public int canCompleteCircuit(int[] supply, int[] burn) {
        int net = 0, tank = 0, start = 0;
        for (int i = 0; i < supply.length; i++) {
            int diff = supply[i] - burn[i];
            net += diff;
            tank += diff;
            if (tank < 0) {          // restart candidate at the next depot
                start = i + 1;
                tank = 0;
            }
        }
        return net < 0 ? -1 : start;
    }
}
C++
CPP
class Solution {
public:
    int canCompleteCircuit(vector<int>& supply, vector<int>& burn) {
        int net = 0, tank = 0, start = 0;
        for (int i = 0; i < (int)supply.size(); ++i) {
            int diff = supply[i] - burn[i];
            net += diff;
            tank += diff;
            if (tank < 0) {          // this start cannot reach i+1
                start = i + 1;
                tank = 0;
            }
        }
        return net < 0 ? -1 : start;
    }
};
C
C
int canCompleteCircuit(int* supply, int supplySize, int* burn, int burnSize) {
    int net = 0;      /* total surplus over the whole loop */
    int tank = 0;     /* running fuel since the current candidate start */
    int start = 0;

    for (int i = 0; i < supplySize; i++) {
        int diff = supply[i] - burn[i];
        net += diff;
        tank += diff;
        if (tank < 0) {          /* candidate start fails at or before i */
            start = i + 1;       /* the answer, if any, is strictly after i */
            tank = 0;
        }
    }
    return net < 0 ? -1 : start;
}

#### Follow-up
questions
Prove why, when the total surplus is non-negative, a valid start is guaranteed to be unique.
Return every valid starting depot instead of one, if uniqueness were not guaranteed.
What if the vehicle could travel in either direction around the loop?
Handle the variant where the tank has a finite capacity cap; does the single-pass greedy still hold?

