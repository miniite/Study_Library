---
type: "practice"
difficulty: "Other"
---

# Evaluate Expressions in a Scoped Prefix Mini-Language

**Difficulty:** Other · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursion](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

You are given a single string program that encodes one expression written in a tiny functional language. Evaluate it and return the resulting 32-bit integer.

The grammar of the language is:

An expression is one of: an integer literal, a variable reference, a let expression, an add expression, or a mult expression. Every expression evaluates to exactly one integer.
An integer literal may be positive or negative (for example 7 or -12).
A variable is a name that begins with a lowercase letter followed by zero or more lowercase letters or digits (for example k, t3, count). The three words let, add, and mult are reserved keywords and are never used as variable names.
An add expression has the form (add e1 e2) and evaluates to the sum of the values of the two sub-expressions e1 and e2.
A mult expression has the form (mult e1 e2) and evaluates to the product of the values of e1 and e2.
A let expression has the form (let v1 e1 v2 e2 ... vk ek body). It binds variable v1 to the value of e1, then v2 to the value of e2, and so on, in order — so a later value expression may use the variables bound before it. The value of the whole let is the value of the final body expression. There is always at least one variable/value pair before the body.

Scope. Bindings introduced by a let are visible only inside that let (its value expressions and its body), including any nested expressions. When a variable is referenced, its value is taken from the innermost enclosing scope that defines it; if it is not found there, progressively outer scopes are searched. A let may rebind (shadow) a variable an outer scope already defined, and may reassign the same variable more than once within itself — the latest assignment wins for later use.

Tokens are separated by single spaces, and the whole program is a legal expression that evaluates to an integer.

Input

program: a string containing the expression.

Output

An integer: the value of the expression.

#### Cons
traints

1 <= program.length <= 2000
No leading or trailing spaces; every pair of adjacent tokens is separated by exactly one space.
The program is guaranteed to be syntactically legal and to evaluate to an integer.
The final answer and every intermediate value fit in a signed 32-bit integer.

#### Example
s & test cases
TEXT
Input: program = "(add 4 (mult 2 5))" -> Output: 14
Why: the inner `(mult 2 5)` is 10, and `add` sums it with 4 — exercises nested compound expressions.

Input: program = "(let a 6 a)" -> Output: 6
Why: the simplest `let` — bind `a` to 6, then the body just references `a`.

Input: program = "(let a 4 a 7 a)" -> Output: 7
Why: the same variable is reassigned within one `let`; sequential processing means the latest value (7) wins.

Input: program = "(let a 2 (mult a (let a 6 b 10 (add a b))))" -> Output: 32
Why: scope test — the inner `let` shadows `a` (a=6, b=10, sum 16), but the `a` multiplied outside that inner `let` still resolves to the outer 2, so 2 * 16 = 32.

Input: program = "(let a 1 b 3 a (add a b) (add a b))" -> Output: 7
Why: `a` is first 1, then rebound to `(add a b)` = 4 (using the current a and b); the body `(add a b)` = 4 + 3 = 7 — checks that later value expressions see earlier bindings.

Input: program = "(mult -3 5)" -> Output: -15
Why: confirms negative integer literals are parsed correctly.

#### Approach

Recursive-descent evaluation with a lexical scope chain. Parse each compound expression (op ...) by stripping the outer parentheses and splitting the inner text into tokens at spaces that sit at parenthesis-depth zero (so nested sub-expressions stay intact as single tokens). Dispatch on the operator: add and mult recursively evaluate their two operands and combine them; let walks its variable/value pairs left to right, evaluating each value in the current scope and pushing the binding onto a stack before continuing (so later expressions see earlier bindings and reassignments), then evaluates the body and pops every binding it introduced. Integer literals are parsed directly. A variable is resolved by scanning the binding stack from the most recent (innermost) entry outward, which yields correct lexical shadowing for free. The underlying pattern is recursive descent parsing plus an interpreter with an explicit scope stack.

Complexity

Time: O(n * d) where n is the program length and d is the maximum nesting depth — each character is re-tokenized once for every compound expression that encloses it, and a variable lookup scans the currently active bindings; within the given bounds this is effectively linear and well inside limits. Space: O(n) for the recursion stack plus the binding stack, whose combined size is bounded by the program length.

Python
PYTHON
def evaluate(program: str) -> int:
    names = []   # binding stack (variable names)
    vals = []    # binding stack (values), parallel to names

    def split_top(s: str):
        tokens, cur, depth = [], [], 0
        for c in s:
            if c == '(':
                depth += 1
            elif c == ')':
                depth -= 1
            if c == ' ' and depth == 0:
                tokens.append(''.join(cur))
                cur = []
            else:
                cur.append(c)
        if cur:
            tokens.append(''.join(cur))
        return tokens

    def lookup(name: str) -> int:
        for i in range(len(names) - 1, -1, -1):
            if names[i] == name:
                return vals[i]
        return 0  # unreachable for legal input

    def eval_expr(e: str) -> int:
        if e[0] == '-' or e[0].isdigit():
            return int(e)
        if e[0] != '(':
            return lookup(e)

        tokens = split_top(e[1:-1])
        op = tokens[0]
        if op == 'add':
            return eval_expr(tokens[1]) + eval_expr(tokens[2])
        if op == 'mult':
            return eval_expr(tokens[1]) * eval_expr(tokens[2])

        # let
        mark = len(names)
        i = 1
        while i + 1 < len(tokens):
            v = eval_expr(tokens[i + 1])
            names.append(tokens[i])
            vals.append(v)
            i += 2
        result = eval_expr(tokens[-1])
        del names[mark:]
        del vals[mark:]
        return result

    return eval_expr(program)
JavaScript
JAVASCRIPT
function evaluate(program) {

#### cons
t names = [];   // binding stack (names)

#### cons
t vals = [];    // binding stack (values), parallel to names

#### cons
t splitTop = (s) => {

#### cons
t tokens = [];
        let cur = "", depth = 0;
        for (const c of s) {
            if (c === '(') depth++;
            else if (c === ')') depth--;
            if (c === ' ' && depth === 0) {
                tokens.push(cur);
                cur = "";
            } else {
                cur += c;
            }
        }
        if (cur.length > 0) tokens.push(cur);
        return tokens;
    };

#### cons
t lookup = (name) => {
        for (let i = names.length - 1; i >= 0; i--)
            if (names[i] === name) return vals[i];
        return 0; // unreachable for legal input
    };

#### cons
t evalExpr = (e) => {

#### cons
t first = e[0];
        if (first === '-' || (first >= '0' && first <= '9'))
            return parseInt(e, 10);
        if (first !== '(')
            return lookup(e);

#### cons
t tokens = splitTop(e.slice(1, -1));

#### cons
t op = tokens[0];
        if (op === "add")
            return evalExpr(tokens[1]) + evalExpr(tokens[2]);
        if (op === "mult")
            return evalExpr(tokens[1]) * evalExpr(tokens[2]);

        // let

#### cons
t mark = names.length;
        for (let i = 1; i + 1 < tokens.length; i += 2) {

#### cons
t v = evalExpr(tokens[i + 1]);
            names.push(tokens[i]);
            vals.push(v);
        }

#### cons
t result = evalExpr(tokens[tokens.length - 1]);
        names.length = mark;
        vals.length = mark;
        return result;
    };

    return evalExpr(program);
}
Java
JAVA
import java.util.*;

class Solution {
    private final List<String> names = new ArrayList<>();
    private final List<Integer> vals = new ArrayList<>();

    public int evaluate(String program) {
        return evalExpr(program);
    }

    private int evalExpr(String e) {
        char first = e.charAt(0);
        if (first == '-' || Character.isDigit(first))
            return Integer.parseInt(e);
        if (first != '(')
            return lookup(e);

        List<String> tokens = splitTop(e.substring(1, e.length() - 1));
        String op = tokens.get(0);
        if (op.equals("add"))
            return evalExpr(tokens.get(1)) + evalExpr(tokens.get(2));
        if (op.equals("mult"))
            return evalExpr(tokens.get(1)) * evalExpr(tokens.get(2));

        // let
        int mark = names.size();
        for (int i = 1; i + 1 < tokens.size(); i += 2) {
            int v = evalExpr(tokens.get(i + 1));
            names.add(tokens.get(i));
            vals.add(v);
        }
        int result = evalExpr(tokens.get(tokens.size() - 1));
        while (names.size() > mark) {
            names.remove(names.size() - 1);
            vals.remove(vals.size() - 1);
        }
        return result;
    }

    private int lookup(String name) {
        for (int i = names.size() - 1; i >= 0; i--)
            if (names.get(i).equals(name))
                return vals.get(i);
        return 0; // unreachable for legal input
    }

    private List<String> splitTop(String s) {
        List<String> tokens = new ArrayList<>();
        StringBuilder cur = new StringBuilder();
        int depth = 0;
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == '(') depth++;
            else if (c == ')') depth--;
            if (c == ' ' && depth == 0) {
                tokens.add(cur.toString());
                cur.setLength(0);
            } else {
                cur.append(c);
            }
        }
        if (cur.length() > 0) tokens.add(cur.toString());
        return tokens;
    }
}
C++
CPP
#include <string>
#include <vector>
#include <cctype>
using namespace std;

