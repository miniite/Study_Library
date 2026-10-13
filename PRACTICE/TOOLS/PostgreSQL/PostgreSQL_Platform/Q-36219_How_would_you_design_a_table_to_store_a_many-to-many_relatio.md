---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a table to store a many-to-many relationship between students and courses?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Many-to-many Modeling](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Many-to-many_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a table for storing a many-to-many relationship between students and courses in PostgreSQL, we need to create an intermediate table that acts as a junction or association table. This table will have foreign keys referencing both the students and courses tables. Let's break down the steps from first principles.

Firstly, we need to understand what a many-to-many relationship is: it means that each student can be enrolled in multiple courses, and each course can have multiple students. To represent this, we create an intermediary table with foreign keys linking to both primary entities (students and courses).

Secondly, assumptions:

Each student can enroll in more than one course.
Each course can accommodate more than one student.

Now let's design the schema step-by-step:

Define the Student Table:

SQL
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL
);

Define the Course Table:

SQL
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT
);

Create the Junction Table for Many-to-Many Relationship: The junction table, enrollments, will have foreign keys to both the students and courses tables.

SQL
CREATE TABLE enrollments (
    student_id INTEGER REFERENCES students(id),
    course_id INTEGER REFERENCES courses(id),
    enrollment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (student_id, course_id)
);

Adding Indexes for Performance: Adding indexes can improve query performance:

SQL
CREATE INDEX idx_student_id ON enrollments(student_id);
CREATE INDEX idx_course_id ON enrollments(course_id);

Inserting Data and Querying: Here’s how you might insert some sample data:

SQL
INSERT INTO students (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO courses (title, description) VALUES ('Database Management Systems', 'An introduction to database management systems');

INSERT INTO enrollments (student_id, course_id) VALUES ((SELECT id FROM students WHERE name = 'Alice'), (SELECT id FROM courses WHERE title = 'Database Management Systems'));

Querying Enrollments: To get a list of all students enrolled in a specific course:

SQL
SELECT s.name
  FROM enrollments e
  JOIN students s ON e.student_id = s.id
 WHERE e.course_id = (SELECT id FROM courses WHERE title = 'Database Management Systems');

#### Common Mistakes
and Follow-Ups:

Mistake: Forgetting to use composite primary keys in the junction table. The combination of student_id and course_id must be unique.

Mistake: Not using foreign key constraints for referential integrity, which can lead to orphaned records.

#### Follow-Up
Discuss the implications of adding more attributes to the junction table (e.g., grade). If there are frequent updates or deletions, consider denormalization strategies like materialized views or summary tables.

#### Follow-Up
Talk about scaling concerns and performance optimizations if this table grows large. Indexing the enrollment_date can be useful for time-based queries.

**Comparison:**

| Students | Courses | Enrollments |
|----------|---------|-------------|
| id       | name    | id          |
| email    | title   | student_id  |
                      | course_id |
                      | enrollment_date |

**Code:**

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL
);

CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT
);

CREATE TABLE enrollments (
    student_id INTEGER REFERENCES students(id),
    course_id INTEGER REFERENCES courses(id),
    enrollment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (student_id, course_id)
);

CREATE INDEX idx_student_id ON enrollments(student_id);
CREATE INDEX idx_course_id ON enrollments(course_id);
```

**Example input / output:**

```text
-- Inserting student and course data
INSERT INTO students (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO courses (title, description) VALUES ('Database Management Systems', 'An introduction to database management systems');

-- Enrolling Alice in the Database Management Systems course
INSERT INTO enrollments (student_id, course_id)
VALUES ((SELECT id FROM students WHERE name = 'Alice'), (SELECT id FROM courses WHERE title = 'Database Management Systems'));

-- Querying all students enrolled in a specific course
SELECT s.name
FROM enrollments e
JOIN students s ON e.student_id = s.id
WHERE e.course_id = (SELECT id FROM courses WHERE title = 'Database Management Systems');
```

**Data:**

```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "title": "Database Management Systems",
    "description": "An introduction to database management systems"
  }
]
```

**Flow:**

1. Students@students
2. Enrollments@end_enrollments
3. Courses@courses

