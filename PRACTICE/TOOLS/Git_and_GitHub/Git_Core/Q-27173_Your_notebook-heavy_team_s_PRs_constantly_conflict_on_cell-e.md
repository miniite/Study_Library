---
type: "practice"
difficulty: "Intermediate"
---

# Your notebook-heavy team's PRs constantly conflict on cell-execution-count and output metadata even when logic is identical. How would you configure nbstripout plus a gitattributes clean filter so those conflicts vanish without losing needed outputs?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the frequent merge conflicts in your notebook-heavy team's pull requests (PRs) due to cell execution counts and metadata, we can leverage nbstripout combined with a .gitattributes clean filter. This setup ensures that notebook outputs are stripped before committing, thereby reducing conflicts while maintaining necessary logic.

#### Triage

The core issue here is that cells in Jupyter Notebooks often contain timestamps or hashes as part of their execution metadata, which can differ across environments and lead to merge conflicts. The goal is to standardize these values so they no longer conflict, without losing essential information needed for the notebook's integrity.

#### Assumptions

Your team uses Jupyter Notebooks (.ipynb files).
nbstripout is installed.
You are familiar with .gitattributes.

#### Solution

Install and Configure nbstripout: First, ensure nbstripout is installed in your environment. This tool strips metadata from notebooks when they are committed to the repository.

Configure .gitattributes for Clean Filter:

Create or edit a .gitattributes file in your project root.
Add a line to specify that all .ipynb files should use nbstripout during commit stages (clean and smudge).
BASH
*.ipynb filter=nbstripout eol=lf

Set Up nbstripout as a Clean Filter:

Ensure nbstripout is set up to be used by Git as a clean filter.
Add the following lines to your .gitattributes file:
PLAINTEXT
* filter=nbstripout eol=lf

Test and Validate:

Test committing notebooks to see if the metadata is stripped correctly.
Validate that PRs no longer contain conflicting cell execution counts or metadata.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By configuring nbstripout via .gitattributes, you ensure consistent notebook files across different environments, reducing merge conflicts. However, it's crucial to verify that the stripped outputs are still useful for the intended purpose of the notebook.

#### Common Mistakes

Forgetting to install and configure nbstripout.
Not adding the correct lines in the .gitattributes file.
Assuming all cells need their metadata stripped; some cells might require preserved output for documentation or debugging purposes.
Code Blocks

No code blocks required for this solution.