class Solution {
public:
    int evaluate(const string& program) {
        return evalExpr(program);
    }

private:
    vector<string> names;   // binding stack (names)
    vector<int>    vals;    // binding stack (values), parallel to names

    static vector<string> splitTop(const string& s) {
        vector<string> tokens;
        string cur;
        int depth = 0;
        for (char c : s) {
            if (c == '(') ++depth;
            else if (c == ')') --depth;
            if (c == ' ' && depth == 0) {
                tokens.push_back(cur);
                cur.clear();
            } else {
                cur += c;
            }
        }
        if (!cur.empty()) tokens.push_back(cur);
        return tokens;
    }

    int lookup(const string& name) {
        for (int i = (int)names.size() - 1; i >= 0; --i)
            if (names[i] == name) return vals[i];
        return 0; // unreachable for legal input
    }

    int evalExpr(const string& e) {
        if (e[0] == '-' || isdigit((unsigned char)e[0]))
            return stoi(e);
        if (e[0] != '(')
            return lookup(e);

        vector<string> tokens = splitTop(e.substr(1, e.size() - 2));

#### cons
t string& op = tokens[0];
        if (op == "add")
            return evalExpr(tokens[1]) + evalExpr(tokens[2]);
        if (op == "mult")
            return evalExpr(tokens[1]) * evalExpr(tokens[2]);

        // let
        int mark = names.size();
        for (int i = 1; i + 1 < (int)tokens.size(); i += 2) {
            int v = evalExpr(tokens[i + 1]);
            names.push_back(tokens[i]);
            vals.push_back(v);
        }
        int result = evalExpr(tokens.back());
        names.resize(mark);
        vals.resize(mark);
        return result;
    }
};
C
C
#include <stdbool.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>

