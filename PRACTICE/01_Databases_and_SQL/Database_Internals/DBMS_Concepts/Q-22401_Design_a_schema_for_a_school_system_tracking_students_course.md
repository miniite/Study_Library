---
type: "practice"
difficulty: "Intermediate"
---

# Design a schema for a school system tracking students courses and enrollments with grades.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Relationship Modeling](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a database schema for a school system that tracks student course enrollments and grades, we need to ensure the schema is both efficient and normalized while supporting various querying needs. The core entities in this system are students, courses, enrollments, and grades.

#### Triage

The question asks us to design a schema for tracking student enrollments and grades. This means our primary focus will be on designing tables for Students, Courses, Enrollments, and Grades. We need to ensure that each entity is properly normalized and that we can efficiently retrieve the required data.

#### First-Principles Solution

Identify Entities:

Students (unique identifiers, names, etc.)
Courses (course IDs, course titles, credits, instructors)
Enrollments (links students to courses with a timestamp for when they enrolled)
Grades (assessing performance in each course)

Define Primary Keys and Foreign Keys:

Use unique identifiers as primary keys.
Use foreign keys to link related entities.

Normalization Levels:

Ensure that the schema is at least in 3NF (Third Normal Form).
Detailed Schema
SQL
CREATE TABLE Students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE NOT NULL,
    email VARCHAR(100),
    phone_number VARCHAR(20)
);

CREATE TABLE Courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_title VARCHAR(100) NOT NULL,
    credits INT NOT NULL,
    instructor_id INT,
    FOREIGN KEY (instructor_id) REFERENCES Staff(staff_id) ON DELETE SET NULL
);

CREATE TABLE Enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    course_id INT,
    enrollment_date DATE NOT NULL,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

CREATE TABLE Grades (
    grade_id INT PRIMARY KEY AUTO_INCREMENT,
    enrollment_id INT,
    grade_value DECIMAL(3,2) CHECK (grade_value >= 0 AND grade_value <= 100),
    submitted_date DATE NOT NULL,
    FOREIGN KEY (enrollment_id) REFERENCES Enrollments(enrollment_id)
);

#### Closing Summary
and Common Mistakes
Primary Keys: Ensure each entity has a unique identifier.
Foreign Key Relationships: Properly link related tables to avoid orphaned records.
Normalization Levels: Check for data redundancy by ensuring the schema is in at least 3NF.

#### Common mistakes
include not using foreign keys, missing constraints on key columns (like email uniqueness), and over-normalizing the schema which can make queries complex and inefficient. It's essential to balance normalization with query performance needs.

**Code:**

```sql
CREATE TABLE Students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE NOT NULL,
    email VARCHAR(100),
    phone_number VARCHAR(20)
);

CREATE TABLE Courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_title VARCHAR(100) NOT NULL,
    credits INT NOT NULL,
    instructor_id INT,
    FOREIGN KEY (instructor_id) REFERENCES Staff(staff_id) ON DELETE SET NULL
);

CREATE TABLE Enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT,
    course_id INT,
    enrollment_date DATE NOT NULL,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);

CREATE TABLE Grades (
    grade_id INT PRIMARY KEY AUTO_INCREMENT,
    enrollment_id INT,
    grade_value DECIMAL(3,2) CHECK (grade_value >= 0 AND grade_value <= 100),
    submitted_date DATE NOT NULL,
    FOREIGN KEY (enrollment_id) REFERENCES Enrollments(enrollment_id)
);
```

