---
type: "practice"
difficulty: "Deep"
---

# How would you use git bisect with an automated test script to pinpoint the commit that broke a deployment?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To pinpoint the commit that introduced a regression in your codebase, git bisect is a powerful tool when combined with an automated test suite. Essentially, it works by using binary search to quickly narrow down which specific commit caused a failure. Here’s how you can use git bisect effectively:

Understanding What Git Bisect Does: Think of git bisect as a way to systematically try commits in the middle of your codebase history until you find the one that introduced a bug.
Setting Up Your Environment: Ensure that your development environment and deployment environment are consistent, as this helps avoid false positives from differing environments.

Now let's walk through how to use git bisect step by step:

Step-by-Step Guide

Identify the Known Good Commit:

Start with a known good commit (let’s call it good-commit-hash) where the automated tests pass.

Identify the First Bad Commit:

Identify the first bad commit (let’s call it bad-commit-hash) where the automated tests fail.

Run Tests on Both Commits:

Run your automated test suite on both commits to confirm their state.

#### Example
make test or running a specific script like ./run-tests.sh.

Start Bisect:

Use the command git bisect start.
Mark the known good commit as such with git bisect good good-commit-hash.
Mark the first bad commit as such with git bisect bad bad-commit-hash.

Run Tests and Let Git Select a Commit to Test:

Run your tests: make test or similar.
Once the tests fail, run git bisect bad. This tells git that the current commit is not good.

Repeat Until You Find the Bad Commit:

Git will automatically pick an intermediate commit for you to check.
After running your tests on this new commit and determining whether it passes or fails, mark it with git bisect good or git bisect bad.

Continue Narrowing Down Until the Culprit is Found:

Repeat steps 5 and 6 until git reports that only one commit remains as a suspect.

Verify the Suspect Commit:

Once you have a single commit, manually verify its state to ensure it is indeed the culprit.
If necessary, re-run tests or check logs to confirm.

#### Example
Workflow

Suppose your current branch has commits A, B, C, and D. You know that D introduced a bug because the automated tests fail there. However, you’re not sure exactly where in between A and D it happened. Here’s how you would start:

SH
git bisect start A D  # Start bisect with known good (A) and bad commit (D)
make test            # Run tests on the current state

# Assuming the tests fail, run:
git bisect bad        # Mark as bad
make test            # Re-run tests

Git will select a midpoint to check. If you mark this as good or bad, it continues narrowing down until:

SH
git bisect good
git bisect bad
# Repeat until the culprit commit is found.

git show <suspected-commit-hash>  # Verify the suspect commit

#### Closing Summary
and Common Mistakes

Using git bisect effectively requires a clear understanding of your codebase history. It’s easy to overlook steps like marking commits correctly or ensuring that tests are run consistently across different environments. Common mistakes include:

Not properly distinguishing between good and bad commits, leading to incorrect results.
Running tests without proper environment setup, which can mask real issues.
Failing to verify the suspect commit manually before moving on.

By following these steps methodically, you can efficiently narrow down the culprit of a regression in your codebase. Remember, git bisect is most effective when used with a well-defined and automated test suite that covers relevant scenarios.

**Code:**

```
SH
Copy
# Example commands for git bisect usage
git bisect start A D  # Start bisect with known good (A) and bad commit (D)
make test            # Run tests on the current state

# Assuming the tests fail, run:
git bisect bad        # Mark as bad
make test            # Re-run tests
```

