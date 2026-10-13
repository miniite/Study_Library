---
type: "practice"
difficulty: "Intermediate"
---

# How would you use pandas merge to perform a lookup enrichment in a Python pipeline, and how do inner, left, and outer joins change which rows survive?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pandas DataFrames](<../../../../TOOLS/Python/Python_Language/Pandas_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When performing data processing tasks as a Data Engineer, pandas.merge is an essential tool for enriching datasets by combining them based on common keys. This operation allows us to merge multiple DataFrames and leverage their contents to enhance our primary dataset. The type of join (inner, left, or outer) significantly impacts which rows are retained in the output.

In this context, let's break down how each type of join works and illustrate its effect using a practical example.

