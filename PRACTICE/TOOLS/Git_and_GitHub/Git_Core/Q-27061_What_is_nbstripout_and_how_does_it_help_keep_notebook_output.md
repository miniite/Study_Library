---
type: "practice"
difficulty: "Intermediate"
---

# What is nbstripout and how does it help keep notebook outputs out of your Git history?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

nbstripout is a tool designed to remove non-source code content from Jupyter Notebook files before committing them to version control systems like Git. This helps maintain clean, reproducible commits by ensuring that large output data and temporary results do not clutter the repository history.

#### Triaging the Question

The question asks you to explain nbstripout and how it benefits Jupyter Notebook users in managing their Git history. The key aspects are:

Understanding what non-source code content means in the context of Jupyter Notebooks.
Explaining why this is beneficial for Git repositories, particularly in terms of reducing file size and improving commit clarity.
Step-by-Step Solution

What is nbstripout?

nbstripout is a command-line tool that can be used to clean Jupyter Notebook files by stripping out certain types of non-source code content. This includes output cells, raw cell magic, and execution counts.

Non-Source Code Content in Jupyter Notebooks:

In a Jupyter Notebook, the main source code is what you write for analysis or data processing. However, there are also "non-source" elements like:
Output cells containing results of running code (e.g., plots, tables, text).
Raw cell magic used to include raw text that's not part of the notebook's execution.
Execution counts indicating which cells have been run.

Why nbstripout is Beneficial for Git:

File Size Management: Large output files can significantly increase the size of your Git repository, making it slower and more cumbersome to clone or push changes.
Commit Clarity: Keeping only source code in commits makes them easier to read and review. It also ensures that any run results are reproducible when others check out the notebook.

How nbstripout Works:

When you commit your Jupyter Notebook, you can use nbstripout to remove these non-source elements before committing.
For example, if a cell with output data is modified and then deleted during review, running nbstripout notebook.ipynb will remove the content from that cell.

Integration with Git Hooks:

You can configure nbstripout to run automatically as part of your pre-commit or pre-push hooks in Git.
This ensures that all notebooks are cleaned before they get committed, making sure you don't accidentally commit large outputs.

#### Closing Summary
+ Common Mistakes
Summary: nbstripout is a useful tool for maintaining clean and reproducible Jupyter Notebook commits by removing non-source code content. By integrating it with Git hooks, you can ensure that your repository remains organized and efficient.

#### Common Mistakes
Forgetting to install or run nbstripout, leading to large output files being committed.
Assuming that all cells are source code and not stripping them before committing.

#### Example
Usage

To use nbstripout, you can simply run:

SH
pip install nbstripout

Then, for a single file:

SH
nbstripout notebook.ipynb

For automation in Git hooks, you might add something like this to your .git/hooks/pre-commit script:

SH
#!/bin/sh
git ls-files -z | xargs -0 nbstripout

