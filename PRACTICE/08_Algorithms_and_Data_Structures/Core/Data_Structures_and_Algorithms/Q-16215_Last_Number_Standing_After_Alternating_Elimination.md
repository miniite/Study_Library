---
type: "practice"
difficulty: "Other"
---

# Last Number Standing After Alternating Elimination

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Cons
ider the list of all integers from 1 to n, written in strictly increasing order: [1, 2, 3, ..., n].

Repeatedly thin the list with alternating sweeps until a single number is left:

Left-to-right sweep: starting from the leftmost number, remove it and then remove every second number after it (i.e. remove the 1st, 3rd, 5th, ... remaining elements).
Right-to-left sweep: starting from the rightmost remaining number, remove it and then remove every second number moving left (remove the last, 3rd-from-last, ... remaining elements).
Continue alternating between a left-to-right sweep and a right-to-left sweep. Each sweep discards exactly half of the current numbers.

Stop when only one number remains. Return that number.

Input: a single integer n, the count of numbers initially present (1, 2, ..., n).

Output: the value of the one number that survives all the sweeps.

#### Cons
traints:

1 <= n <= 10^9
Building the actual list is infeasible at the upper bound, so an O(n) simulation will be far too slow and use too much memory — an O(log n) approach is expected.

#### Example
s & test cases
TEXT
Input: n = 9 -> Output: 6
Why: [1..9] --L→R--> [2,4,6,8] --R→L--> [2,6] --L→R--> [6]. Standard worked example.

Input: n = 1 -> Output: 1
Why: minimum input — the single number trivially survives with no sweeps needed.

Input: n = 2 -> Output: 2
Why: the first (left-to-right) sweep removes 1, leaving 2.

Input: n = 6 -> Output: 4
Why: [1..6] --L→R--> [2,4,6] --R→L (removes 6 then 2)--> [4].

Input: n = 10 -> Output: 8
Why: [1..10] --L→R--> [2,4,6,8,10] --R→L (removes 10,6,2)--> [4,8] --L→R (removes 4)--> [8].

Input: n = 4 -> Output: 2
Why: [1,2,3,4] --L→R--> [2,4] --R→L (removes 4)--> [2].

#### Approach

Never materialize the list. Track just two facts about the survivors: head, the value of the first (leftmost) survivor, and step, the constant gap between consecutive survivors. Both are enough because after any sweep the survivors remain an arithmetic progression. The key observation is when head moves: a left-to-right sweep always deletes the current first element, so head advances by step. A right-to-left sweep deletes the first element only when the current count is odd (an even count leaves the leftmost one untouched). After each sweep, exactly half the numbers are gone, so remaining halves and step doubles, and the direction flips. When one number is left, head is the survivor. Equivalently, by left/right symmetry the answer satisfies the recurrence f(n) = 2 * (1 + n/2 - f(n/2)) with f(1) = 1; the iterative head/step method is the unrolled, O(1)-space form of that idea. Pattern: mathematical elimination (Josephus-style) solved by pointer arithmetic rather than simulation.

Complexity

Time: O(log n) — every sweep halves the count of remaining numbers, so the loop runs about log2(n) times. Space: O(1) — only the head position, the current gap, the remaining count, and a direction flag are stored, never the list itself.

Python
PYTHON
class Solution:
    def lastNumberStanding(self, n: int) -> int:
        head, step, remaining = 1, 1, n
        left_to_right = True
        while remaining > 1:
            if left_to_right or remaining % 2 == 1:
                head += step
            remaining //= 2
            step *= 2
            left_to_right = not left_to_right
        return head
JavaScript
JAVASCRIPT
function lastNumberStanding(n) {
    let head = 1, step = 1, remaining = n;
    let leftToRight = true;
    while (remaining > 1) {
        if (leftToRight || remaining % 2 === 1) head += step;
        remaining = Math.floor(remaining / 2);
        step *= 2;
        leftToRight = !leftToRight;
    }
    return head;
}
Java
JAVA
class Solution {
    public int lastNumberStanding(int n) {
        long head = 1, step = 1, remaining = n;
        boolean leftToRight = true;
        while (remaining > 1) {
            if (leftToRight || remaining % 2 == 1)
                head += step;
            remaining /= 2;
            step *= 2;
            leftToRight = !leftToRight;
        }
        return (int) head;
    }
}
C++
CPP
class Solution {
public:
    int lastNumberStanding(int n) {
        long head = 1, step = 1, remaining = n;
        bool leftToRight = true;
        while (remaining > 1) {
            if (leftToRight || remaining % 2 == 1)
                head += step;
            remaining /= 2;
            step *= 2;
            leftToRight = !leftToRight;
        }
        return (int) head;
    }
};
C
C
/* Track only the head (first survivor) and the gap between survivors. */
int lastNumberStanding(int n) {
    long head = 1, step = 1, remaining = n;
    int leftToRight = 1;
    while (remaining > 1) {
        /* The head is eliminated whenever we sweep left-to-right, or when we
           sweep right-to-left over an odd count. */
        if (leftToRight || remaining % 2 == 1)
            head += step;
        remaining /= 2;
        step *= 2;
        leftToRight = !leftToRight;
    }
    return (int) head;
}

#### Follow-up
questions
Derive the recurrence f(n) = 2 * (1 + n/2 - f(n/2)) from the left/right symmetry and explain why the right-to-left pass over the first n numbers relates to the left-to-right pass over their mirror image.
How does the answer change if every sweep goes left-to-right (no alternation)? Show it collapses to the largest power of two ≤ n.
Return the k-th surviving number (from the left) after all sweeps, not just the last one.
Generalize to removing every third element instead of every second — how does the Josephus-style recurrence change?

