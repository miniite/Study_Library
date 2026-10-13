---
type: "practice"
difficulty: "Intermediate"
---

# What is normalization and can you explain 1NF, 2NF, and 3NF briefly?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Normalization in relational database design is a process of organizing data to reduce redundancy and dependency by dividing it into related tables and establishing relationships between them. It ensures that each piece of information in the database has a single, unique representation.

#### Triaging the Question

The question asks for an explanation of normalization levels: 1NF (First Normal Form), 2NF (Second Normal Form), and 3NF (Third Normal Form). The interviewer is probing your understanding of fundamental concepts in database design, specifically how to ensure data integrity and minimize redundancy. You'll need to explain each form succinctly while providing context on what these levels mean.

#### First-Principles Solution
1NF: First Normal Form

First normal form (1NF) requires that the table does not contain repeating groups of values in a single column. In other words, every non-key field must be atomic (i.e., indivisible). A simple example is ensuring that each row in a "Courses" table has only one course name and one corresponding ID.

#### Example

Before 1NF:
CODE
| Course ID | Course Name |
|-----------|-------------|
| C001      | Math, Science |
After 1NF:
CODE
| Course ID | Course Name       |
|-----------|------------------|
| C001      | Math             |
| C002      | Science          |
2NF: Second Normal Form

Second normal form (2NF) builds on 1NF by ensuring that the table is fully normalized. This means every non-key column must be functionally dependent on the primary key, and no partial dependencies exist. A functional dependency means that if you know the value of one attribute, you can determine the value of another.

#### Example

Before 2NF:
CODE
| Student ID | Course ID | Teacher Name |
|------------|-----------|--------------|
| S001       | C001      | Mr. Smith    |
| S002       | C001      | Mrs. Johnson |
After 2NF (split into two tables):
CODE
Student Table:
| Student ID | Student Name |
|------------|--------------|
| S001       | John         |
| S002       | Jane         |

Course Table:
| Course ID | Course Name    | Teacher ID |
|-----------|----------------|------------|
| C001      | Math           | T001       |
3NF: Third Normal Form

Third normal form (3NF) extends 2NF by eliminating transitive dependencies. A transitive dependency occurs when a non-key column depends on another non-key column, which in turn is dependent on the primary key.

#### Example

Before 3NF:
CODE
| Student ID | Course ID | Teacher Name | Room Number |
|------------|-----------|--------------|-------------|
| S001       | C001      | Mr. Smith    | R12         |
| S002       | C001      | Mrs. Johnson | R13         |
After 3NF (split into three tables):
CODE
Student Table:
| Student ID | Student Name | Room Number |
|------------|--------------|-------------|
| S001       | John         | R12         |
| S002       | Jane         | R13         |

Course Table:
| Course ID | Course Name  | Teacher ID  |
|-----------|--------------|-------------|
| C001      | Math         | T001        |

Teacher Table:
| Teacher ID | Teacher Name |
|------------|--------------|
| T001       | Mr. Smith    |

#### Closing Summary
+ Common Mistakes/Follow-Ups

Normalization is crucial for maintaining data integrity and reducing redundancy, which ultimately improves the performance and scalability of your database. The key differences between 1NF, 2NF, and 3NF are in ensuring atomicity (1NF), functional dependencies (2NF), and eliminating transitive dependencies (3NF).

#### Common mistakes
include:

Not properly identifying the primary key.
Failing to recognize partial or transitive dependencies.
Over-normalizing, which can lead to too many tables and complex queries.

#### Follow-up
questions might explore how you would normalize a real-world dataset or discuss when it's acceptable not to fully normalize a table (e.g., performance concerns in certain contexts).

**Comparison:**

| 1NF Example: Course Names with Multiple Courses in One Row |
|------------------------------------------------------------|
| Student ID | Course Name         |
|------------|--------------------|
| S001       | Math, Science      |
| After Normalization to 1NF |
|-----------------------------|
| Student ID | Course ID          | Course Name  |
|------------|--------------------|--------------|
| S001       | C001               | Math         |
| S002       | C002               | Science      |

**Code:**

```sql
-- Example of 1NF violation:
CREATE TABLE Courses (
    Course_ID VARCHAR(5),
    Course_Name VARCHAR(20)
);

INSERT INTO Courses VALUES ('C001', 'Math, Science');

-- Corrected to 1NF:
CREATE TABLE Courses (
    Course_ID VARCHAR(5),
    Course_Name VARCHAR(20)
);

CREATE TABLE Course_Titles (
    Course_Title_ID INT AUTO_INCREMENT,
    Course_Name VARCHAR(20) NOT NULL
);

INSERT INTO Course_Titles VALUES ('Math'), ('Science');
```

**Example input / output:**

```text
-- Example input -> 3NF output:
INSERT INTO Students VALUES ('S001', 'John'), ('S002', 'Jane');
INSERT INTO Courses VALUES ('C001', 'Math'), ('C002', 'Science');
INSERT INTO Teachers VALUES (T001, 'Mr. Smith'), (T002, 'Mrs. Johnson');

-- Output:
Students Table:
| Student_ID | Student_Name |
|------------|--------------|
| S001       | John         |
| S002       | Jane         |

Courses Table:
| Course ID | Course Name  | Teacher ID  |
|-----------|--------------|-------------|
| C001      | Math         | T001        |
| C002      | Science      | T002        |

Teachers Table:
| Teacher ID | Teacher Name |
|------------|--------------|
| T001       | Mr. Smith    |
| T002       | Mrs. Johnson |
```

**Data:**

```sql
-- Example of 1NF violation:
CREATE TABLE Student_Course_Mapping (
    Student_ID VARCHAR(5),
    Course_ID VARCHAR(5),
    Teacher_Name VARCHAR(20)
);

INSERT INTO Student_Course_Mapping VALUES ('S001', 'C001', 'Mr. Smith');

-- Corrected to 1NF:
CREATE TABLE Students (
    Student_ID VARCHAR(5),
    Student_Name VARCHAR(20)
);

CREATE TABLE Courses (
    Course_ID VARCHAR(5),
    Course_Name VARCHAR(20)
);

CREATE TABLE Teachers (
    Teacher_ID INT AUTO_INCREMENT,
    Teacher_Name VARCHAR(20) NOT NULL
);

-- Example of 2NF violation:
INSERT INTO Student_Course_Mapping VALUES ('S001', 'C001', 'Mr. Smith');

-- Corrected to 2NF (split into two tables):
CREATE TABLE Courses (
    Course_ID VARCHAR(5),
    Course_Name VARCHAR(20)
);

CREATE TABLE Teachers (
    Teacher_ID INT AUTO_INCREMENT,
    Teacher_Name VARCHAR(20) NOT NULL
);

CREATE TABLE Enrollments (
    Student_ID VARCHAR(5),
    Course_ID VARCHAR(5),
    Teacher_ID INT
);
```

**Flow:**

1. No
2. Yes
3. Yes
4. Before 1NF
5. Is each non-key field atomic?
6. Convert to 1NF
7. Are there any partialdependencies?
8. Split into multiple tables andconvert to 2NF
9. Are there any transitivedependencies?
10. Further split into additionaltables and convert to 3NF

