---
type: "practice"
difficulty: "Intermediate"
---

# What does the DAYOFWEEK or WEEKDAY function return and how is it numbered?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The DAYOFWEEK and WEEKDAY functions in SQL are used to determine the day of the week from a date, returning a numeric value. However, they use different numbering systems for weekdays.

DAYOFWEEK returns the day number starting from 1 (Sunday) to 7 (Saturday). This is useful when you need to directly work with the day name in your query results without additional conversion steps.

WEEKDAY, on the other hand, returns the day of the week as a zero-based index, where Monday is represented by 0 and Sunday by 6. This can be more convenient for certain calculations because it aligns better with array indexing in many programming languages.

To illustrate this, let's walk through an example using these functions.

