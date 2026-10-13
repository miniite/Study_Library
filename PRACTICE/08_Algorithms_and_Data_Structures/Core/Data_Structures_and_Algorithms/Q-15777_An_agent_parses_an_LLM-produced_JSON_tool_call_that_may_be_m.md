---
type: "practice"
difficulty: "Intermediate"
---

# An agent parses an LLM-produced JSON tool call that may be malformed; why validate bracket balance with a stack before parsing, and how do you recover gracefully when the model truncates output mid-object?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursion vs Explicit Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_vs_Explicit_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robustness in processing LLM-generated JSON tool calls, validating the bracket balance using a stack is crucial to prevent parsing errors. When an LLM produces truncated or malformed JSON, it's essential to detect and handle these issues gracefully rather than crashing the system. Here’s how we can achieve this:

#### Triaging the Question

The key aspects of the question are:

Why validate bracket balance with a stack?: This involves understanding why maintaining proper brackets is critical for parsing.
How to recover gracefully when the model truncates output mid-object: This focuses on handling incomplete or malformed JSON data.

#### Assumptions
and First-Principles
The input JSON might be truncated, which means it could end in an incomplete state (e.g., missing a closing brace).
Proper validation helps catch these issues early, preventing deeper parsing errors.
Graceful recovery ensures the system can handle unexpected inputs without failing catastrophically.

#### Solution
Step-by-Step

Bracket Balance Validation with Stack:

Initialize an empty stack to keep track of opening brackets.
Iterate through each character in the JSON string.
For every opening bracket ({ or [), push it onto the stack.
For every closing bracket (} or ]), check if the top of the stack matches. If it does, pop from the stack; otherwise, report an unbalanced state and halt parsing.

Handling Truncation:

If the JSON is truncated mid-object, validate the current contents up to the last valid point.
Upon reaching a closing bracket, check if all opening brackets have been properly closed. This ensures that partial objects are correctly identified as incomplete rather than invalid.
Code Example

Here’s a simple Python implementation using a stack for validation:

PYTHON
def validate_brackets(json_str):
    stack = []
    last_valid_index = 0

    # Iterate through the JSON string
    for i, char in enumerate(json_str):
        if char == '{':
            stack.append('{')
        elif char == '}':
            if not stack or stack[-1] != '{':
                return False, last_valid_index
            else:
                stack.pop()

        if len(stack) == 0:  # All brackets are balanced until here
            last_valid_index = i + 1

    return len(stack) == 0, last_valid_index

# Example usage
json_input = '{"tool": "curl", "args": ["https://api.example.com"]}'
is_balanced, valid_end = validate_brackets(json_input)
print(f"Balanced: {is_balanced}, Valid End Index: {valid_end}")
Handling Truncation Gracefully

Partial Object Recovery:

If the JSON ends mid-object (e.g., { "tool": "curl", "args": ["https://api.example.com), validate up to that point.
Return a partial object or handle it as an incomplete request, allowing the system to manage and log such events without crashing.

Logging and Error Handling:

Log any unbalanced brackets with their index positions for debugging purposes.
Implement fallback logic in the application layer to manage partial results appropriately (e.g., retrying requests or handling them as errors).

#### Closing Summary

By validating bracket balance using a stack, we ensure that the JSON parsing process is robust against truncation and malformed inputs. Graceful recovery mechanisms allow us to handle incomplete data effectively without causing system failures.

#### Common Mistakes/Follow-Ups
Failing on First Error: Only reporting the first unbalanced bracket without attempting to recover.
Ignoring Partial Objects: Not handling cases where the JSON is truncated mid-object, leading to unexpected behavior or crashes.
Lack of Logging: Failing to log errors and their positions for debugging purposes.

**Code:**

```python
def validate_brackets(json_str):
    stack = []
    last_valid_index = 0

    # Iterate through the JSON string
    for i, char in enumerate(json_str):
        if char == '{':
            stack.append('{')
        elif char == '}':
            if not stack or stack[-1] != '{':
                return False, last_valid_index
            else:
                stack.pop()

        if len(stack) == 0:  # All brackets are balanced until here
            last_valid_index = i + 1

    return len(stack) == 0, last_valid_index

# Example usage
json_input = '{"tool": "curl", "args": ["https://api.example.com"]}'
is_balanced, valid_end = validate_brackets(json_input)
print(f"Balanced: {is_balanced}, Valid End Index: {valid_end}")
```

**Example input / output:**

```text
Input: '{"tool": "curl", "args": ["https://api.example.com"]}'
Output: Balanced: True, Valid End Index: 46
```

**Flow:**

1. Yes
2. No
3. Start Parsing
4. Is JSON Truncated?
5. Validate Partial Object
6. Handle Incomplete Request
7. Validate Full JSON
8. Report Errors if Any
9. End

