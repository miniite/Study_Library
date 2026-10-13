---
type: "practice"
difficulty: "Deep"
---

# What is the difference between identifying and non-identifying relationships in ER modeling?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ER Modeling Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/ER_Modeling_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Entity Relationship (ER) Modeling, an ER diagram visually represents entities (tables), attributes (columns), and relationships among these entities. Relationships are a fundamental concept that define how different entities interact with each other. The key difference between identifying and non-identifying relationships lies in the ownership of the primary key.

TL;DR

Identifying relationships mean one entity fully owns the primary key, implying it is part of another entity's structure. Non-identifying relationships use a composite or shared primary key to link entities without one owning the other’s primary key.

#### Triaging the Question

This question probes your understanding of ER modeling and how different types of relationships are represented in an ER diagram. The interviewer might be interested in:

Your ability to distinguish between identifying and non-identifying relationships.
How these differences impact data integrity, normalization, and schema design.

#### First Principles Solution
Identifying Relationships

In an identifying relationship, one entity (let's call it Entity A) fully owns the primary key of another entity (Entity B). This means that if you remove the owning entity from the database, all related entities also need to be removed.

For example, consider a university database where students enroll in courses. The Student table might have a foreign key referencing the Course table:

SQL
CREATE TABLE Student (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);

CREATE TABLE Course (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(255)
);

In this case, if a student enrolls in a course, the student_id would be stored within the Course table. The student’s data is intrinsically tied to the course enrollment.

Non-Identifying Relationships

A non-identifying relationship uses a composite or shared primary key that is not owned by any single entity. Both entities have their own unique keys, but they share a common key for linking purposes.

For instance, in the same university database, you might want to model both Student and Course with their own primary keys:

SQL
CREATE TABLE Student (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);

CREATE TABLE Course (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(255)
);

To link them, you introduce a third table (often called a junction or associative table):

SQL
CREATE TABLE Enrollment (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (course_id) REFERENCES Course(course_id)
);

Here, the Enrollment table shares a composite key (student_id, course_id) but neither entity owns this key.

#### Closing Summary

Identifying and non-identifying relationships in ER modeling serve distinct purposes. Identifying relationships are used when one entity is part of another, while non-identifying relationships use shared keys without one entity owning the other's primary key. Understanding these distinctions is crucial for designing robust and normalized schemas that maintain data integrity.

#### Common Mistakes/Follow-Ups
Misunderstanding ownership: Confusing identifying relationships with foreign keys or not recognizing when a composite key should be used.
Over-normalization: Not realizing the benefits of non-identifying relationships to avoid redundant storage and improve query performance.
Ignoring referential integrity: Failing to properly define primary and foreign keys in your schema.

#### Example
Data
SQL
-- Sample data for Student table
INSERT INTO Student (student_id, name, age) VALUES
(1, 'Alice', 20),
(2, 'Bob', 22);

-- Sample data for Course table
INSERT INTO Course (course_id, course_name) VALUES
(1, 'Database Systems'),
(2, 'Data Structures');

-- Sample data for Enrollment table
INSERT INTO Enrollment (student_id, course_id, enrollment_date) VALUES
(1, 1, '2023-09-01'),
(2, 2, '2023-09-05');

**Comparison:**

```
Enrolled
Offers
Student
INT
student_id
PK
VARCHAR(100)
name
INT
age
Enrollment
INT
student_id
PK
INT
course_id
PK
DATE
enrollment_date
in
Course
INT
course_id
PK
VARCHAR(255)
course_name
```

**Code:**

```sql
CREATE TABLE Student (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT
);

CREATE TABLE Course (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(255)
);

CREATE TABLE Enrollment (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (course_id) REFERENCES Course(course_id)
);
```

**Example input / output:**

```text
Input:
SELECT Student.name, Course.course_name FROM Enrollment JOIN Student ON Enrollment.student_id = Student.student_id JOIN Course ON Enrollment.course_id = Course.course_id WHERE Enrollment.enrollment_date BETWEEN '2023-09-01' AND '2023-09-05';

Output:
+--------+------------------+
| name   | course_name      |
+--------+------------------+
| Alice  | Database Systems |
+--------+------------------+
```

**Data:**

```sql
-- Sample data for Student table
INSERT INTO Student (student_id, name, age) VALUES
(1, 'Alice', 20),
(2, 'Bob', 22);

-- Sample data for Course table
INSERT INTO Course (course_id, course_name) VALUES
(1, 'Database Systems'),
(2, 'Data Structures');

-- Sample data for Enrollment table
INSERT INTO Enrollment (student_id, course_id, enrollment_date) VALUES
(1, 1, '2023-09-01'),
(2, 2, '2023-09-05');
```

**Flow:**

1. Student Table
2. Enrollment Table
3. Course Table
4. Enrollment Table
5. Enrollment Date

