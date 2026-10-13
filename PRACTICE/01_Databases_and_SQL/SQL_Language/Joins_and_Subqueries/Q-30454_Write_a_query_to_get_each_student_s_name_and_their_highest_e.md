---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to get each student's name and their highest exam score using a join.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to retrieve each student's name along with their highest exam score from a database where students have multiple exams. The key here is to use SQL joins and subqueries effectively. Let’s break it down step-by-step.

Firstly, the question asks us to combine data from at least two tables: one that stores student information (let’s call it students) and another that holds exam results (let’s name this table exams). The students table likely has columns like student_id and name, while the exams table might have exam_id, student_id, and score.

The core idea is to use a combination of subqueries and joins to fetch the highest score for each student. We can achieve this by:

Using an INNER JOIN to match students with their exams.
Utilizing a subquery to find the maximum score for each student.

Now, let’s reason through the solution from first principles.

