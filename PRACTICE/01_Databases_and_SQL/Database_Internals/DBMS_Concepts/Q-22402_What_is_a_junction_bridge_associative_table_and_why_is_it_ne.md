---
type: "practice"
difficulty: "Basic"
---

# What is a junction (bridge/associative) table and why is it needed for many-to-many relationships?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Relationship Modeling](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Relationship_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A junction table, also known as a bridge or associative table, is used to manage many-to-many relationships in databases by creating an intermediate table that links two other tables together. It's needed because traditional one-to-many and many-to-one relationships can't fully capture the complexity of entities having multiple connections with each other.

#### Triage
The question aims to understand your grasp on database normalization and relationship management. You need to explain what a junction table is, its purpose, and why it’s necessary when dealing with many-to-many relationships.

**Code:**

```sql
-- Example schema for a simple many-to-many relationship using a junction table

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(255)
);

CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(255)
);

CREATE TABLE Enrollments (
    EnrollmentID INT PRIMARY KEY AUTO_INCREMENT,
    CourseID INT,
    StudentID INT,
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
);
```

