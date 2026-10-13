---
type: "practice"
difficulty: "Other"
---

# Deciding the Higher Scorer in a Bonus-Doubling Round Game

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Two contestants each play the same number of rounds in a target-knockdown game. Every round begins with exactly 10 standing targets, and a contestant knocks down some number of them (between 0 and 10) that round.

You are given two 0-indexed integer arrays a and b of equal length n. Entry a[i] is the number of targets the first contestant knocked down in round i, and b[i] is the same for the second contestant.

The value a contestant earns in round i is computed with a bonus rule:

It is 2 * (targets knocked down in round i) if that same contestant achieved a full clear (knocked down all 10 targets) in either round i - 1 or round i - 2.
Otherwise, it is simply the number of targets knocked down in round i.

A contestant's total score is the sum of the values earned across all n rounds. Compare the two totals and return:

1 if the first contestant's total score is strictly greater,
2 if the second contestant's total score is strictly greater,
0 if the two totals are equal.

Input

a: array of n integers, each in [0, 10].
b: array of n integers, each in [0, 10].

Output

An integer 1, 2, or 0 per the rules above.

#### Cons
traints

n == a.length == b.length
1 <= n <= 1000
0 <= a[i], b[i] <= 10

#### Example
s & test cases
TEXT
Input: a = [4, 10, 2, 3], b = [7, 6, 5, 2] -> Output: 1
Why: A full clear in round 1 doubles rounds 2 and 3 for the first contestant: 4 + 10 + 2*2 + 2*3 = 24 vs 7+6+5+2 = 20.

Input: a = [3, 5, 7, 6], b = [8, 10, 10, 2] -> Output: 2
Why: The second contestant's back-to-back clears cascade doubles: 8 + 10 + 2*10 + 2*2 = 42 vs 3+5+7+6 = 21.

Input: a = [2, 3], b = [4, 1] -> Output: 0
Why: A tie — no full clears anywhere, so totals are 5 and 5.

Input: a = [10, 1, 1, 1], b = [1, 1, 1, 1] -> Output: 1
Why: An early full clear doubles the next two rounds: 10 + 2*1 + 2*1 + 1 = 15 vs 4.

Input: a = [10], b = [9] -> Output: 1
Why: Minimum length; with only one round there is no earlier round to trigger doubling, so scores are 10 and 9.

#### Approach

Direct simulation with a small look-back window. Score each contestant independently in one left-to-right pass: at round i, check whether that contestant had a full clear (value equal to 10) in either of the two immediately preceding rounds; if so the current round's targets count double, otherwise they count once. Accumulate into a running total, then compare the two totals and map the comparison to 1, 2, or 0. The pattern is a fixed-size sliding look-back — you never need to store the whole history, only the ability to peek at positions i-1 and i-2, which the index bounds guard against at the start of the array.

Complexity

Time: O(n) — each array is scanned exactly once, doing constant work per round. Space: O(1) — only two running totals are kept; no auxiliary structures scale with the input.

Python
PYTHON
from typing import List

def determine_winner(a: List[int], b: List[int]) -> int:
    def score(arr: List[int]) -> int:
        total = 0
        for i, x in enumerate(arr):
            if (i >= 1 and arr[i - 1] == 10) or (i >= 2 and arr[i - 2] == 10):
                total += 2 * x
            else:
                total += x
        return total

    sa, sb = score(a), score(b)
    if sa > sb:
        return 1
    if sb > sa:
        return 2
    return 0
JavaScript
JAVASCRIPT
function determineWinner(a, b) {

#### cons
t scoreOf = (arr) => {
        let total = 0;
        for (let i = 0; i < arr.length; i++) {
            if ((i >= 1 && arr[i - 1] === 10) || (i >= 2 && arr[i - 2] === 10))
                total += 2 * arr[i];
            else
                total += arr[i];
        }
        return total;
    };

#### cons
t sa = scoreOf(a), sb = scoreOf(b);
    if (sa > sb) return 1;
    if (sb > sa) return 2;
    return 0;
}
Java
JAVA
class Solution {
    private long scoreOf(int[] arr) {
        long total = 0;
        for (int i = 0; i < arr.length; i++) {
            if ((i >= 1 && arr[i - 1] == 10) || (i >= 2 && arr[i - 2] == 10))
                total += 2L * arr[i];
            else
                total += arr[i];
        }
        return total;
    }

    public int determineWinner(int[] a, int[] b) {
        long sa = scoreOf(a), sb = scoreOf(b);
        if (sa > sb) return 1;
        if (sb > sa) return 2;
        return 0;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
    long long scoreOf(const vector<int>& arr) {
        long long total = 0;
        int n = (int)arr.size();
        for (int i = 0; i < n; ++i) {
            if ((i >= 1 && arr[i - 1] == 10) || (i >= 2 && arr[i - 2] == 10))
                total += 2 * arr[i];
            else
                total += arr[i];
        }
        return total;
    }
public:
    int determineWinner(vector<int>& a, vector<int>& b) {
        long long sa = scoreOf(a), sb = scoreOf(b);
        if (sa > sb) return 1;
        if (sb > sa) return 2;
        return 0;
    }
};
C
C
static long long scoreOf(int* arr, int n) {
    long long total = 0;
    for (int i = 0; i < n; i++) {
        if ((i >= 1 && arr[i - 1] == 10) || (i >= 2 && arr[i - 2] == 10))
            total += 2LL * arr[i];
        else
            total += arr[i];
    }
    return total;
}

int determineWinner(int* a, int aSize, int* b, int bSize) {
    long long sa = scoreOf(a, aSize);
    long long sb = scoreOf(b, bSize);
    if (sa > sb) return 1;
    if (sb > sa) return 2;
    return 0;
}

#### Follow-up
questions
What if a full clear doubled only the single immediately following round instead of the next two?
How would you extend this to three or more contestants and return their full ranking?
Could you compute the winner in a single pass over both arrays simultaneously while keeping O(1) extra space?
What if the number of targets per round were variable and a "full clear" meant knocking down all targets available that round?

