---
type: "practice"
difficulty: "Other"
---

# Subtree Removal Game on a Recursive Binary Tree

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Game Theory](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Game_Theory.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Cons
ider a family of binary trees T(k) defined recursively for a non-negative integer k:

T(0) is the empty tree (no nodes).
T(1) is a tree consisting of a single node (just a root).
For k >= 2, T(k) is a tree whose root has its left subtree equal to T(k - 2) and its right subtree equal to T(k - 1).

Two players, Rhea (who moves first) and Sol, play a game on the tree T(k), alternating turns. On a turn, the player picks any node that is still present and removes that node together with its entire subtree (that node and all of its descendants). A player who has no choice but to remove the root on their turn loses the game (removing the root wipes out the whole tree).

Both players play optimally. Given the integer k, return true if Rhea (the first player) wins, or false if Sol (the second player) wins.

A subtree of a node consists of that node and all of its descendants; the whole tree is a subtree of itself.

Input
k: a single positive integer describing which tree T(k) is played on.
Output
A boolean: true if the first player wins under optimal play, otherwise false.

#### Cons
traints
1 <= k <= 100

#### Example
s & test cases
TEXT
- Input: `k = 1` -> Output: `false`
  Why: `T(1)` is a lone root; the first player is immediately forced to remove the root and loses.
- Input: `k = 2` -> Output: `true`
  Why: `T(2)` is a root with a single child (`T(1)` on the right, empty on the left); the first player removes that child, forcing the opponent onto the root.
- Input: `k = 3` -> Output: `true`
  Why: `T(3)` has three removable non-root nodes arranged so the first player can always leave the root for the opponent.
- Input: `k = 6` -> Output: `true`
  Why: `6 mod 6 = 0`, which is a first-player win; a normal winning position.
- Input: `k = 7` -> Output: `false`
  Why: `7 mod 6 = 1`, the unique losing residue; the second player can always mirror optimally and hand the root back to the first player.

#### Approach

This is an impartial combinatorial game, so it is governed by the Sprague-Grundy theorem: the first player wins iff the Grundy (nimber) value of the starting position is non-zero. Removing a node and its subtree is exactly cutting one edge in a rooted green Hackenbush tree, and "forced to remove the root" is the normal-play terminal (only the ground/root left). Using the colon principle, define the branch nimber B(k) of T(k) when its root hangs from a parent by one edge: B(0) = 0 (empty branch), B(1) = 1 (single edge), and for k >= 2, B(k) = 1 + (B(k-2) XOR B(k-1)) (XOR the child branches, replace by a stalk of that length via the colon principle, then add the edge up to the parent). The whole-tree Grundy value is G(k) = B(k-2) XOR B(k-1). Computing the sequence shows G(k) is zero precisely when k is congruent to 1 (mod 6) (e.g. k = 1, 7, 13, ...), and non-zero otherwise. Hence the first player wins iff k % 6 != 1. The pattern/technique: Sprague-Grundy + Hackenbush colon principle, reduced to a periodic closed form.

Complexity

Time: O(1) — a single modulo comparison. Space: O(1) — no auxiliary storage. (Even if one derives the answer by computing the Grundy recurrence up to k, that is only O(k) time, but the closed form collapses it to constant.)

Python
PYTHON
class Solution:
    def firstPlayerWins(self, k: int) -> bool:
        # Grundy(T(k)) == 0  <=>  k % 6 == 1  ->  first player loses there.
        return k % 6 != 1
JavaScript
JAVASCRIPT
function firstPlayerWins(k) {
    // Grundy(T(k)) == 0  <=>  k % 6 == 1  ->  first player loses there.
    return k % 6 !== 1;
}
Java
JAVA
class Solution {
    public boolean firstPlayerWins(int k) {
        // Grundy(T(k)) == 0  <=>  k % 6 == 1  ->  first player loses there.
        return k % 6 != 1;
    }
}
C++
CPP
class Solution {
public:
    bool firstPlayerWins(int k) {
        // Grundy(T(k)) == 0  <=>  k % 6 == 1  ->  first player loses there.
        return k % 6 != 1;
    }
};
C
C
#include <stdbool.h>

/* First player wins unless the Grundy value of T(k) is zero, which happens
   exactly when k is congruent to 1 modulo 6. */
bool firstPlayerWins(int k) {
    return k % 6 != 1;
}

#### Follow-up
questions
Derive the period-6 pattern rigorously from the branch-nimber recurrence B(k) = 1 + (B(k-2) XOR B(k-1)).
How does the answer change if a player may remove only leaf nodes on each turn?
Solve the misere version: the player forced to remove the root wins instead of loses.
Analyze a differently-shaped recursive tree, e.g. both subtrees of T(k) equal to T(k-1).

