---
type: "practice"
difficulty: "Basic"
---

# How do you create a new branch and switch to it in a single command?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fetch vs Pull](<../../../../TOOLS/Git_and_GitHub/Git_Core/Fetch_vs_Pull.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a new branch and switch to it in a single command, you can use the git checkout -b command. This is a concise way to perform two steps that are often done sequentially: creating a new branch and then switching to it.

#### Triage

The question is asking for a Git command syntax to accomplish branching and switching simultaneously. The interviewer might be probing your familiarity with basic Git commands and how they can save time in workflow.

#### First-Principles Solution

Understanding the Command:

git checkout is used to switch branches or to create a working tree for an existing branch.
-b (short for --branch) allows you to create a new branch and then check out that branch at the same time.

Syntax of the Command: The command format is: git checkout -b <new-branch-name>.

#### Example
Usage: If you want to create a new branch called "feature-update" and switch to it immediately, you would run:

BASH
git checkout -b feature-update

#### Closing Summary
+ Common Mistakes/Follow-Ups

This command is very useful in daily development as it simplifies the process of creating and switching branches. A common mistake might be forgetting that git checkout can create a new branch, leading to confusion with other uses of this command.

#### Follow-up
questions could include:

How does this work internally?
What are some alternatives or related commands?
How do you handle conflicts when branching?

