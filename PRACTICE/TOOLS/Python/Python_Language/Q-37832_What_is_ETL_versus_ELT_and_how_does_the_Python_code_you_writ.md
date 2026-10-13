---
type: "practice"
difficulty: "Basic"
---

# What is ETL versus ELT, and how does the Python code you write differ between transforming before load versus pushing raw data into a warehouse first?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Compressed File Handling](<../../../../TOOLS/Python/Python_Language/Compressed_File_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ETL (Extract, Transform, Load) and ELT (Extract, Load, Transform) are both processes used in data engineering to move data from one location to another while performing necessary operations like cleaning or aggregating. In ETL, you transform the data before loading it into a warehouse, whereas in ELT, you push raw data directly into the warehouse and perform transformations later.

The key difference lies in when and where the transformation happens. This choice can significantly impact performance, data quality, and storage costs. Let's dive deeper by considering how Python code would differ between these two paradigms.

