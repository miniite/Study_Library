---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python function that trims and standardizes string categories (like 'IN', 'India', 'india') into a single canonical value during transform?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of trimming and standardizing string categories like "IN", "India", and "india" to a single canonical value, we need to follow these steps:

Trimming Spaces: Remove any leading or trailing spaces from the strings.
Standardization: Convert all characters in the string to a common format (like lowercase) to ensure that different representations of the same value are treated identically.

Here’s how you can implement this in Python:

