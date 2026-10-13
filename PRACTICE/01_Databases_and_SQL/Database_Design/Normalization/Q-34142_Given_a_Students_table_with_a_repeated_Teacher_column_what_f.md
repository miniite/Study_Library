---
type: "practice"
difficulty: "Basic"
---

# Given a Students table with a repeated Teacher column what first step would you take to normalize it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Redundancy and Anomalies](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the normalization issue in the Students table, where the Teacher column is repeatedly listed, we need to start by understanding that this violates First Normal Form (1NF). The first step would be to identify a unique Teacher entity and normalize the data such that each teacher is represented only once. This ensures that our database design adheres to normalization rules, improving data integrity and reducing redundancy.

#### Triage

The question asks for the initial step in normalizing a Students table that contains redundant Teacher entries. The key idea here is recognizing that there are duplicate values in the Teacher column, which need to be cleaned up before further normalization steps can be applied.

#### Assumptions
We assume that the current schema has a simple Students table with columns like StudentID, Name, and Teacher.
The goal is to reduce redundancy and improve data integrity by ensuring each teacher is represented only once in the database.

#### First-Principles Solution
Identify Duplicates: Use SQL queries to find duplicate teachers.
Create a Separate Teacher Table: Extract unique teachers into a new Teachers table, with columns like TeacherID and Name.
Update References: Replace the redundant teacher names in the Students table with references to the Teachers table using TeacherID.
Detailed Steps

Step 1: Find Duplicates

SQL
SELECT Teacher, COUNT(*) AS Count
FROM Students
GROUP BY Teacher
HAVING COUNT(*) > 1;

Step 2: Create Teachers Table

SQL
CREATE TABLE Teachers (
    TeacherID INT PRIMARY KEY,
    Name VARCHAR(50)
);

Step 3: Insert Unique Teachers into Teachers Table

SQL
INSERT INTO Teachers (TeacherID, Name)
SELECT ROW_NUMBER() OVER (ORDER BY Teacher) AS ID, Teacher
FROM Students
GROUP BY Teacher;

Step 4: Update References in Students Table

SQL
UPDATE Students
SET Teacher = t.TeacherID
FROM Students s
     JOIN Teachers t ON s.Teacher = t.Name;

Step 5: Ensure Data Consistency After updating the Students table, verify that all teacher references are consistent and correct.

#### Closing Summary

The initial step to normalize the Students table is to identify and extract unique teachers into a new Teachers table. This process ensures data integrity by eliminating redundant entries and making the database more efficient for querying and management.

#### Common Mistakes/Follow-Ups
Forgetting to Verify Data: After updating references, it’s crucial to verify that all records are correctly linked.
Handling Missing Teachers: If some teachers are missing in the Students table but exist in a different context (e.g., historical data), additional steps may be needed.

#### Performance Considerations
Depending on the size of the dataset, performance optimizations might be necessary during the update phase.

