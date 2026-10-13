---
type: "practice"
difficulty: "Basic"
---

# What is the complexity of validating balanced parentheses using a stack, and what is the space cost?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stack-based Complexity](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Stack-based_Complexity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When validating whether a string of parentheses is balanced, one common approach involves using a stack data structure. The goal is to ensure that every opening parenthesis has a corresponding closing parenthesis in the correct order.

In this context, "time complexity" refers to how the algorithm's runtime grows with input size, while "space complexity" pertains to the memory usage as the input size increases.

#### Triage
Restating the Question and Assumptions

The question asks for two key aspects:

Time Complexity: How long does it take to validate a string of parentheses?
Space Complexity: How much additional space is used during this process?

#### Assumptions
include:

The input consists only of opening (() and closing ()) parentheses.
We aim to use the most straightforward approach, utilizing a stack.

#### First-Principles Solution
Time Complexity Analysis
Initialization: The stack is initialized with an empty state in O(1).
Iteration through String:
For each character in the string, we perform operations that are constant time (O(1)).
We push onto the stack for every opening parenthesis (() and pop from the stack for every closing parenthesis ()).

Since we need to process each character exactly once, the total number of operations is proportional to the length of the string. Hence, the time complexity is O(n), where n is the length of the string.

Space Complexity Analysis
Stack Usage:
The stack stores at most as many elements as there are opening parentheses in the worst case (when all are not immediately followed by closing ones).

Therefore, the space required by the stack can grow up to O(n), where n is the length of the string.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary:

Time Complexity: The validation process takes linear time, i.e., O(n).
Space Complexity: The worst-case space requirement is also O(n) due to stack usage.

#### Common mistakes
include:

Overcomplicating the solution with additional data structures or logic.
Failing to recognize that both complexities are directly related to the length of the input string, leading to incorrect assumptions about their values.

#### Example
Dry-Run

#### Cons
ider a string s = "(()())":

Initialize an empty stack.
Iterate through each character:
Push ( onto stack: [ ( ]
Push ( onto stack: [ ( ( ]
Pop from stack on ) : [ ( ]
Push ( onto stack: [ ( ( ]
Pop from stack on ) : [ ( ]
Pop from stack on ): empty

If the stack is empty at the end, the parentheses are balanced.

**Code:**

```python
def validate_parentheses(s: str) -> bool:
    stack = []
    for char in s:
        if char == '(':
            stack.append(char)
        elif char == ')':
            if not stack:
                return False  # Unmatched closing parenthesis
            stack.pop()
    return len(stack) == 0  # True if all parentheses are balanced
```

