---
type: "practice"
difficulty: "Basic"
---

# What is prompt engineering, and how does an AI coding assistant help you draft better prompts for an LLM feature you are building?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Prompting for Trade-off Analysis](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Trade-off_Analysis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Prompt engineering is a critical process in fine-tuning interactions with large language models (LLMs) like those used in AI-assisted coding assistants such as Copilot. Essentially, it involves crafting clear and concise prompts that guide the LLM to produce useful and relevant responses. For an AI-assisted coding feature, understanding prompt engineering means ensuring your prompts are specific enough to elicit the desired code output while being flexible enough to handle various edge cases.

To start with, let’s triage the question:

What is prompt engineering? - This involves designing and refining input instructions (prompts) for an AI model.
How does it apply in AI-assisted coding? - The goal is to improve code generation accuracy and efficiency by crafting effective prompts.

Now, let’s dive into the first-principles solution:

1. Defining Prompt Engineering

Prompt engineering involves creating structured input that a language model can understand to produce useful outputs. This process includes breaking down complex requests into simpler, more precise instructions. For instance, in AI-assisted coding, you might ask for "a Python function that calculates the factorial of a number" or "an optimized loop to iterate through an array."

2. Key Components of Effective Prompts
Clarity and Specificity: Clear prompts reduce ambiguity and ensure the model understands exactly what is required.
Contextual Clues: Providing relevant context can guide the model towards more accurate responses.
Format and Structure: Proper formatting ensures that the LLM processes your instructions correctly. For example, using code blocks for code snippets or specifying data types.
3. Example Scenario

Suppose you are building a feature where users can generate Python code to solve a specific problem. Here’s how prompt engineering can help:

Initial Prompt: "Generate a Python function that calculates the factorial of a number."
Improved Prompt with Context and Clarity:
PLAINTEXT
I need a Python function named `factorial` that takes an integer as input and returns its factorial value.

The function should handle non-negative integers. Please include appropriate comments in the code.
4. Handling Edge Cases
Negative Numbers: "What if the user inputs a negative number? Should we return an error or calculate for absolute values?"
Large Numbers: "How do we handle large numbers to avoid overflow issues?"
5. Iterative Refinement

Prompt engineering is iterative, requiring multiple rounds of refinement based on feedback from both the model and human users.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Effective prompt engineering ensures that your AI-assisted coding feature provides accurate and useful outputs. Key mistakes include overly vague prompts or missing context, which can lead to incorrect or irrelevant code generation. Always test your prompts with different scenarios to ensure robustness.

Common follow-ups might involve discussing the trade-offs between specificity and generality in prompts or exploring advanced techniques like chaining multiple prompts for complex tasks.

**Comparison:**

```
Initial Prompt	Improved Prompt
"Generate a Python function that calculates the factorial of a number."	"I need a Python function named factorial that takes an integer as input and returns its factorial value. The function should handle non-negative integers. Please include appropriate comments in the code."
```

**Code:**

```python
def factorial(n):
    """Calculate the factorial of a non-negative integer n."""
    if n < 0:
        return "Error: Factorial is not defined for negative numbers."
    elif n == 0 or n == 1:
        return 1
    else:
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result

# Example usage
print(factorial(5))  # Output: 120
```

**Example input / output:**

```

Initial Prompt: "Generate a Python function that calculates the factorial of a number."

Output: A basic, unoptimized function.

Improved Prompt:

PLAINTEXT
Copy
I need a Python function named `factorial` that takes an integer as input and returns its factorial value. The function should handle non-negative integers. Please include appropriate comments in the code.
Optimized Output:
PYTHON
Copy
def factorial(n):
    """Calculate the factorial of a non-negative integer n."""
    if n < 0:
        return "Error: Factorial is not defined for negative numbers."
    elif n == 0 or n == 1:
        return 1
    else:
        result = 1
        for i in range(2, n + 1):
            result *= i
        return result

# Example usage
print(factorial(5))  # Output: 120
```

**Data:**

```json
{
  "prompt": "Generate a Python function that calculates the factorial of a number.",
  "improved_prompt": "I need a Python function named `factorial` that takes an integer as input and returns its factorial value. The function should handle non-negative integers. Please include appropriate comments in the code."
}
```

**Flow:**

1. No
2. Initial Prompt
3. Is clear?
4. Refine Prompt
5. Improved Prompt
6. Test with different scenarios
7. Iterate based on feedback

