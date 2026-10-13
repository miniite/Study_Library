---
type: "practice"
difficulty: "Intermediate"
---

# You have a many-to-many join table student_courses(student_id, course_id). What indexes would you add and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize queries on the student_courses table in a many-to-many relationship, we need to ensure that our indexing strategy minimizes the overhead of joining tables while maximizing query performance. This involves understanding the typical queries and their frequency.

The core idea is to create indexes on columns that are frequently used in join or filter conditions. For this scenario, we will focus on adding indexes on student_id and course_id.

Indexing student_id: This column is likely to be used for filtering students who have taken specific courses.
Indexing course_id: This column is useful when querying which courses a student has enrolled in.

By adding these indexes, we can significantly speed up queries that involve joining the student_courses table with other tables such as students or courses.

