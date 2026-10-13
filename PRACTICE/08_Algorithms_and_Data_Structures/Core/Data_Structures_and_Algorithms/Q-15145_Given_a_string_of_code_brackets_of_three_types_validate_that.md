---
type: "practice"
difficulty: "Basic"
---

# Given a string of code brackets of three types, validate that they are correctly nested.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Balanced Parentheses](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Balanced_Parentheses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to ensure that the given string of code brackets is correctly nested. The three types of brackets are square brackets [], curly braces {}, and parentheses (). We will use a stack-based approach to validate these brackets. Here’s a step-by-step explanation:

Problem Overview: We need to check if the sequence of brackets is valid, meaning that each opening bracket has a corresponding closing bracket in the correct order.
What's Being Probed: The interviewer is testing your understanding of data structures (stacks) and algorithms for solving problems involving nested structures.

#### Assumptions
We assume the input string only contains these three types of brackets.

We'll use a stack to keep track of opening brackets, pushing them onto the stack when we encounter an opening bracket and popping from the stack when we find a corresponding closing bracket. If the stack is empty when we try to pop or if the sequence ends with non-matching brackets, the input is invalid.

Now, let's walk through the solution step-by-step:

Initialize a Stack: We will use a list as our stack.

Iterate Through Each Character in the String:

If it’s an opening bracket ([, {, (), push it onto the stack.
If it’s a closing bracket (], }, )):
Check if the stack is empty. If it is, return false since there's no corresponding opening bracket.
Pop from the stack and compare the top element with the current character to see if they match. For example, [ should have a corresponding ], { should have a }, etc. If they don’t match, return false.

Final Check: After iterating through all characters, check if the stack is empty. If it’s not, there are unmatched opening brackets; hence, return false.

Here's how we can implement this:

PYTHON
def validate_brackets(code: str) -> bool:
    stack = []

    for char in code:
        if char in ['[', '{', '(']:
            stack.append(char)
        elif char in [']', '}', ')']:
            if not stack:
                return False
            top = stack.pop()
            if (char == ']' and top != '[') or \
               (char == '}' and top != '{') or \
               (char == ')' and top != '('):
                return False

    return len(stack) == 0

# Example usage
print(validate_brackets("[{()}]"))  # True
print(validate_brackets("([{]}))")) # False

**Code:**

```python
def validate_brackets(code: str) -> bool:
    stack = []

    for char in code:
        if char in ['[', '{', '(']:
            stack.append(char)
        elif char in [']', '}', ')']:
            if not stack:
                return False
            top = stack.pop()
            if (char == ']' and top != '[') or \
               (char == '}' and top != '{') or \
               (char == ')' and top != '('):
                return False

    return len(stack) == 0
```

**Example input / output:**

```text
Input: "[{()}]"
Output: True

Input: "([{]}))"
Output: False
```

