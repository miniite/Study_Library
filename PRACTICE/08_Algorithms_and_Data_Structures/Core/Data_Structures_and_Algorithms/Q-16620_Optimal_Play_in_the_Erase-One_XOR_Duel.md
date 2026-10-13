---
type: "practice"
difficulty: "Other"
---

# Optimal Play in the Erase-One XOR Duel

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bit Manipulation](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bit_Manipulation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Two players share a board on which a list of non-negative integers is written. They take alternating turns, and the first player moves first. On a turn, the player to move must erase exactly one of the numbers still on the board.

The outcome is governed by the bitwise XOR of all numbers that remain on the board:

Losing rule. If, immediately after a player erases a number, the XOR of every number still on the board equals 0, then that player loses at once. (The XOR of a single remaining number is that number itself, and the XOR of an empty board is 0.)
Instant-win rule. If a player begins their turn and the XOR of all numbers currently on the board is already 0, then that player wins immediately, before erasing anything.

Both players play optimally. Determine whether the first player is guaranteed to win.

Return true if and only if the first player wins.

Input

marks: an array of non-negative integers written on the board.

Output

A boolean: true if the first player wins under optimal play, otherwise false.

#### Cons
traints

1 <= marks.length <= 1000
0 <= marks[i] < 2^16 (i.e. each value is in [0, 65535])

#### Example
s & test cases
TEXT
Input: marks = [3, 3, 5] -> Output: false
Why: total XOR = 3 ^ 3 ^ 5 = 5 (nonzero) and the count 3 is odd; with odd size and nonzero XOR the first player is eventually forced to create a zero XOR and loses.

Input: marks = [1, 2, 4, 8] -> Output: true
Why: total XOR = 15 (nonzero) but the count 4 is even; an even-sized board with nonzero XOR is always winnable for the player to move — this is the case that proves parity, not the XOR value, decides it.

Input: marks = [2, 4, 6] -> Output: true
Why: 2 ^ 4 ^ 6 = 0 at the very start, so the first player wins immediately by the instant-win rule even though the count is odd.

Input: marks = [5] -> Output: false
Why: single nonzero element; the first player must erase it, leaving an empty board whose XOR is 0, so that erase loses the game.

Input: marks = [0] -> Output: true
Why: the board XOR is already 0 when the first player's turn begins, so they win instantly (they never even move).

Input: marks = [0, 7] -> Output: true
Why: even count; parity guarantees the first player survives to hand a losing position back to the opponent.

#### Approach

The answer is simply XOR(all values) == 0 OR n is even. If the board already XORs to 0 at the start, the first player wins immediately by the instant-win rule. Otherwise the total XOR is nonzero, and parity decides everything. Key claim: a position with even size and nonzero total XOR always has a "safe" number to erase that keeps the XOR nonzero. If it did not — if every possible erase drove the XOR to 0 — then every element would have to equal the total XOR; but XOR-ing an even count of identical values yields 0, contradicting the nonzero total. So the player facing an even board can always erase safely, passing an odd board to the opponent, and by induction the opponent is the one who eventually runs out of safe moves. When the size is odd and the XOR is nonzero, the symmetric argument shows the first player is the one forced to zero the board. This is a classic game-theory / brainteaser result whose implementation collapses to an O(n) XOR and a parity test.

Complexity

Time: O(n) — a single pass to XOR all values together. Space: O(1) — only the running XOR and the length are needed. The whole game reduces to one XOR aggregate plus a parity check.

Python
PYTHON
from typing import List
from functools import reduce
from operator import xor

def xor_game(marks: List[int]) -> bool:
    total = reduce(xor, marks, 0)
    # Win iff starting XOR is 0, or the count is even.
    return total == 0 or len(marks) % 2 == 0
JavaScript
JAVASCRIPT
function xorGame(marks) {
    let total = 0;
    for (const v of marks) total ^= v;
    // Win iff starting XOR is 0, or the count is even.
    return total === 0 || marks.length % 2 === 0;
}
Java
JAVA
class Solution {
    public boolean xorGame(int[] marks) {
        int total = 0;
        for (int v : marks)
            total ^= v;
        // Win iff starting XOR is 0, or the count is even.
        return total == 0 || marks.length % 2 == 0;
    }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
public:
    bool xorGame(vector<int>& marks) {
        int total = 0;
        for (int v : marks)
            total ^= v;
        // Win iff starting XOR is 0, or the count is even.
        return total == 0 || marks.size() % 2 == 0;
    }
};
C
C
#include <stdbool.h>

bool xorGame(int* marks, int marksSize) {
    int total = 0;
    for (int i = 0; i < marksSize; i++)
        total ^= marks[i];
    /* First player wins iff the board already XORs to 0,
       or the number of elements is even. */
    return total == 0 || (marksSize % 2 == 0);
}

#### Follow-up
questions
Prove rigorously why parity of the count, not the actual XOR value, decides the winner once the initial XOR is nonzero.
How does the answer change if the player who reduces the XOR to 0 wins instead of loses?
Generalize the game to k players seated in a circle taking turns — who wins?
What if a player may erase up to two numbers on a single turn?

