---
type: "practice"
difficulty: "Other"
---

# Validate a String of Nested Brackets

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string text made up only of the six bracket characters (, ), {, }, [, and ]. Decide whether the string is a well-formed (balanced) bracket sequence.

The string is well-formed if all of the following hold:

Every opening bracket is eventually closed by a closing bracket of the same type.
Brackets are closed in the correct nested order — the most recently opened bracket that is still open must be the next one closed.
Every closing bracket matches an earlier, still-open bracket of the same type (no stray closers).

Return true if the string is well-formed and false otherwise.

Input

text: a string consisting only of the characters (, ), {, }, [, ].

Output

A boolean: true if the bracket sequence is balanced and correctly nested, false otherwise.

#### Cons
traints

1 <= text.length <= 10^4
text contains only the characters (, ), {, }, [, ].

#### Example
s & test cases
TEXT
Input: text = "{}" -> Output: true
Why: a single matched pair of the same type (simplest valid case).

Input: text = "[]{}()" -> Output: true
Why: three independent, correctly matched pairs placed side by side (valid, non-nested).

Input: text = "{[()]}" -> Output: true
Why: properly nested brackets of different types close in reverse order of opening.

Input: text = "(]" -> Output: false
Why: the closer does not match the type of the last opener (type mismatch).

Input: text = "([)]" -> Output: false
Why: the pairs interleave instead of nesting, so the closing order is wrong.

Input: text = ")" -> Output: false
Why: a lone closing bracket has no matching opener (stray closer / empty stack).

#### Approach

Use a stack to track the openers that are still waiting to be closed. Scan the string left to right: push every opening bracket; on a closing bracket, the most recently pushed opener must match it in type, so pop and compare. If the stack is empty when a closer arrives (stray closer) or the popped opener is the wrong type (mismatch), the string is invalid. After the scan, the string is valid only if the stack is empty — any leftover openers were never closed. This "last-opened, first-closed" requirement is exactly LIFO behavior, which is why a stack is the natural fit; a hash map from each closer to its expected opener keeps the matching logic concise.

Complexity

Time: O(n) — each character is processed once with O(1) stack operations, where n is the string length. Space: O(n) — in the worst case (all openers) the stack holds every character before any is matched.

Python
PYTHON
def is_balanced(text: str) -> bool:
    match = {')': '(', ']': '[', '}': '{'}
    stack = []
    for c in text:
        if c in '([{':
            stack.append(c)                # push opener
        else:
            # stack empty (stray closer) or wrong type => invalid
            if not stack or stack.pop() != match[c]:
                return False
    return not stack                       # leftover openers => invalid
JavaScript
JAVASCRIPT
function isBalanced(text) {

#### cons
t match = { ')': '(', ']': '[', '}': '{' };

#### cons
t stack = [];
    for (const c of text) {
        if (c === '(' || c === '{' || c === '[') {
            stack.push(c);                 // push opener
        } else {
            // pop() on an empty array is undefined, which never equals an opener
            if (stack.pop() !== match[c]) return false;
        }
    }
    return stack.length === 0;             // leftover openers => invalid
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public boolean isBalanced(String text) {
        Deque<Character> stack = new ArrayDeque<>();
        for (char c : text.toCharArray()) {
            if (c == '(' || c == '{' || c == '[') {
                stack.push(c);
            } else {
                if (stack.isEmpty()) return false;          // stray closer
                char open = stack.pop();
                if ((c == ')' && open != '(') ||
                    (c == '}' && open != '{') ||
                    (c == ']' && open != '[')) return false; // type mismatch
            }
        }
        return stack.isEmpty();                             // no unclosed openers
    }
}
C++
CPP
#include <string>
#include <stack>
using namespace std;

class Solution {
public:
    bool isBalanced(string text) {
        stack<char> st;
        for (char c : text) {
            if (c == '(' || c == '{' || c == '[') {
                st.push(c);
            } else {
                if (st.empty()) return false;          // stray closer
                char open = st.top();
                st.pop();
                if ((c == ')' && open != '(') ||
                    (c == '}' && open != '{') ||
                    (c == ']' && open != '[')) return false;  // type mismatch
            }
        }
        return st.empty();                             // no unclosed openers
    }
};
C
C
#include <stdbool.h>
#include <string.h>
#include <stdlib.h>

bool isBalanced(char* text) {
    int n = (int)strlen(text);
    char* stack = (char*)malloc(n + 1);
    int top = 0;

    for (int i = 0; i < n; i++) {
        char c = text[i];
        if (c == '(' || c == '{' || c == '[') {
            stack[top++] = c;                 /* push opener */
        } else {
            if (top == 0) { free(stack); return false; }  /* stray closer */
            char open = stack[--top];         /* pop matching opener */
            if ((c == ')' && open != '(') ||
                (c == '}' && open != '{') ||
                (c == ']' && open != '[')) {
                free(stack);
                return false;                 /* type mismatch */
            }
        }
    }

    bool ok = (top == 0);                     /* leftover openers => invalid */
    free(stack);
    return ok;
}

#### Follow-up
questions
How would you extend this to also allow arbitrary non-bracket characters in the string (ignore them but still validate the brackets)?
If only one bracket type were used, could you validate it with O(1) extra space using a counter instead of a stack?
How would you return the index of the first offending bracket instead of just true/false?
Could you determine the minimum number of insertions needed to make an unbalanced string valid?

