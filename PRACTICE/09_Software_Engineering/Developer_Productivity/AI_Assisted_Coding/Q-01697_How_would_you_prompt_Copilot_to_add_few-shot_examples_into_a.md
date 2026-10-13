---
type: "practice"
difficulty: "Intermediate"
---

# How would you prompt Copilot to add few-shot examples into a prompt, and what would you verify about the example format matching your task?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Prompting for Trade-off Analysis](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Trade-off_Analysis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate few-shot examples effectively into an AI-assisted coding tool like Copilot or a similar system, we need to carefully craft our prompts so that the AI can understand and apply the examples appropriately. The key steps involve structuring the prompt in a way that clearly delineates the examples from the main task description, ensuring consistency in the example format, and verifying the appropriateness of the examples for the given task.

Firstly, let's break down what we need to achieve:

Prompt Crafting: We'll construct our prompt by including both a brief task description and one or more few-shot examples.

#### Example
Format Verification: We'll ensure that the format of the examples matches the requirements of the task at hand.

To start with, here’s an example of how we can structure our prompt:

PLAINTEXT
Given the following code snippet for generating a simple calculator function:
```python
def add(a: int, b: int) -> int:
    return a + b

def subtract(a: int, b: int) -> int:
    return a - b

# Here are some examples of how these functions can be used:
add(5, 3) # Returns 8
subtract(7, 2) # Returns 5

# Your task is to implement the following function and include several example calls in your code:

def multiply(a: int, b: int) -> int:
    return a * b

In this prompt, we explicitly define the functions add and subtract, provide their usage examples, and then ask for implementation of a new function multiply. This structure helps Copilot understand what to expect from the examples.

Next, let's discuss what needs to be verified about the example format:

#### Cons
istency in Function Signature: Ensure that all provided examples match the function signature. For instance, if our task involves functions with integer inputs and outputs, then each example should follow this pattern.
Relevance of Examples: The examples should be relevant to the task at hand. For complex tasks, providing a few high-quality examples can help guide Copilot more effectively.
Correctness of Examples: Verify that the examples are syntactically correct and logically valid for the function they illustrate.

To verify these aspects programmatically (assuming we're working within an environment like VSCode with Copilot), we could add checks in our code:

PYTHON
def test_add():
    assert add(5, 3) == 8

def test_subtract():
    assert subtract(7, 2) == 5

# Add similar tests for the `multiply` function after its implementation.

By including these checks, we can ensure that Copilot understands and applies examples correctly. The key is to provide clear and relevant examples in a structured manner.

In conclusion, structuring our prompt properly and verifying example formats are crucial steps in leveraging AI-assisted coding tools effectively. Careful attention to detail ensures that the AI can understand the task and generate high-quality code.

**Code:**

```python
# Example of a prompt for Copilot

def add(a: int, b: int) -> int:
    return a + b

def subtract(a: int, b: int) -> int:
    return a - b

# Here are some examples of how these functions can be used:
add(5, 3) # Returns 8
subtract(7, 2) # Returns 5

# Your task is to implement the following function and include several example calls in your code:

def multiply(a: int, b: int) -> int:
    return a * b

# Example of checks for these functions
def test_add():
    assert add(5, 3) == 8

def test_subtract():
    assert subtract(7, 2) == 5
```

