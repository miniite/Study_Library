---
type: "practice"
difficulty: "Intermediate"
---

# Why is a .gitattributes entry useful for telling Git to treat .ipynb files or model binaries specially?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gitattributes and Merge Drivers](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitattributes_and_Merge_Drivers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A .gitattributes file in Git tells Git how to handle specific file types during operations like checkout, commit, and merge. For example, with a properly configured .gitattributes entry, you can ensure that certain files like Jupyter Notebook (.ipynb) files or model binaries are not accidentally committed to the repository. This is particularly useful for managing large binary files or ensuring consistent handling of file types.

The question asks specifically about treating .ipynb files and model binaries specially. Here’s a step-by-step explanation:

#### Triage
What's being Probed: The interviewer is probing your understanding of Git's capabilities and how to configure it for specific file types.
Key Assumptions: Assume that you are working on a project where Jupyter Notebooks (.ipynb files) and model binaries are critical but should not be committed directly into the repository due to size or sensitivity.
First Principles: Git handles different file types with default settings, but we can override these by configuring .gitattributes.

#### Solution

Understanding git attributes: These are instructions for Git that specify how you want certain file types handled during various operations such as checkout and commit.

#### Example
.gitattributes Entry:

PLAINTEXT
*.ipynb        text eol=lf
model.bin      diff=false

Explanation of Entries:

*.ipynb text eol=lf: This tells Git to treat all files with a .ipynb extension as text files (this is default, so it's included here for clarity) and specifies that line endings should be converted to LF on checkout.
model.bin diff=false: This disables the file contents from being compared during merge conflicts, which can save space in the commit history.

Handling Large Binary Files:

If you have large binary files like model binaries (.bin), you might want to exclude them from the repository entirely and manage their versioning elsewhere.

#### Example
PLAINTEXT
model.bin       export-ignore

Configuration in .gitignore: Ensure that these files are also excluded from committing by adding them to your .gitignore file:

PLAINTEXT
# .gitignore
*.ipynb
model.bin

Committing and Ignoring:

When you want to include a modified Jupyter Notebook in the commit, use git add --update or manually stage it.
For large binaries, ensure they are not committed at all.

#### Closing Summary

Using .gitattributes, you can fine-tune how Git handles specific file types. This is particularly useful for managing sensitive data like model binaries and Jupyter Notebooks. By properly configuring these settings, you maintain the integrity of your repository while ensuring that critical files are handled appropriately.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to include .gitignore rules that might prevent certain files from being ignored.

#### Follow-Up
Question: How do you ensure that non-binary Jupyter Notebook files (e.g., those containing code) are committed properly while excluding model binaries?

**Code:**

```plaintext
# Example .gitattributes file
*.ipynb        text eol=lf
model.bin      diff=false export-ignore
```

