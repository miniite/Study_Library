---
type: "practice"
difficulty: "Other"
---

# Counting First-Player Wins in a Two-Arc Petal Game

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Math](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Math.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Two players play a turn-based game on a circular garden path lined with flowers. The two players stand at two distinct points on the circle, which splits the ring into two arcs between them: one arc holds x flowers and the other arc holds y flowers.

The rules are:

The starting player moves first, and the players then alternate turns.
On each turn, the player to move must pick exactly one flower from either arc (their choice).
After a turn, if there are no flowers left anywhere on the circle, the player who just moved wins (they take the final flower).

You are given two integers a and b. Count the number of configurations (x, y) for which the starting player wins, subject to:

1 <= x <= a (flowers on the first arc),
1 <= y <= b (flowers on the second arc).

Return the number of such winning pairs (x, y).

Input

a: the maximum number of flowers allowed on the first arc.
b: the maximum number of flowers allowed on the second arc.

Output

The count of pairs (x, y) in range for which the starting player wins. The count can be large, so use a 64-bit integer.

#### Cons
traints

1 <= a, b <= 10^5

#### Example
s & test cases
TEXT
- Input: `a = 3, b = 2` -> Output: `3`
  Why: the winning pairs are `(1,2)`, `(2,1)`, `(3,2)` — exactly those with `x + y` odd. Normal small case.
- Input: `a = 1, b = 1` -> Output: `0`
  Why: the only pair `(1,1)` has an even sum, so the starting player never wins (minimum input).
- Input: `a = 4, b = 5` -> Output: `10`
  Why: `floor(4 * 5 / 2) = 10` of the 20 pairs have an odd sum.
- Input: `a = 6, b = 1` -> Output: `3`
  Why: with `y = 1`, `x + y` is odd only when `x` is even, i.e. `x` in `{2, 4, 6}`. Edge with one dimension equal to 1.
- Input: `a = 2, b = 2` -> Output: `2`
  Why: `(1,2)` and `(2,1)` win; `(1,1)` and `(2,2)` have even sums. Balanced tiny case.
- Input: `a = 100000, b = 100000` -> Output: `5000000000`
  Why: `floor(10^10 / 2) = 5 * 10^9`, which overflows 32-bit, so a 64-bit result is required (maximum case).

#### Approach

The game always lasts exactly x + y turns: every turn removes exactly one flower and the game can only end on the turn that takes the last flower, so no choice by either player can shorten or lengthen it. The starting player takes turns 1, 3, 5, ..., so they take the final (winning) flower precisely when the total x + y is odd — the outcome is forced and independent of strategy. Therefore we just count pairs (x, y) with an odd sum: an odd sum needs one coordinate odd and the other even. In [1, a] there are ceil(a/2) odd and floor(a/2) even numbers, and likewise for [1, b], so the count is oddA·evenB + evenA·oddB, which simplifies to floor(a·b/2). This is a parity-counting / game-theory reduction — recognizing the winner depends only on (x + y) mod 2 collapses the whole game to one formula.

Complexity

Time: O(1) — the answer is a single closed-form arithmetic expression, no loops. Space: O(1) — only the returned value is computed. Note: a * b can reach 10^10, so the multiplication must be done in 64-bit (or JavaScript's safe-integer range) to avoid overflow.

Python
PYTHON
class Solution:
    def countFirstPlayerWins(self, a: int, b: int) -> int:
        # pairs with x + y odd = floor(a*b/2)
        return a * b // 2
JavaScript
JAVASCRIPT
function countFirstPlayerWins(a, b) {
    // pairs with x + y odd = floor(a*b/2); a*b <= 1e10 stays within Number's safe range
    return Math.floor((a * b) / 2);
}
Java
JAVA
class Solution {
  public long countFirstPlayerWins(int a, int b) {
    // pairs with x + y odd = floor(a*b/2); cast first so the multiply is 64-bit
    return (long) a * b / 2;
  }
}
C++
CPP
class Solution {
 public:
  long long countFirstPlayerWins(int a, int b) {
    // pairs with x + y odd = floor(a*b/2); multiply in 64-bit to avoid overflow
    return (long long)a * b / 2;
  }
};
C
C
long long countFirstPlayerWins(int a, int b) {
    /* pairs with x + y odd = floor(a*b/2); multiply in 64-bit to avoid overflow */
    return (long long)a * b / 2;
}

#### Follow-up
questions
Prove rigorously that the winner cannot depend on either player's moves, only on the parity of x + y.
How does the answer change if a player may pick 1 or 2 flowers per turn (a Nim-like variant)?
Count instead the configurations where the second player wins.
Generalize to k arcs (a k-dimensional range) and count the first-player wins.

