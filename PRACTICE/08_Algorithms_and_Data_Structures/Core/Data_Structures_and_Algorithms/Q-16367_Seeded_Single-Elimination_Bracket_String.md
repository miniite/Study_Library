---
type: "practice"
difficulty: "Other"
---

# Seeded Single-Elimination Bracket String

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A single-elimination tournament has n contestants, seeded (ranked) from 1 to n, where seed 1 is the strongest and seed n is the weakest. To keep matchups balanced, every round pairs the strongest remaining slot with the weakest, the second-strongest with the second-weakest, and so on.

Represent a match with parentheses and a comma: a pairing of slot A against slot B is written (A,B). In the first round the raw seeds are paired this way; in each subsequent round the (already parenthesized) winners' slots are paired again by the same strongest-with-weakest rule. Repeat until a single slot remains, which describes the whole bracket.

Return the final bracket as a string, with no spaces.

Concretely, before the first round the slots are "1", "2", ..., "n" in order. In one round, if there are m slots s[0], s[1], ..., s[m-1], the new slots become (s[0],s[m-1]), (s[1],s[m-2]), ... — m/2 of them — preserving order. Continue until one slot is left.

Input: an integer n.

Output: the single bracket string.

#### Cons
traints:

n is an exact power of two: n = 2^x for some integer x in the range [1, 12] (so n ranges from 2 to 4096).

#### Example
s & test cases
TEXT
- Input: n = 2 -> Output: "(1,2)". Only one round: the strongest meets the weakest.
- Input: n = 4 -> Output: "((1,4),(2,3))". Round 1 pairs (1,4) and (2,3); round 2 pairs those two winners.
- Input: n = 8 -> Output: "(((1,8),(4,5)),((2,7),(3,6)))". Round 1: (1,8),(2,7),(3,6),(4,5); round 2 pairs (1,8) with (4,5) and (2,7) with (3,6); round 3 pairs the two halves.
- Input: n = 16 -> Output: "((((1,16),(8,9)),((4,13),(5,12))),(((2,15),(7,10)),((3,14),(6,11))))". Verifies the strongest-vs-weakest rule cascades correctly across four rounds and that seeds 1 and 2 land in opposite halves.
- Input: smallest allowed n = 2 (minimum edge) -> Output: "(1,2)". Confirms the base case of a single pairing.

#### Approach

Simulate the bracket with a shrinking array of slot strings. Start with the seeds "1".."n". In each round, use a two-pointer pairing from both ends inward — slot i merges with slot count-1-i into "(" + slot[i] + "," + slot[count-1-i] + ")" — for the first half of the slots, then halve count. After log n rounds a single string remains, which is the whole bracket. The key pattern is the symmetric strongest-with-weakest folding, equivalent to recursively pairing a list with its reverse; the iterative in-place version keeps only the surviving front half each round.

Complexity

Time: O(n log n) — there are log n rounds, and each round rewrites slots whose total character length is O(n), so the concatenation work sums to O(n log n). Space: O(n) for the slot array and the final string, whose total length is O(n).

Python
PYTHON
class Solution:
    def bracketString(self, n: int) -> str:
        slots = [str(i + 1) for i in range(n)]
        count = n
        while count > 1:
            for i in range(count // 2):
                slots[i] = "(" + slots[i] + "," + slots[count - 1 - i] + ")"
            count //= 2
        return slots[0]
JavaScript
JAVASCRIPT
function bracketString(n) {
    let slots = Array.from({ length: n }, (_, i) => String(i + 1));
    for (let count = n; count > 1; count = Math.floor(count / 2)) {
        for (let i = 0; i < Math.floor(count / 2); i++) {
            slots[i] = "(" + slots[i] + "," + slots[count - 1 - i] + ")";
        }
    }
    return slots[0];
}
Java
JAVA
class Solution {
    public String bracketString(int n) {
        String[] slots = new String[n];
        for (int i = 0; i < n; i++)
            slots[i] = String.valueOf(i + 1);
        for (int count = n; count > 1; count /= 2) {
            for (int i = 0; i < count / 2; i++)
                slots[i] = "(" + slots[i] + "," + slots[count - 1 - i] + ")";
        }
        return slots[0];
    }
}
C++
CPP
#include <string>
#include <vector>
using namespace std;

class Solution {
public:
    string bracketString(int n) {
        vector<string> slots(n);
        for (int i = 0; i < n; i++)
            slots[i] = to_string(i + 1);
        for (int count = n; count > 1; count /= 2) {
            for (int i = 0; i < count / 2; i++)
                slots[i] = "(" + slots[i] + "," + slots[count - 1 - i] + ")";
        }
        return slots[0];
    }
};
C
C
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

char* bracketString(int n) {
    char** slots = (char**)malloc(sizeof(char*) * n);
    for (int i = 0; i < n; i++) {
        slots[i] = (char*)malloc(8);          /* seeds up to 4096 -> <= 4 digits */
        sprintf(slots[i], "%d", i + 1);
    }
    for (int count = n; count > 1; count /= 2) {
        int half = count / 2;
        for (int i = 0; i < half; i++) {
            char* a = slots[i];
            char* b = slots[count - 1 - i];
            char* merged = (char*)malloc(strlen(a) + strlen(b) + 4); /* '(' ',' ')' '\0' */
            sprintf(merged, "(%s,%s)", a, b);
            free(a);
            free(b);
            slots[i] = merged;
        }
    }
    char* result = slots[0];
    free(slots);
    return result;   /* caller frees */
}

#### Follow-up
questions
Rewrite the solution recursively (pair a list with its reverse, recurse on the halved list) and compare with the iterative version.
Given a finished bracket string, parse it back into the round-by-round matchups.
Generalize to an n that is not a power of two by introducing byes (some slots advance without a match).
Print each round's pairings separately instead of only the final nested string.

