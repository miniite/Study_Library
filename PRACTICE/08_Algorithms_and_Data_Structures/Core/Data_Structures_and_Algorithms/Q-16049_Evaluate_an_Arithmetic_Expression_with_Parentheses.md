---
type: "practice"
difficulty: "Other"
---

# Evaluate an Arithmetic Expression with Parentheses

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a string expr that holds a valid arithmetic expression. Implement your own evaluator that computes the integer value of the expression and returns it.

The expression may contain:

non-negative integers (one or more decimal digits),
the binary operators + (addition) and - (subtraction),
parentheses ( and ) used for grouping,
and space characters, which should be ignored.

You must not use any built-in facility that parses and evaluates a string as an expression (for example an eval-style function). You must compute the result yourself.

Notes on the operators:

+ is never used as a unary sign (so "+3" or "+(2 + 3)" will not appear).
- may be used as a unary sign, so "-4" and "-(2 + 3)" are valid.
The input never contains two consecutive operators.
Input

A single string expr describing a valid expression.

Output

A single integer: the evaluated result.

#### Cons
traints
1 <= expr.length <= 3 * 10^5
expr consists only of the characters: digits 0–9, '+', '-', '(', ')', and ' '.
expr is guaranteed to be a valid expression.
Every intermediate number and every running partial result fits in a signed 32-bit integer.

#### Example
s & test cases
TEXT
Input: expr = "3 + 5" -> Output: 8
Why: straightforward addition with surrounding spaces ignored.

Input: expr = " 10-2 + 4 " -> Output: 12
Why: multi-digit numbers and mixed +/- with leading/trailing/internal spaces; 10-2+4 = 12.

Input: expr = "(7+(2+6+1)-4)+(3+9)" -> Output: 24
Why: nested parentheses; inner (2+6+1)=9, then 7+9-4=12, plus (3+9)=12, total 24.

Input: expr = "-(4+2)" -> Output: -6
Why: unary minus applied to a whole parenthesized group flips the sign of 4+2.

Input: expr = "2-(5-6)" -> Output: 3
Why: subtracting a group that itself evaluates to -1 gives 2-(-1)=3 (sign propagation across parentheses).

Input: expr = "0" -> Output: 0
Why: minimal single-digit expression; validates the flush-at-end of the last accumulated number.

#### Approach

Scan the string once, building each multi-digit number digit by digit and maintaining a running result plus the sign (+1 or -1) that applies to the number currently being read. Whenever a + or - is hit, the fully-read previous number is committed into result (result += sign * num) and the new sign is set. The key trick for parentheses is a stack that stores the effective sign of the enclosing context: when ( is seen, push the current sign (which is the sign that multiplies the whole group); every operator inside that group multiplies its own +/- by the top of the stack, so a leading -( correctly negates everything within. On ), pop back to the outer context. After the loop, flush the last accumulated number. This is a single-pass stack-based evaluation that avoids explicit recursion and needs no operator-precedence handling because only + and - are present.

Complexity

Time: O(n) where n = expr.length — each character is processed exactly once with O(1) work. Space: O(d) where d is the maximum parenthesis nesting depth (O(n) in the worst case), for the sign stack.

Python
PYTHON
class Solution:
    def evaluate(self, expr: str) -> int:
        result = 0
        num = 0
        sign = 1
        stack = [1]  # top = effective sign of the current parenthesis context

        for c in expr:
            if c.isdigit():
                num = num * 10 + (ord(c) - ord('0'))
            elif c == '+' or c == '-':
                result += sign * num
                num = 0
                sign = (1 if c == '+' else -1) * stack[-1]
            elif c == '(':
                stack.append(sign)
            elif c == ')':
                stack.pop()

        return result + sign * num
JavaScript
JAVASCRIPT
function evaluate(expr) {
    let result = 0, num = 0, sign = 1;

#### cons
t stack = [1]; // top = effective sign of the current parenthesis context

    for (const c of expr) {
        if (c >= '0' && c <= '9') {
            num = num * 10 + (c.charCodeAt(0) - 48);
        } else if (c === '+' || c === '-') {
            result += sign * num;
            num = 0;
            sign = (c === '+' ? 1 : -1) * stack[stack.length - 1];
        } else if (c === '(') {
            stack.push(sign);
        } else if (c === ')') {
            stack.pop();
        }
    }
    return result + sign * num;
}
Java
JAVA
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int evaluate(String expr) {
        int result = 0, num = 0, sign = 1;
        Deque<Integer> ctx = new ArrayDeque<>(); // sign of each enclosing () context
        ctx.push(1);

        for (int i = 0; i < expr.length(); i++) {
            char c = expr.charAt(i);
            if (c >= '0' && c <= '9') {
                num = num * 10 + (c - '0');
            } else if (c == '+' || c == '-') {
                result += sign * num;
                num = 0;
                sign = (c == '+' ? 1 : -1) * ctx.peek();
            } else if (c == '(') {
                ctx.push(sign);
            } else if (c == ')') {
                ctx.pop();
            }
        }
        return result + sign * num;
    }
}
C++
CPP
#include <string>
#include <stack>
#include <cctype>
using namespace std;

class Solution {
public:
    int evaluate(const string& expr) {
        int result = 0, num = 0, sign = 1;
        stack<int> ctx;   // effective sign of each enclosing () context
        ctx.push(1);

        for (char c : expr) {
            if (isdigit(static_cast<unsigned char>(c))) {
                num = num * 10 + (c - '0');
            } else if (c == '+' || c == '-') {
                result += sign * num;
                num = 0;
                sign = (c == '+' ? 1 : -1) * ctx.top();
            } else if (c == '(') {
                ctx.push(sign);
            } else if (c == ')') {
                ctx.pop();
            }
        }
        return result + sign * num;
    }
};
C
C
#include <stdio.h>
#include <stdlib.h>

int evaluate(const char *expr) {
    int result = 0, num = 0, sign = 1;

    /* Stack holds the effective sign of each enclosing () context. */
    int depth = 0;
    for (const char *q = expr; *q; ++q) if (*q == '(') depth++;
    int *stack = (int *)malloc(sizeof(int) * (depth + 2));
    int top = 0;
    stack[top++] = 1;               /* outermost context sign */

    for (const char *p = expr; *p; ++p) {
        char c = *p;
        if (c >= '0' && c <= '9') {
            num = num * 10 + (c - '0');
        } else if (c == '+' || c == '-') {
            result += sign * num;
            num = 0;
            sign = (c == '+' ? 1 : -1) * stack[top - 1];
        } else if (c == '(') {
            stack[top++] = sign;    /* sign that multiplies everything inside */
        } else if (c == ')') {
            --top;
        }
        /* spaces and any other char are ignored */
    }

    result += sign * num;           /* flush the trailing number */
    free(stack);
    return result;
}

int main(void) {
    printf("%d\n", evaluate("(7+(2+6+1)-4)+(3+9)")); /* 24 */
    return 0;
}

#### Follow-up
questions
Extend the evaluator to also support * and / with correct operator precedence.
Support floating-point numbers and stacked unary operators such as --5.
Convert the expression to postfix (Reverse Polish Notation) first, then evaluate the RPN with a value stack.
Given a possibly malformed expression, detect invalid syntax and report the position of the first error.

