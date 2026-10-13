---
type: "practice"
difficulty: "Deep"
---

# How would you find which commit introduced a bug using Git?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the commit that introduced a bug, we need to understand how Git and its tools can help us trace changes in the codebase. This involves leveraging git bisect, which is a powerful tool designed specifically for finding the exact commit where an issue began. Let's break down the process from first principles.

Firstly, it's important to note that git bisect works by automatically dividing the range of commits between two given points and checking whether the bug exists in the midpoint commit. If the bug is present, we focus on the upper half; if not, we look at the lower half. This process continues until the specific commit causing the issue is found.

#### Assumptions

You have a version history with tagged or branch-based commits.
The bug was introduced between two known points (e.g., two tags or branches).

Now, let's go through the steps to use git bisect:

Identify the Range: Determine the first commit where you know the bug doesn’t exist and the last commit where it does. Typically, this could be between a stable tag and a more recent one.

Start Bisecting:

SH
git bisect start <bad-commit> <good-commit>

Replace <bad-commit> with the commit hash or tag where you know the bug is present, and <good-commit> with the commit hash or tag where the bug isn't.

Run the Test: At each midpoint commit, run your tests to see if the bug persists.

Continue Bisecting:

SH
git bisect good  # If the test passes
git bisect bad   # If the test fails

Complete Bisect:

SH
git bisect reset  # To exit bisect mode and return to the original branch
Full Example

Let’s say you have a bug introduced between two tags, v1.0 (good) and v2.0 (bad). Here’s how you’d use git bisect:

SH
# Start the bisect process
git bisect start v1.0 v2.0

# Check out the initial commit to test
git checkout -b bisect-branch

# Run your tests and determine if the bug is present
# If it's not, mark as good
git bisect good  # This runs a test and marks the current commit as "good"

# Continue bisecting until you find the bad commit

#### Closing Summary
+ Common Mistakes/Follow-Ups

git bisect is incredibly useful for pinpointing problematic commits in your codebase. However, it’s crucial to ensure that the tests run on each bisected commit are deterministic and isolated from external factors like environment variables or database states.

#### Common mistakes
include:

Not properly setting up the initial range.
Running non-deterministic tests that can yield false positives or negatives.
Failing to reset your working branch after completing the bisect process.

#### Follow-ups
could involve discussing strategies for managing branches during such debugging sessions, ensuring test environments are consistent across all commits, and integrating git bisect into automated testing pipelines.

