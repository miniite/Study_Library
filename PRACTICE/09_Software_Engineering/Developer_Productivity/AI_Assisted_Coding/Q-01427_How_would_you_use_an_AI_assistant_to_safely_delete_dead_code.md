---
type: "practice"
difficulty: "Deep"
---

# How would you use an AI assistant to safely delete dead code, given the model can't fully know what's reachable via reflection, dynamic dispatch, or configs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Debugging with AI Assistants](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Debugging_with_AI_Assistants.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To safely delete dead code using an AI-assisted coding tool like Copilot/Cursor, we need to address several key challenges: understanding the program’s state, identifying unreachable code, and ensuring that critical paths aren’t inadvertently affected. The core idea is to leverage static analysis tools combined with a dynamic runtime check mechanism to confirm safety before deletion.

The question essentially probes how you can build a robust system that minimizes false positives (incorrectly flagged dead code) while also catching true dead code without disrupting the program’s functionality. Here are the steps and reasoning from first principles:

Static Analysis: Use static analysis tools to identify potential dead code. These tools analyze the source code, looking for unreachable or unused functions, variables, classes, etc.

Dynamic Analysis: Implement a runtime check mechanism to verify that the identified code is indeed dead under all possible execution paths. This can be done by temporarily disabling parts of the code and observing if the application still runs correctly.

Safety Checks via Configuration: Leverage configuration files or environment variables to control when these checks are performed, ensuring they don't interfere with production environments.

Incremental Testing: Run automated tests on modified code segments to ensure that functionality remains intact after deletion.

AI-Assisted Refactoring: Integrate AI tools like Copilot/Cursor to provide suggestions and automate the refactoring process, while still allowing human oversight for critical decisions.

By combining these approaches, we can create a more comprehensive solution that leverages both static and dynamic analysis techniques, ensuring that dead code is safely removed without introducing bugs or instability into the application.

**Comparison:**

```
Issue ID	Function Name	Line Number	Dry-Run Result
1234	unused_function	456	Successful
```

**Code:**

```python
# Example of integrating a static analysis tool with a runtime check

def static_analysis(file_path):
    # Run a static analysis tool to identify potential dead code
    issues = analyze_code(file_path)
    return issues

def dynamic_check(issues, config_env):
    # Implement a dynamic runtime check mechanism
    for issue in issues:
        if should_run_dry_run(issue, config_env):
            dry_run_result = perform_dry_run(issue)
            if not dry_run_result.successful:
                raise Exception(f"Runtime check failed: {issue}")
```

**Example input / output:**

```

Given the input configuration and issues, the system performs a dynamic check:

PLAINTEXT
Copy
Configuration:
- env: development
- run_dry_runs: true

Static Analysis Issues:
1. Unused function 'unused_function' at line 456

Dynamic Check Result:
- Run dry-run for issue_id: 1234 (unused_function)
- No runtime errors detected, safe to proceed with deletion.
```

**Data:**

```yaml
config_env:
  env: "development"
  run_dry_runs: true
issues:
  - issue_id: 1234
    line_number: 456
    function_name: "unused_function"
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is Static Analysis Enabled?
7. Run Static Analysis
8. Are Runtime ChecksConfigured?
9. Perform Dynamic Check
10. Delete Code if Safe
11. Mark as Potential Dead Code
12. No Action Required

