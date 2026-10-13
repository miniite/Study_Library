---
type: "practice"
difficulty: "Basic"
---

# What is a prompt in LLM usage, and how would you build a prompt string in Python from a fixed instruction plus a user-supplied variable?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming LLM Responses](<../../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of using Large Language Models (LLMs), a "prompt" is essentially an input string that you provide to guide the model's response. It can include explicit instructions and context that help shape the direction or content of the output. In Python, constructing a prompt involves combining fixed instruction text with dynamic user inputs. This answer will cover both the basic concept and how to implement this in Python for an AI Engineer at different levels.

#### Triage
The question is asking about the definition of a "prompt" in LLM usage and how to create such a string using Python, combining static instructions with variable data provided by users. Assumptions: We assume you have access to the necessary libraries and that your environment supports basic Python operations. For simplicity, let's use a fixed instruction template and simulate user input as strings.

#### First-principles solution
To build a prompt in Python from a fixed instruction plus a user-supplied variable, follow these steps:

Define your fixed instruction string.
Accept the user-supplied data as a variable.
Concatenate or format the variables into the instruction template to form the final prompt.

For example:

Fixed instruction: "Given the following description of an item, please generate a title for it: {description}."
User input (variable): "A sleek black watch with a leather strap"

Combining these elements would result in a complete prompt string like: "Given the following description of an item, please generate a title for it: A sleek black watch with a leather strap."

#### Closing summary
Building a prompt involves clear communication between the model and user. In Python, this is achieved through concatenation or string formatting techniques. Common mistakes include not handling edge cases where the variable input might be empty or incorrectly formatted.

**Code:**

```python
# Example of building a prompt in Python
fixed_instruction = "Given the following description of an item, please generate a title for it: {description}."

user_description = "A sleek black watch with a leather strap"

# Using f-string (Python 3.6+)
prompt_fstring = f"{fixed_instruction.format(description=user_description)}"

# Alternative using .format method
prompt_format = fixed_instruction.format(description=user_description)

print(prompt_fstring)  # Output: Given the following description of an item, please generate a title for it: A sleek black watch with a leather strap.
```

**Example input / output:**

```text
- Input: `fixed_instruction = "Given the following description of an item, please generate a title for it: {description}."`, `user_description = "A sleek black watch with a leather strap"`
- Output: `"Given the following description of an item, please generate a title for it: A sleek black watch with a leather strap."`
```

**Flow:**

1. Yes
2. No
3. Define fixed instruction
4. Is user input provided?
5. Combine using f-string or.format()
6. Use default prompt
7. Output complete prompt