/* Binding stack: names[i] bound to vals[i]; innermost binding is at the top. */
typedef struct {
    char** names;
    int*   vals;
    int    top;
    int    cap;
} Scope;

static void push(Scope* s, char* name, int val) {
    if (s->top == s->cap) {
        s->cap = s->cap ? s->cap * 2 : 16;
        s->names = (char**)realloc(s->names, s->cap * sizeof(char*));
        s->vals  = (int*)realloc(s->vals, s->cap * sizeof(int));
    }
    s->names[s->top] = name;
    s->vals[s->top]  = val;
    s->top++;
}

/* Split an inner string (outer parens already removed) into top-level tokens.
   Writes NUL terminators in place and records pointers to each token. */
static int splitTop(char* s, char** out) {
    int count = 0, depth = 0;
    char* start = s;
    for (char* p = s;; p++) {
        char c = *p;
        if (c == '\0') {
            if (p > start) out[count++] = start;
            break;
        }
        if (c == '(') depth++;
        else if (c == ')') depth--;
        if (c == ' ' && depth == 0) {
            *p = '\0';
            if (p > start) out[count++] = start;
            start = p + 1;
        }
    }
    return count;
}

static int evalExpr(char* e, Scope* s) {
    if (e[0] == '-' || isdigit((unsigned char)e[0]))
        return atoi(e);
    if (e[0] != '(') {                       /* variable: innermost first */
        for (int i = s->top - 1; i >= 0; i--)
            if (strcmp(s->names[i], e) == 0)
                return s->vals[i];
        return 0;                            /* unreachable for legal input */
    }

    int len = (int)strlen(e);
    e[len - 1] = '\0';                       /* drop trailing ')' */
    char* inner = e + 1;                     /* drop leading '('  */

    int cap = (int)strlen(inner) / 2 + 2;
    char** tokens = (char**)malloc(cap * sizeof(char*));
    int nt = splitTop(inner, tokens);
    char* op = tokens[0];
    int result;

    if (strcmp(op, "add") == 0) {
        result = evalExpr(tokens[1], s) + evalExpr(tokens[2], s);
    } else if (strcmp(op, "mult") == 0) {
        result = evalExpr(tokens[1], s) * evalExpr(tokens[2], s);
    } else {                                 /* let */
        int mark = s->top;
        for (int i = 1; i + 1 < nt; i += 2) {
            int v = evalExpr(tokens[i + 1], s);
            push(s, tokens[i], v);
        }
        result = evalExpr(tokens[nt - 1], s);
        s->top = mark;                       /* pop this let's bindings */
    }
    free(tokens);
    return result;
}

int evaluate(char* program) {
    char* buf = strdup(program);             /* mutable copy for in-place tokenizing */
    Scope s = {0};
    int ans = evalExpr(buf, &s);
    free(s.names);
    free(s.vals);
    free(buf);
    return ans;
}

#### Follow-up
questions
How would you extend the interpreter to support extra operators such as subtraction or integer division, including division-by-zero handling?
The current design re-tokenizes each sub-expression on the fly; how would you parse the program once into an abstract syntax tree and evaluate that instead?
How would you detect and report an illegal program (undefined variable, wrong arity, unbalanced parentheses) rather than assuming legality?
Can you make variable resolution O(1) with a hash-map scope while still preserving correct nested scoping and cheap teardown of a let's bindings on exit?

