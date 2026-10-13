---
type: "practice"
difficulty: "Intermediate"
---

# How do you design a junction table for a many-to-many relationship between students and courses?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Schemas and Databases](<../../../../TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a junction table for a many-to-many relationship between students and courses in MySQL, we need to create an intermediary table that links each student to multiple courses and vice versa. This involves understanding the requirements, designing the schema, and ensuring normalization. Let's break it down step-by-step.

#### Triage
Understanding the Question

The question asks for a junction table design in MySQL to handle a many-to-many relationship between students and courses. The key aspects we need to cover include:

Identifying the entities involved (students and courses).
Determining the relationships between these entities.
Creating a schema that accurately represents this relationship.

#### Assumptions
We assume each student can enroll in multiple courses, and each course can have multiple students enrolled.
We will not store any additional attributes on the junction table itself but only use it to link students and courses.
Step-by-Step Solution

Identify the Entities:

Students
Courses

Determine the Relationships:

A student can be enrolled in multiple courses.
A course can have multiple students enrolled.

Design the Schema: To represent this many-to-many relationship, we will create a junction table that contains foreign keys to both the students and courses tables. Here’s how we can do it:

SQL
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
Explanation of the Schema
student_id and course_id are foreign keys that reference the primary keys in the students and courses tables, respectively.
The primary key constraint on (student_id, course_id) ensures uniqueness, meaning each combination of student and course is represented only once.

#### Example
Data

Here’s some sample data to illustrate how this junction table works:

SQL
INSERT INTO students (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie');

INSERT INTO courses (id, name) VALUES
(101, 'Math'),
(102, 'Physics'),
(103, 'Chemistry');

INSERT INTO student_courses (student_id, course_id) VALUES
(1, 101),
(1, 102),
(2, 101),
(3, 102);
Input-Output Example
Input: SELECT * FROM student_courses;
Output:
SQL
+-------------+------------+
| student_id  | course_id  |
+-------------+------------+
|           1 |        101 |
|           1 |        102 |
|           2 |        101 |
|           3 |        102 |
+-------------+------------+

#### Closing Summary

The junction table student_courses effectively represents the many-to-many relationship between students and courses. By using foreign keys, we ensure referential integrity and can easily query which students are enrolled in which courses.

#### Common Mistakes/Follow-Ups
Using One-to-One or One-to-Many Relationships Instead: Make sure to understand that a student-course relationship is inherently many-to-many.
Forgetting the Composite Primary Key: Ensure both student_id and course_id together form the primary key.
Storing Additional Attributes on the Junction Table: While not necessary for this basic design, in more complex scenarios, storing additional attributes (like enrollment date or grade) might be required.

**Comparison:**

| Student ID | Course ID |
| --- | --- |
| 1          | 101       |
| 1          | 102       |
| 2          | 101       |
| 3          | 102       |

**Code:**

```sql
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

**Example input / output:**

```text
-- Input
SELECT * FROM student_courses;

-- Output
+-------------+------------+
| student_id  | course_id  |
+-------------+------------+
|           1 |        101 |
|           1 |        102 |
|           2 |        101 |
|           3 |        102 |
+-------------+------------+
```

**Data:**

```sql
INSERT INTO students (id, name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie');

INSERT INTO courses (id, name) VALUES
(101, 'Math'),
(102, 'Physics'),
(103, 'Chemistry');

INSERT INTO student_courses (student_id, course_id) VALUES
(1, 101),
(1, 102),
(2, 101),
(3, 102);
```

**Flow:**

1. Enrolls in
2. Has many Enrollments
3. Many enrollments in
4. References
5. References
6. Student
7. Course
8. Student_Courses
9. id
10. id

