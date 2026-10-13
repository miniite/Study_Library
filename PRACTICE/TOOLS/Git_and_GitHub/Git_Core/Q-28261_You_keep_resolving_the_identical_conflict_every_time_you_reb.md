---
type: "practice"
difficulty: "Deep"
---

# You keep resolving the identical conflict every time you rebase a long-lived branch onto main. How does rerere help, how do you enable it, and what are its failure modes on subtle conflicts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rerere and Repeated Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rerere_and_Repeated_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve recurring merge conflicts during rebases, rerere is an invaluable tool in Git that records resolutions from previous merges to automatically apply them when similar conflicts arise again. Enabling rerere allows you to streamline your workflow and avoid redundant manual conflict resolution. However, it's important to understand its limitations, especially with subtle or complex conflicts.

First, let’s break down what rerere is and how it works:

TL;DR: rerere records the outcome of merge conflicts in previous merges, allowing Git to apply those resolutions automatically when faced with similar conflicts again.

#### Triage
The key aspects we need to cover are enabling rerere, understanding its functionality, and identifying potential failure modes.
Enabling Rerere

To enable rerere, you simply run:

SH
git config --global rerere.enabled true

Alternatively, you can enable it per repository by running the command within that repository’s root directory. Once enabled, Git will record conflict resolutions if they are explicitly committed or added to the index using git add ..

How Rerere Works

When a merge conflict occurs during a rebase, Git attempts to resolve the conflicts based on previously recorded resolutions. Here's how it handles different scenarios:

Recording Resolutions:

If you manually resolve a conflict and commit the changes, rerere records this resolution.
You can also use git rerere record to explicitly mark a manual merge as resolved.

Automatically Applying Resolutions:

During subsequent merges, if Git encounters conflicts that match previously recorded resolutions, it will attempt to apply those same resolutions automatically.
Failure Modes

While rerere is incredibly useful, it has its limitations and potential pitfalls:

Complex or Subtle Conflicts: Rerere may not work well with complex or subtle conflicts where the previous resolution might not be directly applicable. For example, if two different conflict resolutions are needed depending on context.
Incorrect Manual Resolutions:
If you manually resolve a conflict incorrectly and commit those incorrect changes, rerere will apply this incorrect resolution in future merges, leading to further issues.
Different Merge Scenarios: Conflicts can sometimes vary significantly based on the merge scenario or branch history. Rerere may not be able to handle these variations effectively.
Practical Example

Suppose you have a conflict during a rebase that requires choosing between two files:

DIFF
<<<<<<< HEAD
file1.txt: line 20
=======
file1.txt: line 25
>>>>>>> main

If you manually resolve this by keeping the main version and commit it, Git records this resolution. If the next time a similar conflict arises due to a different merge scenario but with the same files, Git might apply this resolution again.

#### Common Mistakes
Overreliance on Rerere: Always review manual resolutions before committing to ensure correctness.
Not Handling Complex Conflicts: Be aware that rerere may not handle complex or subtle conflicts effectively and be prepared to manually resolve them.
Incorrect Recording of Resolutions: Ensure that you are recording the correct resolutions, especially when dealing with more intricate merge scenarios.

#### Closing Summary

In summary, while rerere is a powerful tool for streamlining conflict resolution during rebases, it’s crucial to understand its limitations and use it judiciously. By enabling rerere, recording explicit resolutions, and being mindful of potential failure modes, you can significantly improve your workflow and efficiency.

#### Common Mistakes
Overrelying on rerere without reviewing manual resolutions.
Failing to handle complex or subtle conflicts effectively.
Incorrectly recording conflict resolutions, leading to propagation of errors.

**Code:**

```
SH
Copy
git config --global rerere.enabled true
```

