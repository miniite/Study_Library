---
type: "practice"
difficulty: "Basic"
---

# What's the difference between asking Copilot to 'make my chatbot better' versus giving it a specific behavior you want changed, with an example?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Prompting for Trade-off Analysis](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Trade-off_Analysis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with AI-assisted coding tools like Copilot or similar systems, there's a key difference between asking for broad suggestions to improve your chatbot and providing specific instructions for behavior changes. Broad requests allow the system to leverage its understanding of general best practices and context, potentially leading to more innovative solutions but at the cost of predictability. Specific requests, on the other hand, guide the AI towards targeted improvements, ensuring that the desired behavior is accurately implemented.

#### Triage
Understanding the Question

The question asks us to compare two approaches:

Asking Copilot to "make my chatbot better."
Giving it a specific behavior change request.

#### Assumptions
and Scope
We are working with an existing chatbot codebase, which has some flaws or requires improvement.
The chatbot is written in Python, using a standard framework like Flask.

#### First-Principles Solution

Let's break down each approach:

#### Approach
1: "Make My Chatbot Better"

When you ask Copilot to make your chatbot better, it will analyze the existing code and suggest broad improvements. These could include:

Optimizing response times.
Improving user experience through more natural language processing (NLP).
Enhancing security measures.

For example, if there is a function that handles user inputs in a slow manner, Copilot might recommend using async functions or caching mechanisms to improve performance.

#### Approach
2: Specific Behavior Change

In contrast, when you ask for a specific behavior change, such as making the chatbot respond with "Hello" instead of "Hi," Copilot will focus on this exact task. It will look at the relevant code and suggest precise changes. For instance:

PYTHON
# Original Code
def greet_user(user_input):
    if user_input.lower() == 'hi':
        return 'Hi there!'

# Specific Behavior Change Request: Make it respond with "Hello"
Copilot might suggest:
def greet_user(user_input):
    if user_input.lower() == 'hi':
        return 'Hello there!'

# Or, if the function is already handling different greetings:
def handle_greeting(greeting_type):
    if greeting_type.lower() == 'hello':
        return 'Hello there!'

#### Closing Summary
+ Common Mistakes/Follow-Ups
Broad Requests: Useful for general improvements but might not address specific issues.
Specific Requests: Ensures targeted changes, but might miss broader optimizations.

#### Common Mistakes

Over-relying on broad requests without providing context or constraints.
Failing to specify clear and detailed requirements when giving specific requests.

#### Example

Let's consider a chatbot that currently responds with "Hi" to all greetings:

PYTHON
# Existing Code
def handle_greeting(greeting):
    if greeting.lower() == 'hi':
        return 'Hi there!'

If you ask Copilot to make the chatbot say "Hello" instead of "Hi," it would look at this function and suggest a change like this:

PYTHON
# Revised Code
def handle_greeting(greeting):
    if greeting.lower() == 'hello':
        return 'Hello there!'

However, if you want it to respond with both "Hi" and "Hello" but have them trigger different actions (e.g., one for casual users and another for registered users), providing this specific requirement would allow Copilot to suggest more nuanced changes.

Self-Check

This answer covers the core differences between broad and specific requests, provides examples, and highlights common mistakes. It matches the basic concept level by focusing on foundational understanding without unnecessary complexity.

**Code:**

```python
# Example of original code
def greet_user(user_input):
    if user_input.lower() == 'hi':
        return 'Hi there!'

# Revised code with specific behavior change
def greet_user(user_input):
    if user_input.lower() == 'hello':
        return 'Hello there!'

# Handling different greetings more granularly
def handle_greeting(greeting_type, user_input):
    if greeting_type.lower() == 'hi':
        return 'Hi there!'
    elif greeting_type.lower() == 'hello':
        return 'Hello there!'
```

**Example input / output:**

```text
# Original input and expected output for 'hi'
user_input = "hi"
expected_response = greet_user(user_input)  # Returns "Hi there!"

# Specific behavior change request input and expected output for 'hello'
user_input = "hello"
expected_response = greet_user(user_input)  # Now returns "Hello there!"
```

**Data:**

```json
{
  "greetings": [
    {"input": "hi", "expected_response": "Hi there!"},
    {"input": "hello", "expected_response": "Hello there!"}
  ]
}
```

