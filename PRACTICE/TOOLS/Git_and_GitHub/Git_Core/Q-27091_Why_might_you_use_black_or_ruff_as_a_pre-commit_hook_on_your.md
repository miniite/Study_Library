---
type: "practice"
difficulty: "Intermediate"
---

# Why might you use black or ruff as a pre-commit hook on your ML code, and what does that enforce for the team?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pre-commit Hooks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pre-commit_Hooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with machine learning (ML) code in a collaborative environment, using Git hooks like black and ruff as pre-commit hooks can significantly enhance code quality, consistency, and maintainability. A pre-commit hook runs automatically before each commit is pushed to the repository. By configuring these tools, you ensure that your code meets certain standards or formatting rules before it's added to version control.

#### Triage

The question is asking about the benefits of using black for code formatting and ruff for static analysis as pre-commit hooks in an ML project. The key idea here is understanding why these tools are beneficial, what they enforce on a team, and how they contribute to good software engineering practices.

#### First-Principles Solution
Using Black for Code Formatting
Enforced Consistency: black is an opinionated code formatter that enforces consistent formatting rules across all Python files in the project.
Reduced Merge Conflicts: By automatically formatting code to a uniform style, conflicts during merge operations are minimized as developers can focus on actual logic changes rather than formatting disagreements.
Using Ruff for Static Analysis
Automated Linting: ruff is a Python linter that checks your code against various rules and standards, catching potential issues early.
Code Quality Assurance: By ensuring the code adheres to best practices, it helps maintain high-quality code and reduces bugs.
Triaging Further

#### Assumptions
The team uses Python as the primary programming language for ML models.
Scope: These tools are configured in a .git/hooks/pre-commit file to run automatically before each commit.
Detailed Explanation
Why Black?

Black’s consistent formatting can be seen as a type of "code hygiene" that ensures everyone is on the same page, reducing cognitive load when reading or maintaining code. For example:

PYTHON
def function_name(arg1, arg2):
    if condition:
        return result

becomes:

PYTHON
def function_name(arg1, arg2) -> None:
    if condition:
        return result

This transformation might seem minor, but it ensures that all code follows the same style guide, making it easier for new team members to understand and contribute.

Why Ruff?

Ruff’s static analysis capabilities catch issues early in the development process. For instance, consider this snippet:

PYTHON
x = 5
y = "2"
z = x + y

ruff would flag an issue because it detects a type mismatch between int and str. Such linting ensures that common bugs are caught before they reach production.

#### Closing Summary

Incorporating black and ruff as pre-commit hooks in your ML project enforces consistent code formatting and static analysis, respectively. This not only improves the overall quality of the code but also enhances team collaboration by reducing merge conflicts and catching errors early. By automating these checks, developers can focus more on writing effective models rather than dealing with low-level coding issues.

#### Common Mistakes/Follow-Ups
Resistance to Change: Some developers might resist such enforcements because they find it restrictive or prefer their own style.
Misconfiguration: If the hooks are not properly set up, they could fail silently or interfere with the commit process.
Over-Reliance on Linters: While linters help catch errors, they should not replace proper testing and code review processes.

**Code:**

```bash
# Example pre-commit hook configuration in .git/hooks/pre-commit
#!/bin/sh

black .
ruff --fix .
```

