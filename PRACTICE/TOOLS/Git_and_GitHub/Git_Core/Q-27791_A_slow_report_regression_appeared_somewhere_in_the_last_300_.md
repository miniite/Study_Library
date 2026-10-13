---
type: "practice"
difficulty: "Deep"
---

# A slow report regression appeared somewhere in the last 300 commits and each candidate needs a schema apply plus a benchmark run. How would you script git bisect run around a provision-migrate-benchmark loop, and what makes bisect give a wrong answer here?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To identify the commit that introduced a slow report regression in your database codebase, we can use git bisect. This powerful tool helps narrow down the exact commit where a bug or performance issue first appeared by iteratively testing the mid-point commits of the range. Given that each candidate needs to apply a schema change and run benchmarks, scripting this process effectively is crucial.

git bisect works best when you have a binary search space; in our case, it's the 300 recent commits where the issue might be introduced. We'll use git bisect run to automate the testing of each commit by applying schema changes and running benchmarks.

Here’s how we can script this:

Initialize Bisect: Start by marking the range using git bisect start followed by git bisect bad for the current state (assuming it's known that the issue is present in the latest version).
Define the Loop: We'll create a loop to apply schema changes and run benchmarks at each mid-point commit.
Automate with Bisect Run: Use git bisect run to automate running tests, which will help us quickly identify the problematic commit.

