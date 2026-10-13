---
type: "practice"
difficulty: "Basic"
---

# You accept a Copilot suggestion that looks right. Why should you still read it before committing?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Hallucinated APIs and Outdated Suggestions](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Hallucinated_APIs_and_Outdated_Suggestions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with AI-assisted coding tools like Copilot, accepting suggestions without reviewing them can lead to significant issues down the line. This is a fundamental concept in software development that emphasizes the importance of understanding and validating code. Let's break it down step by step.

#### Triage
What the Question Is Asking

The question is essentially probing why you should always review AI-generated suggestions, even when they appear correct on the surface. The core idea here is to understand the potential pitfalls of relying solely on AI without proper verification.

#### First-Principles Solution
Understanding AI Limitations: AI tools like Copilot are not infallible and can make mistakes or generate suboptimal code for various reasons, such as incorrect assumptions about context or limitations in their training data.
Code Context and Intent: Copilot might not always fully understand the broader intent behind your codebase. For instance, it could suggest a function name that is similar to an existing one but has subtle differences.

#### Performance Considerations
The suggested code may perform differently from what you originally intended or expected. This includes potential issues like memory leaks, performance bottlenecks, or unexpected behavior in edge cases.
Security Risks: AI-generated code might introduce security vulnerabilities that are not immediately obvious but can be critical if the codebase is handling sensitive data.

#### Cons
istency and Style: Copilot may suggest a coding style that does not match your team's conventions, which could lead to inconsistencies in the codebase over time.
Detailed Reasoning

Misunderstanding Context: AI tools like Copilot rely on understanding context from surrounding code, documentation, and other sources. Sometimes, these contextual elements can be misleading or insufficient for the tool.

#### Example
If you have a function that handles user authentication, Copilot might suggest a generic logging statement that could lead to confusion about its purpose.

Performance Impact: AI-generated code may not always account for performance optimizations that experienced developers are aware of.

#### Example
Copilot might suggest using a less efficient data structure or algorithm because it does not have the same level of insight into your specific use case and constraints as a human developer.

Security Flaws: AI tools can sometimes overlook security best practices, especially if they rely on generic suggestions rather than specialized knowledge.

#### Example
Copilot might suggest using a less secure method for handling sensitive data because it does not have the same level of understanding about potential security threats as an experienced developer.

#### Cons
istency and Readability: Consistent coding styles are crucial for maintainability and readability. AI tools may not always provide suggestions that align with your team's established conventions.

#### Example
If your team has a strict convention against using certain keywords in function names, Copilot might suggest something that could cause confusion or errors.

#### Closing Summary

In summary, while AI-assisted coding tools like Copilot can be incredibly helpful and save time, it is essential to review their suggestions. This practice ensures the code remains consistent, secure, performant, and aligned with team conventions. Always take a moment to understand why a suggestion was made and verify its correctness.

#### Common Mistakes/Follow-Ups
Mistake: Over-relying on AI-generated suggestions without verification.

#### Follow-Up
Question: How would you integrate code reviews into your workflow when using an AI-assisted coding tool?

**Code:**

```python
# Example of a potentially misleading suggestion from Copilot
def handle_user_input(user_data):
    # Copilot might suggest this, but it's not ideal
    print(f"User input processed: {user_data}")
```

