---
type: "practice"
difficulty: "Basic"
---

# How do you detect missing values in a pandas DataFrame, and what does df.isnull().sum() tell you about your dataset?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pandas DataFrames](<../../../../TOOLS/Python/Python_Language/Pandas_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect missing values in a pandas DataFrame, one of the most common methods is to use df.isnull().sum() where df represents the DataFrame. This method returns the number of missing (NaN) values present in each column. Let's break down how this works and what it tells us about our dataset.

