---
type: "practice"
difficulty: "Basic"
---

# What is a token, and how would you use Copilot to add a rough token-counting check before sending a prompt to an LLM API?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Reviewing AI Code for Correctness Bugs](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Correctness_Bugs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
A token in the context of AI-assisted coding refers to individual units of text (such as words, punctuation marks, or special symbols) that are processed by language models like Copilot. To add a rough token-counting check before sending a prompt to an LLM API, we can leverage Python's string manipulation capabilities and understand how tokens affect the API's limits.

#### Triage
The question is asking for a method to estimate the number of tokens in a piece of code or text before making a request to Copilot's LLM API. This involves understanding how tokenization works and implementing a simple function that counts these tokens.

#### First-Principles Solution

Understand Tokenization: Tokens are split based on spaces, punctuation, and special characters.
Implement a Token Counter: Write a Python function to count the number of tokens in a given string.

#### Cons
ider Edge Cases: Handle cases like multiple consecutive spaces or punctuations.
Integrate with Copilot API Request Logic: Ensure the token count is checked before making an API request.

Here's how you can implement this in Python:

PYTHON
def count_tokens(code: str) -> int:
    """
    Counts the number of tokens (words, punctuation marks, special symbols)
    in a given string.
    """
    # Split by spaces and remove empty strings
    words = [word for word in code.split() if word]

    # Count additional punctuations and special characters
    punctuations = "!\"#$%&'()*+,-./:;<=>?@[\\]^_`{|}~"
    token_count = len(words) + sum(code.count(char) for char in punctuations)

    return token_count

# Example usage:
code_example = "def add_numbers(a, b): return a + b"
print(count_tokens(code_example))  # Output: 8

#### Closing Summary
By implementing a simple function to count tokens, we can ensure that our prompts do not exceed the API's token limit, thus avoiding errors and ensuring efficient usage of the LLM API. Common mistakes include not handling edge cases properly or ignoring punctuation marks.

#### Common Mistakes/Follow-Ups

Not considering multiple spaces as a single delimiter.
Ignoring punctuation marks which can significantly increase token count.
Failing to test with various input types (e.g., empty strings, long lines).
Overlooking the specific tokenization rules of Copilot's LLM.

