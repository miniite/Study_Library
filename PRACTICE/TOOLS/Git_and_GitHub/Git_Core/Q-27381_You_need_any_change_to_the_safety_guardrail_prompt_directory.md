---
type: "practice"
difficulty: "Deep"
---

# You need any change to the safety/guardrail prompt directory to require a review from the trust-and-safety team. How do you express that with CODEOWNERS path rules and what breaks if a glob is wrong?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure any changes to the safety/guardrail prompt directory require review by the trust-and-safety team, we can leverage GitHub's CODEOWNERS mechanism. The goal here is to define path-specific rules that enforce this requirement. Here’s a detailed explanation and solution:

TL;DR: We will use the .github/CODEOWNERS file to specify which teams or individuals should review changes in the safety/guardrail directory.

#### Triage
The question is probing how to set up path-specific rules using GitHub's CODEOWNERS feature and understanding the potential issues that arise from incorrectly configured globs (file patterns).

#### First-Principles Solution
Define the path in .github/CODEOWNERS.
Ensure the glob is correctly specified.
Test the setup to verify it works as intended.
Step-by-Step Solution

Identify the Directory: The directory we want to enforce review for is safety/guardrail.

Configure the CODEOWNERS File:

Navigate to the .github directory in your repository.
Create or edit the CODEOWNERS file if it doesn't exist.
Add a line specifying that the trust-and-safety team should review changes to the safety/guardrail directory. The exact line would look like this:
PLAINTEXT
@trust-and-safety-team safety/**/guardrail/**
This tells GitHub that any file under safety/guardrail in your repository needs a review by members of the trust-and-safety-team.

Test the Setup:

Make a small change to a file within the safety/guardrail directory.
Push the changes and observe the GitHub UI for the pull request (PR) creation prompt. Ensure it correctly requires approval from the specified team.

#### Common Mistakes
and Follow-Ups

Incorrect Glob Pattern: A common mistake is using an overly broad or too specific glob pattern, which can lead to unintended file coverage.

#### Example
of a mistake: @trust-and-safety-team safety/guardrail/* might include files outside the intended directory.

#### Example
of a fix: Ensure the glob matches exactly what you intend, such as safety/**/guardrail/**.

Misconfigured TEAM Name: Double-check the name of the team in your CODEOWNERS file to ensure it is correctly spelled and referenced.

Testing with Different Scenarios:

Test with a nested directory structure: Ensure that changes in subdirectories like safety/guardrail/subdir are also covered.
Test with non-matching files: Make sure other unrelated directories do not trigger the same review process.

#### Closing Summary

In summary, using GitHub's CODEOWNERS file correctly configured can enforce strict review processes for specific directories. Ensuring accurate glob patterns and thorough testing are crucial to avoid unintended behavior and ensure compliance with safety protocols. Mistakes in this setup can lead to security vulnerabilities or unnecessary friction during development.

**Code:**

```plaintext
# Example .github/CODEOWNERS file content
@trust-and-safety-team safety/**/guardrail/**
```

