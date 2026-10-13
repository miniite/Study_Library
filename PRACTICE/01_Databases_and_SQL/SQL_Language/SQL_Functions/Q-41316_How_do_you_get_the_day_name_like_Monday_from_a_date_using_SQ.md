---
type: "practice"
difficulty: "Intermediate"
---

# How do you get the day name (like Monday) from a date using SQL functions?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To get the day name from a date in SQL, we need to use a specific function that returns the day of the week as a string. This is a common requirement when dealing with dates and can be achieved using various SQL functions depending on your database system.

#### Triage

The question is asking for a method to extract the day name (e.g., "Monday") from a given date.
Different databases have different functions for this, but we'll focus on popular systems like MySQL, PostgreSQL, and SQLite.
We need to ensure that the function works correctly across different regions, as the names of days can vary based on language settings.

#### First-principles solution

Understand the Function: Most SQL databases provide a built-in function for this purpose. For example, in MySQL, it's DAYNAME(date_column), and in PostgreSQL, it's to_char(date_column, 'Day').
Execute the Query: Use the appropriate function to get the day name.
Test with Sample Data: Verify that the function works as expected.

#### Closing summary
+ common mistakes/follow-ups:

Ensure you're using the correct function for your SQL database system.
Be mindful of locale settings, especially if working in a multilingual environment.

#### Cons
ider edge cases like NULL values or dates outside the valid range.

