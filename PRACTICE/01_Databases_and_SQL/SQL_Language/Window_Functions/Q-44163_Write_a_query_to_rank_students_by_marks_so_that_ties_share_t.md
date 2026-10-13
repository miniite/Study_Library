---
type: "practice"
difficulty: "Basic"
---

# Write a query to rank students by marks so that ties share the same rank and no numbers are skipped.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query using window functions to assign ranks to students based on their marks such that tied students share the same rank with gaps in ranking if necessary. The key is to use ROW_NUMBER, RANK, or DENSE_RANK depending on the specific requirements and behavior you want.

#### Triage

We are required to rank students by their marks.
Tied ranks should be assigned to multiple students.
No numbers should be skipped in the ranking if there are gaps due to ties.

#### First-principles solution

Identify the core logic: Use a window function to calculate the rank based on the marks column.
Handle ties appropriately: Choose between ROW_NUMBER, RANK, or DENSE_RANK depending on whether we want skipped ranks in case of ties.
Implement the query: Write an SQL statement that uses the chosen window function and ensures correct ranking.

Here’s how to implement this step-by-step:

SQL
SELECT
    student_id,
    marks,
    RANK() OVER (ORDER BY marks DESC) AS rank
FROM
    students;

In this example, I used RANK() because it handles ties by assigning the same rank and then skipping the next number in the sequence. If we wanted to avoid skipped ranks, we would use DENSE_RANK instead:

SQL
SELECT
    student_id,
    marks,
    DENSE_RANK() OVER (ORDER BY marks DESC) AS rank
FROM
    students;

#### Closing summary

We used window functions (RANK or DENSE_RANK) to assign ranks based on the marks.
Ties are handled appropriately, with no numbers skipped in case of ties using RANK, and gaps allowed when necessary.

#### Common mistakes/follow-ups

Not correctly identifying whether you need RANK, ROW_NUMBER, or DENSE_RANK.
Misunderstanding how window functions work and expecting them to behave like regular aggregate functions.
Failing to test the query with sample data to ensure it handles ties as expected.

