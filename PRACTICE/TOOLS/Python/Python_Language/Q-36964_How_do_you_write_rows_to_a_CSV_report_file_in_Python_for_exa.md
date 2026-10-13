---
type: "practice"
difficulty: "Basic"
---

# How do you write rows to a CSV report file in Python, for example a summary of which health checks passed or failed?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Reconciliation Scripts](<../../../../TOOLS/Python/Python_Language/Data_Reconciliation_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write rows to a CSV report file in Python, we typically use the csv module, which is part of Python's standard library. The process involves creating a CSV writer object and then writing each row of data into this file. Here’s how you can do it for a summary of health checks.

Firstly, let's understand what we are doing: we want to create a CSV file where one row contains the status (passed or failed) of each health check. This involves opening the file, setting up the writer object, and then writing data row by row.

