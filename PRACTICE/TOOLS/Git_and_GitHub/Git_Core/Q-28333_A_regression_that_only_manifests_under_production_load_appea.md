---
type: "practice"
difficulty: "Deep"
---

# A regression that only manifests under production load appeared somewhere in 400 commits. Design a git bisect run script that spins up dependencies, and enumerate the pitfalls that make bisect converge on the wrong commit.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to design a robust git bisect script that can efficiently narrow down the source of a regression bug by systematically checking commits between two known good and bad states. The script will set up dependencies, run tests, and determine which commit introduces the issue. However, there are several pitfalls that can cause git bisect to converge on the wrong commit if not handled carefully.

#### Triaging the Question

The core of this question revolves around designing a git bisect script with robustness in mind. The interviewer is probing your understanding of version control systems, automation scripting, dependency management, and debugging techniques. Key assumptions include:

We have two known states: one commit (BAD) where the regression appears and another commit (GOOD) before the bug was introduced.
The script should handle dependencies efficiently to avoid manual setup and ensure consistent testing environments.

#### First Principles Solution
Set Up Dependencies: Automate the installation of necessary libraries, frameworks, and tools required for the project.
Run Tests: Execute tests in a controlled environment to determine if a given commit introduces the regression.
Handle Edge Cases: Ensure the script can handle different failure modes, such as network issues or unexpected test outcomes.
Step-by-Step Solution

Setup Dependencies

We need to install dependencies using a package manager like pip for Python projects or npm for JavaScript projects. This step should be automated.
BASH
# Example for Python project
pip install -r requirements.txt

Run Tests

Define a script that runs tests and checks the output to determine if the regression is present.
BASH
# Example test runner script
python3 run_tests.py

Bisect Script

Write a shell script or use git commands directly for bisecting commits.
BASH
#!/bin/bash

# Initialize bisect
git bisect start

# Set the initial good and bad commits
git bisect good $(git log --oneline --format=%h -1 --grep="initial commit hash")
git bisect bad $(git log --oneline --format=%h -1 --grep="commit hash with regression")

# Define a function to run tests for a given commit
run_tests() {
    echo "Checking out commit: $1"
    git checkout $1
    pip install -r requirements.txt  # Reinstall dependencies in case of changes
    python3 run_tests.py
    if [ $? -ne 0 ]; then
        return 1  # Regression detected
    else
        return 0  # No regression
    fi
}

# Run bisect
while ! git bisect good || ! git bisect bad; do
    commit=$(git rev-list --bisect ^$GIT_BISECT Bad $Good | tail -n +2 | head -n 1)
    if run_tests $commit; then
        git bisect good
    else
        git bisect bad
    fi
done

# Finish bisect
git bisect reset
Pitfalls and Convergence Errors
Dependency Management: Ensure that dependencies are always reinstalled to avoid stale state issues.

#### Cons
istent Test Environment: Make sure the testing environment is identical across all runs. Variations in the environment can cause false positives or negatives.
Network Issues: Automated tests might fail due to network issues, leading git bisect to incorrectly mark a commit as bad.
Unexpected Outcomes: Tests that return non-zero exit codes may not always indicate regression but could be flaky tests. Proper error handling is crucial.

#### Closing Summary

A well-designed git bisect script can significantly reduce the time spent debugging regressions by automatically checking commits and managing dependencies. However, it's essential to handle potential pitfalls such as inconsistent environments or flaky tests to ensure accurate convergence on the faulty commit.

#### Common Mistakes/Follow-Ups
Not handling dependency installation in a consistent manner.
Ignoring test flakiness which can lead to false positives/negatives.
Missing edge cases like network issues during automated testing.
Failing to reset git bisect properly after finding the culprit.

