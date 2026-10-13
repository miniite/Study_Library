---
type: "practice"
difficulty: "Other"
---

# Validate a String Built by Inserting "xyz" Blocks

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Start with an empty string t = "". You are allowed to perform the following operation any number of times (including zero):

Choose any position in the current string t and insert the three-character block "xyz" there. Formally, t becomes t_left + "xyz" + t_right, where t == t_left + t_right. Both t_left and t_right may be empty, so the block can be inserted at the very start, the very end, or anywhere in between.

Given a string s, decide whether it can be produced from the empty string by some sequence of these insertions. Return true if s is reachable this way and false otherwise.

Input: a string s made up only of the characters 'x', 'y', and 'z'. Output: a boolean — true if s is valid, false otherwise.

#### Cons
traints:

1 <= s.length <= 2 * 10^4
s consists only of the letters 'x', 'y', and 'z'.

#### Example
s & test cases
TEXT
Input: s = "xyz" -> Output: true
  Why: a single insertion of the block into the empty string produces exactly "xyz".

Input: s = "xxyzyz" -> Output: true
  Why: "" -> "xyz" -> insert another block after the first char: "x" + "xyz" + "yz" = "xxyzyz".

Input: s = "xyxyzzxyz" -> Output: true
  Why: nested insertions collapse cleanly — scanning left to right, every 'z' closes a preceding "xy", leaving nothing over.

Input: s = "xyzzyx" -> Output: false
  Why: after the first "xyz" cancels, the next character is 'z' with fewer than two symbols before it, so it can never have been part of a valid block.

Input: s = "x" -> Output: false
  Why: an 'x' with no matching 'y' and 'z' cannot be part of any complete inserted block.

Input: s = "zyx" -> Output: false
  Why: the leading 'z' has nothing before it, so no block could have placed it there.

#### Approach

Stack-based matching / reduction. Every valid string reduces to empty by repeatedly deleting an "xyz" substring, so the check is the mirror image: scan left to right pushing non-'z' characters onto a stack. When a 'z' arrives, the only way it could have completed an inserted block is if the two characters directly beneath it are 'x' then 'y' (in stack order, top is 'y', next is 'x'); pop both to "cancel" that block. If a 'z' appears without a valid "xy" pair on top, the string is unreachable, so return false immediately. At the end the string is valid iff the stack is empty. This is the classic balanced-brackets / adjacent- pattern-elimination idea generalized from a single symbol pair to a fixed triple.

Complexity

Time: O(n), where n is the length of s — each character is pushed at most once and popped at most once. Space: O(n) in the worst case for the stack (e.g. a long run of 'x' characters before any 'z').

Python
PYTHON
class Solution:
    def isValidSequence(self, s: str) -> bool:
        stack = []
        for c in s:
            if c == 'z':
                if len(stack) < 2 or stack[-1] != 'y' or stack[-2] != 'x':
                    return False
                stack.pop()
                stack.pop()
            else:
                stack.append(c)
        return not stack
JavaScript
JAVASCRIPT
function isValidSequence(s) {

#### cons
t stack = [];
    for (const c of s) {
        if (c === 'z') {

#### cons
t n = stack.length;
            if (n < 2 || stack[n - 1] !== 'y' || stack[n - 2] !== 'x') return false;
            stack.pop();
            stack.pop();
        } else {
            stack.push(c);
        }
    }
    return stack.length === 0;
}
Java
JAVA
import java.util.*;

class Solution {
  public boolean isValidSequence(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
      if (c == 'z') {
        if (stack.size() < 2) return false;
        if (stack.pop() != 'y') return false;
        if (stack.pop() != 'x') return false;
      } else {
        stack.push(c);
      }
    }
    return stack.isEmpty();
  }
}
C++
CPP
#include <string>
using namespace std;

class Solution {
 public:
  bool isValidSequence(string s) {
    string stack;
    for (char c : s) {
      if (c == 'z') {
        int n = (int)stack.size();
        if (n < 2 || stack[n - 1] != 'y' || stack[n - 2] != 'x')
          return false;
        stack.pop_back();
        stack.pop_back();
      } else {
        stack.push_back(c);
      }
    }
    return stack.empty();
  }
};
C
C
#include <stdbool.h>
#include <string.h>
#include <stdlib.h>

bool isValidSequence(char* s) {
    int n = (int)strlen(s);
    char* stack = (char*)malloc(n > 0 ? n : 1);
    int top = 0;

    for (int i = 0; i < n; i++) {
        char c = s[i];
        if (c == 'z') {
            if (top < 2 || stack[top - 1] != 'y' || stack[top - 2] != 'x') {
                free(stack);
                return false;
            }
            top -= 2;
        } else {
            stack[top++] = c;
        }
    }

    bool ok = (top == 0);
    free(stack);
    return ok;
}

#### Follow-up
questions
Solve it in O(1) extra space (no explicit stack) by tracking counts of open 'x' and pending "xy" pairs instead.
Generalize the inserted block from the fixed triple to an arbitrary given pattern P — what invariant must hold?
What if two or more different blocks could be inserted; how would you validate then?
Beyond a yes/no answer, reconstruct one valid sequence of insertion positions that produces s.

