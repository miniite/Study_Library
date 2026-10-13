---
type: "practice"
difficulty: "Basic"
---

# How would you use Cursor to add a requirements.txt for an ML project, and why should you pin library versions?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Agentic Coding with Cursor](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Agentic_Coding_with_Cursor.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To use Cursor (an AI-assisted coding tool) in an ML project, one of the first steps is adding a requirements.txt file. This file lists all the necessary libraries and their specific versions that your project depends on. Pinning library versions ensures consistency across different development environments and helps avoid version mismatches that can lead to bugs.

To add a requirements.txt using Cursor, you would likely start by creating or modifying an existing cursor in your codebase where it's appropriate (e.g., within the __init__.py file of a project). The tool will then suggest adding the necessary lines to this file. Here’s how you can do it step-by-step:

Identify Required Libraries: Determine which libraries your project needs, such as TensorFlow, NumPy, pandas, etc.
Determine Specific Versions: Use Cursor to add these libraries with specific versions (e.g., tensorflow==2.8.0).

#### Cons
istency Across Environments: By pinning the exact versions, you ensure that all developers and production environments have identical dependencies.

Now let’s dive into a more detailed explanation of why this is important and how to do it in practice.

