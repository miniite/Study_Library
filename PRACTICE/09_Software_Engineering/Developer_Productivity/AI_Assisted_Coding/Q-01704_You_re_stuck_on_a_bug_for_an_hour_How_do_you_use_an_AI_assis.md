---
type: "practice"
difficulty: "Intermediate"
---

# You're stuck on a bug for an hour. How do you use an AI assistant to make progress without just guessing?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Debugging with AI Assistants](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Debugging_with_AI_Assistants.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a stubborn bug, leveraging an AI assistant like GitHub Copilot can be incredibly beneficial. The key is not to treat it as a magic solution that will instantly solve everything but rather as a powerful tool to help you think through the problem methodically.

First, I would restate the core of the question: How do we use AI assistance effectively to make progress when stuck on a bug for an hour without resorting to guesswork? The interviewer is probing our ability to strategically employ tools and our debugging techniques.

#### Assumptions
You have access to an AI assistant like Copilot.
You are familiar with the codebase and context of the issue.
The bug has been isolated, but you're facing a tricky part.
Step-by-Step Solution:

Clarify the Problem: Clearly articulate the issue to the AI assistant by describing the symptoms and the specific line(s) where the problem seems to occur. For example:

CODE
There's an error in this function: `validateInput`. The code throws a TypeError on line 25, but it should not.

Code Context: Provide context around the problematic section of code for the AI assistant to understand better. Highlighting relevant parts can help the assistant generate more targeted suggestions:

PYTHON
def validateInput(input_str: str) -> bool:
    # Some complex logic here
    if not isinstance(input_str, str):
        raise TypeError("Expected a string")
    return len(input_str) > 5

input_data = 1234567890  # This is the problematic line causing the error.
validateInput(input_data)

AI Suggestion: Leverage Copilot’s suggestions to get alternative ways of writing the code or to find potential issues:

CODE
# Copilot suggests:
if not isinstance(input_str, str) or len(input_str) <= 5:
    raise ValueError("Invalid input")
return True

Reasoning with First Principles: Evaluate why this suggestion might be better and how it addresses the root cause of the error. For instance, it catches both type and length issues using ValueError, which could prevent similar bugs in other parts of your codebase.

Refactoring and Testing: Implement the suggested changes, run tests, and observe if the issue persists or if new problems arise:

PYTHON
def validateInput(input_str: str) -> bool:
    if not isinstance(input_str, str):
        raise ValueError("Expected a string")
    elif len(input_str) <= 5:
        raise ValueError("String length should be more than 5")

    return True

input_data = "abc"  # This is the problematic line causing the error.
try:
    validateInput(input_data)
except ValueError as e:
    print(e)

Iterative Debugging: If issues persist, continue to interact with Copilot, providing more context and letting it generate further suggestions. Iterate on your understanding of the problem.

Document and Learn: Once you resolve the issue, document what you learned from this process for future reference:

CODE
Issue: TypeError due to wrong type check

#### Solution
Use ValueError for both type and length checks.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes

Relying too heavily on the AI without critical thinking. It’s crucial to understand why suggestions are made.
Not thoroughly testing or validating changes suggested by the AI.

#### Follow-Ups

Discuss how you would ensure the reliability of the AI-generated code in a larger, more complex project.
Explore how this approach could be integrated into your existing debugging workflow for other developers on the team.

By following these steps, you can effectively use an AI assistant to make progress without guessing and enhance your problem-solving skills.

**Code:**

```python
def validateInput(input_str: str) -> bool:
    if not isinstance(input_str, str):
        raise ValueError("Expected a string")
    elif len(input_str) <= 5:
        raise ValueError("String length should be more than 5")

    return True

input_data = "abc"  # This is the problematic line causing the error.
try:
    validateInput(input_data)
except ValueError as e:
    print(e)
```

**Example input / output:**

```text
String length should be more than 5
```

