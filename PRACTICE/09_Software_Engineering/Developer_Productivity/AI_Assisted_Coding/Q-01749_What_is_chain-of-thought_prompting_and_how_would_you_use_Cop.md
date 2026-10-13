---
type: "practice"
difficulty: "Deep"
---

# What is chain-of-thought prompting, and how would you use Copilot to add a 'think step by step' instruction while keeping the final answer parseable?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Prompting for Trade-off Analysis](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Trade-off_Analysis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Chain-of-thought prompting in AI-assisted coding involves providing detailed reasoning or thought processes within code snippets, making the development process more transparent and understandable. By using this approach with tools like Copilot, we can ensure that each step of the solution is clear and logical, which not only helps in debugging but also makes the code easier for others to review.

To use Copilot for adding a 'think step by step' instruction while keeping the final answer parseable, follow these steps:

Understand the Problem: First, clearly define what needs to be done. For instance, if you are writing a function that calculates the Fibonacci sequence, specify the requirements and constraints.
Plan Your Steps: Break down the problem into smaller, manageable parts. List out each step in your thought process.
Prompt Copilot with Thoughtful Prompts: Use prompts like "Let's start by defining the base cases," "Next, we can use a loop to generate subsequent numbers," and "Finally, return the desired value." This helps Copilot understand that you want it to provide detailed reasoning rather than just the final code.
Review and Refine: After Copilot generates the code, review each part to ensure clarity and correctness. Make any necessary adjustments.

By structuring your prompts in this manner, you can create a more educational coding experience, which is especially useful for learning or teaching AI-assisted coding techniques.

**Code:**

```python
# Example of using chain-of-thought prompting with Copilot

def fibonacci(n):
    # Let's start by defining the base cases.
    if n <= 0:
        return "Input should be a positive integer."
    elif n == 1:
        return [0]
    elif n == 2:
        return [0, 1]

    # Next, we can use a loop to generate subsequent numbers
    result = [0, 1]
    for i in range(2, n):
        next_value = result[i-1] + result[i-2]
        result.append(next_value)

    # Finally, return the desired value.
    return result[-1]

# Test case: Print the 10th Fibonacci number
print(f"The 10th Fibonacci number is {fibonacci(10)}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `print(fibonacci(10))` | `55` |

**Flow:**

1. Start
2. Define the problem
3. Break down into steps
4. Write thought prompts foreach step
5. Use Copilot with theseprompts
6. Review generated code
7. Refine if necessary
8. End

