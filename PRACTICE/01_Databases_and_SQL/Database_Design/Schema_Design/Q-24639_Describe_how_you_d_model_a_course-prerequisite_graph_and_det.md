---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd model a course-prerequisite graph and detect/prevent cycles at the schema or query level.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Modeling Relationships and Schemas](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database to model a course-prerequisite relationship, we need to ensure that the schema can efficiently represent the relationships between courses and their prerequisites while also providing mechanisms to detect and prevent cycles. This involves understanding both the structure of the data and the logical constraints it must adhere to.

Firstly, let's define what we mean by "cycles" in this context: a cycle occurs when a course has a prerequisite that directly or indirectly includes itself as a dependency, leading to an infinite loop of prerequisites. Detecting and preventing such cycles is crucial for maintaining a valid and meaningful course catalog.

To model the relationship, we'll use two main entities: Courses and Prerequisites. The schema should be designed with these key considerations in mind:

Entities: We have Courses, which stores information about each course.
Relationships: A one-to-many relationship between Courses and its prerequisites.

Now, let's delve into the detailed steps to achieve this using first principles:

#### Triaging the Question

The question is asking for a schema design that can represent a directed graph where nodes are courses and edges are prerequisite relationships. The goal is not just to model these relationships but also to ensure they do not form cycles.

#### Assumptions
Each course has one or more prerequisites.
A course cannot be its own prerequisite directly or indirectly (no cycles).
There can be multiple courses with the same name but different identifiers, so we need unique keys for each instance of a course.
The database should support efficient querying and updating.
Step-by-Step Solution

Entity Definitions:

Courses table: This holds basic information about each course.
Prerequisites table: Maps courses to their prerequisites.

Schema Design:

SQL
CREATE TABLE Courses (
    CourseID INT PRIMARY KEY AUTO_INCREMENT,
    CourseName VARCHAR(100) NOT NULL UNIQUE,
    CourseDescription TEXT
);

CREATE TABLE Prerequisites (
    CourseID INT,
    PrereqCourseID INT,
    FOREIGN KEY (CourseID, PrereqCourseID) REFERENCES Courses(CourseID, CourseID)
);

Detecting Cycles:

Using a recursive common table expression (CTE) in SQL to detect cycles.
Alternatively, implementing a cycle detection algorithm in the application layer.

Preventing Cycles at Query Level:

Adding validation logic within transactional operations to check for cycles before inserting new prerequisites.
First-Principles Reasoning

Entities and Relationships: We start by defining two tables: Courses and Prerequisites. The Courses table stores the basic metadata of each course, while the Prerequisites table maps a course to its prerequisite(s).

Data Integrity Constraints:

In Prerequisites, both CourseID and PrereqCourseID are foreign keys referencing Courses.CourseID.
Ensuring uniqueness on CourseName in Courses prevents duplicate courses.

Cycle Detection:

SQL CTEs can be used to detect cycles by traversing the graph of prerequisites.
SQL
WITH RECURSIVE prerequisite_graph AS (
    SELECT CourseID, PrereqCourseID FROM Prerequisites
    UNION ALL
    SELECT pg.CourseID, p.PrereqCourseID
    FROM prerequisite_graph pg JOIN Prerequisites p ON pg.PrereqCourseID = p.CourseID
)
SELECT * FROM prerequisite_graph WHERE CourseID IN (SELECT CourseID FROM Courses);
This query helps in identifying if any course is part of a cycle.

Prevent Cycles:

Before inserting new prerequisites, validate that adding the edge does not create a cycle.
Use a transaction to ensure all operations are atomic and consistent.

#### Closing Summary

In summary, designing a schema for a course-prerequisite relationship involves defining Courses and Prerequisites tables with proper foreign key constraints. Detecting cycles can be done using SQL CTEs or application logic, while preventing them requires careful validation during data insertion. Ensuring the integrity of this model is crucial to maintaining a valid and useful database.

#### Common Mistakes/Follow-Ups
Common Mistake: Not properly validating before inserting prerequisites, which may lead to accidental cycle creation.

#### Follow-Up
Discussing how to optimize query performance for large datasets or handling different types of cycles (direct vs. indirect).

**Comparison:**

```
Course
Prerequisite
```

**Code:**

```sql
CREATE TABLE Courses (
    CourseID INT PRIMARY KEY AUTO_INCREMENT,
    CourseName VARCHAR(100) NOT NULL UNIQUE,
    CourseDescription TEXT
);

CREATE TABLE Prerequisites (
    CourseID INT,
    PrereqCourseID INT,
    FOREIGN KEY (CourseID, PrereqCourseID) REFERENCES Courses(CourseID, CourseID)
);
```

**Example input / output:**

```text
WITH RECURSIVE prerequisite_graph AS (
    SELECT CourseID, PrereqCourseID FROM Prerequisites
    UNION ALL
    SELECT pg.CourseID, p.PrereqCourseID
    FROM prerequisite_graph pg JOIN Prerequisites p ON pg.PrereqCourseID = p.CourseID
)
SELECT * FROM prerequisite_graph WHERE CourseID IN (SELECT CourseID FROM Courses);
```

**Data:**

```json
{
  "Courses": [
    {"CourseID": 1, "CourseName": "Math", "CourseDescription": "Basic Math"},
    {"CourseID": 2, "CourseName": "Algebra", "CourseDescription": "Advanced Algebra"}
  ],
  "Prerequisites": [
    {"CourseID": 2, "PrereqCourseID": 1}
  ]
}
```

**Flow:**

1. User
2. Insert Prerequisites
3. Validate Cycles
4. Deny Insertion if CycleDetected
5. Allow Insertion if No Cycle
6. Throw Error or Log Issue
7. Insert into Database

