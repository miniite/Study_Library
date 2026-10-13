---
type: "practice"
difficulty: "Basic"
---

# Given a string of parentheses, check whether it is balanced.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Balanced Parentheses](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Balanced_Parentheses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Given a string made of bracket characters, determine whether every opening bracket has a correctly-ordered matching closing bracket. This is the classic "valid parentheses" / "balanced brackets" problem.

What the interviewer is probing: Do you recognize this as a Last-In-First-Out (LIFO) matching problem and reach for a stack? Can you translate that recognition into a clean O(n) time / O(n) space solution, handle edge cases, and reason about correctness? At a fresher level this is also a proxy for "can you write correct, bug-free code and talk about complexity."

Clarifying questions I would ask:

Is the input only ( and ), or does it also contain {} and []? (Most versions include all three — I'll solve the general case, which subsumes the single-type case.)
Can the string contain other characters (letters, spaces)? If yes, do I ignore them or is that invalid? (I'll assume brackets only, and mention how to adapt.)
Is an empty string considered balanced? (Standard answer: yes — vacuously true.)

Definition of "balanced": A string is balanced if (a) every closing bracket matches the type of the most recent unmatched opening bracket, and (b) no opening bracket is left unmatched at the end. So ()[]{} and ([{}]) are balanced; (], ([)], and ((( are not.

From first principles to the solution

The key insight is ordering. Brackets must close in the reverse order they open — the most recently opened bracket must be the first one closed. That "most recent first" behavior is exactly the definition of a stack (LIFO). So the algorithm writes itself:

Scan the string left to right, one character at a time:

If the character is an opening bracket ((, {, [), we don't yet know its partner, so we push it onto the stack — it's now the "pending" bracket that must be closed next.
If the character is a closing bracket (), }, ]), it must match the bracket on top of the stack. Two things can go wrong:
The stack is empty — there's a closing bracket with nothing to match it (e.g. )(). Invalid.
The top of the stack is the wrong type (e.g. top is ( but we see ]). Invalid.
Otherwise it matches: pop the top and continue.

After the whole string is processed, the stack must be empty. If anything remains, those are opening brackets that were never closed (e.g. (((). So the final answer is stack.isEmpty().

A common clean trick is to build a map from each closing bracket to its expected opening bracket: { ')' : '(', ']' : '[', '}' : '{' }. Then on a closing bracket you just compare the popped top against the mapped value.

Why this is correct
No unmatched close survives: Every closing bracket either finds a valid matching top (and cancels it) or immediately returns false. So the string can never be accepted with a stray or mismatched close.
No unmatched open survives: Any opening bracket that is never closed stays on the stack, so the final isEmpty() check catches it.
Order is enforced automatically: Because we always match against the top, a case like ([)] fails — when we hit ), the top is [, which doesn't match (, so we correctly reject.

This is a standard, provably-correct approach; nesting depth is naturally tracked by the stack size.

Complexity
Time: O(n) — each character is visited once, and push/pop/lookup are O(1).
Space: O(n) — worst case is all opening brackets (((((), which pushes n items.

Micro-optimization for a single bracket type: If the input is only ( and ), you don't need a stack at all — keep an integer balance, +1 on (, -1 on ), return false immediately if balance ever goes negative, and require balance == 0 at the end. That's O(1) space. I'd mention this because it shows I understand why the stack is needed (to track type/order), and that when there's only one type, a counter suffices.

Edge cases to call out
Empty string → balanced (return true).
Odd length → can never be balanced (nice early-exit optimization).
Starts with a closing bracket ()abc) → stack empty on first close → false.
All opening (((() → stack non-empty at end → false.
Interleaved wrong nesting (([)]) → type mismatch on pop → false.

#### Common mistakes
Forgetting the final empty-stack check (accepts ((().
Forgetting to check for an empty stack before popping on a closing bracket → runtime error / crash on ).
Only counting brackets (open count == close count) without a stack — this wrongly accepts ([)] and )(.
Summary — what a strong answer emphasizes

State the LIFO insight out loud, reach for a stack, use a closing→opening map for clean matching, handle the two failure modes (empty stack / type mismatch), and finish with the empty-stack check. Give O(n)/O(n) complexity, mention the O(1) counter trick for the single-type case, and name the tricky cases (([)], )(, (((). That combination — correct code, correct complexity, and awareness of the subtle cases — is exactly what an interviewer wants here.

**Comparison:**

```
Approach	Handles multiple bracket types	Time	Space	Notes
Stack (map closing→opening)	Yes	O(n)	O(n)	Standard, recommended
Integer counter	No (single type only)	O(n)	O(1)	Only for (); fails on ([)] if misused
Compare open/close counts	No	O(n)	O(1)	WRONG — accepts )( and ([)]
```

**Code:**

```

// General case: (), {}, [] — O(n) time, O(n) space public boolean isBalanced(String s) { Deque<Character> stack = new ArrayDeque<>(); Map<Character, Character> match = Map.of(')', '(', ']', '[', '}', '{');

CODE
Copy
for (char c : s.toCharArray()) {
    if (c == '(' || c == '[' || c == '{') {
        stack.push(c);                       // opening: remember it
    } else if (match.containsKey(c)) {
        // closing: top must exist and be the matching opener
        if (stack.isEmpty() || stack.pop() != match.get(c)) {
            return false;
        }
    }
    // (optional) ignore any non-bracket characters here
}
return stack.isEmpty();                        // no unclosed openers left

}

CODE
Copy

```python
# Python version, same logic
def is_balanced(s: str) -> bool:
    match = {')': '(', ']': '[', '}': '{'}
    stack = []
    for c in s:
        if c in '([{':
            stack.append(c)
        elif c in match:
            if not stack or stack.pop() != match[c]:
                return False
    return not stack
PYTHON
Copy
# Single bracket type only — O(1) space counter trick
def is_balanced_single(s: str) -> bool:
    balance = 0
    for c in s:
        if c == '(':
            balance += 1
        elif c == ')':
            balance -= 1
            if balance < 0:      # a ')' with no matching '('
                return False
    return balance == 0
```

**Example input / output:**

```

Worked examples (general case):

Input	Output	Why
""	true	empty is vacuously balanced
()	true	push (, pop on ), stack empty
()[]{}	true	each pair opens and immediately closes
([{}])	true	properly nested
(]	false	pop ( but expected [ → type mismatch
([)]	false	on ), top is [ ≠ ( → mismatch
)(	false	first char ) → stack empty → false
(((	false	stack non-empty at end

Dry run of ([)]:

CODE
Copy
c='(' -> push        stack: [ (        ]
c='[' -> push        stack: [ ( [      ]
c=')' -> pop '['     '[' != match[')']='('  -> return FALSE

Dry run of ([{}]):

CODE
Copy
c='(' push   -> (
c='[' push   -> ( [
c='{' push   -> ( [ {
c='}' pop {   matches '{'   -> ( [
c=']' pop [   matches '['   -> (
c=')' pop (   matches '('   -> (empty)
end: stack empty -> TRUE
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Yes
6. No
7. No
8. Yes
9. end of string
10. Yes
11. No
12. Start: read next char c
13. c is opening bracket?
14. push c onto stack
15. c is closing bracket?
16. stack empty?
17. return false
18. pop top t
19. t matches c's type?
20. stack empty?
21. return true

