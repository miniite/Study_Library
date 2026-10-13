---
type: "practice"
difficulty: "Intermediate"
---

# What does 'squash and merge' do on a GitHub pull request and when would you use it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Squashing and merging in Git is a technique where multiple commits are combined into a single commit during a pull request (PR) merge. This process helps to maintain a clean history, reducing the number of commits on your master branch. You would use squashing and merging when you want to simplify the PR's history, making it easier to review and understand.

#### Triaging the Question

The question is asking for an explanation of what 'squash and merge' does in GitHub and when this method should be used. The core concepts being probed include understanding commit histories, pull request workflows, and the benefits and drawbacks of different merging strategies.

#### Assumptions
You have a basic understanding of Git workflow.
You are familiar with using GitHub for version control.

#### First-Principles Solution

When you create a PR, it usually contains multiple commits reflecting changes over time. Squashing merges those commits into a single commit before the merge is completed. This process can be done through various methods in GitHub, such as Squash and Merge or by using Git commands like git rebase.

Here’s a step-by-step breakdown:

Initiate the PR: You start working on a feature branch, making several commits.
GitHub UI Option: On the GitHub interface for the PR, you can select "Squash and merge" to combine all commits into one commit before merging them into the main branch.
Git Command Example: Alternatively, using Git commands like git rebase -i on your feature branch will allow you to interactively squash commits and then push them.
When to Use Squashing:
Clean History: When the PR’s history needs simplification for a cleaner commit log.
Feature Development: During iterative development where intermediate steps should be ignored, such as temporary debugging branches.
Code Review Simplification: To make code reviews easier by condensing multiple commits into fewer, more meaningful ones.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Squashing and merging is a powerful feature that can significantly clean up your project history. However, it should be used judiciously:

Avoid Misuse: Do not squash important changes or commit messages.
Conflict Resolution: Be prepared to handle conflicts if squashed commits contain changes in the same files.
Communication: Ensure team members are aware that you will use this technique to avoid confusion.

#### Common Mistakes
Forgetting to communicate with your team about using squashing, leading to misunderstandings.
Squashing critical commit messages or important change details.
Overusing squashing in PRs where it’s not necessary, cluttering history unnecessarily.

#### Example
Workflow

If you have a feature branch feature/new-feature with multiple commits:

CODE
git checkout feature/new-feature
# Perform some changes and commit them
git rebase -i HEAD~5
# Use an interactive editor to squash the last 5 commits into one.
git push --force-with-lease

This would result in a clean, single-commit PR on GitHub.

**Code:**

```bash
# Example Git commands for squashing and rebasing
git checkout feature/new-feature
git rebase -i HEAD~5
# Interactive editor pops up. Choose squash or fixup to combine commits.
git push --force-with-lease
```

**Example input / output:**

```text
Input:
- A PR with multiple commits on a feature branch `feature/new-feature`

Output:
- A single commit in the main branch after squashing and merging

Example Request:
- "I have made several changes to add new features. Please review the PR."

Example Response:
- After squashing, response: "The PR has been updated to include all changes in a single commit for better code review."
```

**Flow:**

1. Initiate PR with multiplecommits
2. Squash and merge option onGitHub
3. Select 'Squash and Merge' oruse Git commands like rebase
4. Merge into main branch with asingle commit
5. Clean history, easier codereview

