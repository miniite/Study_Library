---
type: "practice"
difficulty: "Deep"
---

# A Spark job's config and its transformation code both changed in one PR, and a reviewer can't tell whether a data-quality drop came from the shuffle-partition change or the join logic. How would you split concerns across commits so a bad output is bisectable to config or code?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to ensure that changes in configuration and transformation logic are isolated into distinct commits, making it easier to pinpoint the root cause of any data quality issues. The key idea is to maintain a clear separation of concerns using Git's branching model effectively.

Firstly, let’s understand what the question is probing:

Understanding Commit History: Reviewers need to be able to identify changes that may have introduced bugs or data quality issues.
Git Workflow: Proper use of branches and merges to manage code changes in a structured way.
Bisecting Changes: Being able to trace back changes using Git bisect.

#### Assumptions

The Spark job is part of a larger project that uses Git for version control.
We want to maintain the integrity and clarity of our commit history by isolating configuration and code changes into separate commits.
Step-by-Step Solution

Initial Setup:

Ensure your Git repository is clean, and you have the latest changes from the main branch or other relevant branches.

Create a Feature Branch for Config Changes:

BASH
git checkout -b config-updates main

Make the necessary configuration updates. Once done, commit these changes with a clear message:

BASH
git add conf.properties
git commit -m "Update Spark job configurations"

Create Another Feature Branch for Code Changes:

BASH
git checkout -b code-updates config-updates

Make the necessary transformation logic updates. Commit these changes similarly:

BASH
git add transformations.py
git commit -m "Refactor Spark job transformations"

Merge Both Branches Back to Main: After making both sets of changes, you can merge them back into main or another relevant branch.

BASH
git checkout main
git merge --no-ff code-updates -m "Merge config and code updates"

Here, the --no-ff flag ensures a merge commit is created, making it clear that these changes were intentional.

Run Tests and Verify Data Quality: After merging, run your tests to ensure no regressions occurred.

BASH
./gradlew test

If there are any issues, you can use Git bisect to find the exact commit causing the issue:

BASH
git bisect start <commit_id_before_merge> HEAD
git bisect bad
# Run your tests and identify if they fail. Repeat as necessary.

#### Closing Summary

By splitting configuration updates from code changes into separate commits, we ensure that each commit is logically isolated. This makes it much easier to use Git's bisect tool to trace back any issues directly to the relevant commit. Proper branching and merging practices help maintain a clean and understandable commit history.

#### Common Mistakes/Follow-Ups
Not Using Separate Branches: Failing to create distinct branches for configuration and code changes can lead to mixed-up histories where it’s difficult to pinpoint which change introduced an issue.
Ignoring Bisect Tool: Not using tools like git bisect when troubleshooting can make the process lengthy and error-prone.
Mixed Commits: Keeping all changes in a single commit without proper context makes debugging challenging.

**Comparison:**

| Step | Command                   | Description                                                                 |
|------|---------------------------|-----------------------------------------------------------------------------|
| 1    | `git checkout -b config-updates main` | Create a feature branch for configuration changes.                          |
| 2    | `git add conf.properties`   | Stage the updated configuration file.                                       |
| 3    | `git commit -m "Update Spark job configurations"` | Commit the configuration changes with a clear message.                      |
| 4    | `git checkout -b code-updates config-updates` | Create another feature branch for code changes, based on config updates.   |
| 5    | `git add transformations.py` | Stage the updated transformation logic file.                               |
| 6    | `git commit -m "Refactor Spark job transformations"` | Commit the code changes with a clear message.                             |

**Code:**

```bash
# Example Git commands for branching, committing, and merging
```

**Example input / output:**

```text
+ git checkout -b config-updates main
- # Make configuration updates
- git add conf.properties
- git commit -m "Update Spark job configurations"

+ git checkout -b code-updates config-updates
- # Refactor transformation logic
- git add transformations.py
- git commit -m "Refactor Spark job transformations"
```

**Data:**

```json
{
  "branch_name": "config-updates",
  "commit_message": "Update Spark job configurations"
}
```

**Flow:**

1. Initial Setup
2. Create config-updates branch
3. Make configuration updates,add conf.properties, commit
4. Create code-updates branchfrom config-updates
5. Make code changes, addtransformations.py, commit
6. Merge back to main with--no-ff flag
7. Run tests and verify dataquality
8. Bisect if issues arise

