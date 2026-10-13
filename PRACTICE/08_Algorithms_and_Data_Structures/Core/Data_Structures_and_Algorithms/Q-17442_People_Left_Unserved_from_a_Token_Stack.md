---
type: "practice"
difficulty: "Other"
---

# People Left Unserved from a Token Stack

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A kiosk hands out tokens of two kinds, labeled 0 and 1. The available tokens sit in a single vertical stack, and a group of people waits in a queue to receive one. Each person accepts only a token of the kind they want (either 0 or 1) and refuses the other.

The number of tokens equals the number of people. Service proceeds in steps:

Look at the person at the front of the queue and the token on top of the stack.
If the front person wants that token's kind, they take it and leave the queue (that token is removed from the stack).
Otherwise, the front person declines and walks to the back of the queue; the token stays on top.

This repeats until every person still in the queue declines the current top token (no one remaining wants it). Those people can never be served.

You are given two integer arrays:

wants, where wants[j] is the token kind the j-th person in the queue wants (j = 0 is the front).
stack, where stack[i] is the kind of the i-th token in the stack (i = 0 is the top).

Return the number of people who remain unserved.

Input

wants: array of 0/1 values (queue order, front first).
stack: array of 0/1 values (stack order, top first). Same length as wants.

Output

A single integer: how many people never receive a token.

#### Cons
traints

1 <= wants.length, stack.length <= 100
wants.length == stack.length
Each element of wants and stack is either 0 or 1.

#### Example
s & test cases
TEXT
- Input: `wants = [1,1,0,0]`, `stack = [0,1,0,1]` -> Output: `0`
  Why: the counts of wanted `0`s and `1`s match the supply exactly, so everyone eventually gets served.
- Input: `wants = [1,1,1,0,0,1]`, `stack = [1,0,0,0,1,1]` -> Output: `3`
  Why: after some tokens are handed out, the top becomes a `0` that no remaining person wants; the 3 people still waiting are stuck.
- Input: `wants = [0]`, `stack = [1]` -> Output: `1`
  Why: the only person wants `0` but the only token is `1` — an immediate deadlock (smallest input).
- Input: `wants = [1,1,1]`, `stack = [1,1,1]` -> Output: `0`
  Why: all wants and all tokens are the same kind, so each person is served in turn.
- Input: `wants = [0,0]`, `stack = [1,1]` -> Output: `2`
  Why: nobody wants a `1`, so the top token blocks the whole queue and both people leave unserved.

#### Approach

The crucial observation is that the order of people in the queue does not matter. A person who declines simply cycles to the back without changing anyone's preference, so for any token on top of the stack it will be claimed by some remaining person of the matching kind if one exists — otherwise it blocks everyone. That reduces the whole rotating-queue process to plain counting: tally how many people want kind 0 and how many want kind 1. Then walk the stack top to bottom; each token of kind t is consumed by one waiting person of that kind (decrement count[t]). The instant a token's kind has zero remaining wanters, the process deadlocks — every person still in line refuses this and all further tokens beneath a compatible one are unreachable — so the answer is the number of people left, count[0] + count[1]. If the stack is fully consumed, both counters reach zero and the answer is 0. This is a counting / greedy-simulation pattern that replaces an O(n^2) literal simulation with an O(n) tally.

Complexity

Time: O(n) — one pass to tally how many people want each kind, then one pass over the stack that stops as soon as the top token is unwanted. Space: O(1) — only two running counters are kept, independent of n. Justification: the queue's internal order is irrelevant, so no explicit simulation of the rotating line is needed.

Python
PYTHON
from typing import List

class Solution:
    def countUnserved(self, wants: List[int], stack: List[int]) -> int:
        count = [0, 0]                     # count[k] = people still wanting kind k
        for w in wants:
            count[w] += 1
        for t in stack:
            if count[t] == 0:              # no remaining person wants the top token
                break
            count[t] -= 1
        return count[0] + count[1]
JavaScript
JAVASCRIPT
function countUnserved(wants, stack) {

#### cons
t count = [0, 0];                 // count[k] = people still wanting kind k
    for (const w of wants) count[w]++;
    for (const t of stack) {
        if (count[t] === 0) break;        // top token wanted by no one remaining
        count[t]--;
    }
    return count[0] + count[1];
}
Java
JAVA
class Solution {
  public int countUnserved(int[] wants, int[] stack) {
    int[] count = new int[2];          // count[k] = people still wanting kind k
    for (int w : wants) count[w]++;
    for (int t : stack) {
      if (count[t] == 0) break;        // nobody remaining wants the top token
      count[t]--;
    }
    return count[0] + count[1];
  }
}
C++
CPP
#include <vector>
using namespace std;

class Solution {
 public:
  int countUnserved(vector<int>& wants, vector<int>& stack) {
    int count[2] = {0, 0};
    for (int w : wants) count[w]++;
    for (int t : stack) {
      if (count[t] == 0) break;   // top token wanted by no one remaining
      count[t]--;
    }
    return count[0] + count[1];
  }
};
C
C
/* wants and stack have the same length n. */
int countUnserved(int* wants, int wantsSize, int* stack, int stackSize) {
    int count[2] = {0, 0};                 /* how many people want kind 0 and kind 1 */
    for (int i = 0; i < wantsSize; ++i) count[wants[i]]++;
    for (int i = 0; i < stackSize; ++i) {
        int t = stack[i];
        if (count[t] == 0) break;          /* nobody left wants this top token -> deadlock */
        count[t]--;
    }
    return count[0] + count[1];
}

#### Follow-up
questions
Prove rigorously why the queue's ordering can be ignored (why counting equals the literal simulation).
Generalize to k token kinds instead of just two — how do the counters and the deadlock check change?
Also report which specific people go unserved, not just how many.
If each person had a bounded patience (leaves after declining p times), how would the answer change?

