---
type: "practice"
difficulty: "Basic"
---

# What is the difference between COUNT(DISTINCT column) and COUNT(column), and when do their results differ?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ GROUP BY vs PARTITION BY](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, COUNT(column) and COUNT(DISTINCT column) are two commonly used aggregate functions that serve different purposes. The main difference lies in how they handle duplicates within a column.

COUNT(column) simply counts the number of non-null values in the specified column without considering their uniqueness. This means every distinct value in the column, even if it appears multiple times, is counted once.

On the other hand, COUNT(DISTINCT column) calculates the count based on unique values only. It treats all occurrences of a duplicate value as one and counts each unique value just once, disregarding any repeated instances within the same row.

The results will differ when there are duplicate entries in your column. Let's dive into how this plays out with an example.

**Code:**

```sql
-- Sample data table creation and insertion
CREATE TABLE Employees (
    ID INT,
    Name VARCHAR(255)
);

INSERT INTO Employees (ID, Name) VALUES
(1, 'Alice'),
(2, 'Bob'),
(3, 'Charlie'),
(4, 'Alice'), -- Duplicate name
(5, 'David');
```

**Example input / output:**

```sql
-- Sample query outputs
SELECT COUNT(Name) FROM Employees; -- 5 rows
SELECT COUNT(DISTINCT Name) FROM Employees; -- 4 distinct names

The COUNT(Name) will return 5, as it counts every row in the table. However, COUNT(DISTINCT Name) returns 4, because 'Alice' is only counted once despite appearing twice.
```

