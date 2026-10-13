---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd add type hints and mypy in strict mode to a large untyped legacy Python codebase incrementally without blocking feature work.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Functions Modules and Reuse](<../../../../TOOLS/Python/Python_Language/Functions_Modules_and_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When integrating type hints and leveraging mypy in strict mode into an existing, untold Python codebase, the key is to do so incrementally while minimizing disruption to ongoing development. The goal is to gradually improve code quality and catch as many bugs as possible without introducing new ones.

#### Triage

The question asks for a strategy on how to introduce type hints and mypy in strict mode into an untyped legacy Python application, ensuring that feature work can continue uninterrupted. This involves understanding the current state of the codebase, choosing a sensible entry point, and setting up a gradual migration plan.

#### Assumptions
The existing codebase is large and complex.
There are no static type annotations currently in place.
mypy will be run with strict mode to enforce type checking rigorously.
Feature work is ongoing, so incremental changes must be manageable.
Step-by-Step Solution

Assessment

Start by identifying the codebase's structure and complexity.
Use tools like flake8 or pylint to identify common issues that may hinder the initial use of type hints.

Entry Point Selection

Begin with a well-defined module or function where types are likely to be straightforward and critical for correctness (e.g., functions that interact heavily with data, API endpoints).
Use a small but representative sample area to understand the impact of adding type annotations.

Setup mypy Configuration

Configure mypy to run in strict mode using a .mypy.ini file.
INI
# .mypy.ini
[mypy]
check_untyped_defs = True
disallow_any_generics = True
allow_untyped_calls = False
This configuration ensures that all functions are typed and no untyped calls are allowed.

Incremental Annotation

Start by adding type hints to the function signatures, then gradually add them to variables.
Use # type: ignore comments in areas where refactoring is complex or where the current state of the code makes typing non-trivial (e.g., third-party libraries).
PYTHON
def process_data(data: List[str]) -> str:
    # Some complex logic here
    return data[0]  # type: ignore

Run mypy and Fix Issues

Initially, run mypy without strict mode to catch all issues.
Address each issue systematically, ensuring that changes do not block ongoing feature work.

Iterative Refinement

As more areas are typed, re-run mypy with stricter configurations (e.g., disallow_untyped_defs = True).
Gradually increase the strictness of mypy settings to catch more issues while managing disruptions.
INI
# .mypy.ini (updated)
[mypy]
check_untyped_defs = True
disallow_any_generics = True
allow_untyped_calls = False
disallow_untyped_defs = True  # Increase strictness

Automated Testing

Integrate mypy into the CI/CD pipeline to ensure that type checking is part of every build.
Write unit tests for newly typed functions and modules.

#### Closing Summary
and Common Mistakes
Mistake 1: Failing to manage dependencies on untyped code. Always ensure new features can be developed without blocking type annotation work.
Mistake 2: Overwhelming the development team with too many changes at once, leading to productivity loss.
Mistake 3: Ignoring edge cases and complex interactions that may not be immediately obvious.

By following a structured approach, you can incrementally improve your Python codebase’s type safety without disrupting existing feature work. This process helps maintain the balance between development speed and code quality.

**Code:**

```
INI
Copy
# .mypy.ini example configuration
[mypy]
check_untyped_defs = True
disallow_any_generics = True
allow_untyped_calls = False
```

