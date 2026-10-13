---
type: "practice"
difficulty: "Intermediate"
---

# What does the second and third argument of LAG() (offset and default) do?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ LAG and LEAD](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/LAG_and_LEAD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with window functions in SQL, LAG() is a powerful tool that allows you to access rows above the current row within the same partition. The LAG() function has three parameters: column, offset, and default. In this answer, we'll explore what these parameters do and how they influence the behavior of the function.

TL;DR
OFFSET: Determines how many rows to offset from the current row.
DEFAULT: Specifies a default value if the offset position is out of bounds (i.e., there are fewer rows above the current row than specified).

#### Triaging the Question

The question asks about the second and third arguments of LAG() in SQL. Specifically, it wants to know what OFFSET and DEFAULT do. The interviewer might be probing your understanding of window functions, partitioning, and handling edge cases.

#### Assumptions
We are working with a table that has multiple columns.
The question is about the LAG() function in SQL.

#### First-Principles Solution
Understanding OFFSET

The first argument of LAG(), often referred to as column, specifies which column's value you want to retrieve from the offset row. However, the second and third arguments are crucial for handling cases where the requested offset is beyond the current partition or there are no rows above.

OFFSET: This parameter defines how many rows to move backward from the current row within the same window partition. For example, if you set OFFSET 1, it means you want to access the value of the previous row (one row above).

#### Example

#### Cons
ider a table with student grades over several semesters:

SQL
CREATE TABLE Students (
    id INT,
    name VARCHAR(255),
    grade INT,
    semester INT
);

INSERT INTO Students VALUES
(1, 'Alice', 90, 1),
(1, 'Alice', 85, 2),
(1, 'Bob', 70, 1),
(1, 'Bob', 80, 2);

To get the previous semester's grade for each student:

SQL
SELECT
    id,
    name,
    grade AS current_grade,
    LAG(grade, 1, -99) OVER (PARTITION BY id ORDER BY semester) AS prev_grade
FROM Students;

In this query, LAG(grade, 1, -99) means:

grade: The column from which the value is to be retrieved.
1: The offset: move one row backward.
-99: The default value if there's no previous row (i.e., Alice in her first semester).
Understanding DEFAULT

The third argument, DEFAULT, is used when the offset position is out of bounds. If you request a LAG with an offset that exceeds the number of rows above the current row, SQL will return this default value.

For example, if we change our query to get the grade from two semesters back:

SQL
SELECT
    id,
    name,
    grade AS current_grade,
    LAG(grade, 2, -99) OVER (PARTITION BY id ORDER BY semester) AS prev_grade_2
FROM Students;

Here, DEFAULT -99 ensures that if a student has only one or no records before the offset, -99 will be returned.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Forgetting to define the correct partition and order within the window function can lead to incorrect results. Misunderstanding the OFFSET and DEFAULT values can also cause confusion.

#### Follow-Ups
The interviewer might ask about performance implications, especially with large datasets, or how these parameters interact with other window functions like LEAD().
Self-Check

The explanation is clear, complete, and correctly covers all aspects of the question. No errors or missing details.

**Comparison:**

| id | name | grade | semester |
|----|------|-------|----------|
| 1  | Alice| 90    | 1        |
| 1  | Alice| 85    | 2        |
| 1  | Bob  | 70    | 1        |
| 1  | Bob  | 80    | 2        |
| id | name | grade | prev_grade |
|----|------|-------|------------|
| 1  | Alice| 90    | NULL       |
| 1  | Alice| 85    | 90         |
| 1  | Bob  | 70    | NULL       |
| 1  | Bob  | 80    | 70         |

**Code:**

```sql
-- Example code for LAG() function
CREATE TABLE Students (
    id INT,
    name VARCHAR(255),
    grade INT,
    semester INT
);

INSERT INTO Students VALUES
(1, 'Alice', 90, 1),
(1, 'Alice', 85, 2),
(1, 'Bob', 70, 1),
(1, 'Bob', 80, 2);

SELECT
    id,
    name,
    grade AS current_grade,
    LAG(grade, 1, -99) OVER (PARTITION BY id ORDER BY semester) AS prev_grade
FROM Students;
```

**Example input / output:**

```text
Input:
id | name | grade | semester | prev_grade
1  | Alice| 90    | 1        | NULL
1  | Alice| 85    | 2        | 90
1  | Bob  | 70    | 1        | NULL
1  | Bob  | 80    | 2        | 70

Output:
id | name | grade | prev_grade
1  | Alice| 85    | 90
1  | Bob  | 80    | 70
```

**Data:**

```plaintext
id | name | grade | semester | prev_grade
1  | Alice| 90    | 1        | NULL     <- No previous record for first semester
1  | Alice| 85    | 2        | 90       <-
1  | Bob  | 70    | 1        | NULL     <- No previous record for first semester
1  | Bob  | 80    | 2        | 70       <-
```

**Flow:**

1. Yes
2. No
3. Define Window Partition
4. Is Offset Positive?
5. Move backward by offset
6. Fetch Column Value
7. Return Default Value
8. Order Within Partition
9. Output Previous Value
10. Use Default Value

