---
type: "practice"
difficulty: "Other"
---

# Fewest Suffix Flips to Brighten Every Panel

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Dynamic Programming](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Dynamic_Programming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a strip of n light panels in a row, indexed from left to right. Each panel is either dim (0) or bright (1). The strip is given as an array panels whose entries are all 0 or 1.

You may repeat the following action any number of times (including zero):

Pick any index i and flip every panel from index i through the last index (a suffix). Flipping toggles each chosen panel: 0 becomes 1 and 1 becomes 0.

Return the minimum number of actions needed to make every panel bright (all values equal to 1). Unlike a fixed-width move, a suffix flip can always eventually brighten everything, so an answer always exists.

Input

panels: an array of n integers, each either 0 or 1.

Output

The minimum number of suffix flips required to turn every panel bright.

#### Cons
traints

1 <= n <= 10^5
Each panels[i] is 0 or 1.

#### Example
s & test cases
TEXT
- Input: `panels = [1,0,0,1,0] -> Output: 3`
  Why: reading left to right, the state changes require a flip at the first dim panel, another where brightness would otherwise break, and a final one at the end — three flips in total (normal case with several transitions).
- Input: `panels = [0,1,1,1] -> Output: 2`
  Why: flip the whole suffix from index 0, then from index 1; every panel becomes bright in two flips.
- Input: `panels = [0,0,0,0,0] -> Output: 1`
  Why: all panels are dim, so one flip of the entire strip (suffix starting at index 0) brightens them all (single-flip edge case).
- Input: `panels = [1,1,1,1] -> Output: 0`
  Why: already all bright, so no action is needed (already-solved edge case).
- Input: `panels = [0] -> Output: 1`
  Why: the smallest strip with a single dim panel needs exactly one flip (minimum size).

#### Approach

Process panels left to right and commit each one before moving on. A suffix flip starting at index i toggles every panel from i onward, so a panel's true current state is its original value toggled by the number of flips already started to its left — only the parity of that count matters. Keep a running counter flips; the effective state of the current panel is value XOR (flips AND 1). If that effective state is dim (0), the only way to brighten this panel without ever revisiting the already-settled panels to its left is to flip the suffix that starts here, so increment flips. Because every later action affects this panel identically, this greedy choice is forced and optimal. Panels that are already effectively bright need no action. The total number of flips is the answer, and it always terminates in a valid all-bright state. This is a greedy / running-parity (prefix-XOR) technique.

Complexity

Time: O(n) — one left-to-right pass doing O(1) work per panel. Space: O(1) — only a running flip counter is kept. Justification: each panel is decided once using the parity of flips applied so far, with no extra storage.

Python
PYTHON
from typing import List

class Solution:
    def fewestSuffixFlips(self, panels: List[int]) -> int:
        flips = 0
        for v in panels:
            # effective state = value toggled once per flip already applied (parity)
            if (v ^ (flips & 1)) == 0:   # panel is effectively dim
                flips += 1               # flip the suffix starting here
        return flips
JavaScript
JAVASCRIPT
function fewestSuffixFlips(panels) {
    let flips = 0;
    for (const v of panels) {
        // effective state = value toggled once per flip already applied (parity)
        if ((v ^ (flips & 1)) === 0)  // panel is effectively dim
            flips++;                  // flip the suffix starting here
    }
    return flips;
}
Java
JAVA
class Solution {
  public int fewestSuffixFlips(int[] panels) {
    int flips = 0;
    for (int v : panels)
      if ((v ^ (flips & 1)) == 0)  // panel is effectively dim
        flips++;                   // flip the suffix starting here
    return flips;
  }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
 public:
  int fewestSuffixFlips(vector<int>& panels) {
    int flips = 0;
    for (int v : panels)
      if ((v ^ (flips & 1)) == 0)  // panel is effectively dim
        ++flips;                   // flip the suffix starting here
    return flips;
  }
};
C
C
int fewestSuffixFlips(int* panels, int n) {
    int flips = 0;
    for (int i = 0; i < n; i++) {
        /* effective state = panel toggled once per flip already applied (parity) */
        if ((panels[i] ^ (flips & 1)) == 0) /* panel i is effectively dim */
            flips++;                        /* flip the suffix starting here */
    }
    return flips;
}

#### Follow-up
questions
Show that the minimum flip count equals the number of adjacent state changes when a virtual bright panel is prepended to the left end.
Return the actual sequence of suffix start indices that achieves the minimum, not just the count.
What if a move could flip a prefix (from the start up to index i) instead of a suffix — does the strategy change?
Generalize to make every panel equal to a given target array rather than all-bright; how does the parity condition adapt?

