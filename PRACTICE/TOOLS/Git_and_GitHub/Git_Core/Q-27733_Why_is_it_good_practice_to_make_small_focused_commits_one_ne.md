---
type: "practice"
difficulty: "Basic"
---

# Why is it good practice to make small focused commits (one new table, one new index) rather than one giant commit for a whole schema?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, making small, focused commits is considered best practice because it makes your codebase easier to understand, review, and maintain. Each commit should represent a clear, self-contained change—like adding a single table or creating an index. This approach contrasts sharply with committing large changesets that may include multiple database modifications at once.

Let's break down why small focused commits are beneficial:

#### Triage
What the Question is Really Asking

The question is asking about the advantages of breaking down larger work into smaller, more manageable chunks when using Git. The interviewer might be probing your understanding of version control principles and how they apply to database development.

Step-by-Step Solution from First Principles
Maintainability: Smaller commits make it easier to understand what has changed between any two points in time. For example, if a bug is introduced, having smaller commits allows for quicker isolation of the problematic change.
Revertibility and Rollbacks: If you need to undo changes or roll back a database schema, having small, focused commits makes this process smoother. It's easier to revert just one table or index than an entire set of modifications.
Collaboration and Code Review: When working in a team environment, smaller commits facilitate better collaboration and code reviews. Team members can review specific changes without being overwhelmed by extensive diffs.
History Clarity: A history with small, focused commits provides a clearer picture of the development process. It’s easier to trace the evolution of your schema over time when each commit represents a discrete change.

#### Triage
Recap

In summary, making small, focused Git commits is beneficial because it enhances maintainability, facilitates reversion and rollback, promotes better collaboration through clearer diffs, and creates a cleaner project history. These benefits are particularly crucial in database development where changes can have significant impacts on the application's functionality.

#### Common Mistakes
and Follow-Ups
Large Committing Habit: Some developers might be tempted to make large commits because they believe it saves time. However, this habit often leads to confusion and difficulty in troubleshooting.
Lack of Documentation: While small commits are helpful, it’s also important to document why a particular change was made. This can be achieved through commit messages that provide context.

#### Closing Summary

By adopting the practice of making small, focused commits, you ensure that your Git history is clear and maintainable, which in turn helps in managing database changes more effectively and collaboratively.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Make Small Commits] --> B[Maintainability];
    A --> C[Revertibility & Rollbacks];
    A --> D[Collaboration & Code Reviews];
    A --> E[Clear History];
```

**Flow:**

1. Make Small Commits
2. Maintainability
3. Revertibility & Rollbacks
4. Collaboration & Code Reviews
5. Clear History

