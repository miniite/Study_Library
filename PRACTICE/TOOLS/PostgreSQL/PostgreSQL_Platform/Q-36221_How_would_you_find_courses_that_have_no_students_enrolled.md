---
type: "practice"
difficulty: "Intermediate"
---

# How would you find courses that have no students enrolled?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding courses with no students enrolled in PostgreSQL, we need to focus on querying a table structure where course enrollments are tracked. Let’s break this down from first principles.

#### Triage
and Assumptions

The key assumption here is that there is a database schema containing at least two tables: one for courses (courses) and another for enrollments (enrollments). The courses table will have an ID for each course, and the enrollments table will link students to their enrolled courses. Our goal is to find courses with zero entries in the enrollments table.

#### First-Principles Solution

Understand the Schema:

courses table: Contains a list of all available courses.
SQL
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    department VARCHAR(100)
);
enrollments table: Tracks student enrollments to specific courses.
SQL
CREATE TABLE enrollments (
    enrollment_id SERIAL PRIMARY KEY,
    student_id INTEGER NOT NULL REFERENCES students(student_id),
    course_id INTEGER NOT NULL REFERENCES courses(course_id)
);

Identify the Query Requirement: We need to find all course_ids in the courses table that do not have any corresponding entries in the enrollments table.

Implement the Query:

Use a LEFT JOIN between the courses and enrollments tables.
Filter out courses where there are no matches in the enrollments table using WHERE clause with a COALESCE or similar function to check for null values.
Full Solution

Here’s how we can write this query:

SQL
-- Query to find courses with no enrolled students
SELECT c.course_id, c.name
FROM courses AS c
LEFT JOIN enrollments AS e ON c.course_id = e.course_id
WHERE COALESCE(e.enrollment_id, 0) = 0;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query works by performing a LEFT JOIN on the course_id fields of both tables. If there is no corresponding enrollment for a course (i.e., e.course_id does not match any rows in enrollments), then the result will include all columns from courses with NULL values for the enrollments table, which we filter out using COALESCE.

#### Common Mistakes

Not understanding that a simple JOIN would only return courses that have enrollments.
Forgetting to use LEFT JOIN, resulting in missing courses with no enrollments.
Additional Considerations
Indexes: Ensure that appropriate indexes are created on the course_id columns of both tables to improve performance, especially if dealing with large datasets.
Database Version: Ensure compatibility with the version of PostgreSQL you're using (e.g., newer versions might have more efficient syntax or functions).

**Comparison:**

| course_id | name           |
|-----------|----------------|
| 2         | Data Mining    |

**Code:**

```sql
-- Example schema for courses and enrollments table
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    department VARCHAR(100)
);

CREATE TABLE enrollments (
    enrollment_id SERIAL PRIMARY KEY,
    student_id INTEGER NOT NULL REFERENCES students(student_id),
    course_id INTEGER NOT NULL REFERENCES courses(course_id)
);
```

**Example input / output:**

```text
-- Query execution and result
SELECT c.course_id, c.name
FROM courses AS c
LEFT JOIN enrollments AS e ON c.course_id = e.course_id
WHERE COALESCE(e.enrollment_id, 0) = 0;

-- Expected output:
-- course_id | name
-- 2         | Data Mining
```

**Data:**

```sql
-- Sample data for courses and enrollments table
INSERT INTO courses (name, department) VALUES ('Database Systems', 'Computer Science');
INSERT INTO courses (name, department) VALUES ('Data Mining', 'Statistics');
INSERT INTO courses (name, department) VALUES ('Machine Learning', 'Artificial Intelligence');

INSERT INTO enrollments (student_id, course_id) VALUES (1, 1);
INSERT INTO enrollments (student_id, course_id) VALUES (2, 1);

-- No insert for the second course to demonstrate it has no students
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the Schema Correct?
5. Create Courses andEnrollments Tables
6. Insert Sample Data
7. Run Query to Find Courseswith No Students
8. Get Result: Courses with NoEnrollments
9. Correct Schema First
10. End

