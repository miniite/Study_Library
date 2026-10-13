---
type: "practice"
difficulty: "Other"
---

# Swap Adjacent Slot Assignments

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Database.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A hall has a single row of numbered slots, and every slot is occupied by exactly one participant. The slots are numbered consecutively starting at 1, with no gaps.

Table: Booth

Column Name	Type
slot	int
occupant	varchar

slot is the primary key (unique value). Each row gives the slot number and the name of the participant sitting there. The slot sequence always starts from 1 and increases by exactly 1 with no missing numbers.

Rearrange the seating so that the occupants of every two consecutive slots trade places: the occupant of slot 1 swaps with slot 2, slot 3 swaps with slot 4, and so on. If the total number of participants is odd, the participant in the very last slot keeps their slot (there is no partner to swap with).

Return the resulting table ordered by slot in ascending order, with columns slot and occupant.

#### Cons
traints

1 <= number of rows (n) <= 10^4.
Slot numbers are exactly 1, 2, ..., n (continuous, starting at 1).
occupant is a non-empty string of at most 50 characters.
Only the assignment of occupants to slots changes; the set of participants and the set of slot numbers stay the same.

#### Example
s & test cases
TEXT
Input:
Booth:
| slot | occupant |
| 1 | Nadia |
| 2 | Owen  |
| 3 | Priya |
| 4 | Ravi  |
| 5 | Sana  |
-> Output:
| slot | occupant |
| 1 | Owen  |
| 2 | Nadia |
| 3 | Ravi  |
| 4 | Priya |
| 5 | Sana  |
Why: Slots (1,2) and (3,4) swap occupants; there are 5 (odd) participants so slot 5 is left unchanged.

Input:
Booth: | 1 | Ana | ; | 2 | Beto | ; | 3 | Cai | ; | 4 | Dina |
-> Output: | 1 | Beto | ; | 2 | Ana | ; | 3 | Dina | ; | 4 | Cai |
Why: Even count, so every pair swaps and no one is left over.

Input:
Booth: | 1 | Solo |
-> Output: | 1 | Solo |
Why: A single participant is the odd last slot — nothing to swap.

Input:
Booth: | 1 | Xavi | ; | 2 | Yara |
-> Output: | 1 | Yara | ; | 2 | Xavi |
Why: The minimal swap case — one pair trades places.

Input:
Booth: | 1 | Pam | ; | 2 | Quill | ; | 3 | Remy |
-> Output: | 1 | Quill | ; | 2 | Pam | ; | 3 | Remy |
Why: Only the first pair swaps; the odd third slot keeps its occupant.

#### Approach

The pattern is pairwise swapping / index remapping. Because slots are numbered continuously from 1, "swap every two consecutive slots" means: an occupant in an odd slot s moves to s + 1, an occupant in an even slot s moves to s - 1, and the single unpaired occupant (an odd slot equal to the maximum slot, which only happens when the count is odd) stays put. In array terms, indexing occupants by slot - 1, this is simply swapping elements at positions (0,1), (2,3), (4,5), ...; a loop that steps by 2 and stops before the last element automatically leaves an odd final element alone.

In SQL the same slot remapping is expressed with a CASE on the parity of slot, guarding the last odd slot with a subquery for the maximum:

SQL
SELECT
    CASE
        WHEN slot % 2 = 1 AND slot = (SELECT MAX(slot) FROM Booth) THEN slot
        WHEN slot % 2 = 1 THEN slot + 1
        ELSE slot - 1
    END AS slot,
    occupant
FROM Booth
ORDER BY slot;

The first branch keeps the trailing odd slot, the second bumps other odd slots up by one, and the ELSE pulls every even slot down by one; ordering by the recomputed slot yields the swapped seating. An equivalent window-function form pairs each row with its neighbor via LEAD/LAG, but the parity-CASE approach is the clearest.

Complexity

Time: O(n) — a single pass touches each slot once, swapping in steps of two. Space: O(1) extra — the swap is done in place (the array/list is reused; no auxiliary structure grows with n). The canonical SQL version is one table scan that recomputes each row's slot with a CASE expression, plus an O(n log n) sort to emit rows in ascending slot order.

Python
PYTHON
from typing import List

def exchange_seats(names: List[str]) -> List[str]:
    # names[i] is the occupant of slot i+1; swap every adjacent pair.
    # An odd-length list leaves its final element in place.
    for i in range(0, len(names) - 1, 2):
        names[i], names[i + 1] = names[i + 1], names[i]
    return names
JavaScript
JAVASCRIPT
// names[i] is the occupant of slot i+1; swaps every adjacent pair in place.
// If the length is odd, the last slot keeps its occupant.
function exchangeSeats(names) {
    for (let i = 0; i + 1 < names.length; i += 2) {
        [names[i], names[i + 1]] = [names[i + 1], names[i]];
    }
    return names;
}
Java
JAVA
import java.util.*;

class Solution {
    // names.get(i) is the occupant of slot i+1; swaps every adjacent pair in place.
    // If the size is odd, the last slot keeps its occupant.
    public List<String> exchangeSeats(List<String> names) {
        for (int i = 0; i + 1 < names.size(); i += 2)
            Collections.swap(names, i, i + 1);
        return names;
    }
}
C++
CPP
#include <vector>
#include <string>
#include <utility>
using namespace std;

// occupants[i] is the participant in slot i+1; after the call it is the new occupant.
// Swaps every adjacent pair; an odd final slot is left untouched.
void exchangeSeats(vector<string>& occupants) {
    for (size_t i = 0; i + 1 < occupants.size(); i += 2)
        swap(occupants[i], occupants[i + 1]);
}
C
C
#include <stddef.h>

/* occupants[i] is the participant sitting in slot i+1 (slots are 1..n).
   Rearranges in place so occupants[i] becomes the NEW occupant of slot i+1,
   swapping every adjacent pair; if n is odd the final slot is unchanged. */
void exchangeSeats(char** occupants, int n) {
    for (int i = 0; i + 1 < n; i += 2) {
        char* tmp = occupants[i];
        occupants[i] = occupants[i + 1];
        occupants[i + 1] = tmp;
    }
}

#### Follow-up
questions
Solve it without the MAX(slot) subquery — for example using a window function (LEAD/LAG) or the bit trick new_slot = ((slot - 1) XOR 1) + 1 with a special case for the last row.
What changes if slot numbers are NOT guaranteed continuous (there are gaps)? How do you still pair "adjacent" occupants?
Generalize to swapping in rotating groups of three (1->2->3->1) instead of pairs.
Do it as an in-place UPDATE of the table rather than a SELECT that returns a reordered copy.

