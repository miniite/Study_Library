---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between CEIL (CEILING) and FLOOR functions.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Let's break down the key differences between the SQL CEIL (or CEILING) and FLOOR functions. These functions are used to round numbers, but they do so in opposite ways.

#### Triaging the Question
What is being asked? The question is essentially asking for an explanation of how these two rounding functions behave differently.
What's being probed? The interviewer likely wants to ensure you understand the basic principles and applications of these functions.

#### First-Principles Solution
CEIL Function: This function rounds a number up to the nearest integer or specified precision. For example, CEIL(3.2) results in 4.
FLOOR Function: Conversely, this function rounds a number down to the nearest integer or specified precision. For instance, FLOOR(3.8) would result in 3.

#### Example Scenario
s:
If you have a value of 5.6, using CEIL will round it up to 6, whereas FLOOR will round it down to 5.
Similarly, for negative numbers like -2.4, CEIL(-2.4) rounds to -2 (since we're moving towards zero), and FLOOR(-2.4) rounds to -3 (since we're rounding away from zero).

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Common Mistake: Assuming that both functions work the same way. It’s crucial to understand their opposite nature.

#### Follow-Up
Question: You might be asked about scenarios where these functions are particularly useful, such as in financial calculations or when dealing with timestamps.

