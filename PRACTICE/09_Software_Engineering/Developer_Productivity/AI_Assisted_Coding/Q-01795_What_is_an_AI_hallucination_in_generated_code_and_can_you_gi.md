---
type: "practice"
difficulty: "Basic"
---

# What is an AI "hallucination" in generated code and can you give an example?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reviewing AI Code for Correctness Bugs](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Correctness_Bugs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An AI hallucination in generated code refers to a situation where the AI produces code that appears correct or relevant based on the input, but is actually incorrect or nonsensical. This can happen when the AI misunderstands the context or requirements of the task at hand. For example, consider a scenario where you ask an AI assistant like Copilot to generate Python code for calculating the factorial of a number.

Here's a basic concept triage:

Understanding the Question: The question is asking about "hallucination" in AI-generated code. Specifically, it wants an explanation and an illustrative example.

#### Assumptions
We assume that we are discussing modern AI assistants like Copilot or similar tools used for coding assistance.
Key Idea: A hallucination happens when the AI generates incorrect or unexpected output.

Now let's dive into a step-by-step first-principles solution:

Suppose you ask an AI assistant to generate code for calculating the factorial of a number using recursion in Python:

PYTHON
def factorial(n):
    if n == 1:
        return 1
    else:
        # Hallucination: The AI might generate this line by misunderstanding the context or task.
        return n * (n - 2) + 2

In this case, the AI has generated a line of code that seems to make sense because it involves multiplication and addition, which are common operations in mathematical functions. However, when you run this code with factorial(3), for example, the output will be incorrect:

PYTHON
print(factorial(3))  # Output: 4 (Expected: 6)

This is an example of a hallucination because the generated line of code does not correctly implement the factorial calculation. The AI has misunderstood the recursion logic and provided a different formula that, while plausible in its own right, fails to accurately calculate the factorial.

To illustrate further:

Why It Happened: The AI might have misinterpreted the context or failed to understand the recursive nature of the problem.
Failure Mode: In this case, the failure mode is producing incorrect code that superficially appears relevant but does not meet the requirements.

#### Common Mistakes
and Follow-Ups:

#### Common mistakes
include underestimating the complexity of the task or over-relying on the AI's understanding of context.

#### Follow-ups
could involve discussing strategies to mitigate hallucinations, such as providing more detailed inputs, reviewing generated code, or using multiple tools for verification.

