---
type: "practice"
difficulty: "Basic"
---

# How do you get the current date and current timestamp in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve the current date and current timestamp in PostgreSQL, we need to understand that these are common operations used in database development. The CURRENT_DATE function returns the current date (without time), while NOW() or CURRENT_TIMESTAMP provides both the date and time.

Let's break down each part of this operation:

Understanding the Functions:

CURRENT_DATE: This function retrieves the current date, which is useful for operations that do not require a timestamp.
NOW(), CURRENT_TIMESTAMP: These functions return the current date and time, providing more precision compared to CURRENT_DATE.

#### Example
Usage:

To get the current date: You would use SELECT CURRENT_DATE;.
To get the current timestamp: You can use either SELECT NOW(); or SELECT CURRENT_TIMESTAMP;.

#### First-Principles Solution

SQL
-- Get Current Date
SELECT CURRENT_DATE;

-- Get Current Timestamp
SELECT NOW();

#### Closing Summary
and Common Mistakes/Follow-Ups:

Always ensure you are using the correct function based on your requirement.
While CURRENT_TIMESTAMP is functionally equivalent to NOW(), CURRENT_TIMESTAMP can be set as a default value for a column, making it more useful in certain scenarios.

