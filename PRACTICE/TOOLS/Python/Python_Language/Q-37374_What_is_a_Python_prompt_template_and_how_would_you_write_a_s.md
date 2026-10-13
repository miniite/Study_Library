---
type: "practice"
difficulty: "Basic"
---

# What is a Python prompt template, and how would you write a simple function that fills placeholders like {question} and {context} into a template string?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Token Budgeting for RAG](<../../../../TOOLS/Python/Python_Language/Token_Budgeting_for_RAG.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A Python prompt template is a feature in certain libraries or frameworks, such as prompt-toolkit or rich, which allows you to create interactive command-line interfaces with placeholders that can be dynamically filled with values during runtime. These templates are particularly useful for creating interactive prompts where users can input information based on predefined structures.

To write a simple function that fills placeholders like {question} and {context} into a template string, we'll follow these steps:

Understand the Problem: We need to create a function that takes a template string with placeholders and a dictionary of values to fill those placeholders.
Define the Function Signature: The function will accept two parameters: the template string and a dictionary containing key-value pairs for the placeholders.
Implement the Template Filling Logic: Use Python's f-string formatting or str.format() method to replace the placeholders in the template with actual values from the dictionary.

Here’s how you can implement this:

PYTHON
def fill_template(template, data):
    # Replace {question} and {context} placeholders with their corresponding values
    filled_template = template.format(**data)
    return filled_template

This function uses Python's str.format() method to replace the {question} and {context} placeholders in the template string based on the provided dictionary. For example, if you have a template like:

PLAINTEXT
Question: {question}
Context: {context}

And a data dictionary like:

PYTHON
data = {
    'question': 'What is the capital of France?',
    'context': 'France is located in Europe.'
}

The function would output:

PLAINTEXT
Question: What is the capital of France?
Context: France is located in Europe.

