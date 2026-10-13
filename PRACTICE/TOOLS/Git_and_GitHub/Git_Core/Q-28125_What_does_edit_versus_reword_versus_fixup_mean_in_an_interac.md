---
type: "practice"
difficulty: "Intermediate"
---

# What does 'edit' versus 'reword' versus 'fixup' mean in an interactive rebase todo list?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interactive Rebase](<../../../../TOOLS/Git_and_GitHub/Git_Core/Interactive_Rebase.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git, particularly during an interactive rebase, you might encounter commands like edit, reword, and fixup. These commands are part of the interactive rebase todo list, which allows you to modify commit history interactively. Here’s a brief overview:

TL;DR: During an interactive rebase, edit is used to return a specific commit for editing, reword is used when you want to change the commit message while keeping the same changes, and fixup is used to automatically squash one commit into another.

Now let's dive deeper:

#### Triage
What are We Probing?

The question is asking about the distinctions between these commands in an interactive rebase context. The interviewer might be probing your understanding of Git workflows, specifically around modifying commit history interactively. They may also want to ensure you can explain how and why each command differs.

Step-by-Step Solution from First Principles
1. Understanding Interactive Rebase

Interactive rebase is a powerful feature in Git that allows you to rewrite commits as if they were created after the last successful merge. It's commonly used for cleaning up commit history, especially when preparing a series of commits for public release.

2. The edit Command
Purpose: You use edit when you need to return to a specific commit and make additional changes.
Use Case: Suppose you made a commit but realized there were mistakes or incomplete work in the commit that you need to fix before proceeding with the next commits. By using edit, Git will stop processing the rebase at this point, allowing you to amend the commit.
3. The reword Command
Purpose: You use reword when you want to change the message of a specific commit but not its content.
Use Case: Imagine you made a commit and forgot to include some important information in the commit message. By using reword, Git will stop at that commit, allowing you to update the message without changing the code.
4. The fixup Command
Purpose: You use fixup when you want to squash one small fix into another commit.
Use Case: Suppose you made two consecutive commits: one where you fixed a small typo and another with more significant changes. By using fixup, Git will combine these two commits, effectively removing the minor fix from history.

#### Closing Summary

In summary:

Use edit when you need to go back and make further changes to a specific commit.
Use reword when you want to change the commit message without altering any code.
Use fixup to combine two commits, typically for small fixes that can be squashed into another significant commit.

#### Common Mistakes
and Follow-Ups

#### Common mistakes
include:

Using edit unnecessarily or incorrectly, which can lead to unnecessary complexity in your rebase process.
Overusing reword when a simple amend would suffice.
Not understanding the implications of using fixup, such as how it affects commit history.

#### Follow-up
questions might include scenarios where you'd use these commands together and what impact they have on public repositories versus private ones.

**Comparison:**

| Command | Description | Use Case |
|---------|-------------|----------|
| edit    | Return to a specific commit for further editing | When you need to make additional changes after creating a commit |
| reword  | Change the message of a specific commit without altering code | When you want to correct an error in the commit message |
| fixup   | Squash one small fix into another commit | To combine two commits, typically when one is a minor fix for the other |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Start Interactive Rebase] --> B{Commit to Edit?};
    B -- Yes --> C[Edit Commit];
    B -- No --> D{Commit to Reword?};
    D -- Yes --> E[Reword Commit];
    D -- No --> F{Commit to Fixup?};
    F -- Yes --> G[Fixup Commit];
    F -- No --> H[Continue Rebase];
```

**Example input / output:**

```text
Interactive Rebase:
1. git rebase -i HEAD~3

Todo List:
pick abc123 Initial commit with bug
edit def456 Commit with typo fix
pick ghi789 Final commit with significant changes

# After making the necessary changes
git rebase --continue
```

**Data:**

```json
{
  "commitHistory": [
    {
      "id": "abc123",
      "message": "Initial commit with bug"
    },
    {
      "id": "def456",
      "message": "Commit with typo fix"
    },
    {
      "id": "ghi789",
      "message": "Final commit with significant changes"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start Interactive Rebase
8. Commit to Edit?
9. Edit Commit
10. Commit to Reword?
11. Reword Commit
12. Commit to Fixup?
13. Fixup Commit
14. Continue Rebase

