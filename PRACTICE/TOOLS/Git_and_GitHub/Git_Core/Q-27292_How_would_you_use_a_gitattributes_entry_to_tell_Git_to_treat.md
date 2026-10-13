---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a .gitattributes entry to tell Git to treat your .ipynb prompt-experiment notebooks specially so diffs are cleaner?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gitattributes and Merge Drivers](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitattributes_and_Merge_Drivers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand how .gitattributes can be used to customize Git's behavior with specific file types. Specifically, we want to ensure that when working with Jupyter Notebook files (.ipynb), Git treats them in a way that makes diffs cleaner and more readable.

Git attributes allow us to set special rules for handling files based on their extensions or paths. In this case, we can instruct Git to ignore certain parts of the notebook during diffs by marking those cells as "binary" content, which means they won't be diffed line-by-line but will still be version-controlled.

Let's break down the steps and rationale behind setting up a .gitattributes file for Jupyter Notebooks:

Identify the Parts to Ignore: In an IPython Notebook (.ipynb), cells containing raw Markdown, code, or output can often generate large diffs that are uninformative. We want to ignore these parts but keep track of changes in other areas like cell metadata.

Define the .gitattributes Entry: We will mark specific cells as "binary" content using a .gitattributes entry.

Here is the step-by-step process:

#### Triage
The question asks how to use .gitattributes entries for Jupyter Notebooks.
Git attributes provide fine-grained control over file handling, allowing us to instruct Git on what parts of files should be ignored during diffs.
We need to identify which parts of the .ipynb files can generate large diffs and mark them as binary.

#### Solution
Understand Jupyter Notebook Structure: A Jupyter Notebook is a JSON file containing cells, each with a type (code, markdown), metadata, and cell content. The cells are stored under an "cells" key in the main JSON object.
Create .gitattributes Entry: We need to instruct Git that we want certain parts of the notebook ignored during diffs.
BASH
# Create or edit .gitattributes in your repository root
*.ipynb diff=ipydiff
Define Custom Diff Driver: Next, we define a custom diff driver in .gitconfig which will handle the ipydiff type by ignoring specific cells.

Edit your global .gitconfig file:

INI
[diff "ipydiff"]
    textconv = python -c "import ipynb_diff; print(ipynb_diff.diff_cell_content('$@'))"

The ipynb_diff Python script (which we need to create) will be responsible for generating diffs that ignore specific cell types.

Custom Script: ipynb_diff.py

Here's a basic implementation of the diff script:

PYTHON
def diff_cell_content(filename):
    import json, sys

    with open(filename, 'r') as f:
        notebook = json.load(f)

    # Example logic to ignore certain cells (e.g., output cells)
    for cell in notebook['cells']:
        if cell['cell_type'] == 'code' and 'outputs' in cell:
            del cell['outputs']

    with open(filename, 'w') as f:
        json.dump(notebook, f, indent=2)

if __name__ == "__main__":
    diff_cell_content(sys.argv[1])

#### Closing Summary

By using .gitattributes to specify *.ipynb diff=ipydiff, we can instruct Git to use our custom diff driver for Jupyter Notebook files. The Python script processes the notebook content, ignoring certain cells during diffs while still allowing other changes to be tracked.

#### Common Mistakes
and Follow-Ups
Ignoring Important Changes: Care must be taken not to ignore important parts of the code or output that could affect the functionality of the notebook.
Over-reliance on Diff Drivers: While custom diff drivers can help, they should be used judiciously to ensure meaningful diffs are still produced.

