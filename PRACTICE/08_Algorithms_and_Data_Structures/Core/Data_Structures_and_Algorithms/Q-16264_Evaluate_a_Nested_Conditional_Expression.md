---
type: "practice"
difficulty: "Other"
---

# Evaluate a Nested Conditional Expression

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string expr that encodes an arbitrarily nested conditional (if/else) expression, written with the same syntax as the condition ? then_value : else_value operator found in many programming languages. Evaluate it and return the resulting single character.

The string is built from exactly these symbols:

'T' stands for the boolean true and 'F' stands for the boolean false.
A single digit '0'–'9' is a literal value.
'?' separates a condition from its "true" branch, and ':' separates the "true" branch from the "false" branch.

Formally, an expression is either a single symbol (a digit, 'T', or 'F'), or has the form C ? X : Y where C is 'T' or 'F' and X and Y are themselves expressions. When C is 'T' the whole expression evaluates to the value of X; when C is 'F' it evaluates to the value of Y. Every number that appears is a single digit.

The conditional operator is right-associative: it groups from right to left. So a string like A ? a : B ? b : c must be read as A ? a : (B ? b : c), not as (A ? a : B) ? b : c. The final result is always a single character — a digit, 'T', or 'F'.

Input: a valid conditional-expression string expr.

Output: the single-character result of evaluating it (returned as a length-1 string).

#### Cons
traints:

1 <= expr.length <= 10^4, and the length is always odd.
expr consists only of the characters '0'–'9', 'T', 'F', '?', and ':'.
expr is guaranteed to be a well-formed conditional expression, and every number in it is a single digit.

#### Example
s & test cases
TEXT
Input: expr = "T?7:2" -> Output: "7"
Why: the condition is true, so the expression takes its "true" branch, the digit 7.

Input: expr = "F?8:2" -> Output: "2"
Why: the condition is false, so it takes the "false" branch, the digit 2.

Input: expr = "F?8:T?5:6" -> Output: "5"
Why: right-associative grouping reads it as F ? 8 : (T ? 5 : 6); the outer condition is false, so the value is the inner T ? 5 : 6, which is 5.

Input: expr = "T?T?4:9:1" -> Output: "4"
Why: grouped as T ? (T ? 4 : 9) : 1; the outer condition is true, so the value is the inner T ? 4 : 9, which is 4.

Input: expr = "T?T:F" -> Output: "T"
Why: minimal nesting where a branch value is itself a boolean; true condition selects the "true" branch, the character T.

Input: expr = "F?9:T?F:2" -> Output: "F"
Why: grouped as F ? 9 : (T ? F : 2); the outer false condition selects the inner T ? F : 2, which yields the boolean F.

#### Approach

Scan the string from right to left, pushing each character onto a stack. Because the operator is right-associative, the moment the current character is a condition ('T'/'F') whose immediate right neighbor was '?', that '?' is on top of the stack and the fully-reduced "true" branch, the ':', and the fully-reduced "false" branch sit directly beneath it in known order. Pop '?', the true value, ':', and the false value; then push back the true value if the condition is 'T', otherwise the false value. Non-condition characters (and conditions not yet followed by a '?' on the stack) are simply pushed. When the scan ends, exactly one symbol remains on the stack — the answer. The underlying pattern is stack-based expression evaluation; an equivalent formulation is recursive descent with a moving index that greedily parses each sub-expression.

Complexity

Time: O(n), where n is the length of the string — each character is pushed once and, across the whole run, popped at most a constant number of times, so the single right-to-left pass is linear. Space: O(n) for the stack, which in the worst case (a long right-leaning chain) holds a constant fraction of the characters at once.

Python
PYTHON
class Solution:
    def evaluate(self, expr: str) -> str:
        stack = []
        for c in reversed(expr):
            if stack and stack[-1] == '?':
                stack.pop()               # discard '?'
                first = stack.pop()       # "true" branch value
                stack.pop()               # discard ':'
                second = stack.pop()      # "false" branch value
                stack.append(first if c == 'T' else second)
            else:
                stack.append(c)
        return stack[-1]
JavaScript
JAVASCRIPT
function evaluate(expr) {

#### cons
t stack = [];
    for (let i = expr.length - 1; i >= 0; i--) {

#### cons
t c = expr[i];
        if (stack.length && stack[stack.length - 1] === '?') {
            stack.pop();                  // discard '?'

#### cons
t first = stack.pop();    // "true" branch value
            stack.pop();                  // discard ':'

#### cons
t second = stack.pop();   // "false" branch value
            stack.push(c === 'T' ? first : second);
        } else {
            stack.push(c);
        }
    }
    return stack[stack.length - 1];
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public String evaluate(String expr) {
        Deque<Character> st = new ArrayDeque<>();
        for (int i = expr.length() - 1; i >= 0; --i) {
            char c = expr.charAt(i);
            if (!st.isEmpty() && st.peek() == '?') {
                st.pop();                 // discard '?'
                char first = st.pop();    // "true" branch value
                st.pop();                 // discard ':'
                char second = st.pop();   // "false" branch value
                st.push(c == 'T' ? first : second);
            } else {
                st.push(c);
            }
        }
        return String.valueOf(st.peek());
    }
}
C++
CPP
#include <string>
#include <stack>
using namespace std;

class Solution {
public:
    string evaluate(string expr) {
        stack<char> st;
        for (int i = (int)expr.size() - 1; i >= 0; --i) {
            char c = expr[i];
            if (!st.empty() && st.top() == '?') {
                st.pop();                 // discard '?'
                char first = st.top();  st.pop();   // "true" branch value
                st.pop();                 // discard ':'
                char second = st.top(); st.pop();   // "false" branch value
                st.push(c == 'T' ? first : second);
            } else {
                st.push(c);
            }
        }
        return string(1, st.top());
    }
};
C
C
#include <stdlib.h>
#include <string.h>

/* Returns a freshly allocated length-1 string holding the evaluated symbol. */
char* evaluate(const char* expr) {
    int n = (int)strlen(expr);
    char* stack = (char*)malloc((n + 1) * sizeof(char));
    int top = -1;

    for (int i = n - 1; i >= 0; --i) {
        char c = expr[i];
        if (top >= 0 && stack[top] == '?') {
            top--;                        /* discard '?'                          */
            char first  = stack[top--];   /* value taken when condition is true   */
            top--;                        /* discard ':'                          */
            char second = stack[top--];   /* value taken when condition is false  */
            stack[++top] = (c == 'T') ? first : second;
        } else {
            stack[++top] = c;
        }
    }

    char* res = (char*)malloc(2 * sizeof(char));
    res[0] = stack[top];
    res[1] = '\0';
    free(stack);
    return res;
}

#### Follow-up
questions
Extend the parser to support multi-digit numeric literals instead of only single digits.
Allow variables and simple boolean sub-expressions (e.g., x & y) as conditions rather than just 'T'/'F'.
Solve it with recursion and a single moving index instead of an explicit stack, and compare the memory profile.
Add validation: detect and report malformed input rather than assuming the expression is always well-formed.

