---
type: "practice"
difficulty: "Basic"
---

# Can you list the normal forms in order from 1NF up to BCNF?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ BCNF vs 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/BCNF_vs_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sure! Let's dive into the world of normalization, which is a fundamental concept in database design that helps us organize data efficiently and reduce redundancy.

#### Triaging the Question

The question asks about listing the normal forms from 1NF (First Normal Form) to BCNF (Boyce-Codd Normal Form). This is essentially asking you to recall and explain each level of normalization. The interviewer is likely probing your foundational knowledge in database design principles.

#### First-Principles Solution

Normalization is a process used to organize data in a relational database. It involves restructuring tables to prevent duplication, reduce redundancy, and improve the overall integrity of the database. Here's a step-by-step breakdown from 1NF up to BCNF:

1. First Normal Form (1NF)
Definition: A relation is said to be in 1NF if it satisfies two conditions:
Each column contains atomic values.
The table has no repeating groups.

#### Example
Consider a Students table where each row represents a student, and columns include StudentID, FirstName, and LastName. This is in 1NF if there are no duplicate rows or repeating group structures like multiple entries for the same student.
2. Second Normal Form (2NF)
Definition: A relation is in 2NF if it satisfies these conditions:
It is in 1NF.
All non-key attributes are fully functionally dependent on the primary key.

#### Example
If we add an Enrollments table to link students with their courses, ensuring that each course ID depends only on a student's ID would satisfy this form.
3. Third Normal Form (3NF)
Definition: A relation is in 3NF if it satisfies these conditions:
It is in 2NF.
Non-key attributes are not transitively dependent on the primary key.

#### Example
In our Enrollments table, ensuring that there's no dependency of course fees on student IDs would help achieve this form.
4. Fourth Normal Form (4NF)
Definition: A relation is in 4NF if it satisfies these conditions:
It is in 3NF.
The table does not have multi-valued dependencies.

#### Example
If we have a Departments table and an Employees table, ensuring that each department has multiple roles (like HR, IT) without duplicating these roles across employees would adhere to this form.
5. ** Fifth Normal Form (5NF or Full Normal Form)**
Definition: A relation is in 5NF if it satisfies the conditions of 4NF and does not have join dependencies.

#### Example
Ensuring that all data is atomic and there are no join dependencies across multiple tables would help achieve this form.
6. Boyce-Codd Normal Form (BCNF)
Definition: A relation is in BCNF if it satisfies these conditions:
It is in 3NF.
For every non-trivial functional dependency X → Y, X must be a superkey.

#### Example
In our Enrollments table, ensuring that any subset of the key (student ID and course ID) determines all other attributes would help achieve this form.

#### Closing Summary

In summary, normalization is crucial for maintaining data integrity and reducing redundancy. Each normal form builds upon the previous one to address specific issues in data dependency. From 1NF up to BCNF, each step ensures that the database structure is more robust and efficient.

#### Common Mistakes/Follow-Ups
Common Mistake: Misunderstanding the dependencies required for higher forms (like fully functional dependence or transitive dependence).

#### Follow-Up
The interviewer might ask about real-world examples where you'd prefer 1NF over higher normal forms, or vice versa. Be prepared to discuss trade-offs and practical considerations.

**Comparison:**

```
Table	Normal Form
Students & Courses	1NF + 2NF
Enrollments	3NF + 4NF + BCNF
```

**Code:**

```sql
-- Example of a simple table in 2NF
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Enrollments
);

-- Example of a table in BCNF
CREATE TABLE Enrollments (
    StudentID INT,
    CourseID INT,
    Fee DECIMAL(10, 2),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

**Example input / output:**

```text
Input:
- Students Table (1NF)
- Enrollments Table

Output:
- 2NF: Add foreign keys to ensure dependencies.
- BCNF: Ensure that every non-trivial functional dependency has a superkey as the determinant.
```

**Data:**

```json
{
  "Students": [
    {
      "StudentID": 1,
      "FirstName": "John",
      "LastName": "Doe"
    },
    {
      "StudentID": 2,
      "FirstName": "Jane",
      "LastName": "Smith"
    }
  ],
  "Courses": [
    {
      "CourseID": 101,
      "CourseName": "Data Engineering"
    },
    {
      "CourseID": 102,
      "CourseName": "Database Management"
    }
  ]
}
```

**Flow:**

1. 1NF - Atomic Values
2. 2NF - Fully FunctionalDependency
3. 3NF - No TransitiveDependence
4. 4NF - No Multi-ValuedDependencies
5. 5NF/Full Normal Form - NoJoin Dependencies
6. BCNF - Superkey Determinantfor Non-trivial FunctionalDependencies

