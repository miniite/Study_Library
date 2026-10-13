---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a .gitattributes entry to tell Git to treat your metric-exploration .ipynb notebooks specially so diffs stay cleaner?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gitattributes and Merge Drivers](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitattributes_and_Merge_Drivers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep diffs cleaner when working with Jupyter Notebook (.ipynb) files that are used for metric exploration, we can use a .gitattributes entry to configure Git to handle these files differently. This involves telling Git not to track changes in the cell outputs and hashes so that only code and metadata relevant to the notebook's structure are included in diffs.

Essentially, we want to ensure that Git treats .ipynb files as text files but ignores certain parts of them—specifically, the output cells and metadata that can clutter diffs. This way, the diffs will focus on changes in the actual code and content rather than transient runtime results or formatting issues.

