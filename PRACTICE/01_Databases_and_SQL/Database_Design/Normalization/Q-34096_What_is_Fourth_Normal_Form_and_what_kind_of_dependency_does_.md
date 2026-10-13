---
type: "practice"
difficulty: "Deep"
---

# What is Fourth Normal Form and what kind of dependency does it eliminate?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ BCNF vs 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/BCNF_vs_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Fourth Normal Form (4NF) in database design addresses a specific type of data redundancy known as multidimensional dependencies, ensuring that relationships between tables are properly normalized to avoid anomalies during updates.

#### Triaging the Question

The core of this question is understanding 4NF and identifying the dependency it eliminates. The interviewer might be probing your knowledge of normalization levels and your ability to identify complex relationships in database design.

#### First-Principles Solution
What is Fourth Normal Form?

Fourth Normal Form (4NF) is a level of normalization in relational databases that goes beyond third normal form (3NF). It aims to eliminate multidimensional dependencies, which are more complex than the simple functional dependencies addressed by 2NF and 3NF. A table is said to be in 4NF if it satisfies all the conditions for 3NF and does not have any non-trivial multivalued dependencies.

What Kind of Dependency Does It Eliminate?

The key dependency that 4NF eliminates is a multivalued dependency (MVD). In simpler terms, MVDs are relationships where a set of attributes can have multiple values for another set of attributes. For example, in a table about students and their courses, if each student can take multiple courses from different departments, this would be an instance of a multivalued dependency.

Step-by-Step Reasoning:

Understanding Functional Dependencies (2NF):

In 2NF, all non-key attributes are functionally dependent on the primary key.

Adding Transitive Dependencies to 3NF:

3NF eliminates transitive dependencies where a non-prime attribute depends on another non-prime attribute through the primary key.

Identifying Multivalued Dependencies (4NF):

A multivalued dependency exists if changing one set of values in a relation results in changes to multiple sets of values elsewhere, but not affecting other values.
For example, consider a table STUDENT-COURSE with attributes (StudentID, CourseID, Department). If the fact that a student is taking a course from department A implies they might also be taking courses from department B or C, this indicates a multivalued dependency.

Ensuring 4NF:

To achieve 4NF, we need to decompose the table into smaller relations if there are non-trivial multivalued dependencies.
For example, splitting STUDENT-COURSE into:
STUDENT-COURSE: (StudentID, CourseID)
COURSE-DEPARTMENT: (CourseID, Department)

#### Example
Decomposition:
SQL
-- Original Table: STUDENT-COURSE
CREATE TABLE STUDENT_COURSE (
    StudentID INT,
    CourseID INT,
    Department VARCHAR(100),
    PRIMARY KEY(StudentID, CourseID)
);

-- 4NF Decomposed Tables:
CREATE TABLE STUDENT (
    StudentID INT,
    -- other attributes
    PRIMARY KEY(StudentID)
);

CREATE TABLE COURSE (
    CourseID INT,
    -- other attributes
    PRIMARY KEY(CourseID)
);

CREATE TABLE DEPARTMENT (
    Department VARCHAR(100),
    -- other attributes
    PRIMARY KEY(Department)
);

CREATE TABLE STUDENT_COURSE_MVD (
    StudentID INT,
    CourseID INT,
    FOREIGN KEY (StudentID, CourseID) REFERENCES STUDENT_COURSE(StudentID, CourseID),
    PRIMARY KEY(StudentID, CourseID)
);

#### Common Mistakes
and Follow-Ups:

Mistake: Confusing 4NF with other normalization forms. Remember that it specifically targets multivalued dependencies.

#### Follow-Up
The interviewer might ask you to design a normalized schema for a given dataset or explain the implications of violating 4NF in real-world scenarios.

#### Closing Summary

Understanding Fourth Normal Form is crucial as it helps in designing databases that are free from certain types of data redundancy. By recognizing and addressing multivalued dependencies, we can ensure that our database operations remain consistent and efficient.

**Comparison:**

```
StudentID	Name
101	Alice
102	Bob
CourseID	Title
501	Mathematics
502	Physics
Department	Building
Math	Main Hall
Science	Science Lab
StudentID	CourseID	Department
101	501	Math
102	502	Science
```

**Code:**

```sql
-- Example SQL Schema for 4NF Decomposition
CREATE TABLE STUDENT (
    StudentID INT PRIMARY KEY,
    -- other attributes
);

CREATE TABLE COURSE (
    CourseID INT PRIMARY KEY,
    -- other attributes
);

CREATE TABLE DEPARTMENT (
    Department VARCHAR(100) PRIMARY KEY,
    -- other attributes
);

CREATE TABLE STUDENT_COURSE_MVD (
    StudentID INT,
    CourseID INT,
    FOREIGN KEY (StudentID, CourseID) REFERENCES STUDENT(STUDENT_ID),
    FOREIGN KEY (CourseID) REFERENCES COURSE(COURSE_ID),
    PRIMARY KEY(StudentID, CourseID)
);
```

**Example input / output:**

```text
graph LR;
    A[STUDENT] --> B[COURSE]
    B --> C[DEPARTMENT]
    A --> D[STUDENT_COURSE_MVD]
```

**Data:**

```json
{
  "STUDENT": {
    "StudentID": [101, 102],
    "Name": ["Alice", "Bob"]
  },
  "COURSE": {
    "CourseID": [501, 502],
    "Title": ["Mathematics", "Physics"]
  },
  "DEPARTMENT": {
    "Department": ["Math", "Science"],
    "Building": ["Main Hall", "Science Lab"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify MultivaluedDependency
4. Is it Non-Trivial?
5. Decompose Table
6. Table is in 4NF

