---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a one-to-one and a one-to-many relationship, with an example of each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Schemas and Databases](<../../../../TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, understanding relationships between tables is crucial for organizing data effectively. One-to-one (1:1) and one-to-many (1:M) are two common types of relationships used in relational databases like MySQL.

A 1:1 relationship exists when each record in Table A has exactly one corresponding record in Table B, and vice versa. On the other hand, a 1:M relationship occurs when many records in Table A can correspond to a single record in Table B, but a single record in Table B is only related to one record in Table A.

Let’s dive into examples for each type of relationship:

One-to-One Relationship Example

Imagine we are designing a database for student records. We have two tables: Students and Addresses. Each student has one address, and each address belongs to exactly one student.

Tables:
Students (student_id, name)
Addresses (address_id, street, city)

In this case, we would create a foreign key in the Students table that references the primary key of the Addresses table. This ensures that each student has only one address and vice versa.

SQL Schema:
SQL
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    address_id INT,
    FOREIGN KEY (address_id) REFERENCES Addresses(address_id)
);

CREATE TABLE Addresses (
    address_id INT PRIMARY KEY,
    street VARCHAR(255),
    city VARCHAR(100)
);
One-to-Many Relationship Example

Now let's consider a scenario where we have Courses and Students. A single course can be taken by many students, but each student is enrolled in only one course.

Tables:
Courses (course_id, name, credit_hours)
Enrollments (enrollment_id, student_id, course_id)

In this example, the Courses table will have a foreign key in the Enrollments table that references student_id. This indicates which students are enrolled in each course.

SQL Schema:
SQL
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    name VARCHAR(100),
    credit_hours INT
);

CREATE TABLE Enrollments (
    enrollment_id INT PRIMARY KEY,
    student_id INT,
    course_id INT,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);
Summary and Common Mistakes
1:1 Relationship: Ensure that each record in Table A has exactly one corresponding record in Table B, and vice versa. Use foreign keys to enforce this relationship.
1:M Relationship: Multiple records in Table A can be linked to a single record in Table B. Typically involves an intermediate table (like Enrollments above) or using a foreign key that references the primary key of the parent table.

#### Common mistakes
include:

Not properly defining foreign keys, leading to data integrity issues.
Overusing 1:1 relationships when simpler alternatives like a single column in one table would suffice.
Failing to normalize tables where necessary, which can lead to redundancy and inconsistency.

**Code:**

```

-- Example SQL Schema for One-to-One Relationship CREATE TABLE Students ( student_id INT PRIMARY KEY, name VARCHAR(100), address_id INT, FOREIGN KEY (address_id) REFERENCES Addresses(address_id) );

CREATE TABLE Addresses ( address_id INT PRIMARY KEY, street VARCHAR(255), city VARCHAR(100) );

CODE
Copy

```sql
-- Example SQL Schema for One-to-Many Relationship
CREATE TABLE Courses (
    course_id INT PRIMARY KEY,
    name VARCHAR(100),
    credit_hours INT
);

CREATE TABLE Enrollments (
    enrollment_id INT PRIMARY KEY,
    student_id INT,
    course_id INT,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);
```

