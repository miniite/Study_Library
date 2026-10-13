---
type: "practice"
difficulty: "Intermediate"
---

# What does git blame do and how is it useful during debugging?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

git blame is a powerful command in Git that helps identify who made specific changes to lines of code, which can be incredibly useful when trying to debug issues or understand the history and evolution of your codebase.

#### Triaging the Question

The core question here is about understanding what git blame does and why it's valuable during debugging. The interviewer might want you to demonstrate knowledge of Git's inner workings and explain how this command can help with both tracing changes and resolving bugs more efficiently.

Key Assumptions:
You are familiar with basic Git commands.
You understand the context in which git blame is useful, particularly for debugging.
Step-by-Step Solution

Basic Usage:

The git blame command displays who last modified each line of a file and the commit ID associated with that change.
BASH
git blame <filename>

#### Example
BASH
git blame index.html

Output Explanation:

Each line in the output shows the commit hash, author name, and date of the last modification to that line.
For example:
CODE
f4b8e14a (John Doe 2023-10-01 15:29:30 +0200 1) <!DOCTYPE html>
b7f6c3d5 (Jane Smith 2023-09-30 14:05:02 +0200 2) <html lang="en">

How It Helps in Debugging:

Identifies the last person who changed a particular line of code, which can be crucial for understanding context and intention.
Helps track down bugs by pinpointing where certain changes were made.
Useful when you need to review recent modifications or understand why certain parts of the code behave differently.

Detailed Example:

Suppose you encounter an unexpected behavior in index.html. Running git blame index.html could reveal that the problematic line was last changed by Jane Smith on 2023-09-30.
BASH
git blame index.html | grep "<script src='invalid.js'>"
This can help you identify not only who made the change but also why and under what circumstances.

Advanced Usage:

You can use --show-blame to see a graphical representation of changes.
Use --reverse to trace changes backwards from recent modifications.
BASH
git blame --reverse <filename>

#### Closing Summary

In summary, git blame is an essential tool for debugging and understanding the history of your codebase. It provides insights into who made specific changes, making it easier to track down bugs and understand the evolution of your code.

#### Common Mistakes
and Follow-Ups

Mistake: Confusing git blame with other Git commands.

#### Follow-Up
Highlight the differences between blame, log, and show.

Mistake: Overlooking the context provided by commit messages.

#### Follow-Up
Emphasize the importance of reading commit messages for full context.

**Code:**

```bash
# Example git blame command
git blame index.html | grep "<script src='invalid.js'>"
```

