---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through normalizing a flat spreadsheet of student courses and instructors into 3NF.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To normalize a flat spreadsheet of student courses and instructors into Third Normal Form (3NF), we need to break down the data into multiple tables, each representing a single entity or relationship, ensuring that every non-key attribute is fully dependent on the primary key.

#### Triage

The question asks for normalizing a dataset involving students, courses, and instructors. The goal is to eliminate redundant data, ensure atomicity of non-key attributes, and minimize data anomalies. We will start by identifying the entities and their relationships, then design tables to represent each entity in 3NF.

Triaging Assumptions:
Entities: Students, Courses, Instructors.
Relationships: Each course can have multiple instructors, and students can enroll in multiple courses.
Key Considerations: Avoiding partial dependencies and ensuring that non-key attributes are fully functionally dependent on the primary key.
Step-by-Step Solution from First Principles:

Identify Entities:

Student Entity: Columns: student_id, name, email.
Course Entity: Columns: course_id, title, department.
Instructor Entity: Columns: instructor_id, name, subject.

Relationships and Joining Tables:

A student can enroll in multiple courses.
An instructor can teach multiple courses.

Normalization Steps:

Step 1: First Normal Form (1NF)
Ensure each column contains atomic values (no repeating groups).

#### Example
schema:
PLAINTEXT
Students:
student_id | name | email

Courses:
course_id | title | department

Instructors:
instructor_id | name | subject
Step 2: Second Normal Form (2NF)
Remove partial dependencies, ensuring each non-key column is fully dependent on the primary key.
Since title and department are only related to course_id, we can proceed to 3NF without further changes.
Step 3: Third Normal Form (3NF)
Eliminate transitive dependencies by creating a linking table for relationships between entities.
PLAINTEXT
Enrollments:
student_id | course_id

Instructor_Course_Assignments:
instructor_id | course_id
Detailed Schema:

Students Table:

SQL
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

Courses Table:

SQL
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    title VARCHAR(100),
    department VARCHAR(50)
);

Instructors Table:

SQL
CREATE TABLE Instructors (
    instructor_id INT PRIMARY KEY,
    name VARCHAR(50),
    subject VARCHAR(50)
);

Enrollments Table (Linking Table):

SQL
CREATE TABLE Enrollments (
    student_id INT,
    course_id INT,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

Instructor_Course_Assignments Table (Linking Table):

SQL
CREATE TABLE Instructor_Course_Assignments (
    instructor_id INT,
    course_id INT,
    FOREIGN KEY (instructor_id) REFERENCES Instructors(instructor_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

#### Closing Summary

By normalizing the flat spreadsheet into a 3NF schema, we have broken down the data into multiple tables to eliminate redundancy and ensure that each non-key attribute is fully dependent on the primary key. This approach helps in minimizing data anomalies and making the database more efficient for querying.

#### Common Mistakes/Follow-Ups
Ignoring Transitive Dependencies: Ensure all attributes are directly related to the primary key.
Over-normalization: Avoid splitting tables too much, which can complicate queries unnecessarily.
Correct Foreign Key References: Ensure that foreign keys in linking tables correctly reference the primary keys of other tables.

**Code:**

```sql
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    title VARCHAR(100),
    department VARCHAR(50)
);

CREATE TABLE Instructors (
    instructor_id INT PRIMARY KEY,
    name VARCHAR(50),
    subject VARCHAR(50)
);

CREATE TABLE Enrollments (
    student_id INT,
    course_id INT,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

CREATE TABLE Instructor_Course_Assignments (
    instructor_id INT,
    course_id INT,
    FOREIGN KEY (instructor_id) REFERENCES Instructors(instructor_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);
```

