---
type: "practice"
difficulty: "Basic"
---

# What is a many-to-many relationship and why can't you model it with a single foreign key column?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data modeling, a many-to-many (m:m) relationship occurs when each entity in one table can be associated with multiple entities in another table, and vice versa. For example, consider a scenario where you have Students and Courses. Each student can enroll in many courses, and each course can have many students enrolled. To model such relationships, using just a single foreign key column would not suffice because it would either limit the relationship or lead to incorrect data representation.

Let's break down why we need more than a single foreign key:

Assumption of One-to-Many: A single foreign key implies that each entity in one table can be associated with only one entity in another table, which is not true for m:m relationships.
Data Consistency and Integrity: Using just one foreign key might lead to inconsistencies or orphaned records if the referenced record gets deleted.

In this article, we'll explore the concept of normalization and denormalization in detail and see how they can be applied to model many-to-many relationships effectively.

**Comparison:**

| Student | Course |
|---------|--------|
| Alice   | Math   |
| Alice   | Physics|
| Bob     | Math   |

**Code:**

```sql
-- Example ERD for Students and Courses with a many-to-many relationship
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100)
);

-- The join table to handle the many-to-many relationship
CREATE TABLE Enrollments (
    EnrollmentID INT PRIMARY KEY AUTO_INCREMENT,
    StudentID INT,
    CourseID INT,
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

**Example input / output:**

```text
-- Sample data insertion
INSERT INTO Students (Name) VALUES ('Alice'), ('Bob');
INSERT INTO Courses (CourseName) VALUES ('Mathematics'), ('Physics');

-- Enrollments creation
INSERT INTO Enrollments (StudentID, CourseID) VALUES (1, 101), (1, 102), (2, 101);

-- Query to find all courses enrolled by Alice
SELECT Courses.CourseName FROM Courses
JOIN Enrollments ON Courses.CourseID = Enrollments.CourseID
JOIN Students ON Students.StudentID = Enrollments.StudentID
WHERE Students.Name = 'Alice';
```

**Data:**

```json
{
  "Students": [
    { "StudentID": 1, "Name": "Alice" },
    { "StudentID": 2, "Name": "Bob" }
  ],
  "Courses": [
    { "CourseID": 101, "CourseName": "Mathematics" },
    { "CourseID": 102, "CourseName": "Physics" }
  ],
  "Enrollments": [
    { "EnrollmentID": 1, "StudentID": 1, "CourseID": 101 },
    { "EnrollmentID": 2, "StudentID": 1, "CourseID": 102 },
    { "EnrollmentID": 3, "StudentID": 2, "CourseID": 101 }
  ]
}
```

**Flow:**

1. Students
2. Enrollment?
3. Many-to-Many Relationship
4. Enrollments
5. Courses

