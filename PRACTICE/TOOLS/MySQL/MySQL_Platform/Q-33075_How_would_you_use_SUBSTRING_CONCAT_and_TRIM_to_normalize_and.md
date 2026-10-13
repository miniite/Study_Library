---
type: "practice"
difficulty: "Basic"
---

# How would you use SUBSTRING, CONCAT, and TRIM to normalize and format a user's display name?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To normalize and format a user's display name using MySQL functions like SUBSTRING, CONCAT, and TRIM, you first need to understand what each function does:

SUBSTRING extracts a portion of a string based on start and length.
CONCAT joins multiple strings together.
TRIM removes leading and trailing spaces from a string.

The goal here is to clean up the display name by removing any extra whitespace, extracting relevant parts, and then concatenating them in a specific format. Let's break down how we can achieve this step-by-step.

