---
type: "practice"
difficulty: "Other"
---

# Counting Valid Connection Invitations by Age

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Array](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Array.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A community networking app has n members. You are given an integer array age where age[i] is the age of the i-th member.

A member x will not send a connection invitation to a different member y (x != y) if any of the following conditions holds:

age[y] <= 0.5 * age[x] + 7
age[y] > age[x]
age[y] > 100 and age[x] < 100

Otherwise, member x sends an invitation to member y.

Note that invitations are directed: if x sends an invitation to y, it does not imply that y sends one to x. A member never sends an invitation to themselves.

Return the total number of invitations sent across all ordered pairs of members.

#### Cons
traints:

n == age.length
1 <= n <= 2 * 10^4
1 <= age[i] <= 120

#### Example
s & test cases
TEXT
Input: age = [18,18] -> Output: 2
Why: two members of the same age 18 each invite the other (18 is old enough that 0.5*18+7 = 16 < 18), giving 2 directed invitations.

Input: age = [22,23,24] -> Output: 3
Why: 23->22, 24->22, 24->23 are all valid; no other ordered pair passes all three conditions.

Input: age = [24,34,100,110,120] -> Output: 3
Why: valid invitations are 110->100, 120->100, 120->110; the young ages 24 and 34 are below everyone's threshold and no older member may target a younger-than-half+7 age.

Input: age = [14,14] -> Output: 0
Why: age 14 fails its own gate since 0.5*14+7 = 14 and the target must be strictly greater than that, so no invitations are possible.

Input: age = [101,101,101] -> Output: 6
Why: three members all over 100 and same age; each invites the other two (0.5*101+7 = 57.5 < 101), giving 3 * 2 = 6 directed invitations.

Input: age = [16] -> Output: 0
Why: a single member has no one else to invite.

#### Approach

Because ages fall in the tiny fixed range 1..120, bucket members by age into a frequency array instead of comparing every pair of people (which would be O(n^2)). A member of age a can validly invite a member of age b exactly when b > 0.5*a + 7 and b <= a (the third rule, b > 100 and a < 100, is already implied by b > a and is therefore redundant). Rewriting b > 0.5*a + 7 for integers gives the inclusive lower bound b >= a//2 + 8, so valid targets are b in [a//2 + 8, a]. For distinct ages b < a the number of invitations is count[a] * count[b]; for the same age b == a (which is only reachable when a >= 15) any of the count[a] people can invite any of the other count[a] - 1, contributing count[a] * (count[a] - 1) ordered pairs. Summing over all ages yields the answer. This is a bucket-counting / prefix-of-conditions pattern that collapses a pairwise problem using a bounded key space.

Complexity

Time: O(n + A^2) where n is the number of members and A = 120 is the maximum age; the counting pass is O(n) and the double loop over the fixed age range is O(A^2) = constant work. Space: O(A) = O(1) for the fixed-size frequency array. Justification: ages are bounded by a small constant, so bucketing by age turns the naive O(n^2) pairwise check into constant-bounded age-vs-age arithmetic.

Python
PYTHON
from typing import List

def num_invitations(age: List[int]) -> int:
    count = [0] * 121
    for a in age:
        count[a] += 1

    ans = 0
    for a in range(15, 121):
        if count[a] == 0:
            continue
        # same-age ordered pairs
        ans += count[a] * (count[a] - 1)
        # strictly younger valid targets: b in [a // 2 + 8, a - 1]
        for b in range(a // 2 + 8, a):
            ans += count[a] * count[b]
    return ans
JavaScript
JAVASCRIPT
function numInvitations(age) {

#### cons
t count = new Array(121).fill(0);
    for (const a of age) count[a]++;

    let ans = 0;
    for (let a = 15; a <= 120; a++) {
        if (count[a] === 0) continue;
        ans += count[a] * (count[a] - 1);
        for (let b = (a >> 1) + 8; b < a; b++) {
            ans += count[a] * count[b];
        }
    }
    return ans;
}
Java
JAVA
class Solution {
    public int numInvitations(int[] age) {
        int[] count = new int[121];
        for (int a : age) count[a]++;

        long ans = 0;
        for (int a = 15; a <= 120; a++) {
            if (count[a] == 0) continue;
            ans += (long) count[a] * (count[a] - 1);
            for (int b = a / 2 + 8; b < a; b++)
                ans += (long) count[a] * count[b];
        }
        return (int) ans;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    int numInvitations(vector<int>& age) {
        int count[121] = {0};
        for (int a : age) count[a]++;

        long long ans = 0;
        for (int a = 15; a <= 120; a++) {
            if (count[a] == 0) continue;
            ans += (long long)count[a] * (count[a] - 1);
            for (int b = a / 2 + 8; b < a; b++)
                ans += (long long)count[a] * count[b];
        }
        return (int)ans;
    }
};
C
C
int numInvitations(int* age, int ageSize) {
    int count[121] = {0};
    for (int i = 0; i < ageSize; i++) count[age[i]]++;

    long ans = 0;
    for (int a = 15; a <= 120; a++) {
        if (count[a] == 0) continue;
        /* same-age ordered pairs (x != y) */
        ans += (long)count[a] * (count[a] - 1);
        /* strictly younger valid targets: b in [a/2 + 8, a - 1] */
        for (int b = a / 2 + 8; b < a; b++) {
            ans += (long)count[a] * count[b];
        }
    }
    return (int)ans;
}

#### Follow-up
questions
Prove the third condition (age[y] > 100 and age[x] < 100) is redundant given the first two, and show the range [a//2 + 8, a] is correct for both even and odd a.
If ages were not bounded by a small constant (say up to 10^9), how would you solve it — sort plus binary search or a two-pointer sliding window over the boundaries?
Extend to counting mutual invitations (pairs where both x->y and y->x are valid) instead of directed ones.
What if each member could send at most one invitation total, or you needed the actual list of (sender, receiver) pairs rather than just the count?

