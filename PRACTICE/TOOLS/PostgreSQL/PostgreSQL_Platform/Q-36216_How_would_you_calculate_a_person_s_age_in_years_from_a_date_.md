---
type: "practice"
difficulty: "Intermediate"
---

# How would you calculate a person's age in years from a date_of_birth column?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate a person's age in years from a date_of_birth column in PostgreSQL, we need to determine the difference between the current date and the date of birth, then divide by 365 to convert the days into years. This problem requires understanding date arithmetic and functions available in PostgreSQL.

The core idea is to use PostgreSQL’s date manipulation functions to calculate the age accurately. We will assume that date_of_birth is stored as a DATE type column in our table.

