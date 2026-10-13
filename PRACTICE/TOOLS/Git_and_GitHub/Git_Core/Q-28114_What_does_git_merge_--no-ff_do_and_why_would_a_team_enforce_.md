---
type: "practice"
difficulty: "Intermediate"
---

# What does git merge --no-ff do and why would a team enforce it for merging feature branches?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're diving into git merge --no-ff, which is a common Git command that ensures every branch has its own history with clear merge commits. This approach provides transparency in version control by keeping the commit history clean and explicit.

The question probes your understanding of Git's branching strategies and best practices for maintaining a clear commit history, especially when merging feature branches into the main or development branches. We'll start from first principles to understand why teams might enforce this rule and how it works under the hood.

TL;DR

git merge --no-ff merges two branches without fast-forwarding, ensuring that each branch has its own commit history with a clear merge commit. Teams often enforce this practice to maintain an explicit version control history and prevent accidental fast-forwards.

#### Triaging the Question

The interviewer is probing your knowledge of Git's branching strategies, particularly the differences between fast-forward merges and non-fast-forward (or forced) merges. The concept revolves around understanding why teams might want to have a clear commit history and how --no-ff helps achieve that.

#### Assumptions
You understand basic Git commands like git merge and git checkout.
You know the difference between fast-forward and non-fast-forward merges.
You are familiar with feature branches, master/main branch merging, and the importance of commit history in version control.
Step-by-Step Solution
1. Understanding Fast-Forward Merges

When you use git merge without any additional flags, Git tries to perform a fast-forward merge by moving one branch's tip pointer to match another branch's tip. This process is transparent and doesn't create new commits; instead, it simply adjusts the history.

BASH
# Example of a fast-forward merge
$ git checkout main
$ git merge feature-branch

# Main history now shows just one commit for this merge
2. Non-Fast-Forward Merges with --no-ff

When you use git merge --no-ff, Git creates a new merge commit even if the branch can be fast-forwarded. This ensures that each branch has its own history, making it easier to trace changes and understand the project's development flow.

BASH
# Example of a non-fast-forward merge with --no-ff
$ git checkout main
$ git merge --no-ff feature-branch

# Main history now shows two commits: one for merging feature-branch and another for fast-forwarding existing commits.
3. Why Teams Enforce --no-ff

Teams enforce this practice because it provides a more detailed and explicit commit history, which is beneficial in several ways:

Auditability: It makes it easier to track the origin of changes and who contributed what.
Clarity: Merge commits provide clear markers for feature completion and integration points.
Conflict Resolution: In case conflicts arise during merges, having a merge commit helps identify where issues started.
4. Practical Example

Let's walk through an example:

BASH
# Initial state: main branch is clean
$ git checkout -b feature-branch
$ # Work on the feature and add commits
$ git commit -m "Add new feature"
$ git checkout main
$ # New changes are made in main during this period
$ git merge --no-ff feature-branch

# The resulting history will look like:
# main
#  |
#  |-> (commit for merging feature branch)
#  |   |
#  |--> latest commit on main
#     |
#     (commits from working on the feature)

#### Closing Summary
and Common Mistakes/Follow-Ups

In conclusion, git merge --no-ff is a powerful tool that ensures clear history by creating explicit merge commits. Teams often enforce this practice to maintain auditability and clarity in their version control systems.

#### Common Mistakes
Forgetting the Flag: Accidentally performing fast-forward merges.
Misunderstanding History: Not recognizing the benefits of detailed commit histories.

#### Follow-Ups

You might be asked about alternatives or trade-offs, such as:

How does --no-ff compare to git pull --rebase?
What are the implications for automated deployment pipelines and continuous integration?

**Code:**

```bash
# Example of a fast-forward merge
$ git checkout main
$ git merge feature-branch

# Example of a non-fast-forward merge with --no-ff
$ git checkout main
$ git merge --no-ff feature-branch
```

**Example input / output:**

```text
# Initial state: main branch is clean
$ git checkout -b feature-branch
$ # Work on the feature and add commits
$ git commit -m "Add new feature"
$ git checkout main
$ # New changes are made in main during this period
$ git merge --no-ff feature-branch

# The resulting history will look like:
main
 |
 |-> (commit for merging feature branch)
 |   |
 |--> latest commit on main
    |
    (commits from working on the feature)
```

**Flow:**

1. Yes
2. No
3. Start
4. Is merge --no-ff enforced?
5. Merge with new commit
6. Perform fast-forward merge
7. End with clear history
8. May end up with less detailedhistory

